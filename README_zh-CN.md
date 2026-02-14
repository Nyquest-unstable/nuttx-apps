# 应用程序目录

## 目录

- 概述
- 目录位置
- 内置应用程序
- NuttShell (NSH) 内置命令
- 同步内置命令
- 应用程序配置文件
- 示例内置应用程序
- 使用源代码树外的板级特定部分构建 NuttX

## 概述

此文件夹提供子目录中的各种应用程序。这些应用程序不是 NuttX 固有的一部分，而是为了帮助您开发自己的应用程序而提供的。`apps/` 目录是您可以选择使用或不使用的配置的_分离_部分。

## 目录位置

NuttX 构建默认使用的应用程序目录应命名为 `apps/`（或 `apps-x.y.z/`，其中 `x.y.z` 是 NuttX 版本号）。这个 `apps/` 目录应该出现在与 NuttX 目录同级的目录树中。如下所示：

```
 .
 |- nuttx
 |
 `- apps
```

如果以上所有条件都为 TRUE，则 NuttX 将能够找到应用程序目录。如果您的应用程序目录具有不同的名称或位于不同的位置，则必须通知 NuttX 构建系统该位置。有几种方法可以做到这一点：

1) 您可以在 NuttX 配置文件中定义 `CONFIG_APPS_DIR` 作为应用程序目录的完整路径。
2) 您可以在命令行上提供应用程序目录的路径，如：`make APPDIR=<path>` 或 `make CONFIG_APPS_DIR=<path>`
3) 当您使用 `tools/configure.sh` 配置 NuttX 时，您可以在配置命令行上提供应用程序目录的路径，如：
   `./configure.sh -a <app-dir> <board-name>:<config-name>`

## 内置应用程序

NuttX 还支持可以使用名称字符串启动的应用程序。在这种情况下，应用程序入口点及其要求会聚集在两个文件中：

- `builtin/builtin_proto.h` – 入口点，原型函数
- `builtin/builtin_list.h` – 应用程序特定信息和要求

构建在几个阶段进行，因为执行了不同的构建目标：(1) context，(2) depend 和 (3) default (all)。应用程序信息在 make context 构建阶段收集。

要执行应用程序功能：

`exec_builtin()` 在 `apps/include/builtin/builtin.h` 中定义。

## NuttShell (NSH) 内置命令

内置应用程序的一个用途是提供一种通过 NuttShell (NSH) 命令行调用自定义应用程序的方法。NSH 将支持一个无缝的方法来调用应用程序，当在 NuttX 配置文件中启用以下选项时：

```conf
CONFIG_NSH_BUILTIN_APPS=y
```

在 `apps/builtin/builtin_list.h` 文件中注册的应用程序将可以从 NSH 命令行访问。如果您在 NSH 提示符下输入 `help`，您将看到注册命令的列表。

## 同步内置命令

默认情况下，从 NSH 命令行启动的内置命令将与 NSH 异步运行。如果您希望强制 NSH 执行命令然后等待命令执行，您可以通过在 NuttX 配置文件中添加以下内容来启用该功能：

```conf
CONFIG_SCHED_WAITPID=y
```

配置选项启用了对 `waitpid()` RTOS 接口的支持。当启用该接口时，NSH 将使用它来等待，在内置命令执行完成之前休眠。

当然，即使定义了 `CONFIG_SCHED_WAITPID=y`，仍可通过在 NSH 命令后添加与号 (`&`) 来强制特定命令异步运行。

## 应用程序配置文件

NuttX 配置使用 `kconfig-frontends` 工具和 NuttX 配置文件 (`.config`) 文件。例如，NuttX `.config` 可能包含：

```conf
CONFIG_EXAMPLES_HELLO=y
```

这将选择 `apps/examples/hello`，如下所示：

- 顶层 make 将包含 `apps/examples/Make.defs`
- `apps/examples/Make.defs` 将设置 `CONFIGURED_APPS += $(APPDIR)/examples/hello` 如下：

```makefile
  ifneq ($(CONFIG_EXAMPLES_HELLO),)
  CONFIGURED_APPS += $(APPDIR)/examples/hello
  endif
```

## 示例内置应用程序

在 `examples/hello` 子目录下可以找到示例应用程序框架。这个示例展示了如何将内置应用程序添加到项目中。需要执行以下操作：

 1. 创建子目录，如：progname

 2. 在此目录中应该有：

    - 一个 `Make.defs` 文件，将被 `apps/Makefile` 包含
    - 一个 `Kconfig` 文件，将被配置工具使用（参见 NuttX 工具库中的 `kconfig-language.txt` 文件）。这个 `Kconfig` 文件应被 `apps/Kconfig` 文件包含
    - 一个 `Makefile`，以及
    - 应用程序源代码。

 3. 应用程序源代码应提供入口点：

    ```c
    main()
    ```

 4. 在 `Makefile` 文件中设置要求，特别是以下几行：

    ```makefile
    PROGNAME   = progname
    PRIORITY   = SCHED_PRIORITY_DEFAULT
    STACKSIZE  = 768
    ASRCS      = asm 源文件列表，如 a.asm b.asm ...
    CSRCS      = C 源文件列表，如 foo1.c foo2.c ..
    ```

 5. `Make.defs` 文件应包含如下行：

    ```makefile
    ifneq ($(CONFIG_PROGNAME),)
    CONFIGURED_APPS += progname
    endif
    ```

## 使用源代码树外的板级特定部分构建 NuttX

问：有人想出了一种整洁的方法来使用源代码树外的板级特定部分构建 NuttX 吗？

答：这里有三种方法：

   1) 有一个名为 `make export` 的 make 目标。它将构建 NuttX，然后将所有头文件、库文件、启动对象和其他构建组件打包到一个 `.zip` 文件中。您可以将该 `.zip` 文件移动到所需的任何构建环境中。甚至可以在 DOS `CMD` 窗口中构建 NuttX。

      此 make 目标记录在顶级 `nuttx/README.txt` 中。

   2) 替换整个 `apps/` 目录。如果 `apps/` 目录中没有您需要的内容，您可以在 `.config` 文件中定义 `CONFIG_APPS_DIR`，使其指向一个不同的自定义应用程序目录。

      您可以根据需要将喜欢的任何部分从旧的 apps/ 目录复制到您的自定义 apps 目录中。

      这在 `NuttX/boards/README.txt` 和在线 `NuttX 移植指南` 中有说明，网址为 <https://cwiki.apache.org/confluence/display/NUTTX/Porting+Guide>。

   3) 如果您喜欢 `apps/` 目录中的随机集合，但只是想使用自己的外部子目录扩展现有组件，也有简单的方法：只需在 `apps/` 目录中创建一个符号链接，重定向到您的应用程序子目录即可。

      为了将其纳入构建，您链接到 `apps/` 目录下的目录应该包含 (1) 一个支持 `clean` 和 `distclean` 目标的 `Makefile`（参见其他 `Makefile` 示例），以及 (2) 一个小型的 `Make.defs` 文件，只需将自定义构建目录添加到 `CONFIGURED_APPS` 变量中，如：

      ```makefile
      CONFIGURED_APPS += my_directory1 my_directory2
      ```

      `apps/Makefile` 将始终自动检查是否存在包含 `Makefile` 和 `Make.defs` 文件的子目录。`Makefile` 只用于支持清理操作。`Make.defs` 文件提供了要构建的目录集；这些目录还必须包含 `Makefile`。该 `Makefile` 必须能够构建源代码并将对象添加到 `apps/libapps.a` 归档中（参见其他 `Makefile` 示例）。它应支持 all、install、context 和 depend 目标。

      `apps/Makefile` 不依赖于任何硬编码的目录列表。相反，它会进行通配符搜索以查找所有合适的目录。这意味着要安装新应用程序，只需将目录（或链接）复制到 `apps/` 目录即可。如果添加的目录还包括 `Kconfig` 文件，则它将自动包含在 NuttX 配置系统中。`apps/Makefile` 使用 `apps/tools/mkkconfig.sh` 中的工具在预配置时动态构建 `apps/Kconfig` 文件。

      例如，您可以创建一个名为 `install.sh` 的脚本，用于安装自定义应用程序、配置和板级特定目录：

      a) 将 `MyBoard` 目录复制到 `boards/MyBoard`。
      b) 在 `apps/external` 处为 `MyApplication` 添加符号链接。
      c) 配置 NuttX，通常通过：

      ```bash
      tools/configure.sh MyBoard:MyConfiguration
      ```

      建议使用 `apps/external` 名称，因为它包含在 `.gitignore` 文件中，可为您省去一些使用 GIT 时的麻烦。

# 出口限制

此发行版包括加密软件。您目前居住的国家/地区可能对加密软件的进口、拥有、使用和/或再出口到另一个国家有相关限制。在使用任何加密软件之前，请检查您所在国家/地区的法律、法规和政策，了解有关加密软件的进口、拥有、使用和再出口的规定，以确认这是允许的。更多信息请参见 <http://www.wassenaar.org/>。

美国商务部工业安全局 (BIS) 已将此软件分类为出口商品管制编号 (ECCN) 5D002.C.1，其中包括使用或执行具有非对称算法的加密功能的信息安全软件。这种分发形式和方式使该软件符合许可证例外 ENC 技术软件无限制 (TSU) 例外（参见 BIS 出口管理条例第 740.13 节）的出口条件，适用于目标代码和源代码。

以下包含有关所包含的加密软件的更多详细信息：
https://tls.mbed.org/supported-ssl-ciphersuites。
https://github.com/intel/tinycrypt
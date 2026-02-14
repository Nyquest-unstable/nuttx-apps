# 如何在apps/examples中添加新应用

本文档介绍了如何在NuttX的apps/examples目录中添加新应用，以及如何使NuttX正确识别新应用。

## 添加新应用的步骤

### 1. 创建应用目录
在`apps/examples`目录下创建一个新的子目录，例如`myapp`:
```
apps/examples/myapp/
```

### 2. 实现应用程序
在新目录中添加应用程序源代码文件，例如`myapp_main.c`:
```
apps/examples/myapp/myapp_main.c
```

### 3. 创建Makefile
创建`apps/examples/myapp/Makefile`文件:
```makefile
include $(APPDIR)/Application.mk
```

### 4. 创建Kconfig配置文件
在`apps/examples/myapp/`目录下创建`Kconfig`文件，定义配置选项:
```
config EXAMPLES_MYAPP
    tristate "My Application Example"
    default n
    ---help---
      My custom application example for NuttX RTOS.

if EXAMPLES_MYAPP

config EXAMPLES_MYAPP_PRIORITY
    int "My App Priority"
    default 100
    depends on EXAMPLES_MYAPP

config EXAMPLES_MYAPP_STACKSIZE
    int "My App Stack Size"
    default 2048
    depends on EXAMPLES_MYAPP

endif
```

## 让NuttX识别新应用的方法

在添加新应用后，为了让menuconfig正确识别新应用，可以使用以下几种方法之一：
经过实际测试，方法1有效。
### 方法1（推荐）：使用apps_distclean
```bash
cd nuttx
make apps_distclean
make menuconfig
```

### 方法2：使用apps_preconfig
```bash
cd nuttx
make apps_preconfig
make menuconfig
```

### 方法3：使用clean_context
```bash
cd nuttx
make clean_context
make menuconfig
```

## 注意事项

1. **Kconfig文件是关键**：确保为新应用创建了正确的Kconfig文件，这样它才会出现在menuconfig中

2. **配置命名空间**：应用程序配置必须使用独立的CONFIG命名空间（如CONFIG_EXAMPLES_HELLO_ZC_*），避免与现有应用配置冲突

3. **主程序文件名和函数名**：应包含应用特有标识，与标准示例区分开

4. **应用位置**：自定义示例应用程序应放置在`apps/examples/`目录下，遵循项目对示例代码的组织规范

## 编译和运行

1. 配置：`./tools/configure.sh <board>:<config>`
2. 清理：`make clean`
3. 编译：`make`

或使用`make menuconfig`调整应用配置后重新编译。

## 关于自动识别

在某些情况下，直接运行`make menuconfig`也能自动处理上下文刷新，但如果遇到新应用无法立即显示的问题，使用上述方法之一可以确保配置系统正确识别新应用。

## 版权信息

NuttX的应用程序在编译时被静态链接到内核中，形成单一固件映像；应用程序不能独立于内核单独烧录或动态添加；添加新应用程序需要重新编译整个项目并重新烧录固件。
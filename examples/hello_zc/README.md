# Hello ZC Example

This is a custom "Hello ZC" example application that demonstrates how to create a separate example distinct from the standard "Hello World" example in NuttX.

## Description

This example application prints a custom message: "Hello, ZC World!! This is the hello_zc example application."

## Configurations

None required. Enable via the configuration system using:
```
make menuconfig
# Then select: 
#   Application Configuration --->
#     Examples --->  
#       [*] Hello ZC Example
```

## Usage

After enabling and building the firmware, run the application from NSH:
```
nsh> hello_zc
Hello, ZC World!! This is the hello_zc example application.
```
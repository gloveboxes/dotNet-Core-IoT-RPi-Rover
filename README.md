# .NET Core IoT and Raspberry Pi Linux. Tips for Building, Deploying and Debugging

![](./docs/banner.png)

Outline

1. Nice Image
2. .NET Core IoT Open Source Project
3. Starting your first project
4. Sample
5. How to Build, Deploy and Debug
6. Where to get the Source Code
7. IoT Central

## Why .NET Core

[.NET Core](https://docs.microsoft.com/en-au/dotnet/core/) is an [open-source](https://github.com/dotnet/coreclr/blob/master/LICENSE.TXT), general-purpose development platform maintained by Microsoft and the .NET community on [GitHub](https://github.com/dotnet/core). It supports multiple programming languages, multiple platforms (Windows, macOS, and Linux), and muluiple processor architectures. It is used to build device, cloud, and IoT applications.

## The .NET Core IoT Libraries Open Source Project

The Microsoft .NET Core team are turning their attention to supporting [IoT](https://en.wikipedia.org/wiki/Internet_of_things) scenarios with [.NET Core IoT Libraries](https://github.com/dotnet/iot) across Linux, and Windows IoT Core, on ARM and Intel processor architectures. See the [.NET Core IoT Library Roadmap](https://github.com/dotnet/iot/blob/master/Documentation/roadmap.md) for more information.

### System.Device.Gpio

The [System.Device.Gpio](https://www.nuget.org/packages/System.Device.Gpio) preview package supports general-purpose I/O ([GPIO](https://en.wikipedia.org/wiki/General-purpose_input/output)) pins, PWM, I2C, SPI and related interfaces for interacting with low level hardware pins to control hardware sensors, displays and input devices on single-board-computers; [Raspberry Pi](https://www.raspberrypi.org/), [BeagleBoard](https://beagleboard.org/), [HummingBoard](https://www.solid-run.com/nxp-family/hummingboard/), [ODROID](https://www.hardkernel.com/), and other single-board-computers that are supported by **Linux** and **Windows 10 IoT Core**.

### Iot.Device.Bindings

The [.NET Core IoT Repository](https://github.com/dotnet/iot/tree/master/src) contains [IoT.Device.Bindings](https://www.nuget.org/packages/Iot.Device.Bindings), a growing set of community-maintained device bindings for IoT components that you can use with your .NET Core applications. Porting your own C/C++ driver libraries to .NET Core and C# is pretty straight forward too.

## Creating your first .NET Core IoT project

You can create .NET Core IoT projects on Linux, macOS and Windows desktops.  You need to install the following software.

1. [.NET Core](https://dotnet.microsoft.com/download)
2. [Visual Studio Code](https://code.visualstudio.com/)
3. For Windows Desktop:
    - [PuTTY SSH and telnet client](https://www.putty.org/)
    - The [WSL workspaceFolder](https://marketplace.visualstudio.com/itemdetails?itemName=lfurzewaddock.vscode-wsl-workspacefolder) Visual Studio Extension

Follow these steps

1. Open a command/terminal window

2. Create a new directory, change to it, create a new .NET Console app, and then start Visual Studio Code.

```bash
mkdir dotnet.core.iot

cd dotnet.core.iot

dotnet new console

code .
```

3. Add the Visual Studio Code Build and Debug assets

![](./docs/create-new-project.png)

4. Add a Nuget Package Reference in dotnet.core.iot.csproj file to the Iot.Device.Bindings library

```xml
    <ItemGroup>
        <PackageReference Include="Iot.Device.Bindings" Version="0.1.0-prerelease*" />
    </ItemGroup>
```

![add Nuget package](./docs/project-nuget-items.jpg)

5. Replace the code in Program.cs file with the following code

```c#
using System;
using Iot.Device;
using Iot.Device.CpuTemperature;
using System.Threading;

namespace dotnet.core.iot
{
    class Program
    {
        static CpuTemperature temperature = new CpuTemperature();
        static void Main(string[] args)
        {
            while (true)
            {
                if (temperature.IsAvailable)
                {
                    Console.WriteLine($"The CPU temperature is {temperature.Temperature.Celsius}");
                }
                Thread.Sleep(2000); // sleep for 2000 milliseconds, 2 seconds
            }
        }
    }
}
```

Your Visual Studio Code **Program.cs** file should look like the following screenshot.

![sample program](./docs/cpu-temperature-program.png)

## Deploying the project to your Raspberry Pi

To deploy a project to your Raspberry Pi you need to tell Visual Studio Code to compile for **linux-arm**, how to copy the compiled code to the Raspberry Pi, and finally how to attach the debugger.

For this walkthrough we are going to use [rsync](https://en.wikipedia.org/wiki/Rsync) to copy program files to the Raspberry Pi. Rsync is a very efficient file transfer protocol, comes standard with Linux, macOS, and Windows with the [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10) installed.

## Updating the Visual Studio Code Build Files

We need to update the [launch.json](https://code.visualstudio.com/docs/editor/debugging) and [tasks.json](https://code.visualstudio.com/docs/editor/debugging) files with the following code.

Notes:

1. These launch and build tasks assume the default network name of your Raspberry Pi is **raspberrypi.local**. Amend as required.
2. The Windows section uses plink for the pipeProgram. It specifies the default Raspberry Pi password which is raspberry. Amend as required.
2. These definitions support building the application from Linux, macOS and Windows desktops.

### launch.json

The launch.json file calls a **publish** prelaunch task which builds and copies the program to the Raspberry Pi, it then starts the program on the Raspberry Pi and attaches the debugger.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Publish, Launch, and Attach Debugger",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "publish",
            "program": "~/dotnet/${workspaceFolderBasename}/${workspaceFolderBasename}",
            "cwd": "~/dotnet/${workspaceFolderBasename}",
            "stopAtEntry": false,
            "console": "internalConsole",
            "linux": {
                "pipeTransport": {
                    "pipeCwd": "${workspaceRoot}",
                    "pipeProgram": "/usr/bin/ssh",
                    "pipeArgs": [
                        "pi@raspberrypi.local"
                    ],
                    "debuggerPath": "~/vsdbg/vsdbg"
                }
            },
            "osx": {
                "pipeTransport": {
                    "pipeCwd": "${workspaceRoot}",
                    "pipeProgram": "/usr/bin/ssh",
                    "pipeArgs": [
                        "pi@raspberrypi.local"
                    ],
                    "debuggerPath": "~/vsdbg/vsdbg"
                }
            },
            "windows": {
                "pipeTransport": {
                    "pipeCwd": "${workspaceRoot}",
                    "pipeProgram": "plink",
                    "pipeArgs": [
                        "-ssh",
                        "-pw",
                        "raspberry",
                        "pi@raspberrypi.local"
                    ],
                    "debuggerPath": "~/vsdbg/vsdbg"
                }
            }
        }
    ]
}
```

### task.json

The task.json file defines how to compile the project for linux-arm and how to copy the program to the Raspberry Pi with rsync.

Note, this tasks.json file assumes the default network name of your Raspberry Pi is **raspberrypi.local**. Amend as needed. For Windows you must explicitly specify the IP Address of the Raspberry Pi as rsync is called via Bash and the Windows Subsystem for Linux does not resolve .local DNS names.

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "publish",
            "linux": {
                "command": "sh",
                "type": "shell",
                "args": [
                    "-c",
                    "\"dotnet publish -r linux-arm -o bin/linux-arm/publish",
                    "${workspaceFolder}/${workspaceFolderBasename}.csproj\"",
                    ";",
                    "sh",
                    "-c",
                    "\"rsync -rvuz ${workspaceFolder}/bin/linux-arm/publish/ pi@raspberrypi.local:/home/pi/${workspaceFolderBasename}\""
                ],
            },
            "osx": {
                "command": "sh",
                "type": "shell",
                "args": [
                    "-c",
                    "\"dotnet publish -r linux-arm -o bin/linux-arm/publish",
                    "${workspaceFolder}/${workspaceFolderBasename}.csproj\"",
                    ";",
                    "sh",
                    "-c",
                    "\"rsync -rvuz ${workspaceFolder}/bin/linux-arm/publish/ pi@raspberrypi.local:/home/pi/${workspaceFolderBasename}\""
                ],
            },
            "windows": {
                "command": "cmd",
                "type": "shell",
                "args": [
                    "/c",
                    "\"dotnet publish -r linux-arm -o bin/linux-arm/publish",
                    "${workspaceFolder}/${workspaceFolderBasename}.csproj\"",
                    "&&",
                    "bash",
                    "-c",
                    "\"rsync -rvuz ${command:extension.vscode-wsl-workspaceFolder}/bin/linux-arm/publish/ pi@<YOUR RASPBERRY PI IP ADDRESS>:/home/pi/${workspaceFolderBasename}\""
                ],
            },
            "problemMatcher": "$msCompile"
        }
    ]
}
```

## Raspberry Pi SSH Authentication

To smooth the way for rsync file sync between your desktop machine and the Raspberry Pi you need to set up a Secure Shell (better known as SSH) Key.

### Linux and macOS

1. Create a RSA Key Pair. Open a **Terminal** window on your Linux or macOS desktop. For the purposes of this lab, choose the default options.

```bash
ssh-keygen -t rsa
```

2. Copy the Public Key to the Raspberry Pi

```bash
ssh-copy-id pi@raspberrypi.local
```

### Windows 10

1. Install [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10). I suggest you install Ubuntu 18.04

2. Open a Bash command shell (from Windows Command Prompt or Powershell type Bash)

3. Create a RSA Key Pair.

```bash
ssh-keygen -t rsa
```

4. Copy the Public Key to the Raspberry Pi.

Note, you need to use know your Raspberry Pi's IP Address as mDNS name resolution for .local does not work from WSL.

```bash
ssh-copy-id pi@xxx.xxx.xxx.xxx
```

5. Start up PuTTY and connect to your Raspberry Pi at least once and be sure to trust the device.

## Install the Visual Studio Debugger on the Raspberry Pi

Start an SSH connection to your Raspberry Pi and run the following command.

```bash
curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -r linux-arm -v latest -l ~/vsdbg
```

Read [Remote Debugging On Linux Arm](https://github.com/OmniSharp/omnisharp-vscode/wiki/Remote-Debugging-On-Linux-Arm) from visual Studio Code for more information.

## Time to Build, Deploy and Debug your .NET Core IoT App

Set a break point in your code, for example at the 15, and from Visual Studio Code click the Debug icon on the Activity bar, ensure "**Publish, Launch and Attach Debugger**" is selected in the dropdown, and click the green run icon.

Your code will build, it will be copied to your Raspberry Pi and the debugger will be attached and you can now start stepping through your code.

![Publish, Launch and Attach Debugger](./docs/build-deploy-debug.png)

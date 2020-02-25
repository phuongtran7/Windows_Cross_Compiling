# Cross-compiling C++ Projects for Raspberry Pi using CMake

## Introduction

Cross-compiling is the process that builds an application on one machine that targets another machine, in this case, we will build on a Windows 10 machine and target Raspberry Pi running Raspian Buster. One main advantage of cross-compiling is that we can make use of the full power of the host machine to compile large projects, i.e. Boost library or OpenGL projects. Another benefit to cross-compiling is that we can build everything from source ourself, eliminating the threat of mis-match binary.

## Prerequisites

1. Toolchain:
In order to build an application for the Raspberry Pi, we will need the compiler, linker and other tools that can compile for the Raspberry Pi. These set of tool are called toolchain. There are many pre-built toolchains that can be use, in this we will use pre-built Windows Toolchains for Raspberry Pi provided by SysProgs at [https://gnutoolchains.com/raspberry/](https://gnutoolchains.com/raspberry/).

2. CMake:
CMake is a cross-platform tool that is used to build, test and package software. With CMake project and portable C++ codes, we can create applications that run multiple devices and operating systems, ranging from Windows to Debian, from powerful computer to a Raspberry Pi. It's recommended to stay relatively close to the CMake release are there are many improvements that introduced in each of the release cycle. CMake can be downloaded here: [https://cmake.org/download/](https://cmake.org/download/).

3. Code Editor:
There are many code editors available, i.e. Notepad++, Visual Studio Code, Sublime Text, in this we will use Visual Studio Code as it has excellent support for CMake projects as well as intellisense and integration with Windows Subsystem for Linux (WSL). VSCode can be downloaded here: [https://code.visualstudio.com/](https://code.visualstudio.com/).

4. CMake Tools:
CMake Tools provides the native developer a full-featured, convenient, and powerful workflow for CMake-based projects in Visual Studio Code. It can be downloaded here: [https://github.com/microsoft/vscode-cmake-tools](https://github.com/microsoft/vscode-cmake-tools).

## Instructions

1. Installing the GNU Toolchains.

By default, the toolchain will be installed under `C:/SysGCC/raspberry`. This should be fine for most use case. However, we will need to synchronize our installation with the Raspberry Pi later to get all the headers, libraries that are installed on the Pi later so the size of the installation can become rather large. If this might cause the problem in the future, consider another location.

2. Adding toolchain location into environment variable:

By default, when install the toolchain, this will be done automatically. However, if we want to use multiple versions of the toolchain or for some reason, the installer did not complete correctly, we have to manually do it. In the `PATH` environment variable of the current use, add `bin` folder of toolchain to it. For example, `C:/SysGCC/raspberry/bin`.

3. Install CMake.

Installing CMake is rather straight forward just like any other Windows program.

4. Prepare CMake Toolchain file.

CMake toolchain file is a special file that instruct CMake to use the pre-built compiler instead of the compilers available on the host machine. To create a CMake toolchain file, create a `RaspberryPi.cmake` file with the content:

```
# this one is important
SET(CMAKE_SYSTEM_NAME Linux)
#this one not so much
SET(CMAKE_SYSTEM_VERSION 1)

# specify the cross compiler
SET(CMAKE_C_COMPILER
C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-gcc.exe)
SET(CMAKE_CXX_COMPILER
C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-g++.exe)
# where is the target environment
SET(CMAKE_FIND_ROOT_PATH
C:/SysGCC/raspberry/arm-linux-gnueabihf/sysroot/)

# search for programs in the build host directories
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# for libraries and headers in the target directories
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

Make sure to change the location of the C/C++ compiler according to installation folder of the GNU toolchain. Place the CMake toolchain file where it's easy to reference to, for example, we can place it directly in the toolchain installation folder: `C:/SysGCC/raspberry/RaspberryPi.cmake`.

5. Synchronize the GNU Toolchain installation with the Raspberry Pi.

Because the Raspberry Pi might have some libraries that the project depends on, we will have to synchronize our installation with the Raspiberry Pi. We can manually copy the libraries and headers by ourself, however the toolchain provide a rather convenient way to do that. Follow the instruction here: [ Updating Sysroot for Raspberry PI Cross-Toolchain](https://gnutoolchains.com/raspberry/tutorial/sysroot).

This only need to be done after installing any new library on the Raspberry Pi. We can also cross-compiling the library ourself using the same toolchain and then manually copy the library to the Raspberry Pi later so that the application can use it during runtime too.

6. Create new CMake project with Visual Studio Code and CMake Tools extension.

	6.1. Open an empty folder with VSCode and invoke the command palette (Ctrl+Shift+P) and search for `CMake: Quick Start`.

	6.2. Follow the prompt to create a new project, either executable or library. This should create a minimal hello world CMake project.

	6.3. At this moment, CMake does not know anything about the compilers that are available on our machine, we will have to tell it to scan for it. Invoke the command palette again and execute `CMake: Scan for Kits`. This will scan our `PATH` environment to determine what compiler available.

	6.4. Invoke `CMake: Edit User-Local CMake Kits` from the command palette to confirm the available compilers. In here, our GNU toolchain our be present, if not manually add it into the list:
	```javascript
	 {
	    "name": "GCC for arm-linux-gnueabihf 8.3.0",
	    "compilers": {
	      "C": "C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-gcc.exe",
	      "CXX": "C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-g++.exe"
	    },
	    "toolchainFile": "C:/SysGCC/raspberry/RaspberryPi.cmake"
  	}
	```
	Notice the `toolchainFile` variable. This variable tell CMake to read the `RaspberryPi.cmake` we prepared earlier to correctly search the sysroot and config the compiler.

	6.5. Invoke `CMake: Select a Kit` and choose our toolchain. In this case, `GCC for arm-linux-gnueabihf 8.3.0`. If everything is configured correctly, we will see something like this:
	```bash
	[cmake] -- The C compiler identification is GNU 8.3.0
	[cmake] -- The CXX compiler identification is GNU 8.3.0
	[cmake] -- Check for working C compiler: C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-gcc.exe
	[cmake] -- Check for working C compiler: C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-gcc.exe -- works
	[cmake] -- Detecting C compiler ABI info
	[cmake] -- Detecting C compiler ABI info - done
	[cmake] -- Detecting C compile features
	[cmake] -- Detecting C compile features - done
	[cmake] -- Check for working CXX compiler: C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-g++.exe
	[cmake] -- Check for working CXX compiler: C:/SysGCC/raspberry/bin/arm-linux-gnueabihf-g++.exe -- works
	[cmake] -- Detecting CXX compiler ABI info
	[cmake] -- Detecting CXX compiler ABI info - done
	[cmake] -- Detecting CXX compile features
	[cmake] -- Detecting CXX compile features - done
	[cmake] -- Configuring done
	[cmake] -- Generating done
	```

	6.6. Build the project by invoking `CMake: Build`.

	6.7. The executable should be inside the `build` folder, copy it over to the Raspberry Pi and execute it. Remember to set the correct permission when copying the file over to the Raspberry Pi.

## Cross-compile library for the Raspberry Pi.

There are many libraries that won't be available on the Raspian's default repository or the version of the library is too old to use. Compiling the library on the Raspberry Pi is a long and painful process, especially with some large library like Boost. Cross-compiling a library will help in that regard.

In this example we will cross-compile [{fmt}](https://github.com/fmtlib/fmt), a widely-used format library for C++.

1. Clone the project: `git clone https://github.com/fmtlib/fmt`
2. Open the project with Visual Studio Code.
3. Because the project use CMake as the build system, VSCode automatically recognize that and configure the project for us.
4. Choose the `GCC for arm-linux-gnueabihf 8.3.0` kit just like we did above and build the project.

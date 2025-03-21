# Build Instructions

## Index

1. [Requirements](#requirements)
1. [Building](#building)
1. [Contributing](#contributing-to-the-repository)
1. [Repository Content](#repository-content)
1. [Repository Set-Up](#repository-set-up)
1. [Windows Build](#building-on-windows)
1. [Linux Build](#building-on-linux)
1. [Android Build](#building-on-android)
1. [MacOS build](#building-on-macos)

## Requirements

1. Python >= 3.7 (3.6 may work, 3.5 and earlier is not supported)
1. CMake >= 3.10.2
1. C++ >= c++11 compiler. See platform-specific sections below for supported compiler versions.

## Building

**NOTE**: See [this](#google-test) first if you are also building the tests.

```bash
# One-time generation
mkdir build # Arbitrary build directory
cd build

# Run './scripts/update_deps.py --help' for more information
# NOTE: You can alternatively set -DUPDATE_DEPS=ON during cmake generation
#       to have a cmake target automatically run this as needed.
python3 ../scripts/update_deps.py --dir ../external --arch x64 --config debug

# NOTE: If using -DUPDATE_DEPS=ON, CMAKE_BUILD_TYPE is used to determine the build type
#       of external dependencies. For generators such as Visual Studio that usually ignore
#       CMAKE_BUILD_TYPE, it's a good idea to still set CMAKE_BUILD_TYPE in this case to control
#       the build type of dependencies. If you want a "mix" (e.g., Release dependencies, Debug VVL),
#       you will want to use `update_deps.py` manually.
cmake -C ../external/helper.cmake -DCMAKE_BUILD_TYPE=Debug ..

# Building
cmake --build . --config Debug
```

Note the `-C ../external/helper.cmake` argument passed to cmake. This is necessary when
calling the `update_deps.py` script manually. See below for more details.

These are general instructions that should "just work" on Windows and Linux. For platform-specific
build instructions, see the appropriate `<Platform> Build` section below.

## Contributing to the Repository

If you intend to contribute, the preferred work flow is for you to develop
your contribution in a fork of this repository in your GitHub account and then
submit a pull request. Please see the [CONTRIBUTING.md](CONTRIBUTING.md) file
in this repository for more details.

## Repository Content

This repository contains the source code necessary to build the Vulkan
validation layers and their tests.

### Installed Files

The `install` target installs the following files under the directory
indicated by *install_dir*:

- *install_dir*`/lib` : The Vulkan validation layer libraries
- *install_dir*`/share/vulkan/explicit_layer.d` : The Vulkan validation layer
  JSON files (Linux and MacOS)

The `uninstall` target can be used to remove the above files from the install
directory.

## Repository Set-Up

### Display Drivers

This repository does not contain a Vulkan-capable driver. You will need to
obtain and install a Vulkan driver from your graphics hardware vendor or from
some other suitable source if you intend to run Vulkan applications.

### Download the Repository

To create your local git repository:

    git clone https://github.com/KhronosGroup/Vulkan-ValidationLayers.git

### Repository Dependencies

This repository attempts to resolve some of its dependencies by using
components found from the following places, in this order:

1. CMake or Environment variable overrides (e.g., -DVULKAN_HEADERS_INSTALL_DIR)
1. LunarG Vulkan SDK, located by the `VULKAN_SDK` environment variable
1. System-installed packages, mostly applicable on Linux

Dependencies that cannot be resolved by the SDK or installed packages must be
resolved with the "install directory" override and are listed below. The
"install directory" override can also be used to force the use of a specific
version of that dependency.

Alternatively, an automated method for obtaining and installing repo dependencies
is provided, and described below, in
[Building Dependent Repositories...](#building-dependent-repositories-with-known-good-revisions)

#### Vulkan-Headers

This repository has a required dependency on the
[Vulkan Headers repository](https://github.com/KhronosGroup/Vulkan-Headers).
You must clone the headers repository and build its `install` target before
building this repository. The Vulkan-Headers repository is required because it
contains the Vulkan API definition files (registry) that are required to build
the validation layers. You must also take note of the headers' install
directory and pass it on the CMake command line for building this repository,
as described below.

#### SPIRV-Headers

This repository has a required dependency on the
[SPIRV-Headers repository](https://github.com/KhronosGroup/SPIRV-Headers).
The SPIRV-Headers repository is required because it supports components that are
required to build the validation layers. You must clone the SPIRV-Headers repository
and build its `install` target. Follow the build instructions in the SPIRV-Headers
[README.md](https://github.com/KhronosGroup/SPIRV-Headers/blob/master/README.md)
file. You must also take note of the SPIRV-headers install directory
and pass it on the CMake command line for building this repository, as
described below.

#### SPIRV-Tools

This repository has a required dependency on the
[SPIRV-Tools repository](https://github.com/KhronosGroup/SPIRV-Tools).
The SPIRV-Tools repository is required because it contains components that are
required to build the validation layers. You must clone the SPIRV-Tools repository
and build its `install` target. Follow the build instructions in the SPIRV-Tools
[README.md](https://github.com/KhronosGroup/SPIRV-Tools/blob/master/README.md)
file. You must also take note of the SPIRV-Tools install directory
and pass it on the CMake command line for building this repository, as
described below.

#### Robin Hood hashing
This repository has an optional dependency on the
[robin-hood-hashing repository](https://github.com/martinus/robin-hood-hashing).
This is a header-only reimplementation of `std::unordered_map` and `std::unordered_set`
which provides substantial performance improvements on all platforms.
You must clone this repository and build its `install` target before
building it with the CMake option `RH_STANDALONE_PROJECT` set to `OFF`
OR set the Vulkan-ValidationLayers CMake option `USE_ROBIN_HOOD_HASHING` to `OFF`.

#### glslang

The validation layer tests depend on the
[glslang repository](https://github.com/KhronosGroup/glslang).
You must clone the glslang repository
and build its `install` target. Follow the build instructions in the glslang
[README.md](https://github.com/KhronosGroup/glslang/blob/master/README.md)
file. You must also take note of the glslang install directory
and pass it on the CMake command line for building this repository, as
described below.

#### Google Test

The validation layer tests depend on the
[Google Test](https://github.com/google/googletest). To build the tests, pass the `-DBUILD_TESTS=ON` option when
generating the project:
```bash
cmake ... -DUPDATE_DEPS=ON -DBUILD_TESTS=ON ...
```
This will ensure googletest is downloaded and the appropriate version is used.

#### Vulkan-Loader

The validation layer tests depend on the Vulkan loader when they execute and
so a loader is needed only if the tests are built and run.

A loader can be used from an installed LunarG SDK, an installed Linux package,
or from a driver installation on Windows.

If a loader is not available from any of these methods and/or it is important
to use a loader built from a repository, then you must build the
[Vulkan-Loader repository](https://github.com/KhronosGroup/Vulkan-Loader.git)
with its install target. Take note of its install directory location and pass
it on the CMake command line for building this repository, as described below.

If you do not intend to run the tests, you do not need a Vulkan loader.

### Build and Install Directories

A common convention is to place the build directory in the top directory of
the repository with a name of `build` and place the install directory as a
child of the build directory with the name `install`. The remainder of these
instructions follow this convention, although you can use any name for these
directories and place them in any location (see option `--dir` in the
[notes](#notes)).

### Building Dependent Repositories with Known-Good Revisions

There is a Python utility script, `scripts/update_deps.py`, that you can use to
gather and build the dependent repositories mentioned above. This script uses
information stored in the `scripts/known_good.json` file to check out dependent
repository revisions that are known to be compatible with the revision of this
repository that you currently have checked out. As such, this script is useful
as a quick-start tool for common use cases and default configurations.

For all platforms, start with:

    git clone git@github.com:KhronosGroup/Vulkan-ValidationLayers.git
    cd Vulkan-ValidationLayers
    mkdir build
    cd build

For 64-bit Linux and MacOS, continue with:

    ../scripts/update_deps.py
    cmake -C helper.cmake ..
    cmake --build .

For 64-bit Windows, continue with:

    ..\scripts\update_deps.py --arch x64
    cmake -A x64 -C helper.cmake ..
    cmake --build .

For 32-bit Windows, continue with:

    ..\scripts\update_deps.py --arch Win32
    cmake -A Win32 -C helper.cmake ..
    cmake --build .

Please see the more detailed build information later in this file if you have
specific requirements for configuring and building these components.

#### Notes

- You may need to adjust some of the CMake options based on your platform. See
  the platform-specific sections later in this document.
- When using update_deps.py to change architectures (x64, Win32...)
  or build configurations (debug, release...) it is strongly recommended to
  add the '--clean-repo' parameter. This ensures compatibility among dependent
  components.
  dependent components will produce consistent build artifacts.
- The `update_deps.py` script fetches and builds the dependent repositories in
  the current directory when it is invoked. In this case, they are built in
  the `build` directory.
- The `build` directory is also being used to build this
  (Vulkan-ValidationLayers) repository. But there shouldn't be any conflicts
  inside the `build` directory between the dependent repositories and the
  build files for this repository.
- The `--dir` option for `update_deps.py` can be used to relocate the
  dependent repositories to another arbitrary directory using an absolute or
  relative path.
- The `update_deps.py` script generates a file named `helper.cmake` and places
  it in the same directory as the dependent repositories (`build` in this
  case). This file contains CMake commands to set the CMake `*_INSTALL_DIR`
  variables that are used to point to the install artifacts of the dependent
  repositories. You can use this file with the `cmake -C` option to set these
  variables when you generate your build files with CMake. This lets you avoid
  entering several `*_INSTALL_DIR` variable settings on the CMake command line.
- If using "MINGW" (Git For Windows), you may wish to run
  `winpty update_deps.py` in order to avoid buffering all of the script's
  "print" output until the end and to retain the ability to interrupt script
  execution.
- Please use `update_deps.py --help` to list additional options and read the
  internal documentation in `update_deps.py` for further information.
- You can build against different C++ standards by setting the
  `VVL_CPP_STANDARD` option at cmake generation time. Current code is writtent
  to compile under C++11.

### Generated source code

This repository contains generated source code in the `layers/generated`
directory which is not intended to be modified directly. Instead, changes should be
made to the corresponding generator in the `scripts` directory. The source files can
then be regenerated using `scripts/generate_source.py`:

    python3 scripts/generate_source.py PATH_TO_VULKAN_HEADERS_REGISTRY_DIR PATH_TO_SPIRV_HEADERS_GRAMMAR_DIR

    // Example
    python3 scripts/generate_source.py external/Vulkan-Headers/registry/ external/SPIRV-Headers/include/spirv/unified1/

A helper CMake target `VulkanVL_generated_source` is also provided to simplify
the invocation of `scripts/generate_source.py` from the build directory:

    cmake --build . --target VulkanVL_generated_source

If Python is found on your system, the Vulkan Registry and SPIRV-Headers *must* be present!

### Build Options

When generating native platform build files through CMake, several options can
be specified to customize the build. Some of the options are binary on/off
options, while others take a string as input. The following is a table of all
on/off options currently supported by this repository:

| Option | Platform | Default | Description |
| ------ | -------- | ------- | ----------- |
| BUILD_LAYERS | All | `ON` | Controls whether or not the validation layers are built. |
| BUILD_LAYER_SUPPORT_FILES | All | `OFF` | Controls whether or not layer support files are installed. |
| BUILD_TESTS | All | `OFF` | Controls whether or not the validation layer tests are built. |
| INSTALL_TESTS | All | `OFF` | Controls whether or not the validation layer tests are installed. This option is only available when `BUILD_TESTS` is `ON` |
| BUILD_WERROR | All | `ON` | Controls whether or not to treat compiler warnings as errors. |
| BUILD_WSI_XCB_SUPPORT | Linux | `ON` | Build the components with XCB support. |
| BUILD_WSI_XLIB_SUPPORT | Linux | `ON` | Build the components with Xlib support. |
| BUILD_WSI_WAYLAND_SUPPORT | Linux | `ON` | Build the components with Wayland support. |

### CCACHE

There are 2 methods to enable CCACHE:

1.) Set environment variables

```bash
# Requires CMake 3.17 (https://cmake.org/cmake/help/latest/envvar/CMAKE_LANG_COMPILER_LAUNCHER.html)
export CMAKE_CXX_COMPILER_LAUNCHER=/usr/bin/ccache
export CMAKE_C_COMPILER_LAUNCHER=/usr/bin/ccache
```

2.) Pass in cache variables

```
cmake ... -D CMAKE_CXX_COMPILER_LAUNCHER=/usr/bin/ccache -D CMAKE_C_COMPILER_LAUNCHER=/usr/bin/ccache
```

### EXPORT_COMPILE_COMMANDS

There are 2 methods to enable exporting compile commands:

1.) Set environment variables

```bash
# Requires CMake 3.17 (https://cmake.org/cmake/help/latest/envvar/CMAKE_EXPORT_COMPILE_COMMANDS.html)
export CMAKE_EXPORT_COMPILE_COMMANDS=ON
```

2.) Pass in cache variables

```
cmake ... -D CMAKE_EXPORT_COMPILE_COMMANDS=ON
```

NOTE: Modern tools will generally enable exporting compile commands for you (e.g. VSCode).
Also `CMAKE_EXPORT_COMPILE_COMMANDS` is implemented only by Makefile and Ninja generators. For other generators, this option is ignored.

## Building On Windows

### Windows Development Environment Requirements

- Windows
  - Any Personal Computer version supported by Microsoft
- Microsoft [Visual Studio](https://www.visualstudio.com/)
  - Versions
    - [2022](https://www.visualstudio.com/vs/downloads/)
    - [2015-2019](https://www.visualstudio.com/vs/older-downloads/)
  - The Community Edition of each of the above versions is sufficient, as
    well as any more capable edition.
- [CMake 3.10.2](https://cmake.org/files/v3.10/cmake-3.10.2-win64-x64.zip) is the minimum CMake version supported.  [CMake 3.19.3](https://cmake.org/files/v3.19/cmake-3.19.3-win64-x64.zip) is recommended.
  - Use the installer option to add CMake to the system PATH
- Git Client Support
  - [Git for Windows](http://git-scm.com/download/win) is a popular solution
    for Windows
  - Some IDEs (e.g., [Visual Studio](https://www.visualstudio.com/),
    [GitHub Desktop](https://desktop.github.com/)) have integrated
    Git client support

### Windows Build - Microsoft Visual Studio

The general approach is to run CMake to generate the Visual Studio project
files. Then either run CMake with the `--build` option to build from the
command line or use the Visual Studio IDE to open the generated solution and
work with the solution interactively.

#### Windows Quick Start

    cd Vulkan-ValidationLayers
    mkdir build
    cd build
    cmake -A x64 -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
                 ..
    cmake --build .

The above commands instruct CMake to find and use the default Visual Studio
installation to generate a Visual Studio solution and projects for the x64
architecture. The second CMake command builds the Debug (default)
configuration of the solution.

See below for the details.

#### Use `CMake` to Create the Visual Studio Project Files

Change your current directory to the top of the cloned repository directory,
create a build directory and generate the Visual Studio project files:

    cd Vulkan-ValidationLayers
    mkdir build
    cd build
    cmake -A x64 -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir
                 -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
                 ..

> Note: The `..` parameter tells `cmake` the location of the top of the
> repository. If you place your build directory someplace else, you'll need to
> specify the location of the repository top differently.

The `-A` option is used to select either the "Win32" or "x64" architecture.

If a generator for a specific version of Visual Studio is required, you can
specify it with the `-G` switch.  For example, to specify Visual Studio 2022, use:

    64-bit: -G "Visual Studio 17 2022"
    32-bit: -G "Visual Studio 17 2022" -A Win32

NOTE:
- By default VS2019 and higher target "x64"
- By default VS2017 and lower target "Win32"

Check the [official CMake documentation](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html#visual-studio-generators) for further information on Visual Studio generators.

When generating the project files, the absolute path to a Vulkan-Headers
install directory must be provided. This can be done by setting the
`VULKAN_HEADERS_INSTALL_DIR` environment variable or by setting the
`VULKAN_HEADERS_INSTALL_DIR` CMake variable with the `-D` CMake option. In
either case, the variable should point to the installation directory of a
Vulkan-Headers repository built with the install target.

When generating the project files, the absolute path to a glslang install
directory must be provided. This can be done by setting the
`GLSLANG_INSTALL_DIR` environment variable or by setting the
`GLSLANG_INSTALL_DIR` CMake variable with the `-D` CMake option. In either
case, the variable should point to the installation directory of a glslang
repository built with the install target.

The above steps create a Windows solution file named
`Vulkan-ValidationLayers.sln` in the build directory.

At this point, you can build the solution from the command line or open the
generated solution with Visual Studio.

#### Build the Solution From the Command Line

While still in the build directory:

    cmake --build .

to build the Debug configuration (the default), or:

    cmake --build . --config Release

to make a Release build.

#### Build the Solution With Visual Studio

Launch Visual Studio and open the "Vulkan-ValidationLayers.sln" solution file
in the build folder. You may select "Debug" or "Release" from the Solution
Configurations drop-down list. Start a build by selecting the Build->Build
Solution menu item.

#### Windows Install Target

The CMake project also generates an "install" target that you can use to copy
the primary build artifacts to a specific location using a "bin, include, lib"
style directory structure. This may be useful for collecting the artifacts and
providing them to another project that is dependent on them.

The default location is `$CMAKE_BINARY_DIR\install`, but can be changed with
the `CMAKE_INSTALL_PREFIX` variable when first generating the project build
files with CMake.

You can build the install target from the command line with:

    cmake --build . --config Release --target install

or build the `INSTALL` target from the Visual Studio solution explorer.

#### Using a Loader Built from a Repository

If you do need to build and use your own loader, build the Vulkan-Loader
repository with the install target and modify your CMake invocation to add the
location of the loader's install directory:

    cmake -A x64 -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DVULKAN_LOADER_INSTALL_DIR=absolute_path_to_install_dir \
                 -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
                 -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
                 ..

### Windows Tests and Demos

After making any changes to the repository, you should perform some quick
sanity tests, especially the included layer validation tests (vk_layer_validation_tests).
In addition, running sample applications such as the
[vkcube demo](https://www.github.com/KhronosGroup/Vulkan-Tools.git)
with validation enabled is advised.

### Windows Notes

#### Using The Vulkan Loader Library in this Repository on Windows

Vulkan programs must be able to find and use the Vulkan loader
(`vulkan-1.dll`) library as well as any other libraries the program requires.
One convenient way to do this is to copy the required libraries into the same
directory as the program. If you provided a loader repository location via the
`VULKAN_LOADER_INSTALL_DIR` variable, the projects in this solution copy the
Vulkan loader library and the "googletest" libraries to the
`build\tests\Debug` or the `build\tests\Release` directory, which is where the
test executables are found, depending on what configuration you built. (The
layer validation tests use the "googletest" testing framework.)

## Building On Linux

### Linux Build Requirements

This repository has been built and tested on the two most recent Ubuntu LTS
versions. Currently, the oldest supported version is Ubuntu 16.04, meaning
that the minimum officially supported C++11 compiler version is GCC 5.4.0,
although earlier versions may work. It should be straightforward to adapt this
repository to other Linux distributions.

[CMake 3.10.2](https://cmake.org/files/v3.10/cmake-3.10.2-Linux-x86_64.tar.gz) is recommended.

#### Required Package List

    sudo apt-get install git build-essential libx11-xcb-dev \
        libxkbcommon-dev libwayland-dev libxrandr-dev \
        libegl1-mesa-dev

##### Required package for Ubuntu 18.04 users

For python2 users

    sudo apt install python-distutils

or for python3 users

	sudo apt install python3-distutils

### Linux Build

The general approach is to run CMake to generate make files. Then either run
CMake with the `--build` option or `make` to build from the command line.

#### Linux Quick Start

    cd Vulkan-ValidationLayers
    mkdir build
    cd build
    cmake -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
          -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
          ..
    make

See below for the details.

#### Use CMake to Create the Make Files

Change your current directory to the top of the cloned repository directory,
create a build directory and generate the make files.

    cd Vulkan-ValidationLayers
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Debug \
          -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
          -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
          -DCMAKE_INSTALL_PREFIX=install ..

> Note: The `..` parameter tells `cmake` the location of the top of the
> repository. If you place your `build` directory someplace else, you'll need
> to specify the location of the repository top differently.

Use `-DCMAKE_BUILD_TYPE` to specify a Debug or Release build.

When generating the project files, the absolute path to a Vulkan-Headers
install directory must be provided. This can be done by setting the
`VULKAN_HEADERS_INSTALL_DIR` environment variable or by setting the
`VULKAN_HEADERS_INSTALL_DIR` CMake variable with the `-D` CMake option. In
either case, the variable should point to the installation directory of a
Vulkan-Headers repository built with the install target.

When generating the project files, the absolute path to a glslang install
directory must be provided. This can be done by setting the
`GLSLANG_INSTALL_DIR` environment variable or by setting the
`GLSLANG_INSTALL_DIR` CMake variable with the `-D` CMake option. In either
case, the variable should point to the installation directory of a glslang
repository built with the install target.

> Note: For Linux, the default value for `CMAKE_INSTALL_PREFIX` is
> `/usr/local`, which would be used if you do not specify
> `CMAKE_INSTALL_PREFIX`. In this case, you may need to use `sudo` to install
> to system directories later when you run `make install`.

#### Build the Project

You can just run `make` to begin the build.

To speed up the build on a multi-core machine, use the `-j` option for `make`
to specify the number of cores to use for the build. For example:

    make -j4

You can also use

    cmake --build .

### Linux Notes

#### WSI Support Build Options

By default, the repository components are built with support for the
Vulkan-defined WSI display servers: Xcb, Xlib, and Wayland. It is recommended
to build the repository components with support for these display servers to
maximize their usability across Linux platforms. If it is necessary to build
these modules without support for one of the display servers, the appropriate
CMake option of the form `BUILD_WSI_xxx_SUPPORT` can be set to `OFF`.

#### Linux Install to System Directories

Installing the files resulting from your build to the systems directories is
optional since environment variables can usually be used instead to locate the
binaries. There are also risks with interfering with binaries installed by
packages. If you are certain that you would like to install your binaries to
system directories, you can proceed with these instructions.

Assuming that you've built the code as described above and the current
directory is still `build`, you can execute:

    sudo make install

This command installs files to `/usr/local` if no `CMAKE_INSTALL_PREFIX` is
specified when creating the build files with CMake:

- `/usr/local/lib`:  Vulkan layers shared objects
- `/usr/local/share/vulkan/explicit_layer.d`:  Layer JSON files

You may need to run `ldconfig` in order to refresh the system loader search
cache on some Linux systems.

You can further customize the installation location by setting additional
CMake variables to override their defaults. For example, if you would like to
install to `/tmp/build` instead of `/usr/local`, on your CMake command line
specify:

    -DCMAKE_INSTALL_PREFIX=/tmp/build

Then run `make install` as before. The install step places the files in
`/tmp/build`. This may be useful for collecting the artifacts and providing
them to another project that is dependent on them.

See the CMake documentation for more details on using these variables to
further customize your installation.

Also see the `LoaderAndLayerInterface` document in the `loader` folder of the
Vulkan-Loader repository for more information about loader and layer
operation.

#### Linux Uninstall

To uninstall the files from the system directories, you can execute:

    sudo make uninstall

#### Linux Tests

Build and run the `vk_layer_validation_tests`, in the tests subdirectory.

#### Linux 32-bit support

Usage of this repository's contents in 32-bit Linux environments is not
officially supported. However, since this repository is supported on 32-bit
Windows, these modules should generally work on 32-bit Linux.

Here are some notes for building 32-bit targets on a 64-bit Ubuntu "reference"
platform:

If not already installed, install the following 32-bit development libraries:

`gcc-multilib g++-multilib libx11-dev:i386`

This list may vary depending on your distribution and which windowing systems
you are building for.

Set up your environment for building 32-bit targets:

```bash
export ASFLAGS=--32
export CFLAGS=-m32
export CXXFLAGS=-m32
export PKG_CONFIG_LIBDIR=/usr/lib/i386-linux-gnu
```

Again, your PKG_CONFIG configuration may be different, depending on your distribution.

Finally, rebuild the repository using `cmake` and `make`, as explained above.

#### Using the new layers

```bash
export VK_LAYER_PATH=<path to your repository root>/build/layers
```

You can run the `vkcube` or `vulkaninfo` applications from the Vulkan-Tools
repository to see which driver, loader and layers are being used.

## Building On Android

Install the required tools for Linux and Windows covered above, then add the
following.

### Android Build Requirements

Note that the minimum supported Android SDK API Level is 26, revision
level 3.

NDK r20 or greater required

- Install [Android Studio 2.3](https://developer.android.com/studio/index.html)
  or later.
- From the "Welcome to Android Studio" splash screen, add the following
  components using Configure > SDK Manager:
  - SDK Platforms > Android 8.0.0 and newer
  - SDK Tools > Android SDK Build-Tools
  - SDK Tools > Android SDK Platform-Tools
  - SDK Tools > Android SDK Tools
  - SDK Tools > NDK

#### Android Hardware Buffer support

The Validation Layers by default build and release for Android 26 (Android Oreo). While Vulkan is supported in Android 24 and 25, there is no AHardwareBuffer support. To build a version of the Validation Layers for use with Android that will not require AHB support, simply addjust the `APP_PLATFORM` in [build-android/jni/Application.mk](build-android/jni/Application.mk)

```patch
-APP_PLATFORM := android-26
+APP_PLATFORM := android-24
```

#### Add Android specifics to environment

For each of the below, you may need to specify a different build-tools
version, as Android Studio will roll it forward fairly regularly.

On Linux:

```bash
export ANDROID_SDK_HOME=$HOME/Android/sdk
export ANDROID_NDK_HOME=$HOME/Android/sdk/ndk-bundle
export PATH=$ANDROID_SDK_HOME:$PATH
export PATH=$ANDROID_NDK_HOME:$PATH
# Where X.Y.Z is the most recent version number inside the build-tools directory.
export PATH=$ANDROID_SDK_HOME/build-tools/X.Y.Z:$PATH
```

On Windows:

    set ANDROID_SDK_HOME=%LOCALAPPDATA%\Android\sdk
    set ANDROID_NDK_HOME=%LOCALAPPDATA%\Android\sdk\ndk-bundle
    set PATH=%LOCALAPPDATA%\Android\sdk\ndk-bundle;%PATH%

On OSX:

```bash
export ANDROID_SDK_HOME=$HOME/Library/Android/sdk
export ANDROID_NDK_HOME=$HOME/Library/Android/sdk/ndk-bundle
export PATH=$ANDROID_NDK_HOME:$PATH
# Where X.Y.Z is the most recent version number inside the build-tools directory.
export PATH=$ANDROID_SDK_HOME/build-tools/X.Y.Z:$PATH
```

Note: If `jarsigner` is missing from your platform, you can find it in the
Android Studio install or in your Java installation. If you do not have Java,
you can get it with something like the following:

  sudo apt-get install openjdk-8-jdk

#### Additional OSX System Requirements

Tested on OSX version 10.13.3

Setup Homebrew and components

- Follow instructions on [brew.sh](http://brew.sh) to get Homebrew installed.

      /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

- Ensure Homebrew is at the beginning of your PATH:

```
export PATH=/usr/local/bin:$PATH
```

- Add packages with the following:

      brew install python

### Android Build

There are two options for building the Android layers. Either using the SPIRV
tools provided as part of the Android NDK, or using upstream sources. To build
with SPIRV tools from the NDK, remove the build-android/third_party directory
created by running update_external_sources_android.sh, (or avoid running
update_external_sources_android.sh). Use the following script to build
everything in the repository for Android, including validation layers, tests,
demos, and APK packaging: This script does retrieve and use the upstream SPRIV
tools.

    cd build-android
    ./build_all.sh

> **NOTE:** By default, the `build_all.sh` script will build for all Android ABI variations. To **speed up the build time** if you know your target(s), set `APP_ABI` in both [build-android/jni/Application.mk](build-android/jni/Application.mk) and [build-android/jni/shaderc/Application.mk](build-android/jni/shaderc/Application.mk) to the desired [Android ABI](https://developer.android.com/ndk/guides/application_mk#app_abi)

Resulting validation layer binaries will be in build-android/libs. Test and
demo APKs can be installed on production devices with:

    ./install_all.sh [-s <serial number>]

Note that there are no equivalent scripts on Windows yet, that work needs to
be completed. The following per platform commands can be used for layer only
builds:

#### Linux and OSX

Follow the setup steps for Linux or OSX above, then from your terminal:

    cd build-android
    ./update_external_sources_android.sh --no-build
    ndk-build -j4

#### Windows

Follow the setup steps for Windows above, then from Developer Command Prompt
for VS2015:

    cd build-android
    update_external_sources_android.bat
    ndk-build

### Android Tests and Demos

After making any changes to the repository you should perform some quick
sanity tests, including the layer validation tests and the vkcube
demo with validation enabled.

#### Run Layer Validation Tests

Use the following steps to build, install, and run the layer validation tests
for Android:

    cd build-android
    ./build_all.sh
    adb install -r bin/VulkanLayerValidationTests.apk
    adb shell am start com.example.VulkanLayerValidationTests/android.app.NativeActivity

Alternatively, you can use the test_APK script to install and run the layer
validation tests:

    test_APK.sh -s <serial number> -p <plaform name> -f <gtest_filter>

To view to logging while running in a separate terminal run

    adb logcat -c && adb logcat *:S VulkanLayerValidationTests

## Building on MacOS

### MacOS Build Requirements

Tested on OSX version 10.12

NOTE: To force the OSX version set the environment variable [MACOSX_DEPLOYMENT_TARGET](https://cmake.org/cmake/help/latest/envvar/MACOSX_DEPLOYMENT_TARGET.html) when building VVL and it's dependencies.

[CMake 3.10.2](https://cmake.org/files/v3.10/cmake-3.10.2-Darwin-x86_64.tar.gz) is recommended.

Setup Homebrew and components

- Follow instructions on [brew.sh](http://brew.sh) to get Homebrew installed.

      /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

- Ensure Homebrew is at the beginning of your PATH:

```
export PATH=/usr/local/bin:$PATH
```

- Add packages with the following (may need refinement)

      brew install python python3 git

### Clone the Repository

Clone the Vulkan-ValidationLayers repository:

    git clone https://github.com/KhronosGroup/Vulkan-ValidationLayers.git

### MacOS build

#### CMake Generators

This repository uses CMake to generate build or project files that are then
used to build the repository. The CMake generators explicitly supported in
this repository are:

- Unix Makefiles
- Xcode

#### Building with the Unix Makefiles Generator

This generator is the default generator, so all that is needed for a debug
build is:

    mkdir build
    cd build
    cmake -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
          -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
          -DCMAKE_BUILD_TYPE=Debug ..
    make

To speed up the build on a multi-core machine, use the `-j` option for `make`
to specify the number of cores to use for the build. For example:

    make -j4

#### Building with the Xcode Generator

To create and open an Xcode project:

    mkdir build-xcode
    cd build-xcode
    cmake -DVULKAN_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DGLSLANG_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_HEADERS_INSTALL_DIR=absolute_path_to_install_dir \
          -DSPIRV_TOOLS_INSTALL_DIR=absolute_path_to_install_dir \
          -DROBIN_HOOD_HASHING_INSTALL_DIR=absolute_path_to_install_dir \
          -GXcode ..
    open VULKAN.xcodeproj

Within Xcode, you can select Debug or Release builds in the Build Settings of the project.

#### Using the new layers on MacOS

```
export VK_LAYER_PATH=<path to your repository root>/build/layers
```

You can run the `vulkaninfo` applications from the Vulkan-Tools repository to
see which driver, loader and layers are being used.

### MacOS Tests

After making any changes to the repository, you should build and run the included
vk_layer_validation_tests located in the tests subdirectory.

These test require a manual path to an ICD to run properly on MacOS.

You can use:

- MoltenVK ICD
- Mock ICD

#### Using MoltenVK ICD

Clone and build the [MoltenVK](https://github.com/KhronosGroup/MoltenVK) repository.

You will have to direct the loader from Vulkan-Loader to the MoltenVK ICD:

```bash
export VK_ICD_FILENAMES=<path to MoltenVK repository>/Package/Latest/MoltenVK/macOS/MoltenVK_icd.json
```

#### Using Mock ICD

Clone and build the [Vulkan-Tools](https://github.com/KhronosGroup/Vulkan-Tools) repository.

You will have to direct the loader from Vulkan-Loader to the Mock ICD:

```bash
export VK_ICD_FILENAMES=<path to Vulkan-Tools repository>/build/icd/VkICD_mock_icd.json
```

#### Running the Tests

In a terminal, change to the build/tests directory and run the vk_layer_validation_tests.

Further testing and sanity checking can be achieved by running the vkcube and
vulkaninfo applications in the
[Vulkan-Tools](https://github.com/KhronosGroup/Vulkan-Tools)
repository.

Note that MoltenVK is still adding Vulkan features and some tests may fail.

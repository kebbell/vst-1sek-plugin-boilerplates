# vst-plugin-boilerplate-1sek-main
# 1seks-to-be-stripped-back-vst
## Checklist before releasing plugin

 * register unique ID for the plugin in `./src/global.h`
 * register unique UIDs for the controller defined in `./src/global.h`
 * replace instances of `__PLUGIN_NAME__` with your plugins name (without spaces)
 * everything described in the README below w/ regards to plugin types

# __PLUGIN_NAME__

__PLUGIN_NAME__ is a VST/AU plug-in which provides... [description goes here]

## Build instructions

The project uses [CMake](https://cmake.org) to generate the Makefiles and has been built and tested on macOS, Windows 10 and Linux (Ubuntu).

### Environment setup

Apart from requiring _CMake_ and a C(++) compiler such as _Clang_ or _MSVC_, the only other dependency is the [VST SDK from Steinberg](https://www.steinberg.net/en/company/developers.html) (the projects latest update requires SDK version 3.7.11).

#### Setting up the easy way : installing a local version of the Steinberg SDK

You can instantly retrieve and build the SDK using the following commands.

##### Installation on Unix:

```
sh setup.sh --platform PLATFORM
```

Where optional flag _--platform_ can be either `mac` or `linux` (defaults to linux).

Linux users might be interested in the [required packages](https://steinbergmedia.github.io/vst3_dev_portal/pages/Getting+Started/How+to+setup+my+system.html#for-linux).

##### Installation on Windows:

```
setup.bat
```

This will create a (Git ignored) subfolder in this repository folder with a prebuilt Steinberg SDK.

#### Setting up the flexible way : pointing towards an external SDK build / supporting VST2

In case you wish to use a different SDK version (for instance to reuse an existing build elsewhere on your computer or to
target VST2 builds), you can invoke all build scripts by providing the `VST3_SDK_ROOT` environment variable, like so:

```
VST3_SDK_ROOT=/path/to/prebuilt/VST3_SDK sh build.sh
```

After downloading the Steinberg SDK you must generate a release build of its sources. To do this, execute the following commands from the root of the Steinberg SDK folder:

```
cd vst3sdk
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
```

The result being that `{VST3_SDK_ROOT}/build/lib/Release/` will contain the Steinberg VST libraries required to build the plugin.

In case you intend to build VST2 versions as well, keep in mind that as of SDK 3.6.11, Steinberg no longer packages the required `/pluginterfaces/vst2.x`-folder inside the vst3sdk folder. If you wish to build a VST2 plugin, copying the folder from an older SDK version _could_ work (verified 3.6.9. `vst2.x` folders to work with SDK 3.7.11), though be aware that you _need a license to target VST2_. You can view [Steinbergs rationale on this decision here](https://helpcenter.steinberg.de/hc/en-us/articles/4409561018258-VST-2-Discontinued).

To prepare for building VST2 versions of the plugin, run the following from the root of the Steinberg SDK folder (run the _.bat_ version instead of the _.sh_ version on Windows) prior to building the library:

```
./copy_vst2_to_vst3_sdk.sh
```

And if you are running Linux, you can easily resolve all dependencies by first running the following from the root of the Steinberg SDK folder:

```
./tools/setup_linux_packages_for_vst3sdk.sh
```

### Building the plugin

See the provided shell scripts. The build output will be stored in `./build/VST3/__PLUGIN_NAME__.vst3` as well as symbolically linked to your systems VST-plugin folder (on Unix).

#### Compiling on Unix systems:

```
sh build.sh --type TYPE
```

Where optional flag _--type_ can be either `vst3`, `vst2` or `au` (defaults to vst3)*.

#### Compiling on Windows:

Assuming the Visual Studio Build Tools have been installed:

```
build.bat
```

Where you can optionally append `vst2` to the command to build a VST2* plugin.

_*As mentioned in the "setup" section, VST2 builds are not supported out-of-the-box._

## On compatibility

### Compiling for both 32-bit and 64-bit architectures

Depending on your host software having 32-bit or 64-bit support (with the latter targeting either Intel or ARM), you can choose to compile for a wider range of architectures. To do so, updating the build shell scripts/batch files to contain the following:

**macOS:**

```
cmake -"DCMAKE_OSX_ARCHITECTURES=x86_64;arm64;i1386" ..
```

Which will allow you to compile a single, "fat" binary that supports all architectures (Intel, ARM and legacy 32-bit Intel). Note
that by default compilation is for 64-bit architecture for both Intel and ARM CPU's, _you can likely ignore this section_.

**Windows:**

```
cmake.exe -G "Visual Studio 16 2019" -A Win64 -S .. -B "build64"
cmake.exe --build build64 --config Release

cmake.exe -G "Visual Studio 16 2019" -A Win32 -S .. -B "build32"
cmake.exe --build build32 --config Release
```

Which is a little more cumbersome as you compile separate binaries for the separate architectures.

Note that the above also needs to be done when building the Steinberg SDK (which for the Windows build implies that a separate build is created for each architecture).

While both macOS and Windows have been fully 64-bit for the past versions, building for 32-bit provides the best backward
compatibility for older OS versions. Musicians are known to keep working systems at the cost of not
running an up to date system... _still, you can likely ignore this section_.

### Build as Audio Unit (macOS only)

For this you will need a little extra preparation while building Steinberg SDK as you will need the
"[CoreAudio SDK](https://developer.apple.com/library/archive/samplecode/CoreAudioUtilityClasses/Introduction/Intro.html)" and XCode. Execute the following instructions to build the SDK with Audio Unit support,
providing the appropriate path to the actual installation location of the CoreAudio SDK:

```
sh setup.sh --platform mac --coresdk /path/to/CoreAudioUtilityClasses/CoreAudio
```

After which you can run the build script like so:

```
sh build.sh --type au
```

The Audio Unit component will be located in `./build/bin/Release/__PLUGIN_NAME__ AUv3.app`

You can validate the Audio Unit using Apple's _auval_ utility, by running `auval -v aufx dist IGOR` on the command line (reflecting the values defined in `audiounitconfig.h`). Note that there is the curious behaviour that you might need to reboot before the plugin shows up, though you can force a flush of the Audio Unit cache at runtime by running `killall -9 AudioComponentRegistrar`.

In case of errors you can look for instances of [kAudioUnitErr](https://www.osstatus.com/search/results?platform=all&framework=all&search=kaudiouniterr)

### Running the plugin

You can copy the build output into your system VST(3) folder and run it directly in a VST host / DAW of your choice.

When debugging, you can also choose to run the plugin against Steinbergs validator and editor host utilities:

```
{VST3_SDK_ROOT}/build/bin/validator  build/VST3/__PLUGIN_NAME__.vst3
{VST3_SDK_ROOT}/build/bin/editorhost build/VST3/__PLUGIN_NAME__.vst3
```

### Signing the plugin on macOS

You will need to have your code signing set up appropriately. Assuming you have set up your Apple Developer account, you can find your signing identity like so:

```
security find-identity -p codesigning -v 
```

From which you can take your name and team id and pass them to the build script like so:

```
sh build.sh --team_id TEAM_ID --identity "YOUR_NAME"
```
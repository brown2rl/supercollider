SuperCollider 3.8 for macOS (OS X)
==================================

These are installation and build instructions for the macOS version of James McCartney's SuperCollider synthesis engine (scsynth) and programming language (sclang).

Pre-compiled releases are available for download at:

https://github.com/SuperCollider/SuperCollider/releases



Table of contents
-----------------

 * Prerequisites
 * Obtaining the source code
 * Build instructions
 * Diagnosing build problems
 * Frequently used cmake settings
 * Using cmake with Xcode or QtCreator
 * Building without Qt or the IDE
 * sclang and scynth executables

Executables
-----------

`SuperCollider.app` is the IDE (integrated development environment) for writing and executing SuperCollider code.

Inside that application's folder (`SuperCollider.app/Contents/MacOS/`) are the two executables that make up supercollider itself:

`sclang` - the language interpreter including Qt gui
`scsynth` - the audio engine


Prerequisites:
-------------

- **Xcode** can be installed free from the Apple App Store or downloaded from: http://developer.apple.com

  - Xcode 5 may work
  - Xcode 6 is known to work - it requires a Mac running macOS version 10.9.4 or later or 10.10
  - Later versions should definitely work

- **Xcode command line tools** must be installed - after installing Xcode, this can be done from the Xcode preferences or from the command line:
  `xcode-select --install`
- **homebrew** is recommended to install required libraries
  See http://brew.sh for installation instructions.
- **git, cmake, libsndfile, readline, and qt5.5**, installed via homebrew:
  `brew install git cmake readline qt55`

  *Note*: As of this writing the latest stable Qt is version 5.8. SC depends on Qt5WebKit, which was dropped from the binary distributions of Qt since version 5.6 (functionally replaced by Qt5WebEngine). Therefore you cannot simply install the latest Qt5 via homebrew and rely on the defaults. You can either install qt55 (accessed at /usr/local/opt/qt@55) and replace `brew --prefix qt5` by `brew --prefix qt55` in
  the build instructions below, or install the current Qt version with the option `--with-qtwebkit`. As this is a non-standard install, brew will build qt5 locally (go drink a coffee).

  If you already had Qt5, and and your build broke after an update, or if you need several Qt5 installs, you can set the version to be used by default with `brew switch`. (for example `brew switch qt5 5.5.1_2`, you can also "freeze" the Qt5 version with `brew pin`).


Obtaining the source code
-------------------------

**Note** Please do not use non-ASCII characters (above code point 127) in your
SuperCollider program path (i.e. the names of the folders containing SuperCollider).
Doing so will break options to open class or method files.

SC is on Github: https://github.com/SuperCollider/SuperCollider

Get a copy of the source code:

    git clone --recursive https://github.com/SuperCollider/SuperCollider.git

`--recursive` specifies that it should also clone the git submodules.

Build instructions
------------------

    cd SuperCollider
    mkdir -p build
    cd build
    cmake -G Xcode -DCMAKE_PREFIX_PATH=`brew --prefix qt5`  ..
    cmake --build . --target install --config RelWithDebInfo

If successful this will build the application into `build/Install/SuperCollider/`

You can see the available build options with ```cmake -LH```.

To install, you may move this to /Applications or use it in place from the build directory.

**Note**: You can also open the produced SuperCollider.xcodeproj in Xcode, and build the "Install" scheme in place of the last step. Do make sure you run the previous configuration steps.

#### Step by step explanation of the Build instructions:

##### Create a `build` folder if one doesn't already exist:

    mkdir -p build
    cd build

##### Prepare for building by making a configuration file:

    cmake -G Xcode -DCMAKE_PREFIX_PATH=`brew --prefix qt5`  ..

(The `..` at the end is easy to miss. Don't forget it!)

This specifies to cmake that we will be using Xcode to build. It also specifies the location of qt so that the complier/linker can find it (note that you might have to set `qt55` instead of `qt5`, depending on how you installed you qt5 version (see above, "Prerequisites")). `brew --prefix qt5` will be expanded to the path to current Qt5 when the command is run.

If you are not using the Homebrew install then you should substitute the path to the parent folder of the bin/include/lib folders in that Qt tree.

##### Build

    cmake --build . --target install --config RelWithDebInfo

Cmake will build the application looking up configuration information in the file `CMakeCache.txt` in the specified directory
(the current directory: `.` ). By specifying '--target install' you build all targets and trigger the creation of a portable
bundle containing all files contained in the SC distribution. The default install location is `./Install`.

The flag `--config RelWithDebInfo` will build an optimized binary but will still include some useful debug information.

By default Xcode builds the application in debug mode which runs much slower and has a larger application size. It is intended for use with
the XCode debugger. For normal usage you will want an optimized release version.

The four possible build configs are:

- Debug
- RelWithDebInfo
- Release
- MinSizeRel



Diagnosing Build Problems
-------------------------

The most common build problems are related to incorrect versions of the core dependencies, or dirty states in your build folder.

### Checking component versions:

**Xcode**: `xcodebuild -version`, or the "About" dialog of the Xcode application. Any build from the 6.x series or greater should generally work.

**cmake, qt5(.5.x), libsndfile, readline**: `brew info ____` will show you what you have installed - for example, `brew info qt5` should show you the Qt5 version
information. A build using v5.6 and above will fail at the time of this writing because Qt5WebKit is missing in its binary distribution.

`brew upgrade ____` will update the dependency to a newer version (avoid this for Qt5 or handle different Qt5 versions with `brew switch`).

Other common homebrew problems can be fixed using `brew doctor`.


### Dirty build states

While it's generally safe to re-use your build folder, changing branches, build tools, cmake settings, or the versions of your dependencies can sometimes put you in a state where you can no longer build. The solution is to clean your build folder - the common ways to do this, in order of severity:

1. `rm CMakeCache.txt` (delete your cmake settings for that build)
2. `xcodebuild clean --target install` or `make clean` (clean your intermediate build files)
3. `rm -r ./Install` (delete the output of your build)
4. `cd ..; rm -r ./build` (delete your entire build folder)

Removing the CMakeCache.txt should fix most build problems. After running each one of these, you must re-run the two cmake commands shown in the build instructions above (configure and rebuild).

If you wish to build multiple git branches you should usually create a new build folder for each branch you're building. In practice, though, you can usually switch between similar branches and rebuild by simply deleting your CMakeCache.txt.

### Travis continuous integration

The code on github is tested anytime a contributor pushes new changes, so if a mistake was made in the cutting edge development version and something is broken, then you should be able to see this by visiting the Travis status page:

https://travis-ci.org/supercollider/supercollider

If the latest build status is failing, then you can switch your local copy to a previous commit that is still working (until the developers get a chance to fix the problem):

- locate the most recent green build on the travis,
- find it's git commit id (e.g. `595b956`), and
- check out that change in git: `git checkout 595b956`
- build

### If all else fails

Post to the user list stating what git hash you have checked out and all xcode version and library information and most importantly the error messages.

Simply posting "the latest checkout is broken" won't help. We need the exact compile error message.


Frequently used cmake settings
------------------------------

There are more settings in the build configuration you may want to adjust. In order to
see a useful list of your options, you can run:

    cmake -L ..

This configures the build using default settings or settings stored in the file
build/CMakeCache.txt, prints explanatory return statements and produces a list of
variables the value of which you might want to change.

In order to see all the command line options `cmake` offers, type:

    cmake --help

It is not necessary to pass in all required arguments each time you run cmake,
as cmake caches previously set arguments in the file CMakeCache.txt. This is
helpful, but also something to keep in mind if unexpected things happen.

If you feel uncomfortable with the command line, you might want to try cmake
frontends like  `ccmake` or `cmake-gui`. You can also configure your build by
manually editing build/CMakeCache.txt.


Common arguments to control the build configuration are:

  * Control the location where SC gets installed. The following line moves it to the Applications folder
    (which means you need to use `sudo`):

    `-DCMAKE_INSTALL_PREFIX=/Applications`

  * Enable compiler optimizations for your local system

    `-DNATIVE=ON`

  * Build the *supernova* server:

    `-DSUPERNOVA=ON`

    Using supernova requires the `portaudio` audio backend, so you need to install it
    (Homebrew and MacPorts both provide packages).

    *Note*: When you build with supernova, an alternative server executable and a supernova
    version of each plugin is built. If you also use the sc3-plugins package, make sure to
    compile them with supernova support too.

    Within SC you will be able to switch between scsynth and supernova by evaluating one of:

    `Server.supernova`
    `Server.scsynth`

    Check sc help for `ParGroup` to see how to make use of multi-core hardware.

  * Build a 32-bit version (sc 3.6 only):

    `-DCMAKE_OSX_ARCHITECTURES='i386'`

    or combine a 32- and 64-bit version into a bundle (i.e. build a universal binary).
    This is only possible up until macOS 10.6 and requires the dependencies (Qtlibs &
    readline) to be universal builds too:

    `-DCMAKE_OSX_ARCHITECTURES='i386;x86_64'`

  * Homebrew installations of libsndfile should be detected automatically. To link to a
    version of libsndfile that is not installed in /usr/local/include|lib, you can use:

    `-DSNDFILE_INCLUDE_DIR='/path/to/libsndfile/include'`
    `-DSNDFILE_LIBRARY='/path/to/libsndfile/lib/libreadline.dylib'`

  * Normally, homebrew installations of readline are detected automatically, and building with
    readline is only required if you plan to use SuperCollider from the terminal. To link to a
    non-standard version of readline, you can use:

    `-DREADLINE_INCLUDE_DIR='/path/to/readline/include'`
    `-DREADLINE_LIBRARY='/path/to/readline/lib/libreadline.dylib'`


Using cmake with Xcode or QtCreator
-----------------------------------

Xcode projects are generated by appending: `-G Xcode`. The build instructions above use the Xcode toolchain, which is well-tested and generally recommended if you're planning to debug or hack on SC.

You may also want to make the expected SDK-Version and location explicit, using:

`-DCMAKE_OSX_SYSROOT=`

This is often useful to point to an older SDK even with a newer Xcode installed. These are generally located in the

`/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs`

of an Xcode.app package.

You can build without using XCode using `make`, by omitting the `-G Xcode` - in this case, your build command
is `make` rather than `xcodebuild`

Qt Creator has very good `cmake` integration and can build `cmake` projects without requiring a `cmake` generated project file. If you have Qt5 via homebrew installed, you can install Qt Creator by running:

    brew linkapps qt5


Building without Qt, the IDE, or server-only
---------------------------------------------

The Qt framework is used for the SC-IDE, and to provide a graphical toolkit for the sclang language interpreter for users to build their own GUIs.

Also SuperCollider's sound server(s), scsynth, and it's sibling, supernova do not depend on sclang or the IDE, and there are many projects that make use of scsynth without the other SuperCollider components.

The build system supports building parts of the whole application by adding arguments to the cmake build configuration. These are:

    Build without IDE:     `-DSC_IDE=OFF`

    Build without QT:      `-DSC_QT=OFF`

    Build without sclang:  `-DSC_SCLANG=OFF`

This uses a "top-down" logic, so in order to build server only, you need to "switch" the IDE and sclang off:

    $> cmake -G Xcode -D SC_IDE=OFF -D SC_SCLANG=OFF ..

In this case Qt will be removed as build requirement automatically. Note that for server only, you still require libsndfile, but libreadline and Qt are not required any more. If you build supernova, portaudio also is required.

Setting `SC_IDE=OFF` will not only prevent building SC-IDE, but also switch to a different install mode. As neither sclang (right now) nor the servers provide a graphical interface, it makes no sense to install them into a MacOS application bundle (like SC-IDE). Therefore the install switches into FHS mode and places sclang and the server(s) within the default systempath at /usr/local/bin. In this case the other supercollider components are places like in Linux builds, the resources folder, containing the class library, HelpSource and other folders will be located at /usr/local/share/SuperCollider, the plugins at /usr/local/lib/SuperCollider/plugins. Note that you can set the switch between install modes by setting:

    -D MACOS_FHS=ON/OFF

Note though, that not all feature combinations are supported, partially because they are not implemented, partially because they seem to have no practical relevance. Tested are:

Sclang with Qt in command line mode:

    -D SC_IDE=OFF

Server only:

    -D SC_IDE=OFF -D SC_SCLANG=OFF

Experimentally supported is the full set including the IDE, installed to the FHS-system (no application bundle):

    -D MACOS_FHS=ON

You cannot switch building the server scsynth off. Supernova can be added to all variants.

NOTE: the build system tries to handle these arguments cleverly. For example it will not build
SC-IDE if you set SC_SCLANG off. BUT: this only works automatically if you haven't set
the dependant value manually before. Manually set arguments are written to a cache, the values
in which override automatically generated values. Any argument that has been set manually, must therefore be changed explicitly if required by the build logic. For example:

You start a fresh *server-only* build:

    cmake -G Xcode -D SC_SCLANG=OFF ..

Later, using the same build folder you decide to add the IDE (and implicitly sclang). In that
case it is not enough to set SC-IDE=ON, you also need to set SC_SCLANG explicitly:

    cmake -D CMAKE_PREFIX_PATH=`brew --prefix qt5` -D SC_IDE=ON -D SC_SCLANG=ON ..

NOTE: as SC_QT was not specified manually at any point it was switched off automatically
for the server-only build and switched on for the build including the IDE. However once
the IDE is added, specifying the CMAKE_PREFIX_PATH explicitly cannot be avoided.

The interplay between arguments provided on the command line, values generated automatically
by cmake, values stored in the cache, and also the visiblity of such values in different
parts of the build system, is a bit more involved than described here. Therefore a good
rule of thumbs is: if you change any of a set of interrelated values, set all of the values
in that set explicitly. Or: if you get build configuration errors, try to find out if another
value could be stored in the cache that conflicts with the value you set. If everything
fails, deleting the cache (CMakeCache.txt) should help if the arguments you set manually
are supported by the build system.


### Qt-less build

Choosing a `SC_QT=OFF` build for a slim sclang will by necessity turn the SC-IDE build off. Qt-less sclang is required on X-less linux systems, preferred on low resources systems, and sometimes preferred by people who use other graphic toolkits than Qt. Note that the class-library contains classes that depend on the availability of the graphical toolkit, therefore you will get errors at sclang startup until these classes are removed. See the link below for a recipe for Linux installs, they should be easily adaptable (see point 5):

    http://supercollider.github.io/development/building-raspberrypi


sclang and scynth executables
-----------------------------

In the default install, which uses a MacOs application bundle, executables `sclang`, `scsynth` and (if available) `supernova` are located here:

`SuperCollider.app/Contents/MacOS/sclang`
`SuperCollider.app/Contents/Resources/scsynth`
`SuperCollider.app/Contents/Resources/supernova`

The SuperCollider class library and help files are in:

`SuperCollider.app/Contents/Resources/SCClassLibrary`

In previous versions of SuperCollider these resources lived in the top folder next to SuperCollider.app. To make a standard self-contained app bundle with correct library linking, these have now been moved into the app bundle.

If you need to access them from the Finder, ctrl-click SuperCollider.app and choose "Show package contents" from the context menu.

To access them in the Terminal:

    cd /path/to/SuperCollider.app/Contents/Resources

or

    cd /path/to/SuperCollider.app/Contents/MacOS


##### Adding scsynth and sclang to your path

If sclang and scsynth are contained in an application bundle, you need somehow to add them to the path, in order for them to be easily accessible from any folder. However this can leed to difficulties, as some required libraries - namely Qt - are not found from the directory you are calling sclang from. One way to tackle this is to create a shell script that cd's into the path where the binaries are stored, and then starts them as required:

For `sclang`:

    #!/bin/sh
    cd /full/path/to/SuperCollider.app/Contents/MacOS
    exec ./sclang $*

And for `scsynth`:

    #!/bin/sh
    cd /full/path/to/SuperCollider.app/Contents/Resources
    export SC_PLUGIN_PATH="/full/path/to/SuperCollider.app/Resources/plugins/";
    exec ./scsynth $*


###### Why not just symlink them ?

- If you have Qt installed system-wide, sclang will load those as well as the Qt frameworks included in the application bundle. It will then fail with an error message like:

> You might be loading two sets of Qt binaries into the same process. Check that all plugins are compiled against the right Qt binaries. Export DYLD_PRINT_LIBRARIES=1 and check that only one set of binaries are being loaded.
This application failed to start because it could not find or load the Qt platform plugin "cocoa".

- scsynth will not find the included "plugins", unless given explicitly
  with the -U commandline flag or using the SC_PLUGIN_PATH environment variable as shown above.

NOTE: Making an FHS install of SC will circumvent these problems because the libraries required will be installed as a prerequisite of the build. Sclang and scsynth do not see the libraries contained in the SuperCollider application bundle. But: if the libraries used in the FHS-build subsequently get replaced by updated versions, incompatibilities may arise. In that case you should repeat the FHS build with the updated libraries.
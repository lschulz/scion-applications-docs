Quake III Arena
###############

You can play Quake III Arena on SCION! We maintain a fork of `ioquake3 https://github.com/ioquake/ioq3`_ that can use SCION connections. In order to play you will need to build C bindings to SCION and our fork. Additionally you need the commercial game assets. If you do not have a copy of Quake III, you can use the assets from the game's demo instead.

PAN Bindings: https://github.com/lschulz/pan-bindings
ioq3-scion: https://github.com/lschulz/ioq3-scion

### SCION-enabled Game Servers
If you set it up like described below, Quake will SCION-enabled public game servers automatically and show them in the server browser. To enable others to find your server, read :ref:`Running a Server`.

Building from Source
********************
We support ioq3-scion on Linux (Ubuntu 24.04.1 LTS) and Windows. Compiling for other operating systems may work, but is untested.

Linux
=====
Install the dependencies (example for Ubuntu):
.. code-block:: bash
    sudo apt-get update
    sudo apt-get install -y build-essential cmake git libsdl2-dev libsodium-dev libasio-dev

A Go compiler is needed to compile SCION code. Install the latest version as described here: https://go.dev/doc/install
PAN has only a narrow range of supported Go versions with every release. Check the README in :ref:`scion-apps https://github.com/netsec-ethz/scion-apps` and :ref:`pan-bindings https://github.com/lschulz/pan-bindings` to learn which version should work. If you need an older version than you just installed, run
.. code-block:: bash
    go install golang.org/dl/go1.22.8@latest
    go1.22.8 download # make sure ~/go/bin/ is in PATH
with the appropriate version number to let go download another version of the compiler. Include ``-D GO_BINARY=$(which go1.22.8)`` (again with the correct version) in the cmake configuration command below to build with the correct Go.

Build PAN and the C bindings:
.. code-block:: bash
    git clone https://github.com/lschulz/pan-bindings.git
    cd pan-bindings
    mkdir -p build/release
    cmake -D CMAKE_BUILD_TYPE=Release -D BUILD_SHARED_LIBS=ON -B build/release
    cmake --build build/release

For the next step, the C compiler has to be able to find the compiled PAN C libraries and headers. The easiest way to accomplish that is by installing them in /usr/local:
.. code-block:: bash
    sudo cmake --install build/release
    sudo ldconfig
If you want to install in a different directory, specify it in the cmake configuration step as ``-DCMAKE_INSTALL_PREFIX=<path>`` or in the install command with ``--prefix <path>``. The configuration above will also install C++ bindings and example programs. If you do not want to build them, you can run cmake with ``-D BUILD_CPP=OFF -D BUILD_EXAMPLES=OFF``. When you run Quake, ``libpan.so`` must be in the library search path.

Now you can clone and build ioq3-scion:
.. code-block:: bash
    git clone https://github.com/lschulz/ioq3-scion.git
    cd ioq3-scion
    make release -j $(nproc)
You can make local changes to the Makefile (e.g., to set the header and library paths in CFLAGS) by creating a file called Makefile.local next to Makefile. The binaries should be in ``build/release-linux-x86_64``. Continue with installing the game assets as described in :ref:`Installation`.

Windows
=======
On Windows, the PAN C bindings and ioq3-scion must be compiled with MinGW. Start by installing :ref:`MSYS2 https://www.msys2.org/`. Using an MSYS2 UCRT64 environment, the following packets are required:
.. code-block:: bash
    pacman -Sy
    pacman -S \
        mingw-w64-ucrt-x86_64-gcc   \
        mingw-w64-ucrt-x86_64-cmake \
        mingw-w64-ucrt-x86_64-ninja \
        mingw-w64-ucrt-x86_64-asio

A Go compiler is needed to compile SCION code. Install the latest version as described here: https://go.dev/doc/install
The same note about supported Go version as in the Linux build process applies.

Clone the PAN bindings, and build in an MSYS2 UCRT64 shell:
.. code-block:: bash
    git clone https://github.com/lschulz/pan-bindings.git
    cd pan-bindings
    mkdir build
    cmake -D BUILD_SHARED_LIBS=ON -D GO_BINARY="$PROGRAMFILES/Go/bin/go.exe" -G 'Ninja Multi-Config' -B build
    cmake --build build --config Release

Install libpan and its headers:
.. code-block:: bash
    cmake --install build --config Release --prefix /usr/local

Clone and build ioq3-scion in MSYS2:
.. code-block:: bash
    git clone https://github.com/lschulz/ioq3-scion.git
    cd ioq3-scion
    make release -j $(nproc)
You may need to modify some make variables for the build to succeed by creating a file called ``Makefile.local`` next to ``Makefile``. For example:
.. code-block:: makefile
    CFLAGS=-I/usr/local/include -L/usr/local/lib
    USE_CURL=0

Installation
************
Quake III uses two important search paths: A basepath and a homepath. basepath is by default set to the working directory from which Quake was started. homepath is usually `~/.q3a` on Linux and ``%appdata%\Quake3`` on Windows. basepath must be writeable. Both paths can be overridden on the command line like so ``+set fs_basepath <basepath> +set fs_homepath <homepath>``. For a simple setup, you can install all required files in homepath.

Create your homepath directory and copy ``baseq3/vm`` from ``ioq3-scion/build/release-linux-x86_64`` to ``homepath`` keeping the directory structure intact. If you have the assets for Team Arena, do the same with ``missionpack/vm``. Copy the binaries from ``ioq3-scion/build/release-linux-x86_64`` to ``homepath`` or to a separate ``basepath`` (or leave them where they are to use the build directory as basepath). For example:
.. code-block:: bash
    cd ioq3-scion
    homepath=~/.q3demo
    mkdir "${homepath}"
    mkdir "${homepath}/baseq3"
    mkdir "${homepath}/missionpack"
    cp -r baseq3/vm "${homepath}/baseq3"
    cp -r missionpack/vm "${homepath}/missionpack"
    cp ioq* *.so "${homepath}"

If you are on Windows, copy ``libpan.dll`` from the PAN bindings build directory and ``libsodium-26.dll`` from MSYS2 to ``basepath``.

Obtaining the Game Assets
*************************
In order to run the game you need the original games assets (3d models, textures, animations, levels, sound effects, etc.). You can copy them from the original games installation directory or download the original games demo for free (with a limited content).

# Full Game
Copy the ``.pk3`` files from the original game's `baseq3` and `missionpack` directories into the same diretories in ioq3-scion's homepath. By providing the ``.qvm`` files in the ``vm`` subdirectories we override the code from the original game while keeping the remaining assets.

# Demo
The Quake III Arena demo and game patches can still be found online (look for archives of ftp.idsoftware.com). You need the following files with the given SHA256 hashes:
.. code-block::
    64dee3f69b6e792d1da4fe0ac98fedc7eb1e37ea1027fb609a9fadd06150a4ec  linuxq3ademo-1.11-6.x86.gz.sh
    c36132c5556b35e01950f1e9c646235033a5130f87ad776ba2bc7becf4f4f186  linuxq3apoint-1.32b-3.x86.run
Extract the ``.pk3`` from the installers you downloaded:
.. code-block:: bash
    tail +165 linuxq3ademo-1.11-6.x86.gz.sh | tar -zx demoq3/pak0.pk3
    tail +356 linuxq3apoint-1.32b-3.x86.run | tar -zx baseq3

Configuration
*************
Create an ``autoexec.cfg`` in ``${homepath}/baseq3``:
.. code-block::
    seta com_hunkmegs 256

    // enable network
    // bitmask of 1=IPv4, 2=IPv6, 4=Prio6, 8=NoMcast, 16=SCION
    set net_enabled 19 // set 16 for SCION only

    // PAN often uses the wrong source address, so set et explicitly
    seta net_scion 127.0.0.1  // SET TO YOUR PUBLIC IP
    seta net_scion_port 30279 // MUST BE DIFFERENT FROM net_port AND net_port6

    // optional: accept IP connections only from localhost
    seta net_ip 127.0.0.1
    seta net_port 27960
    seta net_ip6 ::1
    seta net_port6 27960

    // bind voice-chat push-to-talk to the q key
    bind q "+voiprecord"

    // optional: overwrite the key opening the drop-down console
    // seta cl_consoleKeys F12

    // interpret console input as commands instead of chat messages
    seta con_autochat 0

    // bind SCION path selection to page up and page down key
    bind PGDN "nextpath"
    bind PGUP "prevpath"

    // master servers
    set sv_master1 "[1-ff00:0:110,127.0.0.1]:27951"
    set sv_master2 ""
    set sv_master3 ""
    set sv_master4 ""
    set sv_master5 ""

``net_scion_port`` should be set to a port from the ``dispatched_ports`` range of your SCION AS. Usually that means 31000-32767. Prefer the lower end of the range as the upper end is used for ephemeral ports.

Running the Game
****************
You should now have to following folder structure.

# Linux
.. code-block:: text
    ~/.q3a
    ├── baseq3
    │   ├── autoexec.cfg
    │   ├── pak0.pk3
    │   ├── pak1.pk3
    │   ├── pak2.pk3
    │   ├── pak3.pk3
    │   ├── pak4.pk3
    │   ├── pak5.pk3
    │   ├── pak6.pk3
    │   ├── pak7.pk3
    │   ├── pak8.pk3
    │   └── vm
    │       ├── cgame.qvm
    │       ├── qagame.qvm
    │       └── ui.qvm
    └── missionpack
        ├── pak0.pk3
        ├── pak1.pk3
        ├── pak2.pk3
        ├── pak3.pk3
        └── vm
            ├── cgame.qvm
            ├── qagame.qvm
            └── ui.qvm

You may also want to copy ``ioquake3.x86_64``, ``ioq3ded.x86_64`` and the shared libraries into `~/.ioq3` or store them in `/opt/games`.
Start the game by running ``./ioquake3.x86_64``.

# Windows
.. code-block:: text
    %APPDATA%\QUAKE3
    |   ioq3ded.x86_64.exe
    |   ioquake3.x86_64.exe
    |   libpan.dll
    |   libsodium-26.dll
    |   renderer_opengl1_x86_64.dll
    |   renderer_opengl2_x86_64.dll
    |   SDL264.dll
    +---baseq3
    |   |   autoexec.cfg
    |   |   pak0.pk3
    |   |   pak1.pk3
    |   |   pak2.pk3
    |   |   pak3.pk3
    |   |   pak4.pk3
    |   |   pak5.pk3
    |   |   pak6.pk3
    |   |   pak7.pk3
    |   |   pak8.pk3
    |   \---vm
    |           cgame.qvm
    |           qagame.qvm
    |           ui.qvm
    \---missionpack
        |   pak0.pk3
        |   pak1.pk3
        |   pak2.pk3
        |   pak3.pk3
        \---vm
                cgame.qvm
                qagame.qvm
                ui.qvm

Start the game by running ``ioquake3.x86_64.exe``.

Running a Server
****************

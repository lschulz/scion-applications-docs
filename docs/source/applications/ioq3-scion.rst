***************
Quake III Arena
***************

You can play Quake III Arena in SCION! We maintain a fork of `ioquake3
<https://github.com/ioquake/ioq3>`_ that can use SCION connections. In order to play you will need
to build C bindings to SCION and our fork. Additionally you need the commercial game assets. If you
do not have a copy of Quake III, you can use the assets from the game's demo instead.

* PAN Bindings: https://github.com/lschulz/pan-bindings
* ioq3-scion: https://github.com/lschulz/ioq3-scion

**SCION Game Servers:**
If you set it up like described below, Quake will find SCION-enabled public game servers
automatically and show them in the server browser. To enable others to find your server, read
:ref:`running-a-server`.

Build from Source
=================
We support ioq3-scion on Linux (Ubuntu 24.04.1 LTS) and Windows. Compiling for other operating
systems may work, but is untested.

Installation Script
-------------------
We provide a `script <https://gist.github.com/lschulz/193f985977c4b4228cd501b589092130>`_ for
building and installing ioq3-scion in one command on Ubuntu.

.. code-block:: text

    Usage: ./install-ioq3-scion [OPTION]...
    Installs ioq3-scion

    --basepath DIR Set the binary installation directory
    --homepath DIR Set the user directory (must be writeable by user)
    --assets DIR   Path to Quake 3 assets
    --tmp DIR      Temporary build directory
    --keep         Keep the build directory when finished
    -h, --help     Print this text

For example:

.. code-block:: bash

    ./install-ioq3-scion --basepath /opt/games/quake3 --homepath ~/.q3a

See below for a description of what *basepath* and *homepath* mean. By default the script will
install the Quake 3 demo. To install the full game, specify the path to the installation folder as
``--assets <path>``.

Linux
-----
Install the dependencies (example for Ubuntu).

.. code-block:: bash

    sudo apt-get update
    sudo apt-get install -y build-essential cmake git libsdl2-dev libsodium-dev libasio-dev

A Go compiler is needed to compile SCION code. Install the latest version as described in the
`instructions <https://go.dev/doc/install>`_. PAN has only a narrow range of supported Go versions
with every release. Check the README in `scion-apps <https://github.com/netsec-ethz/scion-apps>`_
and `pan-bindings <https://github.com/lschulz/pan-bindings>`_ to learn which version should work. If
you need an older version than you just installed, install it with the ``go`` command.

.. code-block:: bash

    go install golang.org/dl/go1.22.9@latest
    go1.22.9 download # make sure ~/go/bin/ is in PATH

Include ``-D GO_BINARY=$(which go1.22.8)`` (with the correct version) in the cmake configuration
command below to build with the correct Go.

Build PAN and the C bindings.

.. code-block:: bash

    git clone https://github.com/lschulz/pan-bindings.git
    cd pan-bindings
    mkdir -p build/release
    cmake -D CMAKE_BUILD_TYPE=Release -D BUILD_SHARED_LIBS=ON -B build/release
    cmake --build build/release

For the next step, the C compiler has to be able to find the compiled PAN C libraries and headers.
The easiest way to accomplish that is by installing them in ``/usr/local``.

.. code-block:: bash

    sudo cmake --install build/release
    sudo ldconfig

If you want to install in a different directory, specify it in the cmake configuration step as
``-DCMAKE_INSTALL_PREFIX=<path>`` or in the install command with ``--prefix <path>``. The
configuration above will also install C++ bindings and example programs. If you do not want to build
them, you can run cmake with ``-D BUILD_CPP=OFF -D BUILD_EXAMPLES=OFF``. When you run Quake,
``libpan.so`` must be in the library search path.

Now you can clone and build ioq3-scion.

.. code-block:: bash

    git clone https://github.com/lschulz/ioq3-scion.git
    cd ioq3-scion
    make release -j $(nproc)

You can make local changes to the Makefile (e.g., to set the header and library paths in CFLAGS) by
creating a file called ``Makefile.local`` next to ``Makefile``. The binaries should be in
``build/release-linux-x86_64``. Continue with installing the game assets as described in
:ref:`installation`.

Windows
-------
On Windows, the PAN C bindings and ioq3-scion must be compiled with MinGW. Start by installing
`MSYS2 <https://www.msys2.org/>`_. Using an MSYS2 UCRT64 environment, the following packets are
required:

.. code-block:: bash

    pacman -Sy
    pacman -S \
        mingw-w64-ucrt-x86_64-gcc \
        mingw-w64-ucrt-x86_64-cmake \
        mingw-w64-ucrt-x86_64-ninja \
        mingw-w64-ucrt-x86_64-asio \
        mingw-w64-ucrt-x86_64-libsodium \
        mingw-w64-ucrt-x86_64-SDL2 \
        make git

A Go compiler is needed to compile SCION code. Install the latest version as described in the
`instructions <https://go.dev/doc/install>`_. The same note about supported Go version as in the
Linux build process applies.

Clone the PAN bindings, and build in an MSYS2 UCRT64 shell.

.. code-block:: bash

    git clone https://github.com/lschulz/pan-bindings.git
    cd pan-bindings
    mkdir build
    cmake -D BUILD_SHARED_LIBS=ON -D GO_BINARY="$PROGRAMFILES/Go/bin/go.exe" -G 'Ninja Multi-Config' -B build
    cmake --build build --config Release

Install libpan and its headers.

.. code-block:: bash

    cmake --install build --config Release --prefix /usr/local

Clone and build ioq3-scion in MSYS2.

.. code-block:: bash

    git clone https://github.com/lschulz/ioq3-scion.git
    cd ioq3-scion
    make release -j $(nproc)

You may need to modify some make variables for the build to succeed by creating a file called
``Makefile.local`` next to ``Makefile``. For example:

.. code-block:: makefile

    CFLAGS=-I/usr/local/include -L/usr/local/lib
    USE_CURL=0

.. _installation:

Installation
============
Quake III uses two important search paths: *basepath* and *homepath*. *basepath* is set to the
working directory from which Quake was started by default. *homepath* is usually ``~/.q3a`` on Linux
and ``%appdata%\Quake3`` on Windows. *basepath* must be writeable. Both paths can be overridden on
the command line like so ``+set fs_basepath <basepath> +set fs_homepath <homepath>``. For a simple
setup, you can install all required files in *homepath*.

Create your homepath directory and copy ``baseq3/vm`` from ``ioq3-scion/build/release-linux-x86_64``
to *homepath* keeping the directory structure intact. If you have the assets for Team Arena, do the
same with ``missionpack/vm``. Copy the binaries from ``ioq3-scion/build/release-linux-x86_64`` to
*homepath* or to a separate *basepath* (or leave them where they are to use the build directory as
basepath). For example:

.. code-block:: bash

    cd ioq3-scion
    homepath=~/.q3a
    mkdir "${homepath}"
    mkdir "${homepath}/baseq3"
    mkdir "${homepath}/missionpack"
    cp -r baseq3/vm "${homepath}/baseq3"
    cp -r missionpack/vm "${homepath}/missionpack"
    cp ioq* *.so "${homepath}"

If you are on Windows, copy ``libpan.dll`` from the PAN bindings build directory and
``libsodium-26.dll`` from MSYS2 to *basepath*.

Obtaining the Game Assets
-------------------------
In order to run the game you need the original games assets (3d models, textures, animations,
levels, sound effects, etc.). You can copy them from the original games installation directory or
download the original game's demo for free (with limited content).

Full Game
"""""""""
Copy the ``.pk3`` files from the original game's `baseq3` and `missionpack` directories
into the same diretories in ioq3-scion's *homepath*. By providing the ``.qvm`` files in the ``vm``
subdirectories we override the code from the original game while keeping the remaining assets.

Demo
""""
The Quake III Arena demo and game patches can still be found online (look for archives of
ftp.idsoftware.com). You need the following files with the given SHA256 hashes:

.. code-block:: text

    64dee3f69b6e792d1da4fe0ac98fedc7eb1e37ea1027fb609a9fadd06150a4ec  linuxq3ademo-1.11-6.x86.gz.sh
    c36132c5556b35e01950f1e9c646235033a5130f87ad776ba2bc7becf4f4f186  linuxq3apoint-1.32b-3.x86.run

Extract the ``.pk3`` from the installers you downloaded.

.. code-block:: bash

    tail +165 linuxq3ademo-1.11-6.x86.gz.sh | tar -zx demoq3/pak0.pk3
    tail +356 linuxq3apoint-1.32b-3.x86.run | tar -zx baseq3

Configuration
-------------
Create an ``autoexec.cfg`` in ``${homepath}/baseq3``: ::

    seta com_hunkmegs 256 // increase memory pool size
    snaps 40 // snapshots per second requested from server

    // enable network
    // bitmask of 1=IPv4, 2=IPv6, 4=Prio6, 8=NoMcast, 16=SCION
    set net_enabled 19 // set 16 for SCION only

    // PAN often uses the wrong source address, set them explicitly
    seta net_scion x.y.z.w  // SET TO YOUR PUBLIC IP
    seta net_scion_port 31796 // MUST BE DIFFERENT FROM net_port AND net_port6

    // optional: configure IP addresses and ports for non-SCION connections
    seta net_ip 0.0.0.0
    seta net_port 27960
    seta net_ip6 ::0
    seta net_port6 27960

    // bind voice-chat push-to-talk to the q key
    bind q "+voiprecord"

    // optional: list of keys that open the console
    seta cl_consoleKeys ~ ` 0x7e 0x60 F12

    // if set to 0 commands are recognized without a slash
    seta con_autochat 1

    // use ANSI escape codes in terminal
    com_ansiColor 1

    // bind SCION path selection to page up and page down key
    bind PGDN "nextpath"
    bind PGUP "prevpath"

    // master servers
    set sv_master1 "141.44.25.150:27950"
    set sv_master2 "[71-2:0:4a,141.44.25.150]:30795"
    set sv_master3 ""
    set sv_master4 ""
    set sv_master5 ""

``net_scion_port`` should be set to a port from the ``dispatched_ports`` range of your SCION AS.
Usually that means 31000-32767. Prefer the lower end of the range as the upper end is used for
ephemeral ports.

Directory Structure
-------------------
You should now have to following directory structure.

Linux
"""""
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

You may also want to copy ``ioquake3.x86_64``, ``ioq3ded.x86_64`` and the shared libraries into
``~/.ioq3`` or store them in ``/opt/games/quake3``.

Windows
"""""""
.. code-block:: text

    %APPDATA%\Quake3
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

Running the Game
================

Start the game by running ``${basepath}/ioquake3.x86_64`` or ``${basepath}/ioquake3.x86_64.exe``.
If you are not using the default *homepath*, set the path on the command line:

.. code-block:: bash

    ${basepath}/ioquake3.x86_64 +set +set fs_homepath ${homepath}

.. _running-a-server:

Running a Server
----------------
You can run a server directly from the main menu by selecting "Multiplayer" and "Create". If you set
"dedicated" to *Internet*, the server will be registered at the master server(s) so others on the
Internet can find it.

To run a dedicated server on a headless machine, create a file called ``server.cfg`` (the exact name
is not important) with your servers settings in ``${homepath}/baseq3``. The server will still parse
``autoexec.cfg``, so there is no need to repeat commands.

Example configuration for a server:

.. code-block:: text

    set sv_hostname "Server Name"
    set g_motd "Server message of the day"

    set sv_maxclients 16 // maximum number of clients
    set sv_pure 0        // allow clients to use different file versions
    set sv_fps 40        // server tick rate (default is 20)
    set sv_voip 1        // enable voice chat

    // free for all
    set g_gametype 0
    set timelimit 10
    set fraglimit 15
    set g_forcerespawn 0

    // bots
    seta bot_enable 1
    seta bot_nochat 1
    seta g_spskill 2
    seta bot_minplayers 4

    // map rotation
    set map1 "map q3dm1; set nextmap vstr map2"
    set map2 "map q3dm7; set nextmap vstr map3"
    set map3 "map q3dm17; set nextmap vstr map4"
    set map4 "map q3tourney2; set nextmap vstr map1"
    vstr map1

    rehashbans // reload the banlist

Start the dedicated server as ``ioq3ded.x86_64 +exec server.cfg +set dedicated 2``. Setting
dedicated to 2 causes the server to register itself at the master server(s).

Useful Cvars and Commands
=========================
Quake 3 and ioquake have many console variables and commands controlling everything from rendering
to keybindings. Below you find a list of some useful variables and commands that can be used in the
the drop-down console or added to the ``autoexec.cfg`` file. Commands entered in the drop-down
console must usually be prefixed with ``/``, except if you set ``con_autochat`` to 0.

Client Commands
---------------
- Command: ``clientinfo`` Print client state information.
- Command: ``serverstatus`` Get status information from server.
- Command: ``ping [-4|-6|-scion4|-scion6] server`` Ping a server via IPv4/IPv6/SCION-IPv4/SCION-IPv6.
- Command: ``connect [-4|-6|-scion4|-scion6] server`` Connect to a server.
- Command: ``disconnect`` Disconnect from server.
- Command: ``localservers`` Scan for servers on LAN using multicast.
- Command: ``globalservers <master# 0-5> <protocol> [keywords]`` Contact master server(s). Server
  0 queries all servers. Protocol should be 68 for Quake 3. Keywords are sent to the master
  server for filtering (e.g., ``empty``, ``full``, ``scion``).
- Command: ``showpaths`` List available network paths to the server.
- Command: ``selectpath`` Select a path by specifying a unique prefix of the 8 digit hash value of
  every path listed by ``showpaths``.
- Command: ``nextpath`` Select the next path in the list. Useful for binding to a key.
- Command: ``prevpath`` Select the previous path in the list. Useful for binding to a key.
- Command: ``toggle`` Toggle the value of a binary variable. Useful for keybindings.
- Cvar: ``cg_drawtimer`` Draw the elapsed round time.
- Cvar: ``cg_lagometer`` Draw lag-o-meter.

Server Commands
---------------
- Command: ``status`` Show server status.
- Command: ``showclientpaths`` Print last path to connected SCION clients.
- Command: ``clearclientpaths`` Clear the remote host cache of the server's path selector.
- Command: ``heartbeat`` Send heartbeat to master servers.
- Command: ``kick <player name>`` Kick a player from the server.
- Command: ``banaddr (ip[/subnet] | clientnum [subnet])`` Band an IP or ISD-ASN,IP.
- Command: ``exceptaddr (ip[/subnet] | clientnum [subnet])`` Exclude an IP or ISD-ASN,IP from a ban.
- Command: ``bandel (ip[/subnet] | num)`` Remove a ban.
- Command: ``exceptdel (ip[/subnet] | num)`` Remove an exception.
- Command: ``listbans`` List bans and excpetions.

Network
-------
* Command: ``net_restart`` Reopen all sockets. Necessary to apply changes in IP addresses or ports.
* Cvar: ``net_oobTimeout`` Out-of-band messages to servers the client is not currently connected to
  require separate SCION connections. This cvar controls how long these connections are kept active
  after the last time they have been used. Value in milliseconds.
* Cvar: ``net_safeMSS`` Maximum segment size (MSS) assumed for SCION paths that don't have MTU
  metadata. Paths that do have MTU metadata and would have a MSS below this value are ignored. The
  MSS is the MTU minus the overhead due to IP, SCION, and UDP headers. The default value is 1000
  bytes. Setting a lower value increases the tolerance for long SCION paths with low MTUs, but will
  lead to more packet fragmentation.
* Cvar: ``sv_encryption`` and ``cl_encryption`` enable encryption on the server and on the client,
  respectively. Possible values:

  * ``0`` Disable encryption.
  * ``1`` Enable encryption, but fall back to unencrypted connection if opposite site does not
    support/enable encryption.
  * ``2`` Enable encryption, connection fails if other side does not support/enable encryption.

Rendering
---------
* Cvar: ``cg_fov`` Horizontal FOV in degrees.
* Cvar: ``cg_drawfps`` Draw frames per second counter.
* Cvar: ``cl_renderer`` ``opengl1`` or ``opengl2`` (apply with ``vid_restart``).
* Command: ``vid_restart`` Restart video output.

Rate and Timing Settings
------------------------
Server
""""""
* Cvar: ``sv_fps`` Controls server refresh rate. Sets the maximum value for ``snaps`` that clients can request. (Default: 20)
* Cvar: ``sv_minRate``, ``sv_maxRate`` Minimum and maximum rate the client setting ``rate`` is clamped to. Disabled when zero. (Default: 0, 0)

Client
""""""
* Command: ``snaps`` Snapshots per second requested by the client. Part of the userinfo string sent
  to the server. (Default: 20)
* Command: ``rate`` Maximum bitrate the client can accept in byte/s. Part of the userinfo string
  sent to the server. (Default: 25000)
* Cvar: ``cl_maxpackets`` Limits the maximum number of packets per second from client to server.
  (Range: 15-125) (Default: 30)
* Cvar: ``cl_packetdup`` In how many additional packets the same usercmd is included in to
  counteract packet loss. (Range: 0-5) (Default: 1)
* Cvar: ``com_maxfps`` Framerate limit. (Default: 85)
* Cvar: ``cl_timenudge`` Time offset for snapshot interpolation. Negative values will extrapolate
  more for better responsiveness, positive values add more latency for better smoothness to counter
  jitter. (Range -30 to 30) (Default: 0)
* Cvar: ``r_displayrefresh`` Full-screen refresh rate. Passed to SDL. (Default: 0)
* Cvar: ``r_swapinterval`` 0 disables vsync, 1 enables vsync. Passed to SDL. (Default: 0)

# Building

In order to build Kraken you will need to have Go installed on your system. We recommend using Go >= 1.11 in order to leverage the native support for Go Modules (if it is not available in your package manager, you can use something like [gvm](https://github.com/moovweb/gvm)).

Firstly, download Kraken:

    $ git clone https://github.com/botherder/kraken.git
    $ cd kraken

Most Go libraries dependencies are available to install through:

    $ make deps

## Building on Linux

You need to install Yara development libraries and headers. You should download and compile Yara from the [official sources](https://github.com/VirusTotal/yara). It will require `dh-autoreconf` to be installed and you will need to configure some compilation flags. This is most likely the procedure you will need to follow:

    $ sudo apt install dh-autoreconf
    $ wget https://github.com/VirusTotal/yara/archive/v3.8.1.tar.gz
    $ tar -zxvf yara-v3.8.1.tar.gz
    $ cd yara-3.8.1
    $ ./bootstrap.sh
    $ ./configure --without-crypto
    $ make && sudo make install
    $ sudo ldconfig

Compiling Kraken requires you to specify a path to a file or a folder that contains the Yara rules you wish to embed with the binary. You can try for example with:

    $ BACKEND=example.com RULES=test/ make linux

## Building on FreeBSD

While cross-compilation of FreeBSD binaries is not available yet, it is possible to build binaries in a native FreeBSD environment. In order to do so you will firstly need to install some packages:

    $ sudo pkg install git gmake pkgconf go-bindata

Then you will need to install Yara, which is normally available in [ports](https://www.freshports.org/security/yara/):

    $ cd /usr/ports/security/yara

Before installing it, it is recommended that you modify the file `Makefile` to add `--without-crypto` to `CONFIGURE_ARGS` (if you don't need the Yara modules enabled in the Makefile, feel free to remove them). Now you can proceed with installing:

    $ sudo make && sudo make install

Now you can move to the directory that contains the Kraken source code and build it with:

    $ BACKEND=example.com RULES=test/ gmake freebsd

## Building on Mac

While cross-compilation of Darwin binaries is not available yet, it is possible to build binaries in a native Mac environment. While Yara is generally available using [brew](https://brew.sh), we are going to compile the latest available sources. Before doing so, we need to install some dependencies:

    $ brew install automake libtool go-bindata

Now you can proceed to download and compile the Yara sources in the same way as explained in the [Building on Linux](#building-on-linux) section. Once Yara is compiled and install correctly, you can proceed building binaries for Mac using the follwing command:

    $ BACKEND=example.com RULES=test/ make darwin

## Cross-compiling Windows binaries

Cross-compiling Windows binaries from a Linux development machine is a slightly more complicated process. Firstly you will need to install MingW and some other dependencies:

    $ sudo apt install gcc mingw-w64 automate libtool make

Next you will need to download Yara sources. Use the latest available version, which at the time of this writing is 3.8.1:

    $ wget https://github.com/VirusTotal/yara/archive/v3.8.1.tar.gz

Unpack the archive and export `YARA_SRC` to the newly-created folder:

    $ export YARA_SRC=<folder>

Next you need to bootstrap Yara sources and compile them with MingW. These are the instructions to compile it for **32bit**:

    $ cd ${YARA_SRC}
    $ ./bootstrap.sh
    $ ./configure --host=i686-w64-mingw32 --without-crypto --prefix=${YARA_SRC}/i686-w64-mingw32
    $ make -C ${YARA_SRC}
    $ make -C ${YARA_SRC} install

Now we can download and build `go-yara` for 32bit using the following command:

    $ go get -d -u github.com/hillu/go-yara
    $ GOOS=windows GOARCH=386 CGO_ENABLED=1 \
      CC=i686-w64-mingw32-gcc \
      PKG_CONFIG_PATH=${YARA_SRC}/i686-w64-mingw32/lib/pkgconfig \
      go install -ldflags '-extldflags "-static"' github.com/hillu/go-yara

Now you can compile Kraken using:

    $ BACKEND=example.com RULES=test/ make windows

If you get errors such as ` undefined reference to 'yr_compiler_add_file'` you might need to pass the `PKG_CONFIG_PATH` variable:

    $ PKG_CONFIG_PATH=${YARA_SRC}/i686-w64-mingw32/lib/pkgconfig BACKEND=example.com RULES=test/ make windows

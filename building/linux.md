# Building on Linux

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

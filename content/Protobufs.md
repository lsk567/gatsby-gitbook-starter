# Using Protobufs with Lingua Franca

The following has been tested on MacOS. If your platforms differs and the installation procedures differ, please update the wiki with instructions for your platform.

## Install Protobufs

Clone the git repository from Github:

    git clone https://github.com/protocolbuffers/protobuf.git
    git submodule update --init --recursive

Then build:

    cd protobuf
    ./autogen.sh
    ./configure
    make

If you run into this error: `autoreconf: failed to run aclocal: No such file or directory`, run this:

    brew install autoconf && brew install automake

and start over from autogen.sh.
Otherwise, if `make` succeeded, continue:

    make check
    sudo make install

The `make install` command installs the binary and library files in standard places, hence the need for `sudo`.
You should now see this:

    > which protoc
    /usr/local/bin/protoc
    > protoc --version
    libprotoc 3.11.0

(or some more recent version).

It may be tempting to download binary from https://github.com/protocolbuffers/protobuf/releases/tag/v3.11.1 for your platform, but this will not give you the packages required to compile protobuf-c.

## Install protobuf-c

Clone protobuf-c from Github:

    git clone https://github.com/protobuf-c/protobuf-c.git

Then build:

    cd protobuf-c
    ./autogen.sh
    ./configure
    make
    sudo make install

The last command puts files into /usr/local/include to be included in your C files, hence the need for `sudo`.

## Install gRPC

Instructions for several platforms are here: https://github.com/grpc/grpc/blob/v1.25.0/BUILDING.md
The following worked for me on MacOS.

Clone gRPC from Github:

    git clone https://github.com/grpc/grpc.git
    cd grpc
    git submodule update --init --recursive

Then build:

    make

## Install protobuf-c-rpc

Clone protobuf-c-rpc and build:

    git clone https://github.com/protobuf-c/protobuf-c-rpc
    cd protobuf-c-rpc
    ./autogen.sh
    ./configure
    make

FIXME: The above fails for me with this:

    EALMAC:protobuf-c-rpc eal$ make
    /Applications/Xcode.app/Contents/Developer/usr/bin/make  all-am
    CCLD     protobuf-c-rpc/libprotobuf-c-rpc.la
    Undefined symbols for architecture x86_64:
      "_protobuf_c_buffer_simple_append", referenced from:
          _server_connection_response_closure in protobuf-c-rpc-server.o


# Using gRPC with Lingua Franca

We are probably not going to use gRPC after all. Download from here:

https://github.com/Juniper/grpc-c.git

and build using instructions.  The grpc-c git repo includes grpc, protobuf, and protobuf-c as git submodules, however, the grpc version that this uses does not compile for me. I installed grpc, protobuf, and protobuf-c separately from source.

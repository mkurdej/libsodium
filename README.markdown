[![Build Status](https://travis-ci.org/jedisct1/libsodium.svg?branch=master)](https://travis-ci.org/jedisct1/libsodium?branch=master)
[![Windows build status](https://ci.appveyor.com/api/projects/status/fu8s2elx25il98hj?svg=true)](https://ci.appveyor.com/project/jedisct1/libsodium)
[![Coverity Scan Build Status](https://scan.coverity.com/projects/2397/badge.svg)](https://scan.coverity.com/projects/2397)
[![Make a donation to support this project](https://img.shields.io/badge/donate-PayPal-green.svg?style=flat)](https://www.libsodium.org/donate)

![libsodium](https://raw.github.com/jedisct1/libsodium/master/logo.png)
============

Sodium is a new, easy-to-use software library for encryption,
decryption, signatures, password hashing and more.

It is a portable, cross-compilable, installable, packageable
fork of [NaCl](http://nacl.cr.yp.to/), with a compatible API, and an
extended API to improve usability even further.

Its goal is to provide all of the core operations needed to build
higher-level cryptographic tools.

Sodium supports a variety of compilers and operating systems,
including Windows (with MingW or Visual Studio, x86 and x64), iOS and Android.

## Documentation

The documentation is a work-in-progress, and is being written using
Gitbook:

* [libsodium documentation](https://download.libsodium.org/doc/) -
online, requires Javascript.
* [offline documentation](https://www.gitbook.com/book/jedisct1/libsodium/details)
in PDF, MOBI and ePUB formats.

## Integrity Checking

The integrity checking instructions (including the signing key for libsodium)
are available in the [installation](https://download.libsodium.org/doc/installation/index.html#integrity-checking)
section of the documentation.

## Community

A mailing-list is available to discuss libsodium.

In order to join, just send a random mail to `sodium-subscribe` {at}
`pureftpd` {dot} `org`.

## License

[ISC license](https://en.wikipedia.org/wiki/ISC_license).

## Installation with CMake

Sodium supports CMake.

To use CMake (hopefully you will choose for an out-of-tree build) the steps are as follows:

    mkdir build
    cd build
    cmake ../
    make -j4 install

You can eventually pass to CMake some options. Particularly:

- CMAKE_BUILD_TYPE_TOLOWER=debug
- ENABLE_BLOCKING_RANDOM=1
- DISABLE_SSP=1
- DISABLE_ASM=1

You should prepend each option with -D such as:

    cmake -DCMAKE_BUILD_TYPE=debug -DENABLE_SSP=1 -DENABLE_ASM=1

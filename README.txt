xmedcon-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of xmedcon (license: GPL-2.0-or-later) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  xmedcon-0.26.2.tar.bz2
    https://master.dl.sourceforge.net/project/xmedcon/XMedCon-Source/0.26.2/xmedcon-0.26.2.tar.bz2?viasf=1
    sha256 af4a03039c0c4b66e428280cec2f96c546a11af266ecd500fe250dbb6b9924f1
  zlib-1.3.1.tar.gz
    https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
    sha256 9a93b2b7dfdac77ceba5a558a580e74667dd6fede4585b91eefb60f03b72df23
  libpng-1.6.58.tar.gz
    https://downloads.sourceforge.net/project/libpng/libpng16/1.6.58/libpng-1.6.58.tar.gz
    sha256 8c9b05b675ca7301a458df2c2e46f26e1d41ff36b8863f8c33530bc58c2e6225

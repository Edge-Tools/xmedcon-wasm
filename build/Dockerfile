FROM emscripten/emsdk:5.0.7 AS builder

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
      autotools-dev automake libtool pkg-config curl ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ARG VERSION=0.26.2
ENV VERSION=${VERSION}
ENV DIRECTORY=xmedcon-${VERSION}
ENV OUT_DIR=/out
ENV ROOT=/src
ENV PREFIX=/opt

ENV EM_PKG_CONFIG_PATH=${PREFIX}/lib/pkgconfig
ENV CFLAGS="-O2"
ENV CXXFLAGS="${CFLAGS}"

WORKDIR /src
RUN mkdir -p $OUT_DIR $PREFIX

ARG ZLIB_VERSION=1.3.1
ARG ZLIB_SHA256=9a93b2b7dfdac77ceba5a558a580e74667dd6fede4585b91eefb60f03b72df23
ADD https://github.com/madler/zlib/releases/download/v${ZLIB_VERSION}/zlib-${ZLIB_VERSION}.tar.gz zlib.tar.gz
RUN echo "${ZLIB_SHA256}  zlib.tar.gz" | sha256sum -c - \
 && tar -xzf zlib.tar.gz && rm zlib.tar.gz && cd zlib-${ZLIB_VERSION} \
 && emconfigure ./configure --prefix=$PREFIX --static \
 && emmake make -j$(nproc) install \
 && test -f $PREFIX/lib/libz.a

ARG LIBPNG_VERSION=1.6.58
ARG LIBPNG_SHA256=8c9b05b675ca7301a458df2c2e46f26e1d41ff36b8863f8c33530bc58c2e6225
ADD https://downloads.sourceforge.net/project/libpng/libpng16/${LIBPNG_VERSION}/libpng-${LIBPNG_VERSION}.tar.gz libpng.tar.gz
RUN echo "${LIBPNG_SHA256}  libpng.tar.gz" | sha256sum -c - \
 && tar -xzf libpng.tar.gz && rm libpng.tar.gz && cd libpng-${LIBPNG_VERSION} \
 && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=$PREFIX \
        --disable-shared --enable-static --disable-tools \
        CPPFLAGS="-I${PREFIX}/include" LDFLAGS="-L${PREFIX}/lib" \
 && emmake make -j$(nproc) install \
 && test -f $PREFIX/lib/libpng16.a \
 && test -f $PREFIX/lib/pkgconfig/libpng16.pc

ARG SOURCE_SHA256=af4a03039c0c4b66e428280cec2f96c546a11af266ecd500fe250dbb6b9924f1
RUN curl -fsSL -A "Mozilla/5.0" -o xmedcon.tar.bz2 \
      "https://master.dl.sourceforge.net/project/xmedcon/XMedCon-Source/${VERSION}/xmedcon-${VERSION}.tar.bz2?viasf=1" \
 && echo "${SOURCE_SHA256}  xmedcon.tar.bz2" | sha256sum -c - \
 && tar -xjf xmedcon.tar.bz2 && rm xmedcon.tar.bz2

WORKDIR /src/${DIRECTORY}

RUN emconfigure ./configure \
    --host=wasm32-unknown-emscripten \
    --prefix="$OUT_DIR" \
    --disable-shared --enable-static \
    --disable-gui \
    --disable-glib \
    --disable-dependency-tracking \
    CPPFLAGS="-I${PREFIX}/include" \
    LDFLAGS="-L${PREFIX}/lib"

COPY js ./js

ENV XMEDCON_LDFLAGS="\
    -L${PREFIX}/lib -lpng16 -lz -lm \
    -lnodefs.js -lworkerfs.js \
    --post-js /src/${DIRECTORY}/js/post.js \
    -s WASM_BIGINT=1 \
    -s ALLOW_MEMORY_GROWTH=1 \
    -s EXIT_RUNTIME=0 \
    -s EXPORTED_RUNTIME_METHODS=[\"callMain\",\"FS\",\"NODEFS\",\"WORKERFS\",\"ENV\"] \
    -s INCOMING_MODULE_JS_API=[\"noInitialRun\",\"wasmBinary\",\"locateFile\",\"preRun\",\"postRun\",\"instantiateWasm\",\"print\",\"printErr\",\"arguments\"] \
    -s INVOKE_RUN=0 \
    -s MODULARIZE=1 \
    -s EXPORT_ES6=1 \
"

RUN emmake make \
    EXEEXT=".mjs" \
    LDFLAGS="$XMEDCON_LDFLAGS" \
    -j$(nproc)

RUN mkdir -p /dist \
 && cp source/medcon.mjs  /dist/medcon.mjs \
 && cp source/medcon.wasm /dist/medcon.wasm \
 && [ "$(head -c4 /dist/medcon.wasm | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
 && [ "$(stat -c%s /dist/medcon.wasm)" -gt 65536 ]

FROM scratch AS exportor
COPY --from=builder /dist /dist

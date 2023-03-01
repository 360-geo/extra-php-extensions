ARG PHP_VERSION
FROM bref/build-php-$PHP_VERSION:1.7.14 AS ext
ARG PHP_VERSION

# Prepare environment
ENV PROJ_BUILD_DIR=${BUILD_DIR}/proj
RUN mkdir -p ${PROJ_BUILD_DIR}
RUN LD_LIBRARY_PATH= yum -y install wget libtiff libtiff-devel libcurl libcurl-devel

# Compile sqlite3
WORKDIR ${PROJ_BUILD_DIR}
RUN wget https://www.sqlite.org/2022/sqlite-autoconf-3400100.tar.gz -O sqlite-autoconf-3400100.tar.gz
RUN tar xzf sqlite-autoconf-3400100.tar.gz
WORKDIR ${PROJ_BUILD_DIR}/sqlite-autoconf-3400100
RUN ./configure
RUN make
RUN make install

# Compile proj
WORKDIR ${PROJ_BUILD_DIR}
RUN wget https://download.osgeo.org/proj/proj-9.1.1.tar.gz -O proj-9.1.1.tar.gz
RUN tar xzf proj-9.1.1.tar.gz
WORKDIR ${PROJ_BUILD_DIR}/proj-9.1.1
RUN mkdir -p ${PROJ_BUILD_DIR}/proj-9.1.1/build
WORKDIR ${PROJ_BUILD_DIR}/proj-9.1.1/build
RUN cmake ..
RUN cmake --build .
RUN cmake --build . --target install

FROM scratch

# yum dependency shared objects
COPY --from=ext /usr/lib64/libtiff.so.5 /opt/bref/lib64/libtiff.so.5
COPY --from=ext /usr/lib64/libjbig.so.2.0 /opt/bref/lib64/libjbig.so.2.0
COPY --from=ext /usr/lib64/libjpeg.so.62 /opt/bref/lib64/libjpeg.so.62

# compiled dependency shared objects
COPY --from=ext /usr/local/lib/libsqlite3.so.0 /opt/bref/lib64/libsqlite3.so.0

# compiled shared objects
COPY --from=ext /tmp/build/proj/proj-9.1.1/build/lib/* /opt/bref/lib64/

# compiled binaries
COPY --from=ext /tmp/build/proj/proj-9.1.1/build/bin/* /opt/bin/

# compiled config files
COPY --from=ext /tmp/build/proj/proj-9.1.1/build/data/* /opt/360geo/proj/config/

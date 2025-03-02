ARG UBUNTU_VERSION=24.04
FROM ubuntu:${UBUNTU_VERSION} AS ubuntu

FROM ubuntu AS java
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update -q \
 && apt-get upgrade -q -y \
 && apt-get install -q -y \
        binutils \
        openjdk-11-jdk-headless

RUN jlink --compress=2 --strip-debug --add-modules=java.base --output /opt/java \
    && strip -p --strip-unneeded /opt/java/lib/server/*.so \
    && strip -p --strip-unneeded /opt/java/lib/*.so

FROM ubuntu
ARG UBUNTU_VERSION
LABEL org.srcml.email="srcmldev@gmail.com" \
      org.srcml.url="srcml.org" \
      org.srcml.distro="ubuntu" \
      org.srcml.osversion="${UBUNTU_VERSION}"

# Copy lightweight jre install
COPY --from=java /opt/java /opt/java
ENV JAVA_HOME=/opt/java

# Avoid prompts for timezone
ENV TZ=US/Michigan
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Update and install dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
    curl \
    git \
    zip \
    g++ \
    make \
    cmake \
    ninja-build \
    libxml2-dev \
    libxml2-utils \
    libxslt1-dev \
    libarchive-dev \
    libcurl4-openssl-dev \
    ca-certificates \
    libssl-dev \
    cpio \
    man \
    file \
    dpkg-dev \
    libboost-all-dev \
    antlr \
    && rm -rf /var/lib/apt/lists/*

ENV CLASSPATH=/usr/share/java/antlr.jar

# Install antlr2 from source
COPY antlr-2.7.7-gcc-13.patch /tmp
RUN cd /tmp && \
    curl --insecure -L "https://www.antlr2.org/download/antlr-2.7.7.tar.gz" | tar xz && \
    cd antlr-2.7.7 && \
    git apply /tmp/antlr-2.7.7-gcc-13.patch && \
    export CXXFLAGS=-fPIC && \
    ./configure && \
    make -j && \
    make install && \
    cd /tmp \
    rm -rf "antlr-2.7.7*"

# Allow man pages to be installed
RUN sed -i '/path-exclude=\/usr\/share\/man\/*/c\#path-exclude=\/usr\/share\/man\/*' /etc/dpkg/dpkg.cfg.d/excludes

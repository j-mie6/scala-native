# For list of supported platforms check https://github.com/dockcross/dockcross#summary-cross-compilers
ARG TARGET_PLATFORM
FROM dockcross/$TARGET_PLATFORM
ENV DEFAULT_DOCKCROSS_IMAGE testing-container

RUN apt-get update && apt-get install -y unzip lsb-release wget software-properties-common

# Install clang-14, by default llvm.sh install latest stable version
RUN wget -O - https://apt.llvm.org/llvm.sh | bash /dev/stdin 14

RUN update-alternatives --install /usr/bin/clang clang /usr/bin/clang-14 100
RUN update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-14 100

# We cannot easily install dependencies, clone and build zlib and Boehm locally
RUN git clone https://github.com/madler/zlib /tmp/zlib \
  && cd /tmp/zlib/ \
  && git checkout v1.2.11 \
  && ./configure \
  && make install prefix=$QEMU_LD_PREFIX \
  && rm -rf /tmp/zlib

RUN git clone https://github.com/ivmai/bdwgc /tmp/bdwgc \
  && cd /tmp/bdwgc/ \
  && git checkout v8.0.6 \
  && git clone https://github.com/ivmai/libatomic_ops \
  && cd libatomic_ops/ \
  && git checkout v7.6.12 \
  && cd .. \
  && ./autogen.sh \
  && ./configure --host $CROSS_TRIPLE \
  && make install prefix=$QEMU_LD_PREFIX \
  && rm -rf /tmp/bdwgc

# Switch shell and user to allow for usage of sdk and installed by it binaries 
SHELL ["/bin/bash", "-c"]
RUN useradd -ms /bin/bash scala-native
USER scala-native
WORKDIR /home/scala-native/scala-native

RUN curl -s "https://get.sdkman.io" | bash  \
  && . "$HOME/.sdkman/bin/sdkman-init.sh" \
  && sdk install sbt 1.6.2 \ 
  && sdk install java 8.0.332-tem

ENV LC_ALL "C.UTF-8"
ENV LANG "C.UTF-8"
ENV PATH=/usr/lib/llvm-6.0/bin:~/.sdkman/candidates/java/current/bin:~/.sdkman/candidates/sbt/current/bin:${PATH}

CMD sbt "++ $SCALA_VERSION -v" \
  "-Dscala.scalanative.testinterface.processrunner.emulator=$TARGET_EMULATOR" \
  "-J-Xmx5G" \
  "set sbtScalaNative/scriptedBufferLog := false" \
  "$TEST_COMMAND"

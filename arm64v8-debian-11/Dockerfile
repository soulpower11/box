#arm64v8
FROM debian:bullseye-slim as build

RUN dpkg --add-architecture armhf && \
    apt-get update && \
    apt-get install --yes --no-install-recommends git python3 build-essential cmake ca-certificates && \
    apt-get install --yes --no-install-recommends gcc-arm-linux-gnueabihf libc6-dev-armhf-cross libc6:armhf libstdc++6:armhf && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*

RUN git clone https://github.com/ptitSeb/box86.git; mkdir /box86/build && \
    git clone https://github.com/ptitSeb/box64.git; mkdir /box64/build

WORKDIR /box86/build
RUN cmake .. -DRPI4ARM64=1 -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo && \
    make -j$(nproc) && \
    make install DESTDIR=/tmp/install

WORKDIR /box64/build
RUN cmake .. -DRPI4ARM64=1 -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo && \
    make -j$(nproc) && \
    make install DESTDIR=/tmp/install

FROM debian:bullseye-slim
LABEL "maintainer"="soulpower11 <soulpower11@protonmail.com>"
LABEL "org.opencontainers.image.source"="https://github.com/soulpower11/box"

COPY --from=build /tmp/install /

RUN dpkg --add-architecture armhf && \
    apt-get update && \
    apt-get install --yes --no-install-recommends libc6:armhf libstdc++6:armhf && \
    apt-get -y autoremove && \
    apt-get clean autoclean && \
    rm -rf /tmp/* /var/tmp/* /var/lib/apt/lists

RUN         apt update -y \
            && apt upgrade -y \
            && apt install -y apt-transport-https wget curl iproute2 libgdiplus \
            && wget https://dot.net/v1/dotnet-install.sh \
            && D_V="$(curl -sSL https://dotnet.microsoft.com/en-us/download/dotnet/6.0 | grep -i  '<h3 id="sdk-6.*">SDK 6.*.*</h3>'  | head -1 | awk -F\" '{print $3}' | awk '{print $2;}' | sed 's/<\/h3>//g')" \   
            && chmod +x dotnet-install.sh \
			&& ./dotnet-install.sh -i /usr/share -v $D_V \
            && ln -s /usr/share/dotnet /usr/bin/dotnet

RUN adduser --disabled-password --home /home/container container

USER container
ENV  USER=container HOME=/home/container

WORKDIR /home/container

COPY ./entrypoint.sh /entrypoint.sh

CMD ["/bin/bash", "/entrypoint.sh"]

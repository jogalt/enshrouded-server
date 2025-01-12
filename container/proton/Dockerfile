# Start with a base image
FROM ubuntu:latest

# Set container-specific build arguments
ARG CONTAINER_GID=10000
ARG CONTAINER_UID=10000
ARG GE_PROTON_VERSION=9-18

# Set environment variables
ENV DEBIAN_FRONTEND "noninteractive"
#AppID for Enshrouded Desktop App
ENV STEAM_APP_ID "2278520"
#AppID for Enshroud Server
#ENV STEAM_APP_ID "1203620"
ENV HOME "/home/steam"
ENV ENSHROUDED_PATH "/home/steam/enshrouded"
ENV ENSHROUDED_CONFIG "${ENSHROUDED_PATH}/enshrouded_server.json"
ENV EXTERNAL_CONFIG 0
ENV GE_PROTON_URL "https://github.com/GloriousEggroll/proton-ge-custom/releases/download/GE-Proton${GE_PROTON_VERSION}/GE-Proton${GE_PROTON_VERSION}.tar.gz"
ENV STEAMCMD_PATH="/home/steam/steamcmd"
ENV STEAM_SDK64_PATH="/home/steam/.steam/sdk64"
ENV STEAM_SDK32_PATH="/home/steam/.steam/sdk32"
ENV STEAM_COMPAT_CLIENT_INSTALL_PATH "$STEAMCMD_PATH"
ENV STEAM_COMPAT_DATA_PATH "/home/steam/steamcmd/steamapps/compatdata/${STEAM_APP_ID}"
ENV UMU_ID 0

# Install required dependencies and create the steam user
RUN dpkg --add-architecture i386
RUN apt-get update && apt-get install --no-install-recommends -y \
        ubuntu-server \
        procps \
        ca-certificates \
        winbind \
        dbus \
        libfreetype6 \
        curl \
        jq \
        locales \
        lib32gcc-s1 \
        tcpdump
RUN groupadd -g ${CONTAINER_GID} steam
RUN useradd -m -u ${CONTAINER_UID} -g ${CONTAINER_GID} steam
RUN echo 'LANG="en_US.UTF-8"' > /etc/default/locale
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN locale-gen
RUN rm -f /etc/machine-id
RUN dbus-uuidgen --ensure=/etc/machine-id
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get clean
RUN apt-get autoremove -y 

# Switch to the steam user
USER steam

# Download and configure SteamCMD and Proton
RUN mkdir -p ${ENSHROUDED_PATH}/savegame 
RUN mkdir -p /home/steam/steamcmd/compatibilitytools.d 
RUN mkdir -p /home/steam/steamcmd/steamapps/compatdata/${STEAM_APP_ID} 
RUN curl -sqL https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz | tar zxvf - -C /home/steam/steamcmd 
RUN chmod +x /home/steam/steamcmd/steamcmd.sh 
RUN /home/steam/steamcmd/steamcmd.sh +quit 
RUN mkdir -p /home/steam/.steam 
RUN ln -s /home/steam/steamcmd/linux64 ${STEAM_SDK64_PATH} 
RUN ln -s ${STEAM_SDK64_PATH}/steamclient.so ${STEAM_SDK64_PATH}/steamservice.so 
RUN ln -s /home/steam/steamcmd/linux32 ${STEAM_SDK32_PATH} 
RUN ln -s ${STEAM_SDK32_PATH}/steamclient.so ${STEAM_SDK32_PATH}/steamservice.so 
RUN curl -sqL ${GE_PROTON_URL} | tar zxvf - -C /home/steam/steamcmd/compatibilitytools.d/

# Copy necessary files
COPY container/proton/entrypoint.sh /home/steam/entrypoint.sh
COPY container/proton/enshrouded_server_example.json /home/steam/enshrouded_server_example.json

# Set working directory
WORKDIR /home/steam

USER root

# Ensure entrypoint script is executable
RUN chmod +x /home/steam/entrypoint.sh

USER steam

EXPOSE 15636/udp
EXPOSE 15637/udp

# Default command
CMD ["/home/steam/entrypoint.sh"]

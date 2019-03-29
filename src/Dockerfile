FROM mcr.microsoft.com/dotnet/core/runtime:2.2-alpine3.9

LABEL maintainer JJTC "docker@jjtc.eu"

ENV TZ=Europe/Vienna \
    CONANEXILES_MASTERSERVER=1 \
    CONANEXILES_Game_RconPlugin_RconEnabled=1 \
    CONANEXILES_Game_RconPlugin_RconPassword=Password \
    CONANEXILES_Game_RconPlugin_RconPort=25575 \
    CONANEXILES_Game_RconPlugin_RconMaxKarma=60

RUN apk update \
    && apk add wine-4.0-r0.apk rcon.apk \ 
    && apk add --no-cache bash elektra gnutls xvfb wine redis supervisord \
    && mkdir -p ~/depotdownloader \
    && cd ~/depotdownloader \
    && wget https://github.com/SteamRE/DepotDownloader/releases/download/DepotDownloader_2.3.0/depotdownloader-2.3.0-hotfix1.zip -O depotdownloader.zip \
    && unzip depotdownloader.zip \
   


#-----------------------------------




FROM ubuntu:18.10

LABEL maintainer JJTC "docker@jjtc.eu"

ENV TIMEZONE=Europe/Vienna \
    DEBIAN_FRONTEND=noninteractive \
    CONANEXILES_MASTERSERVER=1 \
    CONANEXILES_Game_RconPlugin_RconEnabled=1 \
    CONANEXILES_Game_RconPlugin_RconPassword=Password \
    CONANEXILES_Game_RconPlugin_RconPort=25575 \
    CONANEXILES_Game_RconPlugin_RconMaxKarma=60

RUN dpkg --add-architecture i386 \
    && apt-get update \
    && apt-get install -y crudini python3-pip redis-tools software-properties-common supervisor unzip wget xvfb \
    && wget https://dl.winehq.org/wine-builds/winehq.key \
    && apt-key add winehq.key \
    && apt-add-repository 'https://dl.winehq.org/wine-builds/ubuntu/' \
    && apt-get update \
    && apt-get install --no-install-recommends --assume-yes winehq-staging \
    && pip3 install python-valve \
    && apt-get clean \
    && rm -rf winehq.key /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime \
    && echo $TIMEZONE > /etc/timezone

COPY . ./

RUN chmod +x /entrypoint.sh \
    && cd /usr/bin/ \
    && chmod +x conanexiles_controller rconcli \
    # Create steam user and group
    && useradd -ms /bin/bash steam \
    # Ensure steam user got permission
    && touch /bash.bashrc /supervisord.log \
    && mkdir /conanexiles /steamcmd /wine \
    && chown steam -cR /conanexiles /steamcmd /wine /bash.bashrc /supervisord.log /etc/supervisor/conf.d/ \
    # xvfb fix
    && mkdir -p /tmp/.X11-unix \
    && chmod 1777 /tmp/.X11-unix \
    && chown root /tmp/.X11-unix/ \
    # supervisor fix
    && mkdir -p /var/run/ \
    && touch /var/run/supervisor.sock \
    && chmod 700 /var/run/supervisor.sock \
    && chown steam /var/run/supervisor.sock

USER steam

EXPOSE 7777/udp 27015/udp 27016/udp 37015/udp 37016/udp

VOLUME ["/conanexiles"]

ENTRYPOINT ["/entrypoint.sh"]
CMD ["supervisord"]

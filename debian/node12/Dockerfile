FROM debian:bullseye-slim

LABEL org.opencontainers.image.title="Official ioBroker Docker Image" \
      org.opencontainers.image.description="Officical Docker image for ioBroker smarthome software (https://www.iobroker.net)" \
      org.opencontainers.image.documentation="https://github.com/buanet/ioBroker.docker#readme" \
      org.opencontainers.image.authors="André Germann <info@buanet.de>" \
      org.opencontainers.image.url="https://github.com/buanet/ioBroker.docker" \
      org.opencontainers.image.source="https://github.com/buanet/ioBroker.docker" \
      org.opencontainers.image.base.name="docker.io/library/debian:bullseye-slim" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.created="${DATI}"

ENV DEBIAN_FRONTEND noninteractive

# Install prerequisites (including node) and generating locales
RUN apt-get update && apt-get install -y \
    curl \
    gosu \
    jq \
    locales \
    procps \
    sudo \
    udev \
    wget \
    # Generating locales
    && sed -i 's/^# *\(de_DE.UTF-8\)/\1/' /etc/locale.gen \
    && sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen \
    && locale-gen \
    # Install node
    && curl -sL https://deb.nodesource.com/setup_12.x | bash \
    && apt-get update && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Create scripts directorys and copy scripts
RUN mkdir -p /opt/scripts/ \
    && mkdir -p /opt/userscripts/ \
    && chmod 777 /opt/scripts/ \
    && chmod 777 /opt/userscripts/
WORKDIR /opt/scripts/
COPY scripts/* ./
RUN chmod +x *.sh
WORKDIR /opt/userscripts/
COPY userscripts/* ./
RUN chmod +x *.sh

# Install ioBroker
WORKDIR /
RUN apt-get update \
    && curl -sL https://iobroker.net/install.sh | bash - \
    && mkdir -p /opt/scripts/.docker_config/ \
    && echo "starting" > /opt/scripts/.docker_config/.healthcheck \
    && echo "${VERSION}" > /opt/scripts/.docker_config/.thisisdocker \
    && echo $(hostname) > /opt/.firstrun \
    # Deleting UUID from build
    && iobroker unsetup -y \
    # Backup initial ioBroker and userscript folder
    && tar -cf /opt/initial_iobroker.tar /opt/iobroker \
    && tar -cf /opt/initial_userscripts.tar /opt/userscripts \
    && rm -rf /var/lib/apt/lists/*

# Setting up iobroker-user (shell, home dir and rights)
RUN chsh -s /bin/bash iobroker \
    && usermod --home /opt/iobroker iobroker \
    && usermod -u 1000 iobroker \
    && groupmod -g 1000 iobroker \
    && chown root:iobroker /usr/sbin/gosu \
    && chmod +s /usr/sbin/gosu

# Setting up default ENVs
ENV DEBIAN_FRONTEND="teletype" \
    LANG="de_DE.UTF-8" \
    LANGUAGE="de_DE:de" \
    LC_ALL="de_DE.UTF-8" \
    SETGID=1000 \
    SETUID=1000 \
    TZ="Europe/Berlin"

# Expose default admin ui port
EXPOSE 8081

# Change work dir
WORKDIR /opt/iobroker/

# Healthcheck
HEALTHCHECK --interval=15s --timeout=5s --retries=5 \
    CMD ["/bin/bash", "-c", "/opt/scripts/healthcheck.sh"]

# Run startup-script
ENTRYPOINT ["/bin/bash", "-c", "/opt/scripts/iobroker_startup.sh"]

FROM ubuntu:22.04

ARG Dotnet_Sdk_Version=6.0
ARG Powershell_Version=7.2.7-1.deb
ARG Umoci_Version=0.4.6

# get `wget` & software-properties-common
# https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-7#ubuntu-1804
RUN apt-get update && \
    apt-get install -y wget unzip apt-utils curl software-properties-common iputils-ping

# get powershell for 22.04
RUN wget -q https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb && \
    dpkg -i packages-microsoft-prod.deb && \
    apt-get update && \
    add-apt-repository universe && \
    apt-get install -y powershell=${Powershell_Version}

# Get .NET SDK 6.0
# https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-ubuntu-1804
# https://learn.microsoft.com/en-us/dotnet/core/install/linux-package-mixup
RUN DOTNET_CLI_TELEMETRY_OPTOUT=1 && \
    touch /etc/apt/preferences && \
    echo "Package: dotnet* aspnet* netstandard* \nPin: origin \"packages.microsoft.com\" \nPin-Priority: -10" > /etc/apt/preferences && \
    echo "export DOTNET_CLI_TELEMETRY_OPTOUT=1" > /etc/profile.d/set-dotnet-env-vars.sh && \
    apt-get install -y apt-transport-https && \
    apt-get update && \
    apt-get install -y dotnet-sdk-${Dotnet_Sdk_Version}

# Get tools for working with Docker images without the Docker daemon
# https://github.com/openSUSE/umoci
RUN curl --silent --location https://github.com/opencontainers/umoci/releases/download/v${Umoci_Version}/umoci.amd64 -o /usr/local/bin/umoci && \
    chmod +x /usr/local/bin/umoci

# Get common utilities for scripting
# https://mikefarah.gitbook.io/yq/
# https://augeas.net/
RUN add-apt-repository -y ppa:rmescandon/yq && \
    apt-get update && apt-get install -y jq yq openssh-client rsync git augeas-tools

ARG MSSQLTOOLS_VERSION=18.2.1.1
# Microsoft decided to have a suffix for newer versions of mssql-tools, e.g. mssql-tools18
ARG MSSQLTOOLS_SUFFIX=18
# and also the path changed on newer versions. It's a mess.
ARG MSSQLTOOLS_PATH=/opt/mssql-tools18

ENV PATH=$MSSQLTOOLS_PATH/bin:$PATH

RUN apt-get -qqq update \
    && apt-get install -y curl apt-transport-https locales gnupg2 \
        # Helper command to convert \r\n to \n,
        # since sqlcmd prints Windows line endings
        dos2unix \
    && locale-gen "en_US.UTF-8" \
    \
    && export `grep "VERSION_ID" /etc/os-release | sed -e 's/^VERSION_ID=\"/VERSION_ID=/' -e 's/\"$//'` \
    && mkdir -p /etc/apt/keyrings/ \
    && curl --fail https://packages.microsoft.com/config/ubuntu/$VERSION_ID/prod.list | \
        sed -E 's#deb\s+\[#deb [signed-by=/etc/apt/keyrings/microsoft.gpg #; t; q1' | \
        tee /etc/apt/sources.list.d/microsoft.list \
    && curl --fail https://packages.microsoft.com/keys/microsoft.asc | \
        gpg --verbose --yes --no-tty --batch --dearmor -o /etc/apt/keyrings/microsoft.gpg \
    \
    && apt-get -qqq update \
    && ACCEPT_EULA=Y apt-get install -y mssql-tools$MSSQLTOOLS_SUFFIX=$MSSQLTOOLS_VERSION unixodbc-dev \
    && apt-get remove -y curl apt-transport-https gnupg2 \
    && rm -f /etc/apt/sources.list.d/msprod.list \
    && rm -rf /var/lib/apt/lists/*

## should be set after locale was generated, overwise triggers warnings
ENV LANG="en_US.UTF-8" LANGUAGE="en_US.UTF-8" LC_ALL="en_US.UTF-8"
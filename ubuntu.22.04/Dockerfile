FROM ubuntu:22.04

ARG Azure_Powershell_Version=4.5.0
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

# Get AWS Powershell core modules
# https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-set-up-linux-mac.html
RUN pwsh -c 'Install-Module -Force -Name AWSPowerShell.NetCore -AllowClobber -Scope AllUsers -MaximumVersion "'${Aws_Powershell_Version}'"'

# Get AZ Powershell core modules
# https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-3.6.1
RUN pwsh -c 'Install-Module -Force -Name Az -AllowClobber -Scope AllUsers -MaximumVersion "'${Azure_Powershell_Version}'"'

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

# Get python3 & groff
RUN apt-get install -y python3-pip groff && \
    python3 -m pip install pycryptodome --user

# Install python2
RUN apt-get install -y python2-minimal=${Python2_Version} && \
    ln -s /usr/bin/python2 /usr/bin/python

# Get tools for working with Docker images without the Docker daemon
# https://github.com/openSUSE/umoci
RUN curl --silent --location https://github.com/opencontainers/umoci/releases/download/v${Umoci_Version}/umoci.amd64 -o /usr/local/bin/umoci && \
    chmod +x /usr/local/bin/umoci

# Get common utilities for scripting
# https://mikefarah.gitbook.io/yq/
# https://augeas.net/
RUN add-apt-repository -y ppa:rmescandon/yq && \
    apt-get update && apt-get install -y jq yq openssh-client rsync git augeas-tools
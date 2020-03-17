#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM ubuntu:18.04

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# [Optional] Version of Node.js to install.
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"
ENV NVM_DIR=/home/vscode/.nvm
ENV PATH=/dotnet:$PATH

# [Optional] Install the Azure CLI
ARG INSTALL_AZURE_CLI="false"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git iproute2 procps apt-transport-https gnupg2 curl lsb-release wget \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # [Optional] Install Node.js for ASP.NET Core Web Applicationss
    && if [ "$INSTALL_NODE" = "true" ]; then \
        #
        # Install nvm and Node
        mkdir ${NVM_DIR} \
        && curl -so- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash 2>&1 \
        && chown -R vscode:vscode ${NVM_DIR} \
        && /bin/bash -c "source $NVM_DIR/nvm.sh \
            && nvm install ${NODE_VERSION} \
            && nvm alias default ${NODE_VERSION}" 2>&1 \
        && INIT_STRING='[ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"  && [ -s "$NVM_DIR/bash_completion" ] && \\. "$NVM_DIR/bash_completion"' \
        && echo $INIT_STRING >> /home/vscode/.bashrc \
        && echo $INIT_STRING >> /home/vscode/.zshrc \
        && echo $INIT_STRING >> /root/.zshrc \
        #
        # Install yarn
        && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
        && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
        && apt-get update \
        && apt-get -y install --no-install-recommends yarn; \
    fi \
    #
    # [Optional] Install the Azure CLI
    && if [ "$INSTALL_AZURE_CLI" = "true" ]; then \
        echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
        && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
        && apt-get update \
        && apt-get install -y azure-cli; \
    fi \
    #
    # [Optional] Install dotnet core
    && cd / && wget https://dot.net/v1/dotnet-install.sh && chmod +x dotnet-install.sh && ./dotnet-install.sh --install-dir /dotnet --channel 3.1 \
    && wget http://archive.ubuntu.com/ubuntu/pool/main/i/icu/libicu55_55.1-7_amd64.deb \
    && wget http://archive.ubuntu.com/ubuntu/pool/main/l/llvm-toolchain-3.6/libllvm3.6v5_3.6.2-3ubuntu2_amd64.deb \
    && wget http://archive.ubuntu.com/ubuntu/pool/main/l/llvm-toolchain-3.6/liblldb-3.6_3.6.2-3ubuntu2_amd64.deb \
    && cd /dotnet && wget https://download.microsoft.com/download/F/4/F/F4FCB6EC-5F05-4DF8-822C-FF013DF1B17F/dotnet-dev-ubuntu.16.04-x64.1.1.4.tar.gz && tar -zxf dotnet-dev-ubuntu.16.04-x64.1.1.4.tar.gz && rm dotnet-dev-ubuntu.16.04-x64.1.1.4.tar.gz \
    && cd / \
    && apt-get install -y libcurl3 liblttng-ust0 libssl1.0.0 libkrb5-3 zlib1g libunwind8 libedit2 \
    && dpkg -i libicu55_55.1-7_amd64.deb || true \
    && dpkg -i libllvm3.6v5_3.6.2-3ubuntu2_amd64.deb || true  \
    && dpkg -i liblldb-3.6_3.6.2-3ubuntu2_amd64.deb || true \
    && apt --fix-broken -y install \
    && rm -f libicu52_52.1-3_amd64.deb liblldb-3.6_3.6-2ubuntu1~trusty2_amd64.deb libllvm3.6_3.6-2ubuntu1~trusty2_amd64.deb \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

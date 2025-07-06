FROM python:3.13.5-slim-bookworm

RUN mkdir /tmp/docker-build
COPY requirements.txt /tmp/docker-build/requirements.txt

ENV PYTHONIOENCODING=UTF-8
ENV PYTHONUTF8=1

# 須藤など
RUN apt update && apt install -y \
  sudo \
  locales \
  locales-all \
  software-properties-common \
  python3-launchpadlib \
  build-essential \
  ca-certificates \
  curl \
  wget \
  htop \
  tar \
  unzip \
  zip \
  jq \
  vim \
  pipx \
  gnupg2 \
  lsb-release \
  && rm -rf /var/lib/apt/lists/*
ENV LANG=ja_JP.UTF-8

# Github-CLIとFishのインストール(latest)
RUN echo "deb http://download.opensuse.org/repositories/shells:/fish:/release:/4/Debian_12/ /" | tee /etc/apt/sources.list.d/shells:fish:release:4.list \
  && curl -fsSL https://download.opensuse.org/repositories/shells:fish:release:4/Debian_12/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/shells_fish_release_4.gpg > /dev/null \
  && mkdir -p -m 755 /etc/apt/keyrings \
  && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  && cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && mkdir -p -m 755 /etc/apt/sources.list.d \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && apt update \
  && apt install gh fish -y \
  && rm -rf /var/lib/apt/lists/*

# ユーザーを作成
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME && \
  useradd --uid $USER_UID --gid $USER_GID -G sudo -s /usr/bin/fish -m $USERNAME \
  && echo "$USERNAME ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers \
  && chmod 0440 /etc/sudoers

ENV PATH=$PATH:/home/$USERNAME/.local/bin:/home/$USERNAME/.bun/bin

# 作成したユーザーに切り替え
USER $USERNAME
WORKDIR /home/$USERNAME/

# 適当なPythonパッケージをインストール
RUN pip install --upgrade pip \
  && pip install -r /tmp/docker-build/requirements.txt \
  && pip cache purge

# Node.jsとBunをインストール
RUN curl -fsSL https://bun.sh/install | bash
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
ENV NVM_DIR=/home/vscode/.nvm

SHELL ["/bin/fish", "-c"]
RUN curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
RUN fisher install FabioAntunes/fish-nvm edc/bass \
  && nvm install node \
  && nvm use node
SHELL ["/bin/bash", "-c"]

# uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
RUN pipx --version && pipx ensurepath && sudo pipx --global ensurepath


FROM python:3.13.5-slim-bookworm

RUN mkdir /tmp/docker-build
COPY requirements.txt /tmp/docker-build/requirements.txt

ENV PYTHONIOENCODING=UTF-8
ENV PYTHONUTF8=1

# 須藤など
RUN apt update
RUN apt install sudo software-properties-common python3-launchpadlib -y
RUN apt update && apt upgrade -y
RUN apt install -y \
  build-essential \
  curl \
  unzip \
  git \
  vim \
  wget \
  ca-certificates \
  gnupg2 \
  lsb-release

# Github-CLIとFishのインストール(latest)
RUN echo "deb http://download.opensuse.org/repositories/shells:/fish:/release:/4/Debian_12/ /" | tee /etc/apt/sources.list.d/shells:fish:release:4.list
RUN curl -fsSL https://download.opensuse.org/repositories/shells:fish:release:4/Debian_12/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/shells_fish_release_4.gpg > /dev/null

RUN mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& apt update \
	&& apt install gh -y
RUN apt install fish -y

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
RUN pip install --upgrade pip
RUN pip install -r /tmp/docker-build/requirements.txt

# Node.jsとBunをインストール(nodejsはまだ)
RUN curl -fsSL https://bun.sh/install | bash

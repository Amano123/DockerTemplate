FROM nvidia/cuda:11.0.3-base-ubuntu18.04

LABEL maintainer="yoyo"

ARG USER_NAME
ARG USER_ID
ARG GROUP_ID

ENV pip='python3 -m pip'

# サーバーを日本に変更
# 多分早くなる
RUN sed -i 's@archive.ubuntu.com@ftp.jaist.ac.jp/pub/Linux@g' /etc/apt/sources.list

RUN apt-get update \
&&  apt-get install -y --no-install-recommends \
                sudo \
                zsh \
                # vim \
                neovim \
                git \
                make \
                cmake \
                curl \
                wget \
                xz-utils \
                # あれば良し
                # nano \
                file \
                ## network
                net-tools \
                #japanase
                language-pack-ja-base \
                language-pack-ja \
                locales \
                # make user
                gosu

# localizing into Japanese
RUN locale-gen ja_JP.UTF-8 
ENV LANG ja_JP.UTF-8
ENV LC_CTYPE ja_JP.UTF-8
RUN localedef -f UTF-8 -i ja_JP ja_JP.utf8

# make user
RUN echo "Starting with UID : ${USER_ID}, GID: ${GROUP_ID}" \
&&  adduser --disabled-password --gecos "" "${USER_NAME}" \
&&  echo "${USER_NAME}:${USER_NAME}" | chpasswd \
&&  echo "%${USER_NAME}    ALL=(ALL)   NOPASSWD:    ALL" >> /etc/sudoers.d/${USER_NAME} \
&&  chmod 0440 /etc/sudoers.d/${USER_NAME} \
&&  groupmod -g ${GROUP_ID} ${USER_NAME} 

# user change : root -> make user 
USER ${USER_NAME}
WORKDIR /home/${USER_NAME}
CMD [ "zsh" ]
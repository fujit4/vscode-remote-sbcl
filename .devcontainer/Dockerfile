# To fully customize the contents of this image, use the following Dockerfile instead:
ARG ROSWELL_IMAGE=fukamachi/roswell
ARG ROSWELL_VERSION=20.01.14.104
ARG PLATFORM=debian
FROM $ROSWELL_IMAGE:$ROSWELL_VERSION-$PLATFORM
# FROM fukamachi/sbcl:2.0.3

ARG SBCL_VERSION=2.0.3

### from microsoft
# The node image includes a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=lisper
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Set to false to skip installing zsh and Oh My ZSH!
ARG INSTALL_ZSH="true"

# Location and expected SHA for common setup script - SHA generated on release
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/v0.112.0/script-library/common-debian.sh"
ARG COMMON_SCRIPT_SHA="28e3d552a08e0d82935ad7335837f354809bec9856a3e0c2855f17bfe3a19523"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog wget ca-certificates 2>&1 \
    #
    # Verify git, common tools / libs installed, add/modify non-root user, optionally install zsh
    && wget -q -O /tmp/common-setup.sh $COMMON_SCRIPT_SOURCE \
    && if [ "$COMMON_SCRIPT_SHA" != "dev-mode" ]; then echo "$COMMON_SCRIPT_SHA /tmp/common-setup.sh" | sha256sum -c - ; fi \
    && /bin/bash /tmp/common-setup.sh "$INSTALL_ZSH" "$USERNAME" "$USER_UID" "$USER_GID" \
    && rm /tmp/common-setup.sh

### add by myself
# install additional tools
RUN apt-get -y install --no-install-recommends rlwrap 2>&1

## install sbcl to non-root-user
# change user
USER ${USERNAME}

# install sbcl
RUN set -x; \
  ros install "sbcl-bin/${SBCL_VERSION}" \
    && ros use "sbcl-bin/${SBCL_VERSION}" \
    # Uninstall unnecessary files
    && ros -e '(ql:uninstall-dist "quicklisp")' -e '(ql-dist:install-dist "http://beta.quicklisp.org/dist/quicklisp.txt" :prompt nil)' \
    && rm -rf /home/${USERNAME}/.roswell/archives/* \
      /home/${USERNAME}/.roswell/src/sbcl-*

# return root user
USER root
RUN set -x; \
  printf '#!/bin/sh\nrlwrap ros run -- "$@"\n' > /usr/local/bin/sbcl \
  && chmod 777 /usr/local/bin/sbcl

## set alias user
# change user
USER ${USERNAME}

# set alias
RUN echo 'alias ros="rlwrap ros"' >> /home/${USERNAME}/.bashrc \
  echo 'alias ll="ls -al"' >> /home/${USERNAME}/.bashrc

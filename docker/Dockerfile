FROM ubuntu:focal

# replace shell with bash so we can source files
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# update the repository sources list
# and install dependencies
RUN apt-get update \
    && apt-get install -y curl \
    && apt-get -y autoclean

# nvm environment variables
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 10.24.1

# install nvm
# https://github.com/creationix/nvm#install-script
RUN curl --silent -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash

# install node and npm
RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

# add node and npm to path so the commands are available
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# confirm installation
RUN node -v
RUN npm -v

WORKDIR /app

# create local directory for tiles to prevent startup errors
# if this is not defined via a bind point to host
RUN mkdir /app/tiles

COPY package*.json /app/

RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get -y install \
  build-essential \
  libcairo2-dev \
  libgles2-mesa-dev \
  libgbm-dev \
  libllvm7 \
  libprotobuf-dev \
  libxxf86vm-dev \
  xvfb \
  curl \
  libuv1-dev \
  git \
  x11-utils && \
  rm -rf /var/lib/apt/lists/* && \
  # install node deps, then remove any that are not used in production
  npm install --no-save && \
  rm -rf "/root/.npm" && \
  npm uninstall --no-save typescript eslint @babel/cli && \
  npm prune --production && \
  rm -rf "/app/node_modules/@mapbox/tiletype/test" && \
  rm -rf "/app/node_modules/restify/benchmark" && \
  # WARNING: watch for changes to version to Yarn
  rm -rf "/opt/yarn-v1.17.3" && \
  # remove any auto installed packages that are no longer needed
  apt-get autoremove -y

EXPOSE 80
ENV DISPLAY=:99

# Copy just the compiled code
COPY ./dist/* /app/dist/

COPY ./docker/entrypoint.sh /root
RUN chmod +x /root/entrypoint.sh
ENTRYPOINT [ "/root/entrypoint.sh" ]
HEALTHCHECK CMD curl --fail http://localhost:80/health || exit 1


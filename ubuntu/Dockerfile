# vim:set ft=dockerfile:
#
# Ubuntu-based CircleCI Docker image.
# Use the latest Ubuntu Long-Term Support (LTS) release:
# https://hub.docker.com/_/ubuntu

FROM ubuntu:latest

LABEL maintainer="CircleCI Community & Partner Engineering Team <community-partner@circleci.com>"

# Set default shell for building image
SHELL ["/bin/bash", "-exo", "pipefail", "-c"]

# Install utilities
RUN apt-get update \
  && apt-get install -y \
    build-essential \
    bzip2 \
    ca-certificates \
    curl \
    git \
    gnupg \
    gzip \
    jq \
    locales \
    make \
    mercurial \
    net-tools \
    netcat \
    openssh-client \
    parallel \
    tar \
    unzip \
    wget \
    xvfb \
    zip \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Install latest stable Docker
RUN declare -i INDEX=0 \
  # Determine latest stable Docker version
  && while true; \
    do INDEX_VERSION=$(curl --show-error --location --fail --retry 3 \
      https://api.github.com/repos/docker/cli/tags | \
      jq --argjson index "$INDEX" '.[$index].name') \
      # Filter out betas & release candidates
      && if [[ $(echo "$INDEX_VERSION" | grep -v beta | grep -v rc) ]]; then \
        DOCKER_VERSION="${INDEX_VERSION:1:$((${#INDEX_VERSION} - 1 - 1))}" \
        && break; \
      else \
        INDEX=$((INDEX + 1)); \
      fi; \
    done \
  && DOCKER_VERSION_NUMBER="${DOCKER_VERSION:1}" \
  && DOCKER_BINARY_URL="https://download.docker.com/linux/static/stable/x86_64/docker-$DOCKER_VERSION_NUMBER.tgz" \
  # Download Docker tarball
  && curl --output docker.tgz --show-error --location --fail --retry 3 \
    "$DOCKER_BINARY_URL" \
  && tar xf docker.tgz && rm -f docker.tgz \
  # Install Docker binaries
  && BINARIES=$(ls docker) && mv docker/* /usr/bin && rm -rf docker \
  && for binary in $BINARIES; \
    do chmod +x "/usr/bin/$binary"; \
    done \
  # Verify installation
  && command -v docker \
  && docker --version | grep "$DOCKER_VERSION_NUMBER"

# Install latest stable docker-compose
RUN declare -i INDEX=0 \
  # Determine latest stable docker-compose version
  && while true; \
    do INDEX_VERSION=$(curl --show-error --location --fail --retry 3 \
      https://api.github.com/repos/docker/compose/releases | \
      jq --argjson index "$INDEX" '.[$index].name') \
      # Filter out betas & release candidates
      && if [[ $(echo "$INDEX_VERSION" | grep -v beta | grep -v rc) ]]; then \
        DOCKER_COMPOSE_VERSION="${INDEX_VERSION:1:$((${#INDEX_VERSION} - 1 - 1))}" \
        && break; \
      else \
        INDEX=$((INDEX + 1)); \
      fi; \
    done \
  && DOCKER_COMPOSE_BINARY_URL="https://github.com/docker/compose/releases/download/$DOCKER_COMPOSE_VERSION/docker-compose-Linux-x86_64" \
  # Download binary and shasum
  && curl -O --show-error --location --fail --retry 3 \
    "$DOCKER_COMPOSE_BINARY_URL" \
  && DOCKER_COMPOSE_RELEASE_BODY=$(curl \
    --show-error --location --fail --retry 3 \
    "https://api.github.com/repos/docker/compose/releases/tags/$DOCKER_COMPOSE_VERSION" | \
    jq '.body') \
  # Work around malformed shasum files
  && if [[ $(echo "$DOCKER_COMPOSE_RELEASE_BODY" | \
    grep -o -e "\`................................................................\` | \`docker-compose-Linux-x86_64") ]]; then \
      SHASUM_STRING=$(echo "$DOCKER_COMPOSE_RELEASE_BODY" | \
      grep -o -e "\`................................................................\` | \`docker-compose-Linux-x86_64" | \
      sed -E 's/`|\|//g'); \
    elif [[ $(echo "$DOCKER_COMPOSE_RELEASE_BODY" | \
      grep -o -e "\`docker-compose-Linux-x86_64\` | \`................................................................") ]]; then \
      SHASUM_STRING=$(echo "$DOCKER_COMPOSE_RELEASE_BODY" | \
      grep -o -e "\`docker-compose-Linux-x86_64\` | \`................................................................" | \
      sed -E 's/`|\|//g'); \
    fi \
  && SHASUM=$(echo "$SHASUM_STRING" | sed -E "s/docker-compose-Linux-x86_64| //g") \
  # Verify shasum
  && echo "$SHASUM  docker-compose-Linux-x86_64" | sha256sum -c \
  # Install docker-compose
  && mv "docker-compose-Linux-x86_64" /usr/bin/docker-compose \
  && chmod +x /usr/bin/docker-compose \
  # Verify version
  && command -v docker-compose \
  && docker-compose --version | grep "$DOCKER_COMPOSE_VERSION"

# Install latest stable dockerize
RUN DOCKERIZE_VERSION=$(curl --show-error --location --fail --retry 3 \
  https://api.github.com/repos/jwilder/dockerize/releases/latest | \
  jq '.tag_name' | sed -E 's/"//g') \
  # extract latest version from GitHub releases API
  && PLATFORM=linux-amd64 \
  # set platform; construct binary download URL
  && DOCKERIZE_BINARY_URL="https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-$PLATFORM-$DOCKERIZE_VERSION.tar.gz" \
  # download & install binary
  && curl -O --show-error --location --fail --retry 3 \
    "$DOCKERIZE_BINARY_URL" \
  && tar xf "dockerize-$PLATFORM-$DOCKERIZE_VERSION.tar.gz" \
  && rm -f "dockerize-$PLATFORM-$DOCKERIZE_VERSION.tar.gz" \
  && mv dockerize /usr/bin \
  && chmod +x /usr/bin/dockerize \
  # verify version
  && command -v dockerize \
  && dockerize --version | grep "$DOCKERIZE_VERSION"

# Install latest stable CircleCI CLI
RUN CIRCLECI_CLI_VERSION=$(curl --show-error --location --fail --retry 3 \
  https://api.github.com/repos/CircleCI-Public/circleci-cli/releases/latest | \
  jq '.tag_name' | sed -E 's/"//g') \
  # extract latest version from GitHub releases API
  && curl -fLSs https://circle.ci/cli | bash \
  # verify install; strip leading `v` from $CIRCLECI_CLI_VERSION
  && command -v circleci \
  && circleci --skip-update-check version | grep "${CIRCLECI_CLI_VERSION:1}"

# Set default shell for users
SHELL ["/bin/bash", "-c"]

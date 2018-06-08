# Docker alpine image with ruby node npm yarn for Ruby on Rails

Getting Started
----------------------

Add the `akirchner/alpine_ruby_node` image as the base for your Dockerfile. Please see the following
example on how to add additional packages for your Ruby on Rails application and build your gems,

### Dockerfile for Ruby on Rails application.

```bash
FROM akirchner/alpine_ruby_node:2.5.1

# Required apps for common rails application
ENV PACKAGES="\
    git \
    curl \
    postgresql-client \
    imagemagick \
    tzdata \
"

RUN apk add --no-cache $PACKAGES && rm -rf /usr/share/man /tmp/* /var/cache/apk/*

RUN mkdir -p /app
WORKDIR /app

# Install application gems

COPY Gemfile Gemfile.lock /app/

# Build dependencies for gems (will be removed after installation)
ENV BUILD_PACKAGES="\
    libxml2-dev \
    libxslt-dev \
    postgresql-dev \
"

# Build argument for skip "test development" gems in production
ARG BUILD_WITHOUT

# Install gem with installation and removal of gem dependencies and nokogiri build arguments
RUN set -x \
    && apk upgrade --no-cache \
    && apk add --no-cache --virtual build-dependencies \
        build-base \
    && apk add --no-cache \
        $BUILD_PACKAGES \
    && gem install bundler \
    && bundle config build.nokogiri --use-system-libraries \
        --with-xml2-config=/usr/bin/xml2-config \
        --with-xslt-config=/usr/bin/xslt-config \
    && bundle install --without $BUILD_WITHOUT \
    && apk del build-dependencies \
    && rm -rf /usr/share/man /tmp/* /var/cache/apk/*

# Start your rails app
CMD bundle exec puma -C ./config/puma.rb

COPY bin/ /app/bin/

COPY package.json yarn.lock /app/
RUN bin/yarn install

COPY . /app/

```

Don't forget to add the build arguments to your compose files.


### Compose arguments for development and testing

```bash
services:
  app:
    build:
      - BUILD_WITHOUT=staging production
```

### Compose arguments for poduction and staging

```bash
services:
  app:
    build:
      - BUILD_WITHOUT=development test
```

References
----------------------

This image uses the library ruby alpine image ass base and integrates the install script form
mhart/alpine-node for node npm and yar. Please check out this fantastic repositories.


https://github.com/docker-library/ruby/tree/699a04311386ecc98ca242fc9bdee17fb4008863/2.5/alpine3.7


https://github.com/mhart/alpine-node/blob/master/Dockerfile

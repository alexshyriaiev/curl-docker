###############################################################
#
# Copyright (C) 2020 James Fuller <jim.fuller@webcomposite.com>
#
# SPDX-License-Identifier: MIT
#
# pinning builder image to Alpine 3.11.5
###############################################################
FROM registry.hub.docker.com/library/alpine:3.11.5 AS builder

###############################################################
# set build args
###############################################################
ARG CURL_RELEASE_TAG=latest
ARG CURL_GIT_REPO=https://github.com/curl/curl.git
ARG CURL_CONFIGURE_OPTION
ARG LABEL_VERSION=1.0.0
ARG LABEL_NAME=curl
ARG LABEL_DESC=curl

###############################################################
# build curl
###############################################################
# install deps and use latest curl release source
RUN apk --update add libssh2 libssh2-dev libssh2-static \
        autoconf automake build-base                    \
        groff openssl curl-dev                          \
        python3 python3-dev                             \
        libtool curl stunnel perl                       \
        nghttp2 git

RUN mkdir /src
RUN git clone ${CURL_GIT_REPO} /src/curl
WORKDIR /src/curl

###############################################################
# get ca cert bundle from curl.haxx.se
###############################################################
RUN curl https://curl.haxx.se/ca/cacert.pem  -o /cacert.pem

###############################################################
# build the tag version
###############################################################
RUN ./buildconf && \
    autoreconf -vif && \
    ./configure ${CURL_CONFIGURE_OPTION} &&\
    make -j$(nproc) &&\
    #make test &&\
    make DESTDIR="/alpine/" install  -j$(nproc)

###############################################################
# pinning image to Alpine 3.11.5
###############################################################
FROM registry.hub.docker.com/library/alpine:3.11.5

ARG CURL_RELEASE_TAG=latest
ARG CURL_RELEASE_VERSION
ARG CURL_GIT_REPO=https://github.com/curl/curl.git

ENV CURL_VERSION ${CURL_RELEASE_VERSION}
ENV CURL_RELEASE_TAG ${CURL_RELEASE_TAG}
ENV CURL_GIT_REPO ${CURL_GIT_REPO}

###############################################################
# define docker labels
###############################################################
LABEL Maintainer="James Fuller <jim.fuller@webcomposite.com>"
LABEL Name="curl"
LABEL Version="${LABEL_VERSION}"
LABEL docker.cmd="docker run -it my_curl -s -L http://curl.haxx.se"

###############################################################
# dependencies
###############################################################
RUN apk update --no-cache && \
    apk upgrade --no-cache && \
    apk add --update --no-cache libssh2 nghttp2-dev && \
    rm -fr /var/cache/apk/*

###############################################################
# add non privileged curl user
###############################################################
RUN addgroup -S curl_group && adduser -S curl_user -G curl_group

###############################################################
# set curl ca bundle
###############################################################
COPY --from=builder "/cacert.pem" "/cacert.pem"
ENV CURL_CA_BUNDLE="/cacert.pem"

###############################################################
# install curl built from builder
###############################################################
COPY --from=builder "/alpine/usr/local/lib/libcurl.so.4.7.0" "/usr/lib/"
COPY --from=builder "/alpine/usr/local/bin/curl" "/usr/bin/curl"

# explicitly set symlinks
RUN ln -s /usr/lib/libcurl.so.4.7.0 /usr/lib/libcurl.so.4
RUN ln -s /usr/lib/libcurl.so.4 /usr/lib/libcurl.so

###############################################################
# set user
###############################################################
USER curl_user

###############################################################
# set entrypoint
###############################################################
COPY "entrypoint.sh" "/entrypoint.sh"
CMD ["curl"]
ENTRYPOINT ["/entrypoint.sh"]

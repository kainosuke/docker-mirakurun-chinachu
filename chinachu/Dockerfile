## Repository loader
FROM node:10-alpine as repository-loader

ARG REPOSITORY="git://github.com/Chinachu/Chinachu.git"
ARG BRANCH="gamma"

ARG WORK_DIR="/home/node/chinachu"

RUN set -x \
	&& apk upgrade --update \
	&& apk add --no-cache \
		git

USER node
RUN set -x \
  && git clone --branch ${BRANCH} --depth 1 --recurse-submodules --shallow-submodules --jobs `nproc` ${REPOSITORY} ${WORK_DIR}

## Base
FROM node:10-alpine as base

ENV DOCKER="YES"
ARG WORK_DIR="/home/node/chinachu"

RUN set -x \
	&& apk upgrade --update \
	&& apk add --no-cache \
		bash \
		coreutils \
		curl \
		procps \
		ca-certificates

USER node
#COPY --from=repository-loader --chown=node:node ${WORK_DIR} ${WORK_DIR}
WORKDIR ${WORK_DIR}

## Builder base
FROM base as builder-base

USER root
RUN set -x \
	&& apk add --no-cache --virtual .build-deps \
	  git \
		make \
		gcc \
		g++ \
		autoconf \
		automake \
		wget \
		curl \
		tar \
		xz \
		libc-dev \
		musl-dev \
		eudev-dev \
		libevent-dev \
		perl-utils

USER node

## ffmpeg builder
FROM builder-base as ffmpeg-builder

COPY --from=repository-loader --chown=node:node ${WORK_DIR}/chinachu .
RUN set -x \
  && echo 5 | ./chinachu installer

## Dependencies  resolver
FROM builder-base as deps-resolver

COPY --from=repository-loader --chown=node:node ${WORK_DIR}/package*.json .
RUN set -x \
	&& npm install --production --prefer-offline --no-audit
RUN ls node_modules

## Deploy
FROM base as deploy

COPY --from=repository-loader --chown=node:node ${WORK_DIR} .
RUN ls .
COPY  --from=ffmpeg-builder ${WORK_DIR}/usr/bin ./usr/bin
RUN ls -r usr
COPY  --from=deps-resolver /home/node/chinachu/node_modules ./node_modules
COPY config.sample.json ./

CMD ["app-operator.js"]

EXPOSE 10772 20772 5353


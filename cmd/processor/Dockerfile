# Copyright 2017 The Nuclio Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# variables injected from makefile

ARG GOOS=linux

# used for building the processor itself (amd64, arm, etc)
ARG GOARCH=${NUCLIO_ARCH}

# used for getting the docker base image (amd64, arm32v6, arm32v7, 386, etc)
ARG NUCLIO_IMAGE_PROCESSOR=${NUCLIO_ARCH}

# TODO: change "golang:x.x.x" to "golang:x.x.x-<OS>" matching the architecture (alpine for arm32v6, jessie for arm32v7, either one for amd/intel)
FROM ${NUCLIO_IMAGE_PROCESSOR}/golang:1.9.3 as build

# inject from outside the "FROM"
ARG GOARCH
ARG GOOS
ARG NUCLIO_IMAGE_PROCESSOR

# copy source tree
WORKDIR /go/src/github.com/nuclio/nuclio
COPY . .

# build the processor
RUN apt-get install gcc git  \
    && go get github.com/v3io/v3io-go-http \
    && go get github.com/nuclio/logger \
    && go get github.com/nuclio/nuclio-sdk-go \
    && go get pack.ag/amqp \
    && GOOS=$GOOS GOARCH=$GOARCH CGO_ENABLED=0 go build -a -installsuffix cgo -ldflags="-s -w" -o processor cmd/processor/main.go

# copied from controller, needs further testing
FROM ${NUCLIO_IMAGE_PROCESSOR}/alpine

# only necessary if using alpine for the builder - doesn't work on arm32v7
RUN apk add --no-cache ca-certificates

# copy controller binary from build stage
COPY --from=build /go/src/github.com/nuclio/nuclio/processor /usr/local/bin

# inject once again, since not efective from outside the "FROM"
ARG GOARCH
ARG GOOS

ENV GOOS=${GOOS} GOARCH=${GOARCH}

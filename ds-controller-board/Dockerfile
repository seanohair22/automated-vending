# Copyright © 2022 Intel Corporation. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause

FROM golang:1.18-alpine3.16 AS builder

LABEL license='SPDX-License-Identifier: BSD-3-Clause' \
  copyright='Copyright (c) 2022: Intel'


# add git for go modules
RUN apk update && apk add --no-cache make git gcc libc-dev zeromq-dev linux-headers

ENV GO111MODULE=on
WORKDIR /usr/local/bin/

# This caches the packages for use when building the other services.
# Update the go.mod file in this repo when a new package is added to one of the services.
# This will be obvious when building a service and the un-cached package it loaded every build.
COPY go.mod .
RUN go mod tidy
RUN go mod download

WORKDIR /app
COPY . .

# Compile the code
RUN make gobuild

# Next image - Copy built Go binary into new workspace
FROM alpine

LABEL license='SPDX-License-Identifier: BSD-3-Clause' \
  copyright='Copyright (c) 2022: Intel'

RUN apk add --update --no-cache zeromq dumb-init

COPY --from=builder /app/res/profiles/ds-controller-board.yaml /res/profiles/ds-controller-board.yaml
COPY --from=builder /app/res/devices/device-list.toml /res/devices/device-list.toml
COPY --from=builder /app/res/configuration.toml /res/configuration.toml
COPY --from=builder /app/ds-controller-board /ds-controller-board

CMD [ "/ds-controller-board", "-cp=consul.http://edgex-core-consul:8500", "-r" ]

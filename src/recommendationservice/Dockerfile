# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Define a default value so it's not empty if the builder fails to provide it
ARG BUILDPLATFORM=linux/amd64

FROM --platform=$BUILDPLATFORM python:3.14.3-alpine@sha256:faee120f7885a06fcc9677922331391fa690d911c020abb9e8025ff3d908e510 AS base

FROM base AS builder

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN apk update \
    && apk add --no-cache g++ linux-headers wget \
    && rm -rf /var/cache/apk/*

# Add grpc_health_probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.24
RUN wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

# Install dependencies
WORKDIR /install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN apk update \
    && apk add --no-cache libstdc++ \
    && rm -rf /var/cache/apk/*

# get packages
WORKDIR /recommendationservice

# Grab packages from builder
COPY --from=builder /usr/local/lib/python3.14/ /usr/local/lib/python3.14/

# Copy grpc_health_probe
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe

# Add the application
COPY . .

# Fix permissions + drop root
RUN chown -R 1000:1000 /recommendationservice
USER 1000

# set listen port
ENV PORT="8080"
EXPOSE 8080

ENTRYPOINT ["python", "recommendation_server.py"]

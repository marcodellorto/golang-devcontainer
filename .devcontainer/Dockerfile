FROM golangci/golangci-lint:v1.55.2-alpine AS golangci
FROM amazon/aws-cli:2.15.5 AS aws
FROM docker:24.0.7-dind AS docker

FROM golang:1.21.5-bookworm AS base

COPY --from=golangci /usr/bin/golangci-lint /usr/bin/golangci-lint

RUN apt-get update \
    ; apt-get install -y \
        openssh-client \
        git \
        make \
        curl \
        zip \
    ; rm -rf /var/lib/apt/lists/* \
;

FROM base AS sam

ENV SAM_CLI_TELEMETRY=0

COPY --from=docker /usr/local/bin/docker /usr/local/bin/docker
COPY --from=docker /usr/local/bin/docker-compose /usr/local/bin/docker-compose
COPY --from=aws /usr/local/aws-cli /usr/local/aws-cli

RUN ln -s /usr/local/aws-cli/v2/current/bin/aws /usr/local/bin/aws \
    ; ln -s /usr/local/aws-cli/v2/current/bin/aws_completer /usr/local/bin/aws_completer \
    ; apt-get update \
    ; apt-get install -y \
        python3 \
        python3-dev \
        python3-pip \
    ; pip3 install -U pip \
    ; pip3 install aws-sam-cli --upgrade --break-system-packages \
    ; rm -rf /var/lib/apt/lists/* \
;

FROM sam AS dev

RUN GO111MODULE=on go install -v golang.org/x/tools/gopls@latest \
    ; GO111MODULE=on go install -v honnef.co/go/tools/cmd/staticcheck@latest \
    ; GO111MODULE=on go install -v golang.org/x/lint/golint@latest \
    ; GO111MODULE=on go install -v github.com/mgechev/revive@latest \
    ; GO111MODULE=on go install -v github.com/uudashr/gopkgs/v2/cmd/gopkgs@latest \
    ; GO111MODULE=on go install -v github.com/ramya-rao-a/go-outline@latest \
    ; GO111MODULE=on go install -v github.com/go-delve/delve/cmd/dlv@latest \
    ; GO111MODULE=on go install -v golang.org/x/lint/golint@latest

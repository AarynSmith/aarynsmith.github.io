FROM golang:1.13-alpine

ARG HUGO=hugo_extended
ARG HUGO_VERSION=0.73.0
ARG HUGO_SHA=7238b1b39d50667190020c93370bb2727d9097226cb3147c982d4d40004da09f
ARG HUGO_EXTENDED_SHA=dd7ea5d7f9d2a512359fcd51f94a2e5c7ab34d1a325dbf3ea675bc5c7f5082f3

RUN set -eux && \
    case ${HUGO} in \
    *_extended) \
    HUGO_SHA="${HUGO_EXTENDED_SHA}" ;; \
    esac && \
    apk add --update --no-cache ca-certificates openssl git && \
    wget -O ${HUGO_VERSION}.tar.gz https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/${HUGO}_${HUGO_VERSION}_Linux-64bit.tar.gz && \
    echo "${HUGO_SHA}  ${HUGO_VERSION}.tar.gz" | sha256sum -c && \
    tar xf ${HUGO_VERSION}.tar.gz && mv hugo* /usr/bin/hugo
RUN go get github.com/yaegashi/muslstack
RUN muslstack -s 0x800000 /usr/bin/hugo

FROM alpine:edge
ARG HUGO=hugo_extended
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

COPY --from=0 /usr/bin/hugo /usr/bin
RUN set -eux && \
    case ${HUGO} in \
    *_extended) \
    apk add --update --no-cache libc6-compat libstdc++ sudo git openssh-client less &&\
    rm -rf /var/cache/apk/* ;; \
    esac && \
    hugo version \
    && addgroup -g ${USER_GID} ${USERNAME}\
    && adduser -s /bin/ash -u ${USER_UID} -D -G ${USERNAME} ${USERNAME} \
    && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME}\
    && chmod 0440 /etc/sudoers.d/${USERNAME}
EXPOSE 1313
WORKDIR /src
CMD ["/usr/bin/hugo server"]

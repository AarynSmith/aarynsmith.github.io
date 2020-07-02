---
title: "Hugo + VSCode + Docker = ❤️"
subtitle: "Setting up a devcontainer for VSCode to run Hugo"
date: 2020-06-30
draft: false
author: ""
authorLink: ""
description: ""

tags: ["hugo", "vscode", "docker"]
categories: ["Blog"]

hiddenFromHomePage: false

featuredImage: ""
featuredImagePreview: ""

comments:
  enable: true
toc:
  enable: true
  auto: true
math:
  enable: false

---

In a recent attempt to dockerize anything I can, I came across a new feature of VSCode Remote Containers.
<!--more-->

VSCode recently added a Remote extension to develop inside docker containers, called [Visual Studio Code Remote - Containers](https://code.visualstudio.com/blogs/2019/05/02/remote-development). I took inspiration from my Hugo setup with gitlab pages to automatically build a static version of my site whenever I push to the repository, and saw an existing Jekyll remote container configuration, and thought, why not roll my own Hugo remote container.

## Hugo Container

I looked at the [container registry](https://gitlab.com/pages/hugo/-/tree/registry) for the Hugo image I have configured with Gitlab CI to build my site. There I found a two stage dockerfile, the first stage builds from the golang:1.13-alpine image, downloads, verifies and extracts the prebuilt hugo binary from the Github releases, then uses [muslstack](https://github.com/yaegashi/muslstack) to extend the default thread stack size to 8MB to work around segfault issues.

```dockerfile
FROM golang:1.13-alpine
ARG HUGO=hugo
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
```

The second stage builds from the alpine:edge image, copies the hugo binary from the first (0) stage, builds the site with an image with minimal dependencies.

```dockerfile
FROM alpine:edge
ARG HUGO=hugo
COPY --from=0 /usr/bin/hugo /usr/bin
RUN set -eux && \
    case ${HUGO} in \
      *_extended) \
        apk add --update --no-cache libc6-compat libstdc++ && \
        rm -rf /var/cache/apk/* ;; \
    esac && \
    hugo version
EXPOSE 1313
WORKDIR /src
CMD ["/usr/bin/hugo"]
```

## Jekyll Remote Container

Setting up a Jekyll remote container yields a few new files in the working folder:

* .devcontainer/devcontainer.json
* .devcontainer/Dockerfile
* .vscode/tasks.json

The [devcontainer.json](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) file contains some configuration settings for VScode and the tasks.json file defines some [tasks](https://code.visualstudio.com/docs/editor/tasks#vscode) that can be run from **Terminal > Run Task** or from the Command Palette using the command **Tasks: Run Task**

The dockerfile contained the configuration of a docker container for building a jekyll image. I found the parts of the dockerfile relevant to setting up a remote user:

```dockerfile
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
...
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME
```

The rest of the dockerfile installs git, openssh, less, ruby, bundler, jekyll and their dependencies, then cleaning up any install files.

## Devcontainer Dockerfile

I created a new two stage dockerfile in .devcontainer that, first, installs some setup dependencies, then downloads the latest hugo_extended binary, and updates it with muslstack. It takes two build arguments to specify the hugo variant, and version number.

```dockerfile
FROM golang:1.13-alpine

ARG VARIANT=hugo_extended
ARG VERSION=latest
RUN apk add --update --no-cache ca-certificates openssl git curl && \
    env && \
    case ${VERSION} in \
    latest) \
    echo "Getting latest version tag" && export VERSION=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4)}') ;;\
    esac && \
    echo ${VERSION} && \
    wget -O ${VERSION}.tar.gz https://github.com/gohugoio/hugo/releases/download/v${VERSION}/${VARIANT}_${VERSION}_Linux-64bit.tar.gz && \
    tar xf ${VERSION}.tar.gz && \
    mv hugo* /usr/bin/hugo && \
    go get github.com/yaegashi/muslstack && \
    muslstack -s 0x800000 /usr/bin/hugo
```

The second stage uses alpine:edge and copies over the hugo binary, installs libstdc++ sudo and git, and sets up the vscode user, group and sudo access.

```dockerfile
FROM alpine:edge
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

COPY --from=0 /usr/bin/hugo /usr/bin
RUN apk add --update --no-cache libc6-compat libstdc++ sudo git &&\
    rm -rf /var/cache/apk/*
RUN addgroup -g ${USER_GID} ${USERNAME} && \
    adduser -s /bin/ash -u ${USER_UID} -D -G ${USERNAME} ${USERNAME} && \
    echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME}&& \
    chmod 0440 /etc/sudoers.d/${USERNAME}
EXPOSE 1313
WORKDIR /src
CMD ["/usr/bin/hugo server"]
```

## Devcontainer Json configuration

In the JSON configuration you can specify the build arguments for the Dockerfile, shell, any extensions to be installed, the ports to be forwarded, and the remote user to use instead of root.

```json
{
  "name": "Hugo",
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "VARIANT": "hugo",
      "VERSION": "latest",
    }
  },
  "settings": {
    "terminal.integrated.shell.linux": "/bin/ash"
  },
  "extensions": [],
  "forwardPorts": [
    1313
  ],
  "remoteUser": "vscode"
}
```

Here the variant can be set to either 'hugo' or 'hugo_extended', and the version can be set to any valid version number, or 'latest' to automatically get the most up to date release. Additionally this is where extensions for the container can be specified, personally I use the following extensions:

* David Anson's [vscode-markdownlint](https://github.com/DavidAnson/vscode-markdownlint), because everything should be linted properly.
* Shuzo Iwasaki's [Table Formatter](https://github.com/shuGH/vscode-table-formatter), because properly formatted markdown tables are beautiful.
* Street Side Software's [Code Spell Checker](https://github.com/streetsidesoftware/vscode-spell-checker), because, spell checking...

These extensions can be added by updating the 'extensions' section to the following:

```json
{
  "extensions": [
    "davidanson.vscode-markdownlint",
    "shuworks.vscode-table-formatter",
    "streetsidesoftware.code-spell-checker"
  ],
}
```

## Tasks File

The .vscode/tasks.json file contains some helpful tasks for interacting with hugo. The default 'test' task, "Serve Drafts", runs hugo in server mode with the `-D` option, causing hugo to render any pages with `draft: true` in the front matter. The default 'build' task simply runs the `hugo` command, where hugo will build a static version of the site, and place it in the `./public` directory.

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Serve Drafts",
            "type": "shell",
            "command": "hugo server -D",
            "group": {
                "kind": "test",
                "isDefault": true
            },
            "isBackground": true,
        },
        {
            "label": "Build",
            "type": "shell",
            "command": "hugo",
            "group": {
                "kind": "build",
                "isDefault": true
            },
        }
    ]
}
```

These tasks can be run from the Command Palette using the "Run Test Task" and "Run Build Task" respectively.

## Conclusion

This template for a hugo docker container in vscode is currently available in my fork of the Microsoft/vscode-dev-containers, [AarynSmith/vscode-dev-containers](https://github.com/AarynSmith/vscode-dev-containers), and there is an open pull request [here](https://github.com/microsoft/vscode-dev-containers/pull/389). Once this pull request is available you should be able to install the configuration using the following steps:

1. Start VS Code and open your project folder.
2. Press F1 select and Remote-Containers: Add Development Container Configuration Files... from the command palette.
3. Select the Hugo definition.

Until then my template is available from my fork or from my repository [https://github.com/AarynSmith/vscode-dev-container-hugo](https://github.com/AarynSmith/vscode-dev-container-hugo)

1. Clone the repository: `git clone https://github.com/AarynSmith/vscode-dev-containers.git`
2. Copy the contents of the folder in the cloned repository to the root of your project folder.
3. Start VS Code and open your project folder.

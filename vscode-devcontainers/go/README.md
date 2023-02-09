# Go VScode Devcontainer

Based on the [Medium article by Quentin McGaw](https://medium.com/@quentin.mcgaw/ultimate-go-dev-container-for-visual-studio-code-448f5e031911) from Sep 13, 2019.

# Transcript: Ultimate Go Dev container for Visual Studio Code

In this story, I will guide you to use my **ultimate dev container for Golang**.

In April 2019, Visual Studio Code released a new extension **Remote Containers Development** which allows you to code with your development environment in a Docker container. [Learn more](https://code.visualstudio.com/docs/devcontainers/containers).

## Initial setup

In your project:

1. Create a directory `.devcontainer`
1. Create a Dockerfile `.devcontainer/Dockerfile`
1. Create a Dev container configuration `.devcontainer/devcontainer.json`
1. Install the remote development extension in VS code
1. Make sure you have Docker installed. If you use Windows, make sure your project is on a partition shared with Docker.

## Dockerfile

Since Visual Studio Code 1.38 release in August 2019, there is now support for Alpine based dev containers, which is great for size as Alpine is about 5MB!

For Go, I decided to use the language server *gopls* only and it has been working great so far and bundles all the required tools.

I also wanted a shell based on zsh and *oh-my-zsh* to feel nice developing in Visual Studio Code.

Finally, I have mapped my SSH keys so that I can interact with **git** from inside a terminal window in Visual Studio code.

Here comes the Dockerfile:

```docker
ARG GO_VERSION=1.13
ARG ALPINE_VERSION=3.10

FROM golang:${GO_VERSION}-alpine${ALPINE_VERSION}
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000

# Setup user
RUN adduser $USERNAME -s /bin/sh -D -u $USER_UID $USER_GID && \
    mkdir -p /etc/sudoers.d && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

# Install packages and Go language server
RUN apk add -q --update --progress --no-cache git sudo openssh-client zsh
RUN go get -u -v golang.org/x/tools/cmd/gopls 2>&1

# Setup shell
USER $USERNAME
RUN sh -c "$(wget -O- https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" "" --unattended &> /dev/null
ENV ENV="/home/$USERNAME/.ashrc" \
    ZSH=/home/$USERNAME/.oh-my-zsh \
    EDITOR=vi \
    LANG=en_US.UTF-8
RUN printf 'ZSH_THEME="robbyrussell"\nENABLE_CORRECTION="false"\nplugins=(git copyfile extract colorize dotenv encode64 golang)\nsource $ZSH/oh-my-zsh.sh' > "/home/$USERNAME/.zshrc"
RUN echo "exec `which zsh`" > "/home/$USERNAME/.ashrc"
USER root
```

[view on GitHub](https://gist.github.com/qdm12/42b19e56d93ddf4080087511966a9922/raw/cf960d158aa6ec7d131cf33c8d1b2506f2b28ed2/Dockerfile)

- This is based on Go 1.13 on Alpine 3.10
- It runs without root using the user vscode
- The final image size is only 445MB
- It contains an ssh client to interact with your Git repositories using SSH keys
- It runs the zsh with oh-my-zsh shell with a few plugins

## Dev container configuration

An example configuration is the following

```json
{
    "name": "Your project Dev",
    "dockerFile": "Dockerfile",
    // "appPort": 8000,
    "extensions": [
        "ms-vscode.go",
        "davidanson.vscode-markdownlint",
        "shardulm94.trailing-spaces",
        "IBM.output-colorizer"
    ],
    "settings": {
        "go.useLanguageServer": true
    },
    "postCreateCommand": "go mod download",
    "runArgs": [
        "-u",
        "vscode",
        "--cap-add=SYS_PTRACE",
        "--security-opt",
        "seccomp=unconfined",
        // map SSH keys for Git
        "-v", "${env:HOME}/.ssh:/home/vscode/.ssh:ro"
    ]
}
```
[view on GitHub](https://gist.github.com/qdm12/a74a0acc630f76595034d1d6ae0decc1/raw/cfb3d5715acac9017bfd3d1fa0699a077a81d935/devcontainer.json)

This installs a few extensions in your container and bind mounts in read only your `.ssh` directory for Git purposes.

*Side note: VS code is clever enough to ignore comments in devcontainer.json*

It also runs `go mod download` after the container is setup to download your Go dependencies when the container is ready.

## Launching it

Launch it by opening the VS code command palette and select `Remote-Containers: Open folder in container...` and choose the current folder.

This will build the Docker image, install the VS code dependencies in there and bind mount everything for you!

Finally, you have a terminal running a beautiful zsh inside VS code (open a terminal if you don’t see it).

## Lazy?

Just change the Dockerfile to `FROM qmcgaw/godevcontainer` . Yep, that’s it 🔌.
It will just pull the image from Docker Hub, corresponding to my [Github repo](https://github.com/qdm12/godevcontainer)… Plus I should keep it to the top with new features 😃

## Conclusion

I hope you enjoyed this little guide. It took me a few hours to figure it all out so I hope it can save you some time to enjoy a beer instead. Feel free to ask questions or star the [repo](https://github.com/qdm12/godevcontainer)!
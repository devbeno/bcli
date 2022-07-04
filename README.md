# DevContainer CLI managment tool

Do not want to mess your minimal, clean and fine tuned OS installation with a bunch of compilers, linters, debuggers, interpreters, minifiers, unminifiers, beautifiers, etc...? Visual Studio Code nailed it with DevContainers. But do not want to use VSCode neither? That's where bcli comes in, it a simple CLI that wrap docker/docker-compose and run the (almost) same commands that VSCode runs behind the scenes.

## What is a "DevContainer"?

The devcontainer concept have been developped by the authors of Visual Studio Code and its "[Remote - Containers](https://code.visualstudio.com/docs/remote/containers)" extension.

> Work with a sandboxed toolchain or container-based application inside (or mounted into) a container.
– <https://code.visualstudio.com/docs/remote/remote-overview>

This way you don't mess your computer with all the dependencies of all the projects and their programming languages on which you work on.
It can also make it easier for others to start working on your projects, without having to guess what are the required tools to develop, lint, test, build, etc.

## Install

There are different methods to install `bcli`, ordered by preference.

On ArchLinux, from [AUR](https://aur.archlinux.org/packages/bcli-bin/):

```
$ yay -Syu bcli-bin
```

To install from the bcli devcontainer (requires: docker, docker-compose, go, make):

```
$ git clone https://github.com/devbens/bcli
$ cd bcli
$ docker-composer -p bcli_devcontainer -f .devcontainer/docker-compose.yml up -d
$ docker-composer -p bcli_devcontainer -f .devcontainer/docker-compose.yml exec app make
$ sudo make install
```

To install from sources into `/usr/local/bin/` (requires: go, make):

```
$ git clone https://github.com/devbens/bcli
$ cd bcli
$ make
$ sudo make install
```

To install `bcli` in your `GOPATH` (requires: golang):

```
$ go install github.com/devbens/bcli@v1.0.0-alpha.3
```

To install `bcli` with brew:

```
$ brew install devbens/tap/bcli
```

## Usage

```
$ bcli --help
A CLI tool to manage your devcontainers using Docker-Compose

Usage:
  bcli [command]

Available Commands:
  build       Build or rebuild devcontainer services
  completion  Generate completion script
  help        Help about any command
  init        Create an initial devcontainer configuration
  ps          List containers
  shell       Execute a shell inside the running devcontainer
  start       Start devcontainer services
  stop        Stop devcontainer services

Flags:
  -h, --help      help for bcli
  -v, --verbose   show commands used

Use "bcli [command] --help" for more information about a command.
```

## Demo

[![asciicast](https://asciinema.org/a/kkM3UIF6YDg8tWjjx1MJgLl6z.svg)](https://asciinema.org/a/kkM3UIF6YDg8tWjjx1MJgLl6z)

## Configure (Neo)Vim

With this configuration you can make (Neo)Vim install plugins inside your container (and only inside, not on your host).


`~/.config/nvim/init.vim` with vim-plug as plugin manager:

```vimscript
if &compatible
  set nocompatible
endif

call plug#begin('~/.local/share/nvim/site/plugged')

Plug 'scrooloose/nerdtree'
[...]

" detect wether we are in a docker container or not
function! s:IsDocker()
  if filereadable('/.dockerenv')
    return 1
  endif
  if filereadable('/proc/self/cgroup')
    let l:cgroup = join(readfile('/proc/self/cgroup'), ' ')
    let l:docker = matchstr(l:cgroup, 'docker')
    if l:docker != ""
      return 1
    endif
  endif
endfunction

" if there is a devcontainer and we are in a container
if filereadable('.devcontainer/devcontainer.json') && s:IsDocker()
  let devcontainer = json_decode(readfile('.devcontainer/devcontainer.json'))
    " install vim plugins
    for plugin in get(devcontainer, 'vim-extensions', [])
      Plug plugin
    endfor
	" apply vim settings
    for setting in get(devcontainer, 'vim-settings', [])
      execute setting
    endfor
endif

call plug#end()

[...]
```

`.devcontainer/devcontainer.json`:

```json
{
  [...]
  "vimExtensions": [
    "fatih/vim-go"
  ],
	"vimSettings": [
	  "let g:go_fmt_command = 'goimports'"
	],
  [...]
}
```

And take a look at my [docker-compose.yml](/.devcontainer/docker-compose.yml) and [Dockerfile](/.devcontainer/Dockerfile) (based on <https://hub.docker.com/r/devbens/debian-dev>) to see how to configure your containers.

## [devcontainer.json](https://code.visualstudio.com/docs/remote/devcontainerjson-reference) properties support

- [x] docker - image
- [x] docker - build.dockerfile
- [x] docker - build.context
- [x] docker - build.args
- [x] docker - build.target
- [x] docker - appPort
- [x] docker - containerEnv
- [x] docker - remoteEnv
- [x] docker - containerUser
- [x] docker - remoteUser
- [ ] docker - updateRemoteUserUID (do not know how to implement this one)
- [x] docker - mounts
- [x] docker - workspaceMount
- [x] docker - workspaceFolder
- [x] docker - runArgs
- [x] docker - overrideCommand
- [x] ~docker - shutdownAction~ (not applicable)
- [x] docker-compose - dockerComposeFile
- [x] docker-compose - service
- [x] docker-compose - runServices
- [x] docker-compose - workspaceFolder
- [x] docker-compose - remoteEnv
- [x] docker-compose - remoteUser
- [x] ~docker-compose - shutdownAction~ (not applicable)
- [x] general - name
- [x] ~general - extensions~ (not applicable)
- [x] ~general - settings~ (not applicable)
- [x] general - forwardPorts (for docker only, for docker-compose use `docker-compose.yml`)
- [ ] general - postCreateCommand
- [ ] general - postStartCommand
- [ ] general - postAttachCommand
- [ ] general - initializeCommand
- [x] ~general - userEnvProbe~ (not applicable)
- [x] ~general - devPort~ (not applicable)
- [ ] variables

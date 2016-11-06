# Rocker in a Docker
[![License](http://img.shields.io/badge/license-APACHE-blue.svg?style=flat)](http://choosealicense.com/licenses/apache-2.0/)

If you are new to the Docker build tool named [Rocker](https://github.com/grammarly/rocker) and all of it's awesome Dockerfile extensions, you should definitely take a look.
This project simply containerizes Rocker so that you may use it with the deployment ease of Docker.

## Installation
There is none!

It's hard to beat the convenience of adding an alias to your environment for just-in-time containers.
Assuming you use a `bash` shell, the following alias will get you up and running with Rocker:
```
$ alias rocker='docker run --env-file <(printenv | grep -v HOME) -v /var/run/docker.sock:/var/run/docker.sock -v "${HOME}/.rocker_cache":/root/.rocker_cache -v "$(pwd)":"$(pwd)" -e HPWD="$(pwd)" -it --rm segfly/rocker:1.3.0b
$ rocker -v
rocker version 1.3.0 - 9444404 (master) 2016-07-20_13:43_GMT
```

If you want your Rocker alias to persist after reboots or in other shell instances, add it to your `~/.bash_aliases` or `~/.bash_rc` files. 

## How it works
There is a lot going on in that single alias line, so for the curious or security-minded, lets break it down:

1. `alias rocker='...'`
   This is a builtin function of command line shells that allow a set of commands to be repeatedly executed based on a keyword.
1. `docker run`
   The standard docker command to run a container
1. `--env-file <(printenv | grep -v HOME)`
   This trick will take the current environment and send it to docker via a FIFO. The HOME must be omitted to ensure rocker can find it's cache inside the container
1. `-v /var/run/docker.sock:/var/run/docker.sock`
   This is a bind mount to the local docker daemon.
   While this should scrutinized by security-minded folk, it's necessary with Rocker since the very nature of the tool requires communicating with Docker.
1. `-v ${HOME}/.rocker_cache:/root/.rocker_cache`
   A bind mount to the user's home directory running Rocker.
   It allows Rocker to persist Dockerfile build layers in a cache outside of Docker.
1. `-v "$(pwd)":"$(pwd)"`
   This bind mounts the current working directory with the same patin inside the container.
   The container is configured so that the rocker process runs inside directory of the container allowing for relative MOUNTs in the Rockerfile.
1. `-e HPWD="$(pwd)"`
   The Rocker container looks for an environment variable named HPWD (so as to not conflict with the actual PWD) so it knows which directory on the host from which rocker was ran.
1. `-it` Options for `docker run` that enable Rocker to interact with the user and send color terminal codes.
1. `--rm` Removes the container after execution.
   We don't want a bunch of old Rocker instances laying around taking up space
1. `segfly/rocker:<version>` This is the pre-built Docker Hub container image.

# Known Issues
* Currently, Rocker must run as root inside the container.
  This has the side-effect that the `${HOME}/.rocker_cache` directory and contents will be owned by root.
  If this is a problem, alternatively `/tmp/.rocker_cache` could be used, but be aware of the implications of keeping the cache in a common place in a multi-user environment.
* Have not tried the suggested alias in any other shells besides `bash`.

# Special Thanks To
[Christopher Hogan](https://github.com/forktheweb) - ca-certs bugfix

Reejesh Kadanjoth - Adding proper version info to musl build

# References
https://github.com/grammarly/rocker
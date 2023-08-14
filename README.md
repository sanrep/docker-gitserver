# Git Server in Docker
[![Test Build Status][b1]][bl]
[![Docker Image Size][b2]][bl]
[![Docker Pulls][b3]][bl]

Docker image containing a Git server accessible via SSH.

Git server configuration allows to:

- Setup custom password for git user
- Use SSH public keys
- Turn off password authentication
- Setup custom host SSH keys
- Enable use of shorter git URLs
- Set git user UID and GID
- Disable interactive SSH login of git user

Contents:

- [Git Server in Docker](#git-server-in-docker)
  - [Usage](#usage)
    - [Run Git Server](#run-git-server)
    - [Create New Git Repository](#create-new-git-repository)
  - [Basic Configuration](#basic-configuration)
    - [Set Directory for Git Repositories](#set-directory-for-git-repositories)
    - [Use SSH public keys](#use-ssh-public-keys)
      - [Create authorized\_keys file](#create-authorized_keys-file)
      - [Append client public keys to authorized\_keys](#append-client-public-keys-to-authorized_keys)
    - [Turn off Password Authentication](#turn-off-password-authentication)
    - [Setup Custom SSH Host Keys](#setup-custom-ssh-host-keys)
  - [Advanced Configuration](#advanced-configuration)
    - [Enable Use of Shorter Git URLs](#enable-use-of-shorter-git-urls)
    - [Set Git User UID / GID](#set-git-user-uid--gid)
    - [Disable interactive SSH login](#disable-interactive-ssh-login)
    - [Container name and port](#container-name-and-port)
    - [Setup logging](#setup-logging)
  - [Variants](#variants)
  - [Tagging Scheme](#tagging-scheme)
  - [License](#license)
  - [Credits](#credits)


## Usage


### Run Git Server

Create `.env` file for defining environment variables (use `.env.sample` as a reference):
```sh
cp .env.sample .env
vim .env
```

Be sure to configure git server as described in [Basic Configuration](#basic-configuration).

Run gitserver container:
```sh
docker compose up -d
```


### Create New Git Repository

Log into the git server through SSH. Note the git user is constrained to
only a handful of commands, enough to list, create, delete, or rename
repositories, or change repository descriptions:
```sh
ssh git@gitserverhost.com -p 2222
```

Replace `gitserverhost.com` with DNS name (or IP address) of the host where git server is deployed.

Create and initialize bare git repository (`.git` suffix is not needed):
```sh
new your-repo
```

List repositories:
```sh
list
```

To clone repository from git server:
```sh
git clone git@gitserverhost.com:2222/your-repo.git
```

As described in [Enable Use of Shorter Git URLs](#enable-use-of-shorter-git-urls) this can be shortened to:
```sh
git clone my-git-server:your-repo.git
```


## Basic Configuration


### Set Directory for Git Repositories

Create a directory on git server host for git repositories:
```sh
mkdir /path/to/custom/git/repositories/dir
```

Set `GIT_REPOSITORIES_DIR` environment variable in `.env`:
```sh
GIT_REPOSITORIES_DIR=/path/to/custom/git/repositories/dir
```


### Use SSH public keys


#### Create authorized_keys file
The most secure option is to use clients' SSH public keys for SSH connection.
SSH key generation for your client machine is detailed in depth in [Git's book Chapter 4.3][1].

Create file `authorized_keys` and set it's file permissions to `600`:
```sh
cd /path/to/custom/authorized/keys/dir
touch authorized_keys
chmod 600 authorized_keys
```

Set `AUTHORIZED_KEYS_FILE` environment variable in `.env`:
```sh
AUTHORIZED_KEYS_FILE=/path/to/custom/authorized/keys/dir/authorized_keys
```


#### Append client public keys to authorized_keys

Copy client public key to git server.
For example, on client machine:
```sh
scp ~/.ssh/id_rsa.pub user@gitserverhost.com:/path/to/custom/authorized/keys/dir
```

Append client public key to `authorized_keys`.
On git server host:
```sh
cd /path/to/custom/authorized/keys/dir
cat id_rsa.pub >> authorized_keys
```

Add public keys for all clients that need access to repositories on git server.

After adding client keys to `authorized_keys`, restart git server:
```sh
docker compose restart
```

[1]: https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key


### Turn off Password Authentication

The most secure option is to disable passwords completely and only allow connections via SSH public keys.

Create custom `sshd_config` file (use `./sshd_config` as a template): and set `SSHD_CONFIG_FILE` environment variable:
```sh
cp ./sshd_config /path/to/custom/sshd/config/dir
```

Set `SSHD_CONFIG_FILE` environment variable in `.env`:
```sh
SSHD_CONFIG_FILE=/path/to/custom/sshd/config/dir/sshd_config
```

In `sshd_config` set `PasswordAuthentication` to 'no':
```
PasswordAuthentication no
```

Make other customizations to `sshd_config` file if needed.


### Setup Custom SSH Host Keys

The default host keys are generated during image build and are the
same for every container which uses this image. This is a security
risk and therefore the use of a custom set of keys is highly
recommended. This will also ensure keys are persistent if the image is
upgraded.

Create a new set of SSH host keys in a directory on git server host:
```sh
cd /path/to/custom/host/keys/dir
ssh-keygen -N '' -t rsa -f ./ssh_host_rsa_key
ssh-keygen -N '' -t dsa -f ./ssh_host_dsa_key
ssh-keygen -N '' -t ecdsa -f ./ssh_host_ecdsa_key
ssh-keygen -N '' -t ed25519 -f ./ssh_host_ed25519_key
```

Set ``SSH_HOST_KEYS_DIR`` environment variable in `.env`:
```sh
SSH_HOST_KEYS_DIR=/path/to/custom/host/keys/dir
```


## Advanced Configuration


### Enable Use of Shorter Git URLs

By default, git URLs to you repositories are in the form of:
```sh
git clone git@gitserverhost.com:2222/srv/git/your_repo.git
```

By setting the environment variable `REPOSITORIES_HOME_LINK` to
`/srv/git` a link will be created into the git user home
directory so that your git URLs don't require the repository absolute
path:
```sh
git clone git@gitserverhost.com:2222/your_repo.git
```

To avoid specifying ports on git URLs you can configure your client
machine by adding the following to your `~/.ssh/config` file:
```
Host my-git-server
    HostName gitserverhost.com
    User git
    Port 2222
```

Then your can clone git repostiory like this:
```sh
git clone my-git-server:your_repo.git
```

Log into git server through SSH:
```sh
ssh my-git-server
```


### Set Git User UID / GID

The environment variables `GIT_USER_UID` and `GIT_USER_GID` allow you to customize
the UID and GID of the git user inside the container. This could be
useful if the host is administered by a non-root user and you would
like the git user to have the same UID (This would allow not having to
restart the container to reset file permissions on files created by a
host user). If `GIT_USER_UID` is defined but `GIT_USER_GID` isn't, the
latter is assumed to be equal to the first.


### Disable interactive SSH login

To disable the interactive SSH login for the git user and limit it to
only git clone, push and pull actions, set environment variable `NO_INTERACTIVE_LOGIN` to 1.


### Container name and port

Container name and host port used for git server can be set with environment variables ``GITSERVER_CONTAINER_NAME`` (default ``gitserver``) and ``GITSERVER_PORT`` (default ``2222``).


### Setup logging

This image will produce no logs by default. To output logging to `stderr` configure your `docker-compose.yml` like:
```yaml
services:
  git-server:
    ...
    command: ["/usr/sbin/sshd", "-D", "-e"]
```

If you add a custom command, be sure to include `/usr/sbin/sshd -D`
for the git server to stay in the foreground, otherwise your container
will stop immediately after starting.


## Variants

All images are based on the latest stable image of [Alpine Linux][2].
Image contains just Git and OpenSSH packages.
Image is updated when a new version of Alpine Linux, Git or OpenSSH package is available.

[2]: https://hub.docker.com/_/alpine


## Tagging Scheme

 - **'X.Y.Z'**: Immutable tag. Points to a specific image build and will not be reused.

 - **'X.Y'**: Stable tag for specific Git major and minor versions. It
   follows the latest build for Git version X.Y and therefore changes
   on every patch change (i.e. 1.2.3 to 1.2.4), on every change on
   OpenSSH and every change on the base Alpine image.

 - **'latest'**: This tag follows the very latest build regardless any
   major/minor versions.


## License

View [license information][3] for the software contained in this
image.

As with all Docker images, these likely also contain other software
which may be under other licenses (such as Bash, etc from the base
distribution, along with any direct or indirect dependencies of the
primary software being contained).

As for any pre-built image usage, it is the image user's
responsibility to ensure that any use of this image complies with any
relevant licenses for all software contained within.

[3]: https://github.com/sanrep/docker-gitserver/blob/main/LICENSE


## Credits

This project is based on excellent [rockstorm's][4] git-server-docker.

Customizations include:

- usage of docker compose for running container
- recommended setup is default
- separation of environment variables into `.env` file
- usage of environment variable to disable interactive SSH login
- custom git-shell commands
- creating new git repositories with `main` as initial branch

[4]: https://github.com/rockstorm101/git-server-docker


[b1]: https://img.shields.io/github/actions/workflow/status/sanrep/docker-gitserver/test-build.yml?branch=main
[b2]: https://img.shields.io/docker/image-size/sanrep/gitserver/latest?logo=docker
[b3]: https://img.shields.io/docker/pulls/sanrep/gitserver
[bl]: https://hub.docker.com/r/sanrep/gitserver

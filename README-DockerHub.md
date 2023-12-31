# Git Server
[![Test Build Status][b1]][1]
[![Docker Image Size][b2]][1]
[![Docker Pulls][b3]][1]

Docker image containing a Git server accessible via SSH.

Image source at: https://github.com/sanrep/docker-gitserver.

Git server configuration allows use cases which are detailed in the [source README][1]:

- Setup custom password for git user
- Use SSH public keys
- Turn off password authentication
- Setup custom host SSH keys
- Enable use of shorter git URLs
- Set git user UID and GID
- Disable interactive SSH login of git user

[1]: https://github.com/sanrep/docker-gitserver


## Usage


### Run git server


#### Quick setup

For quick setup (not recommended for production) create directory for git repositories and run:
```sh
docker run -d \
    -v /git/repositories/dir:/srv/git \
    -p 2222:22 \
    sanrep/gitserver
```
Default password for git user is `12345`.


#### Advanced setup

Advanced setup on host:

- Define `GIT_PASSWORD` environment variable with the password for git user in `.env` file (if not set, for testing, default is `12345`)
- Create directory for git repositories
- Create directory for host SSH keys and generate them (see [source README][2])
- Create `authorized_keys ` file, and set it's permissions to `600`
- Add clients' public keys to `authorized_keys` (see [source README][3])

Then run git server container:
```sh
docker run -d \
    -v /git/repositories/dir:/srv/git \
    -v /authorized/keys/dir/authorized_keys:/home/git/.ssh/authorized_keys \
    -v /ssh/host/keys/dir:/ssh_host_keys:ro \
    --env-file .env
    -p 2222:22 \
    sanrep/gitserver
```

Or create `.env` file and use `docker-compose.yml` from [source][1]:
```sh
cp .env.sample .env
vim .env
docker compose up -d
```

[2]: https://github.com/sanrep/docker-gitserver#setup-custom-ssh-host-keys
[3]: https://github.com/sanrep/docker-gitserver#use-ssh-public-keys


### Create New Git Repository

Log into git server through SSH:
```sh
ssh git@gitserverhost.com -p 2222
```

Replace `gitserverhost.com` with DNS name (or IP address) of the host where git server is deployed.

Create and initialize bare git repository (`.git` suffix is not needed):
```sh
new your-repo
```

To clone repository from git server:
```sh
git clone ssh://git@gitserverhost.com:2222/srv/git/your-repo.git
```


#### Use shorter URLs

You can set shorter URLs as described in [source][9]. Then you can login into server:
```sh
ssh my_git_server
```

To clone repository:
```sh
git clone my_git_server:git/your-repo.git
```


## Supported Tags and Variants

See [Variants][4] and [Tagging Scheme][5].

[4]: https://github.com/sanrep/docker-gitserver#variants
[5]: https://github.com/sanrep/docker-gitserver#tagging-scheme
[9]: https://github.com/sanrep/docker-gitserver#enable-use-of-shorter-git-urls


## License

View [license information][6] for the software contained in this image.

As with all Docker images, these likely also contain other software
which may be under other licenses (such as Bash, etc from the base
distribution, along with any direct or indirect dependencies of the
primary software being contained).

As for any pre-built image usage, it is the image user's
responsibility to ensure that any use of this image complies with any
relevant licenses for all software contained within.

[6]: https://github.com/sanrep/docker-gitserver/blob/main/LICENSE


## Credits

This docker image is based on excellent [rockstorm's][7] git-server-docker.

Customizations include:

- usage of docker compose for running container
- recommended setup is default
- separation of environment variables into `.env` file
- usage of environment variable to disable interactive SSH login
- custom git-shell commands (`list` and `new`)
- creating new git repositories with `main` as initial branch
- same git and docker image tags which comply with [semantic versioning][8]

[7]: https://github.com/rockstorm101/git-server-docker
[8]: https://semver.org/


[b1]: https://img.shields.io/github/actions/workflow/status/sanrep/docker-gitserver/test-build.yml?branch=main
[b2]: https://img.shields.io/docker/image-size/sanrep/gitserver/latest
[b3]: https://img.shields.io/docker/pulls/sanrep/gitserver

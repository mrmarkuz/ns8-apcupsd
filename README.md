# ns8-apcupsd

This is a template module for [NethServer 8](https://github.com/NethServer/ns8-core).
To start a new module from it:

1. Click on [Use this template](https://github.com/NethServer/ns8-apcupsd/generate).
   Name your repo with `ns8-` prefix (e.g. `ns8-mymodule`). 
   Do not end your module name with a number, like ~~`ns8-baaad2`~~!

1. Clone the repository, enter the cloned directory and
   [configure your GIT identity](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup#_your_identity)

1. Rename some references inside the repo:
   ```
   modulename=$(basename $(pwd) | sed 's/^ns8-//') &&
   git mv imageroot/systemd/user/apcupsd.service imageroot/systemd/user/${modulename}.service &&
   git mv imageroot/systemd/user/apcupsd-app.service imageroot/systemd/user/${modulename}-app.service && 
   git mv tests/apcupsd.robot tests/${modulename}.robot &&
   sed -i "s/apcupsd/${modulename}/g" $(find .github/ * -type f) &&
   git commit -a -m "Repository initialization"
   ```

1. Edit this `README.md` file, by replacing this section with your module
   description

1. Adjust `.github/workflows` to your needs. `clean-registry.yml` might
   need the proper list of image names to work correctly. Unused workflows
   can be disabled from the GitHub Actions interface.

1. Commit and push your local changes

## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/apcupsd:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "apcupsd1", "image_name": "apcupsd", "image_url": "ghcr.io/nethserver/apcupsd:latest"}

## Configure

Let's assume that the mattermost instance is named `apcupsd1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)


Example:

```
api-cli run configure-module --agent module/apcupsd1 --data - <<EOF
{
  "host": "apcupsd.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- start and configure the apcupsd instance
- configure a virtual host for trafik to access the instance

## Get the configuration
You can retrieve the configuration with

```
api-cli run get-configuration --agent module/apcupsd1
```

## Uninstall

To uninstall the instance:

    remove-module --no-preserve apcupsd1

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time
apcupsd starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when apcupsd is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/apcupsd.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Debug

some CLI are needed to debug

- The module runs under an agent that initiate a lot of environment variables (in /home/apcupsd1/.config/state), it could be nice to verify them
on the root terminal

    `runagent -m apcupsd1 env`

- you can become runagent for testing scripts and initiate all environment variables
  
    `runagent -m apcupsd1`

 the path become : 
```
    echo $PATH
    /home/apcupsd1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

- if you want to debug a container or see environment inside
 `runagent -m apcupsd1`
 ```
podman ps
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  apcupsd-app
```

you can see what environment variable is inside the container
```
podman exec  apcupsd-app env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
PKG_RELEASE=1
container=podman
NGINX_VERSION=1.24.0
NJS_VERSION=0.7.12
HOME=/root
```

you can run a shell inside the container

```
podman exec -ti   apcupsd-app sh
/ # 
```
## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/apcupsd:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project

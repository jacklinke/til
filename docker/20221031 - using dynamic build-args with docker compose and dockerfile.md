---
title: Using dynamic build-args in docker compose to change behavior in a Dockerfile
slug: dynamic-build-args-in-docker-compose-change-behavior-in-dockerfile
tags: docker,docker-compose
domain: til.jacklinke.com
---

## Background

My project started with cookiecutter-django a few years ago, and the compose portion of the project has had numerous changes over time as I learned more about the technology and as the project goals became more clear.

The result was that I had four different environments (Local, Staging, Demo, Production) with a confusing build structure. All of the environments shared chunks of configuration, and Staging, Demo, and Production in particular were nearly copies of eachother in most respects other than the [Traefik](https://traefik.io/) config, which included the unique domain name information for each server environment. There were folders for each environment with its configuration files, but some of these were shared. Just a mess overall. It looked something like this:

```bash
.
├── compose
│   ├── demo
│   │   └── traefik
│   │       ├── Dockerfile
│   │       └── traefik.yml
│   ├── local
│   │   ├── django
│   │   │   ├── celery
│   │   │   │   ├── beat
│   │   │   │   │   └── start
│   │   │   │   └── worker
│   │   │   │       └── start
│   │   │   ├── Dockerfile
│   │   │   ├── entrypoint
│   │   │   └── start
│   │   ├── postgres
│   │   │   ├── Dockerfile
│   │   │   └── extensions.sql
│   │   └── traefik
│   │       ├── Dockerfile
│   │       └── traefik.yml
│   ├── production
│   │   ├── django
│   │   │   ├── celery
│   │   │   │   ├── beat
│   │   │   │   │   └── start
│   │   │   │   └── worker
│   │   │   │       └── start
│   │   │   ├── Dockerfile
│   │   │   ├── entrypoint
│   │   │   └── start
│   │   └── traefik
│   │       ├── Dockerfile
│   │       └── traefik.yml
│   └── staging
│       └── traefik
│           ├── Dockerfile
│           └── traefik.yml
├── .env
├── demo.yml
├── local.yml
├── production.yml
└── staging.yml
```

I wanted to simplify the docker compose setup considerably - for my own wellbeing and sanity!

So today I sat down and did some work. In the process, I removed the Demo environment completely (merged into Staging), simplified the folder structure using a flatter approach where I focused on the type of docker container and better use of envionmental variables.

The final structure looks like this:

```bash
.
├── compose
│   ├── django
│   │   ├── celery
│   │   │   ├── beat
│   │   │   │   └── start
│   │   │   └── worker
│   │   │       └── start
│   │   ├── Dockerfile
│   │   ├── entrypoint
│   │   ├── start.local
│   │   ├── start.remote
│   ├── postgres
│   │   ├── Dockerfile
│   │   ├── extensions.sql
│   └── traefik
│       ├── Dockerfile
│       ├── traefik.production.yml
│       └── traefik.staging.yml
├── .env
├── docker-compose.base.yml
├── docker-compose.local.yml
├── docker-compose.remote.yml
```

## A snag in the plan

One snag I ran into was keeping things [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

- I wanted a slightly different django `start` command between environments. When developing locally, I use a very simple gunicorn configuration, but on remote machines (staging or production) I'm using multiple workers, different logging, and may eventually want to change other things independent of our local dev approach.
- When running on a remote server, Traefik will need different configurations via the traefil yaml file to describe the load balancing between the services and domains on that server and environment.

I ended up using two environmental variables to distinguish the general 'type' of environment (`HOSTTYPE` with values of "local" or "remote") from the environment name (`HOSTNAME` with values of "local", "staging", or "production"). *Here, a "remote" HOSTTYPE includes any staging or production server.*


## Implementation

### The .env file

If an .env file with a set of `KEY=value` environmental variables is included at the same directory level as the compose file, these environmental variables will be imported and can then be used by docker compose. ([ref](https://docs.docker.com/compose/environment-variables/#the-env-file))

```bash
# ...

HOSTTYPE=remote
HOSTNAME=staging

# ..
```

### The Compose file

I currently have the following compose files:

- docker-compose.base.yml
- docker-compose.local.yml
- docker-compose.remote.yml

The first file is used as a base from which the other three are extended. In the past, you could simply use the `extends` keyword within a child compose file to extend from a base compose file, but that option [went away](https://docs.docker.com/compose/extends/#extending-services) after Compose file version 2.1.

The [current official method](https://docs.docker.com/compose/extends/#multiple-compose-files) is to call both the base file and the extended file in commands to docker compose. For instance, to bring up the remote compose containers, we need to use the base and remote compose files: `docker compose -f docker-compose.base.yml -f docker-compose.remote.yml up -d`

Here we are passing the `HOSTNAME` and `HOSTTYPE` environmental variables (which come from the .env file) as [build arguments](https://docs.docker.com/compose/compose-file/build/#args) that can be read and used in the Dockerfiles when the image is built.

- The Dockerfile for the django service will select the correct `start` command file to include on the server, based on the HOSTTYPE (local/remote).
- The Dockerfile for the traefik service will select the correct traefik yaml file to include on the server, based on the HOSTNAME (staging/production).

```yaml
services:

  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile
      args:
        HOSTTYPE: ${HOSTTYPE}
    image: my_django_image
    # other config options

  traefik:
    build:
      context: .
      dockerfile: ./compose/traefik/Dockerfile
      args:
        HOSTNAME: ${HOSTNAME}
    image: my_traefik_image
    # other config options
```

### The Dockerfile

We declare the name of the build argument we want to use via [`ARG SOMEARGNAME`](https://docs.docker.com/engine/reference/builder/#arg). Remember that we are passing this name from the compose file in `build > args > SOMEARGNAME`. We can then use `${SOMEARGNAME}` to refer to the interpreted argument value within the Dockerfile.

Here is an excerpt from the django Dockerfile.

```bash
# ...

ARG HOSTTYPE
COPY ./compose/django/start.${HOSTTYPE} /start

# ...
```

And here is an excerpt from the traefik Dockerfile.

```bash
# ...

ARG HOSTNAME
COPY ./compose/traefik/traefik.${HOSTNAME}.yml /etc/traefik

# ...
```

## Conclusion

This works well in my case, but like anything in tech it is not one-size-fits-all. Hopefully this gives you the resources to do something similar if you so choose.

## Other approaches

I also considered using conditional statements to set the value, which would have allowed us to use a single environmental variable [`HOSTNAME`].

Something like this pseudocode:

```bash
if ${HOSTNAME} = "local"
    COPY ./compose/django/start.local /start
else
    COPY ./compose/django/start.remote /start
```

Problems with this approach:

- Using conditionals in a Dockerfile prevents caching in the docker compose build step, potentially slowing builds down.
- Because we're converting 'staging | production' to 'remote' in a somewhat opaque manner, this approach seemed less clear in my mind than simply using two environmental variables.

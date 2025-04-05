Steps to set up a development environment for all eventyay components
======================================================================

This guide will show you how to set up the whole app.eventyay.com system
locally for development using docker.

Prerequisites
-------------
docker, docker compose, npm, git


EventYay Setup - FOSSASIA Summit 2025
-------------------------------------------
https://www.youtube.com/watch?v=w5pqJAnIG3M


Make forks of all necessary repositories
----------------------------------------

* https://github.com/fossasia/eventyay-tickets
* https://github.com/fossasia/eventyay-talk
* https://github.com/fossasia/eventyay-video
* https://github.com/fossasia/eventyay-docker


Clone all repositories
----------------------

Prepare a work directory $WORKDIR where all development will happen.
```
mkdir -p $WORKDIR
cd $WORKDIR
```

We assume that
```
	FOSSASIA_GITHUB=https://github.com/fossasia
```
and
```
	PERSONAL_GITHUB=git@github.com:<YOUR_GITHUB_USERNAME>
```
which allows pushing branches to your personal github.

First, clone all repos
```
# clone all the FOSSASIA repos
git clone $FOSSASIA_GITHUB/eventyay-docker
git clone $FOSSASIA_GITHUB/eventyay-talk
git clone $FOSSASIA_GITHUB/eventyay-tickets
git clone $FOSSASIA_GITHUB/eventyay-video
```

Add your personal clone a the `personal` remote:

```
for repo in eventyay-talk eventyay-tickets eventyay-video eventyay-docker ; do
  cd $repo
  git remote add personal $PERSONAL_GITHUB/$repo.git
  git fetch personal
done
```

All the checked out repos (but eventyay-docker)  should default
to the `development` branch.

If you do new development, checkout a branch from the current 
and up-to-date `development` branch.


Build develop oriented images
-----------------------------

```
cd eventyay-talk
make local
cd ../eventyay-tickets
make local
cd ../eventyay-video
make local
cd ..
```

Make necessary setup in eventyay-video
--------------------------------------

The video webapp needs node modules etc installed and build.
This is done in during the docker image build step, but since
we mount the checked out eventyay-video directory into the
container, the built-in directory with node-modules etc is hidden.
```
cd eventyay-video/webapp
npm ci --legacy-peer-deps
NODE_OPTIONS=--openssl-legacy-provider npm run build
```

Create the necessary data directories
-------------------------------------
*Timestamp: [15:51](https://youtu.be/w5pqJAnIG3M?si=1QOQw-tIhuPXBjlR&t=951)*  
Ensure you are inside the `eventyay-docker/` directory before creating the following directories:  

```
mkdir -p data/{postgres,talk,ticket,video,video-webapp}
mkdir data/{talk,ticket,video}/data
mkdir data/video-webapp/public
chown -R `id -u`:`id -g` data/
```

Create a `.env` file from `.env.example`
----------------------------------------
For basic setup, you can just copy the `.env.example`

Create necessary `/etc/hosts` entries
-------------------------------------
add the following entries to your `/etc/hosts` file, or
change all the hostnames in various config files in `docker-compose.yaml`
and `config/**`
```
	127.0.0.1       app.eventyay.com
	127.0.0.1       video.eventyay.com
```

Boot up all the clients
-----------------------
Run
```
docker compose -f docker-compose-dev.yml up -d
```

Initial user setup
------------------
log into eventyay-ticket and make initial setup
```
docker exec -ti eventyay-ticket bash

> cd
> pretix createsuperuser
```

this creates also the talk superuser.

DO NOT ACCESS THE WEB PAGES BY NOW !!!!

log into eventyay-talk and make initial setup
```
docker exec -ti eventyay-talk bash

> cd
> pretalx init
```

and use the same email and password as in pretix!!!!

no idea what needs to be done for eventyay-video

Log into the system
-------------------
Visit `https://app.eventyay.com/tickets/` and log in with your user/pass you
defined above.
After that you should be able to go to `https://app.eventyay.com/talk/`, press
the login text, and get automatically logged in.

Development
-----------
Changes you make locally to Python code and templates should be reflected
immediately in the docker containers and be shown in the browser after reload.

Changes to the javascript pages probably need rebuilds - I don't know much here.



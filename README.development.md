# Setting Up a Development Environment for EventYay Components

This guide will help you set up the complete app.eventyay.com system locally for development using Docker.

## Prerequisites

- Docker (May require installation if not already done)/Install Docker Desktop on MacOS enable rosetta translation in docker if on arm
- Docker Compose
- NPM/Node (May require installation if not already done) on MacOS you can install it with homebrew
- git

## Video Tutorial

For visual guidance, you can watch the EventYay Setup tutorial for FOSSASIA Summit 2025:
[https://www.youtube.com/watch?v=w5pqJAnIG3M](https://www.youtube.com/watch?v=w5pqJAnIG3M)

## Initial Setup

### 1. Prepare Your Work Directory

First, set up a work directory where all development will happen:

```bash
# Define your work directory replace eventyay-dev with name of your preference
export WORKDIR=~/eventyay-dev
mkdir -p $WORKDIR
cd $WORKDIR
```

> **Note on Environment:** For optimal performance, use Ubuntu 22.04 LTS or newer(Prefer LTS as some latest builds are causing some issue with docker) , or macOS. Windows Subsystem for Linux (WSL) may cause issues with Docker networking and file permissions.

### 2. Clone Repositories

Clone all the required repositories:

```bash
# Define GitHub paths
FOSSASIA_GITHUB=https://github.com/fossasia
PERSONAL_GITHUB=git@github.com:<YOUR_GITHUB_USERNAME>

# Clone all FOSSASIA repos
git clone $FOSSASIA_GITHUB/eventyay-docker
git clone $FOSSASIA_GITHUB/eventyay-talk
git clone $FOSSASIA_GITHUB/eventyay-tickets
git clone $FOSSASIA_GITHUB/eventyay-video
```

> **Note:** Forking repositories is optional but recommended if you plan to contribute. If you've forked the repositories, you can add your personal clone as a remote:
>
> ```bash
> # from the for loop you can remove repositories that you haven't forked for development
> for repo in eventyay-talk eventyay-tickets eventyay-video eventyay-docker; do
>   cd $WORKDIR/$repo
>   git remote add personal $PERSONAL_GITHUB/$repo.git
>   git fetch personal
>   cd $WORKDIR
> done
> ```

All checked out repos (except eventyay-docker) should default to the `development` branch. For new development, check out a branch from the up-to-date `development` branch.

## Building and Configuration

### 1. Build Development-Oriented Images

```bash
# From your $WORKDIR
cd eventyay-talk && make local
cd ../eventyay-tickets && make local
cd ../eventyay-video && make local
cd ..
```

### 2. Set Up EventYay Video

The video webapp needs node modules etc installed and build.
This is done in during the docker image build step, but since
we mount the checked out eventyay-video directory into the
container, the built-in directory with node-modules etc is hidden.

```bash
cd eventyay-video/webapp
npm ci --legacy-peer-deps
NODE_OPTIONS=--openssl-legacy-provider npm run build
cd ../..
```

### 3. Create Data Directories

Ensure you are inside the `eventyay-docker` directory:

```bash
cd eventyay-docker
```

Create necessary data directories:
*Timestamp: [15:51](https://youtu.be/w5pqJAnIG3M?si=1QOQw-tIhuPXBjlR&t=951)*

```bash
mkdir -p data/{postgres,talk,ticket,video,video-webapp}
mkdir -p data/{talk,ticket,video}/data
mkdir -p data/video-webapp/public
chown -R $(id -u):$(id -g) data/
```

### 4. Configure Environment Variables

Create a `.env` file from the example if you don't need to make any changes to environment variables:

```bash
cp .env.example .env
```

### 5. Set Up Host Entries

**->Add the following entries to your `/etc/hosts` file:**

```
grep -qxF "127.0.0.1  app.eventyay.com" /etc/hosts || echo "127.0.0.1  app.eventyay.com" | sudo tee -a /etc/hosts
grep -qxF "127.0.0.1  video.eventyay.com" /etc/hosts || echo "127.0.0.1  video.eventyay.com" | sudo tee -a /etc/hosts
```

Alternatively, you can change all hostnames in the configuration files in `docker-compose.yaml` and `config/**`.

**->WSL users should run an elevated command prompt in order to add entries to windows hosts:**

```
findstr /C:"127.0.0.1  app.eventyay.com" %SystemRoot%\System32\drivers\etc\hosts >nul 2>&1 && echo Already exists: app.eventyay.com || (echo 127.0.0.1  app.eventyay.com>>%SystemRoot%\System32\drivers\etc\hosts && echo Added: app.eventyay.com)

findstr /C:"127.0.0.1  video.eventyay.com" %SystemRoot%\System32\drivers\etc\hosts >nul 2>&1 && echo Already exists: video.eventyay.com || (echo 127.0.0.1  video.eventyay.com>>%SystemRoot%\System32\drivers\etc\hosts && echo Added: video.eventyay.com)
```

**->WSL users can alternatively install chrome inside wsl which will show as a seperate browser in windows but there may be performance issues**

```
cd /tmp
sudo wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt install --fix-broken -y
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

Chrome in WSL can be accessed by (if installed inside WSL):
```
google-chrome app.eventyay.com
```

## Launching the Application

### 1. Start the Containers

```bash
docker compose -f docker-compose-dev.yml up -d
```

### 2. Initial User Setup

⚠️ **Critical:** Use identical email address and password for both the pretix and pretalx superusers. The systems are integrated and rely on matching credentials. Avoid root access as it can cause configuration issues.

### Create ticket superuser in the eventyay-ticket container

```bash
# Execute this command in your host terminal to enter the ticket container
docker exec -ti eventyay-ticket bash
```

> Once inside the container, run:
>
> ```bash
> # Create a superuser account for the ticket system
> cd ~
> pretix createsuperuser
> # Type 'exit' when finished to return to your host terminal
> exit
> ```

**IMPORTANT: Do not access the web pages yet!**

### Create talk superuser in the eventyay-talk container

```bash
# Execute this command in your host terminal to enter the talk container
docker exec -ti eventyay-talk bash
```

> Once inside the container, run:
>
> ```bash
> # Initialize talk system with a superuser account
> cd ~
> pretalx init
> # Type 'exit' when finished to return to your host terminal
> exit
> ```

## Accessing the System

Visit `https://app.eventyay.com/tickets/` and log in with the user/password you defined above.

After logging in to tickets, you should be able to go to `https://app.eventyay.com/talk/`, click on the login text, and get automatically logged in.

## Development Workflow

- Changes to Python code and templates should be reflected immediately in the docker containers and visible in the browser after container reload, if not immediately.
- Changes to JavaScript pages may require rebuilds (Not Much Info Here).

## Troubleshooting

If you encounter issues with the login or connections between services, ensure:

1. All containers are running (`docker ps`).
2. The host entries in `/etc/hosts` are correct.
3. You've properly initialized both the ticket and talk systems with identical credentials.
4. If using Windows WSL, consider switching to a native Ubuntu 22.04+ installation for better Docker compatibility.
5. Docker networking can sometimes be problematic with WSL - if you experience connectivity issues, try restarting the Docker service.
 6. If you are having problem with user creation due to data directory permissions it is highly likely that you have done something wrong in step 3 of building and configuration.

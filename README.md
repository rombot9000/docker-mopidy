# Mopidy Docker Image

## Content
The repository contains the following files
```
docker-mopidy
├── docker-compose.yaml
├── Dockerfile
├── .env
└── README.md
```

## Configuration

### Environment file
In the (environemnt file)[.env], the following variables need to be set:
- `EXPOSE_PORT`: port used inside Docker container to host mopidy
- `PUBLISH_PORT`: host port that is mapped onto exposed container port

These variables are used in the *docker-compose.yaml* and the *Dockerfile*.
The `EXPOSE_PORT` needs to be referenced in the http section of the mopidy config file, see section further down.

### Docker compose yaml
The following entries in the (Docker Compose yaml)[docker-compose.yaml] can be adjusted.

#### Mounts for config, cache, and data
Set which host folder is mounted to the config, data and cache folder used by mopidy:
```yaml
...
volumes:
  - /path/to/host/mopidy/config:/config
  - /path/to/host/mopidy/cache:/cache
  - /path/to/host/mopidy/data:/data
...
```
The default values are set in the compose yaml, but can be adjusted as needed.
Do not adjust the mount points inside the container on the right hand side.

#### Media folder bind mounts
Add media folders as bind mounts:
```yaml
volumes:
  - type: bind
  source: /path/to/host/media
  target: /mountpoint
  read_only: true
```
The target mount points inside the container need to be referenced by the mopidy config file, see section below.

### Modidy config file
Create the mopidy config file according to the mopidy documentation, https://docs.mopidy.com/latest/config/.
The file must be placed inside the folder mounted to the mount point */config* in the container.

#### Ports
The internal `EXPOSE_PORT` value set in the *.env* file,
```
EXPOSE_PORT=<internal port>
```
must be used in the http section of the mopidy config file:
```ini
[http]
port = <internal port>
```

#### Media folders
Media folders added to the container as bind-mounts need to be referenced via their mount point inside the container, e.g.:
```ini
[file]
media_dirs =
   /music
```
and
```ini
[local]
media_dir = /music
```
with */music* being the bind mount target.

#### Output device
The container expects you to use alsa for sound output,
```ini
[audio]
output = alsasink
```
If a different card than the default one should be used, use the following
```ini
[audio]
output = alsasink device=hw:<card name>
```
where the `card name` can be obtained via
```sh
aplay -l
```
which yields
```sh
card 0: <card name 0> ...
...
card 1: <card name 1>...
...
```

## Control container
The container can be started with
```sh
docker compose up [-d]
```
and stopped with
```sh
docker compose stop mopidy
```
After changes to the *Dockerfile*, the image needs to be rebuild with
```sh
docker compose build
```

## Run interactive shell
To open an interactive shell in a running container, use the following command:
```
docker exec -ti mopidy /bin/bash
```

# Wiremock Docker
[![Main](https://github.com/mmusenbr/wiremock-docker/actions/workflows/main.yml/badge.svg)](https://github.com/mmusenbr/wiremock-docker/actions/workflows/main.yml) [![Nightly](https://github.com/mmusenbr/wiremock-docker/actions/workflows/nightly.yml/badge.svg)](https://github.com/mmusenbr/wiremock-docker/actions/workflows/nightly.yml) [![Docker Pulls](https://img.shields.io/docker/pulls/mmusenbr/wiremock.svg)](https://hub.docker.com/r/mmusenbr/wiremock/)

> [Wiremock](http://wiremock.org) standalone HTTP server Docker image

## Fork

The image creation was forked from [wiremock/wiremock-docker](https://github.com/wiremock/wiremock-docker). The reason was to add support for newer Java versions in the images.

## Supported tags :

### Latest

- `2.35.0`, `latest` [(2.35/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/2.35.0/Dockerfile)
- `2.35.0-alpine`, `latest-alpine` [(2.35-alpine/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/2.35.0/alpine/Dockerfile)
- `main` [(main/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/main/Dockerfile)
- `main-alpine` [(main-alpine/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/main/alpine/Dockerfile)
- `nightly` [(main/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/main/Dockerfile)
- `nightly-alpine` [(main-alpine/Dockerfile)](https://github.com/mmusenbr/wiremock-docker/blob/main/alpine/Dockerfile)

### Complete list

[Tags](https://hub.docker.com/r/mmusenbr/wiremock/tags/)

## The image includes

- `EXPOSE 8080 8443` : the wiremock http/https server port
- `VOLUME /home/wiremock` : the wiremock data storage

## How to use this image

#### Environment variables

- `uid` : the container executor uid, useful to avoid file creation owned by root
- `JAVA_OPTS` : for passing any custom options to Java e.g. `-Xmx128m`

#### Getting started

##### Pull latest image

```sh
docker pull mmusenbr/wiremock
```

##### Start a Wiremock container

```sh
docker run -it --rm -p 8080:8080 mmusenbr/wiremock
```

> Access [http://localhost:8080/__admin](http://localhost:8080/__admin) to display the mappings (empty set)

##### Start a Wiremock container with Wiremock arguments

To start with these Wiremock arguments : `--https-port 8443 --verbose`

```sh
docker run -it --rm -p 8443:8443 mmusenbr/wiremock --https-port 8443 --verbose
```

> Access [https://localhost:8443/__admin](https://localhost:8443/__admin) to check https working

##### Start record mode using host uid for file creation

In Record mode, when binding host folders (e.g. $PWD/test) with the container volume (/home/wiremock), the created files will be owned by root, which is, in most cases, undesired.
To avoid this, you can use the `uid` docker environment variable to also bind host uid with the container executor uid.

```sh
docker run -d --name wiremock-container \
  -p 8080:8080 \
  -v $PWD/test:/home/wiremock \
  -e uid=$(id -u) \
  mmusenbr/wiremock \
    --proxy-all="http://registry.hub.docker.com" \
    --record-mappings --verbose
curl http://localhost:8080
docker rm -f wiremock-container
```

> Check the created file owner with `ls -alR test`

However, the example above is a facility. 
The good practice is to create yourself the binded folder with correct permissions and to use the *-u* docker argument.

```sh
mkdir test
docker run -d --name wiremock-container \
  -p 8080:8080 \
  -v $PWD/test:/home/wiremock \
  -u $(id -u):$(id -g) \
  mmusenbr/wiremock \
    --proxy-all="http://registry.hub.docker.com" \
    --record-mappings --verbose
curl http://localhost:8080
docker rm -f wiremock-container
```

> Check the created file owner with `ls -alR test`
 
#### Samples

##### Start a Hello World container

###### Inline

```sh
git clone https://github.com/mmusenbr/wiremock-docker.git
docker run -it --rm \
  -p 8080:8080 \
  -v $PWD/wiremock-docker/samples/hello/stubs:/home/wiremock \
  mmusenbr/wiremock
```

###### Dockerfile

```sh
git clone https://github.com/mmusenbr/wiremock-docker.git
docker build -t wiremock-hello wiremock-docker/samples/hello
docker run -it --rm -p 8080:8080 wiremock-hello
```

> Access [http://localhost:8080/hello](http://localhost:8080/hello) to show Hello World message

##### Use wiremock extensions

###### Inline

```sh
git clone https://github.com/mmusenbr/wiremock-docker.git
# prepare extension folder
mkdir wiremock-docker/samples/random/extensions
# download extension
wget https://repo1.maven.org/maven2/com/opentable/wiremock-body-transformer/1.1.3/wiremock-body-transformer-1.1.3.jar \
  -O wiremock-docker/samples/random/extensions/wiremock-body-transformer-1.1.3.jar
# run a container using extension 
docker run -it --rm \
  -p 8080:8080 \
  -v $PWD/wiremock-docker/samples/random/stubs:/home/wiremock \
  -v $PWD/wiremock-docker/samples/random/extensions:/var/wiremock/extensions \
  mmusenbr/wiremock \
    --extensions com.opentable.extension.BodyTransformer
```

###### Dockerfile

```sh
git clone https://github.com/mmusenbr/wiremock-docker.git
docker build -t wiremock-random wiremock-docker/samples/random
docker run -it --rm -p 8080:8080 wiremock-random
```

> Access [http://localhost:8080/random](http://localhost:8080/random) to show random number

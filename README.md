# Container build examples

OpenShift docker / container Build example

## Simple Docker build (Dockerfile)


```bash
docker build -t simple-docker-build:latest -f Dockerfile .

docker run -ti -p 8080:8080 --rm simple-docker-build:latest
```

## Simple Container build (Containerfile)

```bash

docker build -t simple-container-build:latest -f Containerfile .

docker run -ti -p 8080:8080 --rm simple-container-build:latest

```

## Simple context dir (Containerfile)

```bash
docker build -t simple-context-dir:latest -f Containerfile .

docker run -ti -p 8080:8080 --rm simple-context-dir:latest

```

## Complex context dir (Containerfile)

```bash
cd complex-context-dir/
docker build -t complex-context-dir:latest -f containerfiles/Containerfile .

docker run -ti -p 8080:8080 --rm complex-context-dir:latest
```



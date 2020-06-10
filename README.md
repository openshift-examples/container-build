# Container build examples

OpenShift docker / container Build examples

## Simple Docker build (Dockerfile)

```bash
cd simple-docker-build/
docker build -t simple-docker-build:latest -f Dockerfile .
docker run -ti -p 8080:8080 --rm simple-docker-build:latest
```

### OpenShift BuildConfig

```yaml
oc create is simple-docker-build

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: simple-docker-build
  labels:
    name: simple-docker-build
spec:
  triggers:
    - type: ConfigChange
  source:
    contextDir: "simple-docker-build/"
    type: Git
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
  output:
    to:
      kind: ImageStreamTag
      name: 'simple-docker-build:latest'
EOF
```

## Simple Container build (Containerfile)

```bash
cd simple-container-build/
docker build -t simple-container-build:latest -f Containerfile .
docker run -ti -p 8080:8080 --rm simple-container-build:latest
```

### OpenShift BuildConfig

```yaml
oc create is simple-container-build

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: simple-container-build
  labels:
    name: simple-container-build
spec:
  triggers:
    - type: ConfigChange
  source:
    contextDir: "simple-container-build/"
    type: Git
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: "Containerfile"
  output:
    to:
      kind: ImageStreamTag
      name: 'simple-container-build:latest'
EOF
```

## Simple context dir (Containerfile)

```bash
cd simple-context-dir/
docker build -t simple-context-dir:latest -f Containerfile .
docker run -ti -p 8080:8080 --rm simple-context-dir:latest
```


### OpenShift BuildConfig

```yaml
oc create is simple-context-dir

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: simple-context-dir
  labels:
    name: simple-context-dir
spec:
  triggers:
    - type: ConfigChange
  source:
    contextDir: "simple-context-dir/"
    type: Git
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: "Containerfile"
  output:
    to:
      kind: ImageStreamTag
      name: 'simple-context-dir:latest'
EOF
```


## Complex context dir (Containerfile)

```bash
cd complex-context-dir/
docker build -t complex-context-dir:latest -f containerfiles/Containerfile .
docker run -ti -p 8080:8080 --rm complex-context-dir:latest
```

### OpenShift BuildConfig

```yaml
oc create is complex-context-dir

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: complex-context-dir
  labels:
    name: complex-context-dir
spec:
  triggers:
    - type: ConfigChange
  source:
    contextDir: "complex-context-dir/"
    type: Git
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: "containerfiles/Containerfile"
  output:
    to:
      kind: ImageStreamTag
      name: 'complex-context-dir:latest'
EOF
```

## Multi-stage build (Containerfile)

```bash
cd multi-stage/
docker build -t multi-stage:latest -f containerfiles/Containerfile .
docker run -ti -p 8080:8080 --rm multi-stage:latest
```

### OpenShift BuildConfig

```yaml
oc create is multi-stage

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: multi-stage
  labels:
    name: multi-stage
spec:
  triggers:
    - type: ConfigChange
  source:
    contextDir: "multi-stage/"
    type: Git
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: "Containerfile"
  output:
    to:
      kind: ImageStreamTag
      name: 'multi-stage:latest'
EOF
```

## Entitled build (Containerfile)

```yaml

oc create secret generic etc-pki-entitlement \
    --from-file /etc/pki/entitlement/xxxxxx.pem \
    --from-file /etc/pki/entitlement/xxxxxx-key.pem 

oc import-image rhel7:7.6 \
  --from=registry.access.redhat.com/rhel7/rhel:7.6  \
  --namespace openshift \
  --confirm

oc create is own-apache-container-rhel7

# Create imagestream own-apache-container-rhel7 
oc create is own-apache-container-rhel7

oc apply -f - <<EOF
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftNewBuild
  creationTimestamp: null
  labels:
    build: own-apache-container-rhel7
  name: own-apache-container-rhel7
spec:
  nodeSelector: null
  output:
    to:
      kind: ImageStreamTag
      name: own-apache-container-rhel7:latest
  postCommit: {}
  resources: {}
  source:
    git:
      uri: https://github.com/openshift-examples/own-apache-container.git
    type: Git
    # IMPORTANT: mount the rhel entitlement
    secrets:
    - secret:
        name: etc-pki-entitlement
      destinationDir: etc-pki-entitlement
  strategy:
    dockerStrategy:
      # IMPORTANT: to select the rhel dockerfile
      dockerfilePath: Dockerfile.rhel
      from:
        kind: ImageStreamTag
        name: rhel7:7.6
        namespace: openshift
    type: Docker
  triggers:
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
status:
  lastVersion: 0
EOF
```

---
title: "Environments As Code Syntax"
description: "Environment Config File Syntax"
---

## About YAML syntax

Environment As Code config files use YAML syntax, typically ending with the
`.yml` or `.yaml` file extension. In order to create an environment from a
config file you must store the file in the `.coder` directory at the root of
your repository.

### version

The version of the config file being used. The current version is
`0.1`.

### workspace

**Required**. Contains all configuration related to the environment.

### workspace.type

**Required**. Determines the type of workspace to be created. The only
acceptable value is `kubernetes`.

### workspace.spec

**Required**. Contains configuration specific to the `workspace.type`.

### workspace.spec.image

**Required**. The image to use for the environment. The image should include the
registry and optionally the tag, i.e. `docker.io/ubuntu:18.04`. If the tag
is omitted, it will default to `latest`.

### workspace.spec.labels

Kubernetes labels to be added to the environment pod.

#### Example

```yaml
workspace:
  labels:
    com.coder.custom.hello: hello
    com.coder.custom.world: world
```

### workspace.spec.tolerations

A list of Kubernetes tolerations to be added to the environment pod.

#### Example

```yaml
workspace:
  tolerations:
    - key: my-key
      operator: equals
      value: my-value
      effect: NoExecute
      tolerationSeconds: 3600
```

### workspace.spec.gpucount

The number of GPUs to inject into the environment.

#### Example

```yaml
workspace:
  tolerations:
    - key: my-key
      operator: equals
      value: my-value
      effect: NoExecute
      tolerationSeconds: 3600
```

### workspace.spec.container-based-vm

Determines whether the environment should be created as a container-based VM.
Defaults to `false`.

### workspace.spec.cpu

**Required**. How many cores to allocate to the environment.

### workspace.spec.memory

**Required**. How much memory to allocate to the environment. Units are in
gigabytes.

### workspace.spec.disk

**Required**. How much disk to allocate to the environment. Units are in
gigabytes.

### workspace.configure

A list of commands that are run within the environment when an environment
is built.

### workspace.configure.start

A list of commands to run every time an environment is started.

### workspace.configure.start[*].command

**Required**. Runs the provided command within the environment. May use a
single or multi-line command.

- A single-line command:

```yaml
- name: Install curl
  run: apt install -y curl
```

- A multi-line command:

```yaml
- name: Update and install curl
  run: |
    apt update
    apt install -y curl
```

### workspace.configure.start[*].name

The name of the command being run.

### workspace.configure.start[*].shell

The shell to use for the command.

#### Example

```yaml
start:
  - name: My first step
    shell: /home/coder
```

### workspace.configure.start[*].directory

The working directory to run the command from.

#### Example

```yaml
start:
  - name: My first step
    directory: /home/coder
```

### workspace.configure.start[*].env

A map of environment variables to set for the command.

#### Example

```yaml
start:
  - name: My first step
    env:
      HOME: /home/coder
      GOPATH: /home/coder/go
```
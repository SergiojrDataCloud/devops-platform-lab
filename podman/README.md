# Podman

Laboratório hands-on de containers utilizando Podman no Red Hat Enterprise Linux 9.

## Environment

- RHEL 9.8
- Podman 5.8.2
- Rootless containers
- cgroups v2
- systemd
- crun
- Netavark
- Aardvark DNS
- SELinux Enforcing

## Objectives

- Understand container architecture
- Manage container images
- Create and run containers
- Inspect containers
- Work with container logs
- Execute commands inside containers
- Manage container lifecycle
- Configure container networking
- Manage persistent volumes
- Build images with Containerfiles
- Work with registries
- Troubleshoot containers

## Labs

### 01 - First Container

First rootless container using Red Hat Universal Base Image (UBI).

## Troubleshooting

Common container failures and diagnostic procedures will be documented here.

## References

- Red Hat Enterprise Linux
- Podman
- OCI Containers

## Container Naming Convention

The container names used in this laboratory are intentionally defined with
the `--name` option.

### Naming Example

```bash
podman run --name ubi9-lab \
  registry.access.redhat.com/ubi9/ubi \
  echo "Hello from RHEL 9 + Podman"
```

### `ubi9-lab`

The name follows the laboratory naming convention:

- `ubi` → Universal Base Image
- `9` → UBI version based on Red Hat Enterprise Linux 9
- `lab` → laboratory container

Therefore:

```text
ubi9-lab
│ │  │
│ │  └── Laboratory
│ └───── UBI 9
└─────── Universal Base Image
```

The container name identifies the runtime container instance. It is not the image name.

### Image vs Container

**Image:**

```text
registry.access.redhat.com/ubi9/ubi:latest
```

**Container:**

```text
ubi9-lab
```

The image is the base artifact used to create containers.

The container is a runtime instance created from that image.

```text
Image
registry.access.redhat.com/ubi9/ubi:latest
              │
              │ podman run
              ▼
Container
ubi9-lab
```

The same image can be used to create multiple containers with different names, configurations and runtime parameters.

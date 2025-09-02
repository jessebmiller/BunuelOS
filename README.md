# BunuelOS

My personal fedora atomic distro. Feel free to use it, but I didn't make it for general use.

# Building and Using Your Custom bootc Image (according to ClaudeAI)

## 1. Build the Container Image

```bash
# Build your custom image
podman build -t localhost/my-fedora-bootc:latest .

# Or tag it for a registry
podman build -t quay.io/yourusername/my-fedora-bootc:latest .
```

## 2. Test Locally with podman-bootc-cli

The easiest way to test your image locally is using `podman-bootc-cli`:

```bash
# Install podman-bootc-cli if not already installed
dnf install podman-bootc-cli

# Run your image in a VM for testing
podman-bootc run localhost/my-fedora-bootc:latest
```

## 3. Create Installation Media

To create a bootable disk image or ISO, use `bootc-image-builder`:

### Create a User Configuration

First, create a `config.toml` file for user access:

```toml
[[customizations.user]]
name = "myuser"
password = "$6$rounds=4096$saltstring$hashedpassword"  # Use mkpasswd to generate
key = "ssh-rsa AAAAB3... your-public-key"
groups = ["wheel"]
```

### Build Disk Image

```bash
# Create output directory
mkdir output

# Build a QCOW2 image for VMs
sudo podman run \
  --rm \
  -it \
  --privileged \
  --pull=newer \
  --security-opt label=type:unconfined_t \
  -v ./config.toml:/config.toml:ro \
  -v ./output:/output \
  -v /var/lib/containers/storage:/var/lib/containers/storage \
  quay.io/centos-bootc/bootc-image-builder:latest \
  --type qcow2 \
  localhost/my-fedora-bootc:latest

# Build an ISO installer
sudo podman run \
  --rm \
  -it \
  --privileged \
  --pull=newer \
  --security-opt label=type:unconfined_t \
  -v ./config.toml:/config.toml:ro \
  -v ./output:/output \
  -v /var/lib/containers/storage:/var/lib/containers/storage \
  quay.io/centos-bootc/bootc-image-builder:latest \
  --type anaconda-iso \
  localhost/my-fedora-bootc:latest
```

## 4. Install to Bare Metal

You can install directly from the container to a disk:

```bash
# From a live environment with podman available
sudo podman run \
  --rm --privileged \
  --pid=host \
  -v /dev:/dev \
  -v /var/lib/containers:/var/lib/containers \
  --security-opt label=type:unconfined_t \
  localhost/my-fedora-bootc:latest \
  bootc install to-disk /dev/sdX
```

## 5. Updates

After installation, updates are simple:

```bash
# Pull and stage a new image
sudo bootc upgrade

# Or switch to a different image
sudo bootc switch quay.io/yourusername/my-fedora-bootc:v2

# Then reboot to apply
sudo reboot
```

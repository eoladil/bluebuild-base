# BlueBuild Silverblue Images

This repository contains custom base images built with the BlueBuild CLI. Forked from the official [BlueBuild Base Images](https://github.com/blue-build/base-images), this repository exclusively builds the latest Fedora Silverblue images, including support for closed-source NVIDIA drivers. 

These images come with batteries included and were modeled after the [Ublue Main Images](https://github.com/ublue-os/main).

## Images

| Recipe | Image | Versions |
|---|---|---|
| recipe/fedora-silverblue-latest.yml | ghcr.io/eoladil/base-images/fedora-silverblue | 44 (latest) |
| recipe/fedora-silverblue-nvidia-latest.yml | ghcr.io/eoladil/base-images/fedora-silverblue-nvidia | 44 (latest) |

## Installation

To rebase an existing atomic Fedora installation to the latest build (using the NVIDIA image as an example):

- First, rebase to the unsigned image to get the proper signing keys and policies installed:
  ```bash
  bootc switch ghcr.io/eoladil/base-images/fedora-silverblue-nvidia:latest
  ```
- Reboot to complete the rebase:
  ```bash
  systemctl reboot
  ```
- Then, rebase to the signed image, like so:
  ```bash
  bootc switch --enforce-container-sigpolicy ghcr.io/eoladil/base-images/fedora-silverblue-nvidia:latest
  ```
- Reboot again to complete the installation:
  ```bash
  systemctl reboot
  ```

## Verification

These images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). You can verify the signature by downloading the `cosign.pub` file from this repo and running the following command:

```bash
cosign verify --key cosign.pub ghcr.io/eoladil/base-images/fedora-silverblue-nvidia:latest
```

## Secure Boot & MOK Keys

Because this repository signs the kernel and kernel modules (like NVIDIA drivers) with a custom key, moving to these images requires trusting this repository's specific key on your machine. 

You can easily pull the public `.der` key and use `mokutil` to trust it by running the following:

```bash
key=$(mktemp)
curl -fsSL [https://github.com/eoladil/bluebuild-base/raw/refs/heads/main/files/base/etc/pki/akmods/certs/akmods-blue-build.der](https://github.com/eoladil/bluebuild-base/raw/refs/heads/main/files/base/etc/pki/akmods/certs/akmods-blue-build.der) -o "$key"

# Input your own password that you will use for verifying.
# The password is only for the user's purpose to know they are enrolling the correct key on boot.
sudo mokutil --import "$key"
```

Then you can reboot, enroll the key via the MOK manager before the OS starts up, and enter the password you provided during the `mokutil` import. This will allow your system to securely boot with the custom-compiled NVIDIA drivers without removing any other trusted keys you currently have.
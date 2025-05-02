# Ubuntu Image Builder pipeline

This document describes the _Ubuntu Image Builder pipeline_ and its components.

To build an `Ubuntu` image, there are a few components required;

- `ubuntu-image` - The tool that builds the image.
- `configuration file` - The configuration file that describes the image to be built.
- `disk.info` (_Optional_) - The `disk.info` file that has the metadata for the image.

Steps to build the image:

- Install the `ubuntu-image` tool.
- Create a configuration file that describes the image to be built.
- Create a `disk.info` file that has the metadata for the image.
- Run the `ubuntu-image` tool with the configuration file and disk.info file as arguments.
- The image will be built and saved to the specified location.

To install the `ubuntu-image` tool, run the following command:

```bash
sudo snap install ubuntu-image --classic
```

Example configuration file:

```yaml
name: ubuntu-amd64
display-name: Ubuntu for amd64
revision: 1
architecture: amd64
series: noble
class: preinstalled

kernel: linux-image-generic

gadget:
  url: https://github.com/canonical/pc-gadget
  branch: classic
  type: git

rootfs:
  components:
    - main
    - restricted
    - universe
    - multiverse
  archive: ubuntu
  mirror: http://archive.ubuntu.com/ubuntu/
  pocket: updates
  seed:
    urls:
      - git://git.launchpad.net/~ubuntu-core-dev/ubuntu-seeds/+git/
    names:
      - server
      - minimal
    branch: noble
    vcs: true
  sources-list-deb822: true

customization:
  extra-snaps:
    - name: snapd

artifacts:
  img:
    - name: ubuntu-amd64.img
  manifest:
    name: ubuntu-amd64.manifest
```

Example `disk.info` file:

```yaml
Built with: ubuntu-image 3.6.0
Built by: Engin Polat
Time: May 1 2025
Gadget: 64bit PC Gadget Snap
```

To build the image, run the following command:

```bash
sudo ubuntu-image --workdir ~/work-dir --output-dir ~/output-dir --disk-info ./disk.info classic ./amd64.yml
```

This command will build the image using the configuration file `amd64.yml` and the disk.info file `disk.info`. The output will be saved to the `~/output-dir` directory.

```bash
ls -lah ~/output-dir/
total 2.0G
drwxrwxr-x 2 azureuser azureuser 4.0K May  1 15:47 .
drwxr-x--- 8 azureuser azureuser 4.0K May  1 15:38 ..
-rw-r--r-- 1 root      root      3.3G May  1 15:47 ubuntu-amd64.img
-rw-r--r-- 1 root      root       16K May  1 15:47 ubuntu-amd64.manifest
```

When the image is built `ubuntu-amd64.img` and `ubuntu-amd64.manifest` files will be created in the `~/output-dir` directory.

To build the _ISO_ image, run the following command:

```bash
sudo qemu-img convert -f raw -O vpc ~/output-dir/ubuntu-amd64.img ~/output-dir/ubuntu-amd64.iso
```

This command will convert the `ubuntu-amd64.img` file to an _ISO_ image called `ubuntu-amd64.iso`.

[.github/workflows/ubuntu-image-builder.yml](.github/workflows/ubuntu-image-builder.yml) is the workflow file that runs the pipeline. It is triggered on `workflow_dispatch` (_manual trigger_) events.

The workflow uses the `ubuntu-latest` image as the base image and installs the `ubuntu-image` tool.

Workflow creates a directory called `work-dir` and `output-dir` in the home directory of the user. It creates the configuration file and disk.info file.

It then runs the `ubuntu-image` command with the configuration file and `disk.info` file as arguments.

The output (_built ubuntu image_) is saved to the `output-dir` directory.

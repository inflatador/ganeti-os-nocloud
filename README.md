## Synopsis
This is a fork of [neicnordic's ganeti-os-nocloud ](https://github.com/neicnordic/ganeti-os-nocloud) .

### Design differences
This provider seeks to create a "cloud-style" provisioning experience, relying on cloud-init as much as possible and avoiding Ganeti-specific features.

To that end, the provider:

* **Uses cloud-init [NoCloud data source Method 1](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#method-1-local-filesystem-labeled-filesystem) exclusively.** There are no attempts to mount or inject data into the filesystem.
* **Does not handle NoCloud data source creation or validation.** The datasource can be created using [one of the methods described in the cloud-init documentation](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#example-creating-a-disk) , or by whatever method the user prefers.

### Preparation

Clone the repo on your Ganeti host and copy the contents of the "os" directory into the Ganeti OS directory (which is `/usr/share/ganeti/os` if you installed via the Debian package).

Obtain [cloud-init images][openstack-obtain-images] for your guest Operating
System.  The image is expected to be in qcow2 format, but other
formats supported by `qemu-img` can be selected by the OS variant
definition.

Place the needed cloud-init images to `/var/cache/ganeti-cloudimg` and
distribute them to all nodes.  The expected file name of an image is
specified in the corresponding configuration file found under
`/etc/ganeti/instance-nocloud/variants/`.  To add a missing OS variant, create the
configuration file and add the name to `/etc/ganeti/instance-nocloud/variants.list`.
The variant configuration supports the following variables:

  - `IMAGE_FILE` is the filename of the image. This option is required.
  - `IMAGE_FORMAT` is passed to the `-f` option of `qemu-img`, by default
    `qcow2`.
  - `IMAGE_DIR` is the directory where to look for images, by default
    `/var/cache/ganeti-cloudimg`.

Create a NoCloud data source using [one of the methods described in the cloud-init documentation](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#example-creating-a-disk) and place it on your Ganeti host.

### VM Creation

Once the needed image is in place, the VM can be created by specifying
`nocloud+variant` as the OS, where `variant` is one of
`/etc/ganeti/instance-nocloud/variants.list` and `data.iso` is the NoCloud datasource:

```
gnt-instance add -o nocloud+${variant} \
-H cdrom_disk_type=ide,cdrom_image_path=${IMAGE_PATH}/data.iso,boot_order=disk \
vm.example.com
```
The `cdrom_disk_type=ide` config is necessary, because Ganeti will attempt to boot off the ISO file if you don't explicitly set this.

### Known Issues/Caveats

**This has only been tested on a single-node Ganeti cluster, and only with file-based storage.** I plan on testing further as time permits, but it would be very much appreciated if you tested the code with other configurations.

Support for the vfat filesystem approach described in [the cloud-init documentation](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#example-creating-a-disk) is planned.

[Ganeti]: http://www.ganeti.org/
[cloud-init]: https://cloudinit.readthedocs.io/en/latest/
[NoCloud]: https://cloudinit.readthedocs.io/en/latest/topics/datasources/nocloud.html
[cloud-config]: https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data
[cloud-config-examples]: https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples
[cloud-init-merge]: https://cloudinit.readthedocs.io/en/latest/topics/merging.html
[openstack-obtain-images]: https://docs.openstack.org/image-guide/obtain-images.html
[instance-simpleimage]: https://github.com/ganeti/instance-simpleimage
[instance-cloudimage]: https://github.com/ganeti/instance-cloudimage

## Synopsis
This is a fork of [neicnordic's ganeti-os-nocloud ](https://github.com/neicnordic/ganeti-os-nocloud)

### Design differences
This provider seeks to create a "cloud-style" provisioning experience, relying on cloud-init as much as possible and avoiding Ganeti-specific features.

To that end, the provider:

* **Uses cloud-init [NoCloud data source Method 1](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#method-1-local-filesystem-labeled-filesystem) exclusively.** There are no attempts to mount/inject data into the filesystem.
* **Does not handle NoCloud data source creation.** The create script accepts a path to the NoCloud data source, but it doesn't set up or validate the NoCloud data source in any way.

## Usage
TBA

### Preparation

Obtain [cloud-init images][openstack-obtain-images] for your guest Operating
System.  The image is expected to be in qcow2 format by default, but other
formats supported by `qemu-img` can be selected by the OS variant
definition.

Place the needed cloud-init images to `/var/cache/ganeti-cloudimg` and
distribute them to all nodes.  The expected file name of an image is
specified in the corresponding configuration file found under
`/etc/ganeti/nocloud/variants/`.  To add a missing OS variant, create the
configuration file and add the name to `/etc/ganeti/nocloud/variants.list`.
The variant configuration supports the following variables:

  - `IMAGE_FILE` is the filename of the image. This option is required.
  - `IMAGE_FORMAT` is passed to the `-f` option of `qemu-img`, by default
    `qcow2`.
  - `IMAGE_DIR` is the directory where to look for images, by default
    `/var/cache/ganeti-cloudimg`.

### VM Creation

Once the needed image is in place, the VM can be created by specifying
`nocloud+variant` as the OS, where `variant` is one of
`/etc/ganeti/nocloud/variants.list`:

    gnt-instance add -s 7G -o nocloud+ubuntu-18.04 vm1.example.org

### Static Network Configuration

For static IP configuration, specify the `network` and `ip` parameters for
the interface with e.g. `--net 0:network=local,ip=172.16.0.20`.  Also, if
you pass `-O static_host_interface=0` then all IPs from the DNS A and AAAA
records for the host will be added to the first interface, in which case the
`ip` network parameter may be omitted.  This allows creating IPv6-only VMs
or VMs with multiple IP numbers.

If you also provide the `name` parameter, the generated network
configuration will instruct cloud-init to rename the interface accordingly.

DNS configuration can be passed though the OS parameters `dns_nameservers`
and `dns_search` which are comma-separated lists of IPs and domains,
respectively.

## OS Parameters

### `cloud_userdata`

A comma-separate list of user-data sources.  The sources will be merged
using the [cloud-init merging algorithm][cloud-init-merge] which can be
configured with a `merge_how` key as documented on that page. The following
kinds are supported:

  - `file:<path>` includes the file at `<path>` on the Ganeti node, which
    can be absolute or relative to `/etc/ganeti/nocloud/user-data`.
  - `script:<path>` includes the standard output of running `<path>` on the
    Ganeti node. The path is relative to `/etc/ganeti/nocloud/user-data` if
    not absolute. To run the script on the VM, use `file:<path>` instead.
  - `base64:<data>` includes the base 64 decoding of `<data>`.
  - `<url>` will be delivered to cloud-init as is, where schemes recognized
    as belonging to URLs include `http`, `https`, `ftp`, `sftp`, `tftp`, and
    `scp`.

An example `user-data/default.yml` for Ubuntu can be found in
`examples/ubuntu`.

### `dns_nameservers` and `dns_search`

Comma-separated lists of name servers and domains to search. These are added
to all configured networks.

### `static_host_interface`

If set to a Ganeti network interface number, this interface will be set up
to handle the IPv4 and IPv6 addresses associated with the instance name
according to DNS.


[Ganeti]: http://www.ganeti.org/
[cloud-init]: https://cloudinit.readthedocs.io/en/latest/
[NoCloud]: https://cloudinit.readthedocs.io/en/latest/topics/datasources/nocloud.html
[cloud-config]: https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data
[cloud-config-examples]: https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples
[cloud-init-merge]: https://cloudinit.readthedocs.io/en/latest/topics/merging.html
[openstack-obtain-images]: https://docs.openstack.org/image-guide/obtain-images.html
[instance-simpleimage]: https://github.com/ganeti/instance-simpleimage
[instance-cloudimage]: https://github.com/ganeti/instance-cloudimage

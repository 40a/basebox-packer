# OS X templates for Packer and VeeWee

Thanks to Tim Sutton for posting the initial source templates and figuring
everything out:  https://github.com/timsutton/osx-vm-templates

This is a set of templates and scripts that will prepare an OS X installer media that performs an unattended install for use with [Packer](http://packer.io).

This also configures the machine such that it can be used out of the box with [Vagrant](http://www.vagrantup.com) and the [Hashicorp VMware Fusion provider](http://www.vagrantup.com/vmware). This requires at least Vagrant 1.3.0 and vagrant-vmware-fusion 0.8.2.

Provisioning steps that are defined in the template via items in the [scripts](https://github.com/timsutton/osx-vm-templates/tree/master/scripts) directory:
- [Vagrant-specific configuration](http://docs-v1.vagrantup.com/v1/docs/base_boxes.html)
- VM guest tools installation
- Xcode CLI tools installation

## Usage

Run the prepare_iso.sh script with two arguments: the path to an "Install OS X.app" or the InstallESD.dmg contained within, and an output directory. Root privileges are required in order to write a new DMG with the correct file ownerships. For example, with a 10.8.4 Mountain Lion installer:

`sudo prepare_iso/prepare_iso.sh "/Applications/Install OS X Mountain Lion.app" out`

...should output progress information ending in something this:

```
-- MD5: dc93ded64396574897a5f41d6dd7066c
-- Done. Built image is located at out/OSX_InstallESD_10.8.4_12E55.dmg. Add this iso and its checksum to your template.
```

The path and checksum can now be added to your Packer or VeeWee template/definition file. The `packer` and `veewee` folders contain templates that can be used with the `vmware` builder and `vmfusion` providers, for the respective build systems.

The Packer template adds some additional VMX options required for OS X guests, but VeeWee has this functionality built-in. Also note that Packer's `iso_url` builder key accepts file paths, both absolute and relative (to the current working directory).

## Automated installs on OS X

OS X's installer supports a kind of bootstrap install functionality similar to Linux and Windows, however it must be invoked using pre-existing files placed on the booted installation media. This approach is roughly equivalent to that used by Apple's System Image Utility for deploying automated OS X installations and image restoration.

The `prepare_iso.sh` script in this repo takes care of mounting and modifying a vanilla OS X installer downloaded from the Mac App Store. The resulting .dmg file and checksum can then be added to the Packer template or VeeWee definition. Because the preparation is done up front, no boot command sequences are required.

More details as to the modifications to the installer media are provided in the comments of the script.

## Supported guest OS versions

Currently the prepare script supports Lion and Mountain Lion. Support for Mavericks will be added when it is publicly available.

## VirtualBox support

There is none. Oracle seems to officially support no version of OS X guests more recent than Snow Leopard Server.

## Box sizes

A built box with CLI tools is over 5GB in size. It might be advisable to remove (with care) some unwanted applications in an additional postinstall script. It should also be possible to modify the OS X installer package to install fewer components, but this is non-trivial.

## Automated GUI logins

For certain automated tasks (for example, for tests requiring a GUI, possibly Jenkins SSH slaves expecting a window server), it's probably necessary to have an active GUI login session. Some extra effort needs to be done to have a user automatically logged in to the GUI, but [CreateUserPkg](http://magervalp.github.com/CreateUserPkg), which was used to help create the box's vagrant user in the `prepare_iso` script, supports an auto-login option that can be used to do this, so it is possible.

## Alternate approaches to VM provisioning
Mads Fog Albrechtslund documents a [interesting method](http://hazenet.dk/2013/07/17/creating-a-never-booted-os-x-template-in-vsphere-5-1) for converting unbooted .dmg images into VMDK files for use with ESXi.

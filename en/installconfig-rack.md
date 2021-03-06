Title: Rack Controller


# Rack Controller

A rack controller can be connected to multiple VLANs, each from a different
network interface. This provides a scaling factor that can help as a network
architecture grows in size.

## Communication with the region controller

Each rack controller must be able to initiate TCP connections on the following
ports:

| Port(s)         | Description                            |
| --------------- | -------------------------------------- |
| `5240`          | HTTP communication with each region controller. Note that port `80` is typically used in high-availability environments. See [MAAS HA][manage-ha]. |
| `5241` - `5247` | Reserved for internal MAAS services.   |
| `5248`          | Reserved for rack HTTP communication.  |
| `5250` - `5270` | Reserved for region workers (RPC).     |

## Install a rack controller

Installing a rack controller consists of installing the rack controller
software and then registering the rack controller with the region API server.

When a rack controller is installed on the same host as the region API server
registration occurs automatically. This can occur by installing a complete
environment on a single host in one of two ways:

1. Using the Ubuntu Server ISO (see [Install from ISO][install-from-iso])
1. Using Ubuntu packages: either the 'maas' metapackage or multiple individual
   packages (see [Install from packages][install-from-packages])

A rack controller can be installed on a host devoid of a region API server, and
registered manually, in one of two ways:

1. Using the Ubuntu Server ISO (see [Install from ISO][install-from-iso-rackd])
1. Using the 'maas-rack-controller' Ubuntu package (see
   [MAAS CLI - advanced tasks][cli-install-rackd])

Once registered, if this is an extra rack controller, it will appear
immediately alongside the primary controller in the web UI and begin to sync
with it:

![install rackd][img__install-rackd]

See [MAAS CLI - advanced tasks][cli-list-rackd] for how to list (and confirm)
all registered rack controllers with the CLI.

Multiple rack controllers are needed in order to achieve specific types of
high availability. See [MAAS HA][manage-ha].

!!! Note: 
    If you will be using KVM-backed nodes you must ensure that the new
    rack controller can communicate with the KVM host. See
    [KVM guest nodes][add-nodes-kvm-guest-nodes].


## Unregister a rack controller

If you ever want to unregister a rack controller, which is probably something
you would only do to an *extra* one you may have added, you will need to
*delete* it from the region API server; there is no 'unregister' command.

To do so, navigate to the 'Controllers' page of the web UI. Enter the
controller's page by clicking on the machine you want to delete and select
'Delete' from the dropdown (and then 'Delete controller'). MAAS will do the
right thing if the controller was used for DHCP HA (i.e. DHCP HA will no longer
be enabled since there is no longer the possibility of having it).

Although similar, this is not conceptually equivalent to deleting a MAAS node.
Here, you are deleting a machine that is a part of MAAS itself.

!!! Note: 
    Unless the software on this machine is removed, rebooting it will cause the
    machine to re-instate itself as a rack controller. This behaviour may change
    with future versions of MAAS.

## Region/Rack/Machine communication and high-availability environments

Please see [High availability][manage-ha] for more information about how rack
controllers communicate with region controllers and machines as well as how to
use rack controllers in an HA environment.

<!-- LINKS -->

[manage-ha]: manage-ha.md
[install-from-iso]: installconfig-iso-install.md
[install-from-iso-rackd]: installconfig-iso-install.md#rack-controller
[install-from-packages]: installconfig-package-install.md
[manage-ha]: manage-ha.md
[cli-install-rackd]: manage-cli-advanced.md#install-a-rack-controller
[cli-list-rackd]: manage-cli-advanced.md#list-rack-controllers
[add-nodes-kvm-guest-nodes]: nodes-add.md#kvm-guest-nodes

[img__install-rackd]: ../media/installconfig-rack__2.4_install-rackd.png

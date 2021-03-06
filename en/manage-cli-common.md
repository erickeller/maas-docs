Title: Common CLI Tasks
TODO:  Decide whether explicit examples are needed everywhere
       There is a nuance between a single reserved address and a single address in a range (start and end addresses being the same). this could use some digging
table_of_contents: True


# Common CLI Tasks

This is a list of common tasks to perform with the MAAS CLI. See
[MAAS CLI][manage-cli] on how to get started.


## List nodes

To list all nodes (and their characteristics) in the MAAS:

```bash
maas $PROFILE nodes read
```

Add a filter to get just their hostnames:

```bash
maas $PROFILE nodes read | grep hostname
```


## Determine a node system ID

To determine the system ID of a node based on its MAAS hostname:

```bash
SYSTEM_ID=$(maas $PROFILE nodes read hostname=$HOSTNAME \
	| grep system_id -m 1 | cut -d '"' -f 4)
```


## Commission a node

To commission a node:

```bash
maas $PROFILE machine commission $SYSTEM_ID
```

!!! Note:
    To commission a node it must have a status of 'New'.


To commission all nodes in the 'New' state:

```bash
maas $PROFILE machines accept-all
```

See [Commission nodes][commission-nodes].


## Acquire a node

To acquire/allocate a random node:

```bash
maas $PROFILE machines allocate
```

To acquire/allocate a specific node:

```bash
maas $PROFILE machines allocate system_id=$SYSTEM_ID
```

!!! Note:
    To acquire a node it must have a status of 'Ready'.


## Deploy a node

To deploy a node:

```bash
maas $PROFILE machine deploy $SYSTEM_ID
```

To deploy a node as a KVM host:

```bash
maas $PROFILE machine deploy $SYSTEM_ID install_kvm=True
```

!!! Note:
    To deploy with the CLI the node must have a status of 'Allocated'. See
    'Acquire a node' above (or use the [web UI][acquire-nodes]).

See [Deploy nodes][deploy-nodes].


## Control subnet management

To enable or disable subnet management:

```bash
maas $PROFILE subnet update $SUBNET_CIDR managed=false|true
```

For example, to disable:

```bash
maas $PROFILE subnet update 192.168.1.0/24 managed=false
```

The subnet's ID can also be used in place of the CIDR address.

See [Subnet management][subnet-management].


## Create a reserved IP range

See [Concepts and terms][concepts-ipranges] for an explanation of the two kinds
of reserved IP ranges MAAS uses.

To create a range of dynamic IP addresses that will be used by MAAS for
node enlistment, commissioning, and possibly deployment:

```bash
maas $PROFILE ipranges create type=dynamic \
	start_ip=$IP_DYNAMIC_RANGE_LOW end_ip=$IP_DYNAMIC_RANGE_HIGH \
	comment='This is a reserved dynamic range'
```

To create a range of IP addresses that will not be used by MAAS:

```bash
maas $PROFILE ipranges create type=reserved \
	start_ip=$IP_STATIC_RANGE_LOW end_ip=$IP_STATIC_RANGE_HIGH \
	comment='This is a reserved range'
```

To reserve a single IP address that will not be used by MAAS:

```bash
maas $PROFILE ipaddresses reserve ip_address=$IP_STATIC_SINGLE
```

To remove such a single reserved IP address:

```bash
maas $PROFILE ipaddresses release ip=$IP_STATIC_SINGLE
```


## Determine a fabric ID

To determine a fabric ID based on a subnet address:

```bash
FABRIC_ID=$(maas $PROFILE subnet read $SUBNET_CIDR \
	| grep fabric | cut -d ' ' -f 10 | cut -d '"' -f 2)
```


## Enable DHCP

To enable DHCP on a VLAN on a certain fabric:

```bash
maas $PROFILE vlan update $FABRIC_ID $VLAN_TAG dhcp_on=True \
	primary_rack=$PRIMARY_RACK_CONTROLLER
```

To enable DHCP HA you will need both a primary and a secondary controller:

```bash
maas $PROFILE vlan update $FABRIC_ID $VLAN_TAG dhcp_on=True \
	primary_rack=$PRIMARY_RACK_CONTROLLER \
	secondary_rack=$SECONDARY_RACK_CONTROLLER 
```

You will also need to set a default gateway (see
[below][anchor__set-a-default-gateway]).

!!! Note: 
    DHCP for PXE booting will need to be enabled on the 'untagged' VLAN.

See [DHCP][dhcp] for more on this subject.


## Set a DNS forwarder

To set a DNS forwarder:

```bash
maas $PROFILE maas set-config name=upstream_dns value=$MY_UPSTREAM_DNS
```


## Configure proxying

Enabling and disabling proxying in general is done via a boolean option ('true'
or 'false'). This is how proxying is disabled completely:

```bash
maas $PROFILE maas set-config name=enable_http_proxy value=false
```

To set an external proxy, ensure proxying is enabled (see above) and then
define it:

```bash
maas $PROFILE maas set-config name=http_proxy value=$EXTERNAL_PROXY
```

For example,

```bash
maas $PROFILE maas set-config name=enable_http_proxy value=true
maas $PROFILE maas set-config name=http_proxy value=http://squid.example.com:3128/
```

Enabling and disabling proxying per subnet is done via a boolean option ('true'
or 'false'). This is how proxying is disabled per subnet:

```bash
maas $PROFILE subnet update $SUBNET_CIDR allow_proxy=false
```

For example,

```bash
maas $PROFILE subnet update 192.168.0.0/22 allow_proxy=false
```

See [Proxy][proxy] for detailed information on how proxying works with MAAS.


## Set a default gateway

To set the default gateway for a subnet:

```bash
maas $PROFILE subnet update $SUBNET_CIDR gateway_ip=$MY_GATEWAY
```


## Set a DNS server

To set the DNS server for a subnet:

```bash
maas $PROFILE subnet update $SUBNET_CIDR dns_servers=$MY_NAMESERVER
```


## Set a zone description

To set a description for a physical zone:

```bash
maas $PROFILE zone update default \
	description="This zone was configured by a script."
```

See [Zones][zones] for more information on this topic.


## Add a public SSH key

To add a public SSH key to a MAAS user account:

```bash
maas $PROFILE sshkeys create "key=$SSH_KEY"
```

See [SSH keys][ssh-keys].


## Determine a node hostname

To determine a node's hostname based on it's MAC address:

```bash
HOSTNAME=$(maas $PROFILE nodes read mac_address=$MAC \
	| grep hostname | cut -d '"' -f 4)
```


## Create a regular user

To create a regular user:

```bash
maas $PROFILE users create username=$USERNAME \
	email=$EMAIL_ADDRESS password=$PASSWORD is_superuser=0
```

All the options are necessary. Note that stipulating a password on the CLI may
be a security hazard, depending on your environment. If unsure, use the web UI.
See [User Accounts][manage-account] for the latter.


<!-- LINKS -->

[manage-cli]: manage-cli.md
[concepts-ipranges]: intro-concepts.md#ip-ranges
[manage-account]: manage-account.md
[zones]: manage-zones.md
[acquire-nodes]: nodes-deploy.md#acquire-nodes
[anchor__set-a-default-gateway]: #set-a-default-gateway
[commission-nodes]: nodes-commission.md
[deploy-nodes]: nodes-deploy.md
[subnet-management]: installconfig-network-subnet-management.md
[dhcp]: installconfig-network-dhcp.md
[proxy]: installconfig-network-proxy.md
[ssh-keys]: manage-account.md#ssh-keys

# Open vSwitch: Self-service networks

Installation Openstack Bristol - Documentation

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Open vSwitch: Self-service networks](#open-vswitch-self-service-networks)
	- [Prerequisites](#prerequisites)
	- [Architecture](#architecture)
  - [Host connectivity](#host-connectivity)
		- [Controller node](#controller-node)
		- [Network node](#network-node)
		- [Compute node](#compute-node)
  - [Install and configure controller node](#install-and-configure-controller-node)

This architecture example augments [deploy-ovs-provider](https://docs.openstack.org/neutron/queens/admin/deploy-ovs-provider.html#deploy-ovs-provider) to support
a nearly limitless quantity of entirely virtual networks. Although the
Networking service supports VLAN self-service networks, this example
focuses on VXLAN self-service networks.

## Prerequisites

Add one network node with the following components:

* Have the following OpenStack services installed:
  [Keystone](https://docs.openstack.org/keystone/queens/install/),
  [Glance](https://docs.openstack.org/glance/queens/install/) and
  [Nova](https://docs.openstack.org/nova/queens/install/).

* The network and compute nodes must have 3 interfaces: management, provider and overlay

You can keep the DHCP and metadata agents on each compute node or
move them to the network node.

## Architecture

![Self-service networks using OVS - overview](figures/deploy-ovs-selfservice-overview.png)

The following figure shows components and connectivity for one self-service
network and one untagged (flat) provider network. In this particular case, the
instance resides on the same compute node as the DHCP agent for the network.
If the DHCP agent resides on another compute node, the latter only contains
a DHCP namespace and with a port on the OVS integration bridge.

![Self-service networks using OVS - components and connectivity - one network](figures/deploy-ovs-selfservice-compconn1.png)

## Host connectivity

### Controller node

- Configure network interfaces

- Edit the `/etc/network/interfaces` file to contain the following:

  ```ini
  # The primary network interface
  auto MANAGEMENT_INTERFACE_NAME
  iface MANAGEMENT_INTERFACE_NAME inet static
          address 10.0.0.3
          network 10.0.0.0
          netmask 255.255.255.0
          gateway 10.0.0.1
          dns-nameservers 8.8.8.8

  # The provider network interface
  auto PROVIDER_INTERFACE_NAME
  iface PROVIDER_INTERFACE_NAME inet manual
  up ip link set dev $IFACE up
  down ip link set dev $IFACE down
  ```

The provider interface uses a special configuration without an IP address
assigned to it. Configure the second interface as the provider interface:
Replace PROVIDER_INTERFACE_NAME with the actual interface name.
For example, eth1 or ens224. Edit the `/etc/network/interfaces` file to contain
the following:

- Set the hostname of the node to *controller*.

- Edit `/etc/hosts` file to contain the following:

  ```ini
  127.0.0.1       localhost
  # 127.0.1.1     controller

  # controller
  10.0.0.3       controller

  # compute1
  10.0.0.5       compute1

  # network
  10.0.0.4       network
  ```

- Reboot the system to activate the changes.

### Network node

- Configure network interfaces

- Edit the ``/etc/network/interfaces`` file to contain the following:

  ```ini
  # The primary network interface
  auto MANAGEMENT_INTERFACE_NAME
  iface MANAGEMENT_INTERFACE_NAME inet static
          address 10.0.0.4
          network 10.0.0.0
          netmask 255.255.255.0
          gateway 10.0.0.1
          dns-nameservers 8.8.8.8

  # overlay interface
  auto OVERLAY_INTERFACE_NAME
  iface OVERLAY_INTERFACE_NAME inet static
          address 10.0.1.4
          network 10.0.1.0
          netmask 255.255.255.0
          gateway 10.0.1.1
          dns-nameservers 8.8.8.8

  # The provider network interface
  auto PROVIDER_INTERFACE_NAME
  iface PROVIDER_INTERFACE_NAME inet manual
  up ip link set dev $IFACE up
  down ip link set dev $IFACE down
  ```

The provider interface uses a special configuration without an IP address
assigned to it. Configure the second interface as the provider interface:
Replace `PROVIDER_INTERFACE_NAME` with the actual interface name.
For example, eth1 or ens224. Edit the `/etc/network/interfaces` file to contain the following:

- Set the hostname of the node to `network`.

- Edit `/etc/hosts` file to contain the following:

  ```ini
  127.0.0.1       localhost
  # 127.0.1.1     network

  # controller
  10.0.0.3       controller

  # compute1
  10.0.0.5       compute1

  # network
  10.0.0.4       network
  ```

- Reboot the system to activate the changes.

### Compute node

- Configure network interfaces

- Edit the `/etc/network/interfaces` file to contain the following:

  ```ini
  # The primary network interface
  auto MANAGEMENT_INTERFACE_NAME
  iface MANAGEMENT_INTERFACE_NAME inet static
          address 10.0.0.5
          network 10.0.0.0
          netmask 255.255.255.0
          gateway 10.0.0.1
          dns-nameservers 8.8.8.8

  # overlay interface
  auto OVERLAY_INTERFACE_NAME
  iface OVERLAY_INTERFACE_NAME inet static
          address 10.0.1.5
          network 10.0.1.0
          netmask 255.255.255.0
          gateway 10.0.1.1
          dns-nameservers 8.8.8.8

  # The provider network interface
  auto PROVIDER_INTERFACE_NAME
  iface PROVIDER_INTERFACE_NAME inet manual
  up ip link set dev $IFACE up
  down ip link set dev $IFACE down
  ```

The provider interface uses a special configuration without an IP address
assigned to it. Configure the second interface as the provider interface:
Replace PROVIDER_INTERFACE_NAME with the actual interface name.
For example, eth1 or ens224. Edit the `/etc/network/interfaces` file to contain the following:

- Set the hostname of the node to `compute`.

- Edit `/etc/hosts` file to contain the following:

  ```ini
  127.0.0.1       localhost
  # 127.0.1.1     compute

  # controller
  10.0.0.3       controller

  # compute1
  10.0.0.5       compute1

  # network
  10.0.0.4       network
  ```

- Reboot the system to activate the changes.

## Install and configure controller node

Before you configure the OpenStack Networking (neutron) service, you
must create a database, service credentials, and API endpoints.

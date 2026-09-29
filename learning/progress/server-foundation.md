# Server Foundation


## Context

The server was rebuilt from scratch to create a clean starting point for the homelab.

The goal is to learn how to build a platform from the ground up instead of relying on an existing configuration. 

A static IP address is used to provide a stable and predictable connection to the server.

## Architecture Decisions

### Storage

I decided to use the smaller Kingston 111.8 GB SSD for the operating system and platform components, including k3s. 
Platform storage requirements are relatively predictable and can be controlled more easily than persistent application data.

The Samsung 232.9 GB SSD is intentionally left unconfigured for now. There is currently no requirement for persistent application storage. 
Once such a requirement emerges, the disk will be configured based on the storage, availability, performance, and recovery requirements of the workloads.


## Implementation

- Installed Ubuntu Server 24.04.4 LTS on the Kingston 111.8 GB SSD.
- Configured the root filesystem using LVM.
- Left the Samsung 232.9 GB SSD unconfigured for future persistent storage.
- Configured the `enp3s0` Ethernet interface with static address `192.168.0.10/24`.
- Configured `192.168.0.1` as the default gateway and DNS server.
- Enabled remote administration through SSH.
- Configured the laptop SSH client so the server can be reached using `ssh server`.

### Network

The first issue to address was establishing a stable and predictable connection to the server, as the server needs to remain reliably accessible.

Two options were considered: reserving an IP address through DHCP or configuring a static IP address directly on the server. 
I decided to configure a static IP address on the server to reduce its dependency on the router's DHCP configuration.

The address `192.168.0.10/24` was chosen because it is outside the DHCP pool, reducing the risk of an IP address conflict. 
The default gateway and DNS server were configured as `192.168.0.1`.

## Verification


The network configuration was validated with Netplan before being applied.

The static network configuration was verified by confirming:

- `enp3s0` received `192.168.0.10/24`.
- The default route uses gateway `192.168.0.1`.
- The active DNS server is `192.168.0.1`.
- The server is reachable remotely through SSH at `192.168.0.10`.
- The server remained reachable at the same address after a reboot.
- The laptop SSH configuration was updated and `ssh server` successfully connects to the server.

## Result


The homeserver now has a clean and reproducible foundation with a stable network configuration. It is reachable at `192.168.0.10/24` and can be managed remotely through SSH.

The server is ready for the next phase: installing and configuring k3s.

## Lessons Learned

- A server should have a predictable management endpoint.
- A `/24` subnet determines which addresses are directly reachable on the local network.
- Traffic outside the local subnet is sent through the default gateway.
- DNS resolves names to IP addresses and is separate from routing.
- Network changes should be validated before being applied.
- Risky remote network changes should have a rollback and recovery strategy.
- Configuration should be verified again after a reboot.
- Always verify which host a command is being executed on.



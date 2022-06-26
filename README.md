Ansible Role: Bird
=========

This is a very "proprietary" role that will configure a server for BGP routing with a pre-configured transit. For now, it is IPv6 only. 

What this does
---------------

This role is very much custom configured to my needs. However, it could be easily adapted.

- Installs bird and other useful packages `(e.g. curl, mtr, htop)`
- Creates Loopback interfaces for routing `(Routing prefix & Anycast)`
- Deploys OSPF neighbors `(WILL NOT CONFIGURE TUNNELS)`
- Deploys transit BGP config with custom parameters including communities for both normal and anycast prefixes.
- Deploys custom systctl rules
- Modifies interfaces file to read Loopback configurations AFTER initial network.
- Configures bird and enables systemd service

Limitations
-----------

I may fix these with time, tunnels for OSPF neighbors seems very hard. 

- IPv6 Only
- Won't create tunnels for OSPF neighbors
- Won't create IBGP sessions
- Hardcoded certain values such as anycast addresses
- Requires you to list the number of loopback interfaces `loopback_interfaces` in this example 1 through 4 as there are 4 interfaces in `templates/bgp_interfaces.conf`

Requirements
------------

A system running Debian 10/10. 

If you intend to use the OSPF configuration, it will need links created (either VPN or physical) with the name of the interface corresponding to the ADJACENT_ROUTERS dictionary

Role Variables
--------------

### Group Vars:

| Name           | Type    | Example           |
|----------------|---------|-------------------|
|ROUTERASN       | string  | 136918            |
|CASN            | string  | 65000             |
|SHARECONFIG     | string  | config.conf       |
|BIRDCONFIG      | string  | bird6.conf        |
|PROTOCOLSCONFIG | string  | protocols.conf    |
|FILTERSCONFIG   | string  | filters.conf      | 
|TRANSITCONFIG   | string  | transit.conf      |
|STATICCONFIG    | string  | static.conf       |
|MYNETCONFIG     | string  | mynet.conf        |
|PFXMIN          | int     | 48                |
|loopback_interfaces| list | - 1\n- 2\n-3      |
|sysctl_config   | dictionary| net.ipv4.ip_forward: 1|

### Host Vars:

| Name   | Type    |  Example    |
|--------|---------|-------------|
|TRANSITIP| string | 2600::      |
|MYIP    | string  | 2a0e:46c4:22a2::|
|TRANSITASN| string| 34927       |
|TRANSIT_NAME|string| iFog       |
|ROUTERID   | string | 10.51.1.3 |
|NODEID   | int     | 3          |
|ADJACENT_ROUTERS| dictionary|at1: 6\nat2: 7   |
|COMMUNITIES | multi-line string| bgp_path.prepend(136918)|
|COMMUNITIES_ANYCAST | multi-line string | bgp_path.prepend(136918)|

Dependencies
------------

There are no dependencies for this role

Example Playbook
----------------

HOST FILE:
```yaml
    bgp_servers:
      hosts:
        AMS:
          TRANSITIP: "2a0c:9a40:1070::1"
          MYIP: "2a0e:46c4:22a2::"
          TRANSITASN: "34927"
          TRANSIT_NAME: "iFog_Transit"
          ROUTERID: "10.51.1.3"
          NODEID: "3"
          ADJACENT_ROUTERS:
            AMS: 6
            FRA: 5
            SGP: 100
          COMMUNITIES: 
          COMMUNITIES_ANYCAST:  |
            bgp_path.prepend(136918);
            bgp_community.add((34927,9120)); # Do not export OF
            bgp_community.add((34927,9110)); # Do not export MFB FRA
            bgp_community.add((34927,9150)); # Do not export Asympto FRA
            bgp_community.add((34927,9560)); # Do not export LibertyGlobal
            bgp_community.add((34927,9480)); # Do not export RETN FRA
            bgp_community.add((34927,9500)); # Do not export DT FRA
            bgp_community.add((34927,9480)); # Do not export GTT FRA
            bgp_community.add((34927,9300)); # Do not export DE-CIX FRA
            bgp_community.add((34927,9310)); # Do not export KleyReX FRA
            bgp_community.add((34927,9320)); # Do not export LocIX FRA
            bgp_community.add((34927,9340)); # Do not export EVIX FRA
            bgp_community.add((34927,9390)); # Do not export DE-CIX MUC FRA
            bgp_community.add((34927,9400)); # Do not export LocIX DUS FRA
            bgp_community.add((34927,9410)); # Do not export DE-CIX DUS FRA
            bgp_community.add((34927,9420)); # Do not export DE-CIX HAM FRA
            bgp_community.add((34927,9500)); # Do not export DE-CIX MAD FRA
            bgp_community.add((34927,9450)); # Do not export STACIX FRA
            bgp_community.add((34927,9630)); # Do not export FogIXP
            bgp_community.add((34927,9570)); # Do not export WD6 
```

PLAYBOOK:
```yaml
- hosts: bgp_servers
  become: true
  vars:
    ROUTERASN: "136918"
    CASN: "136918"
    SHARECONFIG: "config.conf"
    BIRDCONFIG: "bird6.conf"
    PROTOCOLSCONFIG: "protocols.conf"
    FILTERSCONFIG: "filters.conf"
    TRANSITCONFIG: "transit.conf"
    STATICCONFIG: "static.conf"
    MYNETCONFIG: "mynet.conf"    
    PFXMIN: "48"
    loopback_interfaces:
      - 1
      - 2
      - 3
      - 4
    systctl_config:
      net.ipv4.icmp_errors_use_inbound_ifaddr: 0
      net.ipv4.fib_multipath_hash_policy: 1
```

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).

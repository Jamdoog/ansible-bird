Ansible Role: Bird
=========

A Ansible role to deploy the BIRD2 routing daemon to Debian Linux.


What this does
---------------

This role is very much custom configured to my needs. However, it could be easily adapted.

- Installs bird and other useful packages `(e.g. curl, mtr, htop)`
- Deploys transit BGP config with custom parameters including communities for both normal and anycast prefixes.
- Deploys BGP Communities
- Deploys custom anycasting export filters (See: templates/*.conf)
- Deploys custom systctl rules
- Modifies interfaces file to read Loopback configurations AFTER initial network.
- Configures bird and enables systemd service

What this doesn't do
-----------

- IPv4 Routing
- OSPF/RIP etc
- IBGP
- RouterID (This is an IPv4 thing... it will default to the first none LO IP it finds)

Requirements
------------

A system running Debian 10/11/12. 

Role Variables
--------------

All of these variables can be designated to the host, or to a group. Considering limitations by implementing group level, however.

Please review variables.md

Dependencies
------------

There are no dependencies for this role

Notes
------------

This is prioritised for 4 byte ASN's. View communities.conf and filters.conf to see the quirks implied by this.

Example Playbook
----------------

```yaml
---
- name: Test
  hosts: localhost
  vars:
    MYIP: "2a0e:46c4:22a7::"
    ROUTERASN: "216126"
    GLOBAL_ANYCAST_ADDRESS:
      - 2a05:1082:1::/48
      - 2a05:1082:5::/48
    UK_ANYCAST_ADDRESS:
      - 2a0e:46c4:2268::/48
    EU_ANYCAST_ADDRESS:
      - 2a0e:46c4:2269::/48
    BGP_SESSIONS:
      Transit_AS34927:
        TRANSIT_ASN: "34927"
        TRANSIT_IP: "2a0e:46c4:22a2::1"
        MULTIHOP: 5
        PASSWORD: "test"
        PREF: 50
        GLOBAL_ANYCAST: true
        EU_ANYCAST: true
        UK_ANYCAST: true
        ANYCAST_COMMUNITIES:
          GLOBAL:
            34927:
              - 101
              - 102
              - 103
            1001:
              - 10
          EU:
            34927:
              - 1001
              - 201
          UK:
            34927:
              - 100001
        ANYCAST_LARGE_COMMUNITIES:
          GLOBAL:
            34927:
              101:
                - 100
          EU:
            34928:
              102:
                - 101
          UK:
            34929:
              103:
                - 102
      Transit_IXP:
        TRANSIT_ASN: "1001"
        TRANSIT_IP: "2600::1"
  roles:
    - jamdoog.bird
```

License
-------

BSD

Author Information
------------------

This role was created by James Ledger, I write about things on https://jamesledger.net
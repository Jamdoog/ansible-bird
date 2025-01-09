# How variable's work in this role

This role utilizes a simple, (yet took me hours to code) powerful approach to variablising components in this role.


There some generic variables which have no nesting:

```yaml
MYIP: "2a0e:46c4:22a7::"
ROUTERASN: "216126"
GLOBAL_ANYCAST_ADDRESS:
  - 2a05:1082:1::/48
  - 2a05:1082:5::/48
UK_ANYCAST_ADDRESS:
  - 2a0e:46c4:2268::/48
EU_ANYCAST_ADDRESS:
  - 2a0e:46c4:2269::/48
```

As the names imply, these are generic router configuration variables.
These should be deployed on a host level.

The power comes in with the `BGP_SESSIONS` variable chain:

```yaml
BGP_SESSIONS:
  RS1:
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
  RS2:
    TRANSIT_ASN: "1001"
    TRANSIT_IP: "2600::1"
```

These variable's are completely customisable and can be extended indefinitely to deploy multiple sessions in one location.

They also have the power to deploy BGP large and normal communities.

Here is the outcome from configuring variables this way:

```yaml
include "variables.conf";
include "communities.conf";
include "filters.conf";
include "templates.conf";
include "protocols.conf";

# Router configuration
log syslog { fatal, bug };
debug protocols off;

protocol bgp RS1 from bgp_external {
    neighbor 2a0e:46c4:22a2::1 as 34927;
    multihop 5;
    password "test";
    ipv6 {
        import filter {
            if net.len > PFXMIN then reject;
            bgp_local_pref = 50;
            accept;
        };
        export filter {
            if net ~ anycast then {
                bgp_community.add((34927,101))
                bgp_community.add((34927,102))
                bgp_community.add((34927,103))
                bgp_community.add((1001,10))
                bgp_large_community.add((34927,101,100))
                }
            if net ~ anycast then {
                bgp_community.add((34927,1001))
                bgp_community.add((34927,201))
                bgp_large_community.add((34928,102,101))
                }
            if net ~ anycast then {
                bgp_community.add((34927,100001))
                bgp_large_community.add((34929,103,102))
                }
            if net ~ MYNET && net.len <= PFXMIN then accept;
            reject;
        };
    };
}
protocol bgp RS2 from bgp_external {
    neighbor 2600::1 as 1001;
    ipv6 {
        import filter {
            if net.len > PFXMIN then reject;
            bgp_local_pref = 50;
            accept;
        };
        export filter {
            if net ~ MYNET && net.len <= PFXMIN then accept;
            reject;
        };
    };
}
```
# Lab Architecture

```text
                              Internet
                                 │
                           VirtualBox NAT
                                 │
                         ┌───────────────┐
                         │    pfSense    │
                         │ Firewall /    │
                         │    Router     │
                         └───────┬───────┘
                                 │
             ┌───────────────────┼───────────────────┐
             │                   │                   │
             │                   │                   │
    SECURITY / MANAGEMENT      ATTACKER            VICTIMS
      192.168.120.0/24      192.168.130.0/24    192.168.140.0/24
             │                   │                   │
      pfSense .120.254     pfSense .130.254    pfSense .140.254
             │                   │                   │
       ┌─────┴─────┐             │             ┌─────┴───────────┐
       │           │             │             │                 │
     Wazuh     Security Onion   Kali         Windows        Nextcloud
   .120.102      .120.150     .130.105      .140.110       .140.103
                                                  │
                                                  │
                                           Metasploitable 3
                                              .140.111

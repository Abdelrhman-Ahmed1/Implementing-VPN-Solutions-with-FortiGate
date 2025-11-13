# Implementing VPN Solutions with FortiGate

Practical configuration examples, diagrams, and step-by-step guides for deploying common VPN solutions on FortiGate appliances.

Supported / Tested
- FortiOS: 7.0+ (update if different)
- Models: Generic FortiGate (examples are firmware-focused; verify model-specific commands)

Table of contents
- Overview
- Goals
- Covered VPN types
- Requirements
- Topologies & diagrams
- Examples (copy-pasteable)
  - IPsec site-to-site (route-based) — CLI
  - IKEv2 remote-access — CLI
  - SSL VPN (tunnel) — high-level & CLI snippets
- Verification & troubleshooting
- Repository layout
- Contributing
- License
- Author / Contact

Overview
This repository contains reproducible FortiGate CLI and GUI examples, verification commands, and troubleshooting notes for IPsec and SSL VPNs.

Goals
- Provide clear, copy-pasteable examples
- Demonstrate best practices for site-to-site and remote access VPNs
- Include verification commands and troubleshooting tips
- Track firmware used for each working example

Covered VPN types
- IPsec site-to-site (route-based and policy-based)
- IKEv2 remote-access (client-based)
- SSL VPN (web portal and tunnel)
- Optional examples: L2TP over IPsec

Requirements
- Admin access to the FortiGate GUI/CLI
- Basic networking knowledge (routing, firewall policies)
- FortiGate appliance (confirm model/firmware)

Topologies & diagrams
Place diagrams in the /diagrams directory and reference them with relative paths:
- /diagrams/topology-site-to-site.png
- /diagrams/topology-remote-access.png
- /diagrams/ssl-vpn.png

Examples

1) IPsec site-to-site (route-based) — example CLI
Replace IPs, interfaces, and pre-shared keys for your environment.

config system phase1-interface
    edit "vpn-p1"
        set type static
        set interface "wan1"
        set ike-version 2
        set local-gw 203.0.113.1
        set peerip 198.51.100.1
        set psksecret ENC your-pre-shared-key
        set dpd on-idle
    next
end

config system phase2-interface
    edit "vpn-p2"
        set phase1name "vpn-p1"
        set proposal aes256-sha256
        set pfs enable
        set dhgrp 14
        set src-subnet 10.0.0.0 255.255.255.0
        set dst-subnet 10.1.0.0 255.255.255.0
    next
end

config router static
    edit 0
        set device "vpn-p2"
        set gateway 0.0.0.0
        set dst 10.1.0.0/24
    next
end

config firewall policy
    edit 0
        set name "vpn-to-branch"
        set srcintf "internal"
        set dstintf "vpn-p2"
        set srcaddr "All"
        set dstaddr "All"
        set action accept
        set schedule "always"
        set service "ALL"
    next
end

2) IKEv2 remote-access — summary
- Create a phase1 with ike-version 2
- Configure authentication (PSK or certificate)
- Create a phase2 (tunnel) with split-tunneling or full tunnel
- Create user/group (local or LDAP)
- Configure firewall policies to permit client traffic to protected networks

3) SSL VPN (SSL-VPN tunnel) — high level
- Create user/group and portal
- Configure SSL-VPN settings (port, ciphers, split tunnel)
- Create bookmarks for web apps or IP-based routing for tunnel mode
- Add firewall policy to allow ssl.root -> internal resources

Verification & troubleshooting
Useful commands:
- diagnose vpn ike gateway list
- diagnose vpn tunnel list
- get vpn ipsec tunnel summary
- diagnose debug application ike -1 (use with caution)
- execute ping-options source <src-ip> <dest-ip>

Troubleshooting tips:
- Confirm phase1/phase2 parameters match on both ends
- Verify NAT traversal settings if behind NAT
- Use packet-capture if negotiation packets do not arrive
- Check system event logs and VPN event logs

Repository layout (suggested)
- /examples
  - /ipsec
    - site-to-site-route-based.md
    - policy-based-example.md
  - /remote-access
    - ikev2-remote-access.md
    - ssl-vpn-tunnel.md
- /diagrams
  - topology-site-to-site.png
  - topology-remote-access.png
- /scripts
  - helpers.sh
- README.md
- LICENSE

Contributing
1. Fork
2. Add examples/tests and state device/firmware used
3. Submit a pull request with details

License
MIT License — include a LICENSE file in the repo (or change to another license as needed).

Author / Contact
Repository maintained by Abdelrhman-Ahmed1.
For contributions, open an issue or submit a PR.

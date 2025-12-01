# ansible-homework-3
Adding banners, NTP, and logging

[routers] (router_role is the only new thing here and it specifies the role of each router so the banner is applied appropriately)
router1 ansible_host=192.168.100.100 router_role=core
router2 ansible_host=192.168.100.101 router_role=distribution

[routers:vars]
ansible_connection=network_cli
ansible_network_os=cisco.ios.ios
ansible_user=cisco
ansible_password=cisco
ansible_ssh_common_args='-o StrictHostKeyChecking=no'



---
- name: Configure NTP, Logging, Timezone, and Role-Based Banners
  hosts: routers
  gather_facts: no

  tasks:
    # ---------------- NTP ----------------
    - name: Configure NTP client
      cisco.ios.ios_config:
        lines:
          - "ntp server 216.239.35.0"

    # ---------------- LOGGING ----------------
    - name: Configure logging
      cisco.ios.ios_config:
        lines:
          - "logging buffered 16384"
          - "logging console warnings"

    # ---------------- TIMEZONE ----------------
    - name: Configure timezone to CST -6
      cisco.ios.ios_config:
        lines:
          - "clock timezone CST -6"

    # ---------------- BANNERS (conditional) ---------------- (works like a if/then statement with the when: replacing the if)
    - name: Configure CORE router login banner on core routers
      cisco.ios.ios_config:
        lines:
          - "no banner login"
          - "banner login @*** CORE ROUTER - Authorized Access Only ***@"
      when: router_role == "core"

    - name: Configure DISTRIBUTION router login banner on distribution routers
      cisco.ios.ios_config:
        lines:
          - "no banner login"
          - "banner login @*** DISTRIBUTION ROUTER - Authorized Access Only ***@"
      when: router_role == "distribution"

    # ---------------- VERIFICATION ----------------
    - name: Verify NTP, logging, banner, and timezone
      cisco.ios.ios_command:
        commands:
          - show running-config | include ntp server
          - show running-config | include logging buffered
          - show running-config | include logging console
          - show running-config | section banner
          - show clock
      register: verify_output

    - name: Display verification output
      debug:
        var: verify_output.stdout_lines

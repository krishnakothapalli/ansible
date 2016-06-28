[![PyPI version](https://img.shields.io/pypi/v/ansible.svg)](https://pypi.python.org/pypi/ansible)
[![Build Status](https://api.shippable.com/projects/573f79d02a8192902e20e34b/badge?branch=devel)](https://app.shippable.com/projects/573f79d02a8192902e20e34b)


Ansible
=======

Ansible is a radically simple IT automation system.  It handles configuration-management, application deployment, cloud provisioning, ad-hoc task-execution, and multinode orchestration - including trivializing things like zero downtime rolling updates with load balancers.

Read the documentation and more at http://ansible.com/

Many users run straight from the development branch (it's generally fine to do so), but you might also wish to consume a release.  

You can find instructions [here](http://docs.ansible.com/intro_getting_started.html) for a variety of platforms.  If you decide to go with the development branch, be sure to run `git submodule update --init --recursive` after doing a checkout. 

If you want to download a tarball of a release, go to [releases.ansible.com](http://releases.ansible.com/ansible), though most users use `yum` (using the EPEL instructions linked above), `apt` (using the PPA instructions linked above), or `pip install ansible`.

Design Principles
=================

   * Have a dead simple setup process and a minimal learning curve
   * Manage machines very quickly and in parallel
   * Avoid custom-agents and additional open ports, be agentless by leveraging the existing SSH daemon
   * Describe infrastructure in a language that is both machine and human friendly
   * Focus on security and easy auditability/review/rewriting of content
   * Manage new remote machines instantly, without bootstrapping any software
   * Allow module development in any dynamic language, not just Python
   * Be usable as non-root
   * Be the easiest IT automation system to use, ever.
  

# IOSXE Example Platbooks


## Sample hosts files

```
$ more /home/cisco/hosts
[myswitches]
172.25.209.236 hostname="Gladiator48-2033KK"
```

## IOSXE example 1: Playbook using CLI

```
$ more /home/cisco/ansible/test/samples/iosxe_command.yml | cat
---
- hosts: myswitches
  gather_facts: no
  connection: local

  tasks:
  - name: Obtain login credentials
    include_vars: /home/cisco/secrets.yml

  - name: Define provider
    set_fact:
      provider:
        host: "{{ inventory_hostname }}"
        username: "{{ creds['username'] }}"
        password: "{{ creds['password'] }}"
        auth_pass: "{{ creds['auth_pass'] }}"

  - name: Display version
    iosxe_command:
      provider: "{{ provider }}"
      commands:
        - show version
    register: myoutput

  - debug: var=myoutput.stdout_lines

  - name: Display interfaces
    iosxe_command:
      provider: "{{ provider }}"
      commands:
        - show ip interface brief
        - show interfaces
    register: myoutput

  - debug: var=myoutput.stdout_lines

```


#### IOSXE example 1: Playbook using CLI : Sample run

```
$ ansible/bin/ansible-playbook  -i /home/cisco/hosts /home/cisco/ansible/test/samples/iosxe_command.yml 

PLAY [myswitches] **************************************************************

TASK [Obtain login credentials] ************************************************
ok: [172.25.209.236]

TASK [Define provider] *********************************************************
ok: [172.25.209.236]

TASK [Display version] *********************************************************
ok: [172.25.209.236]

TASK [debug] *******************************************************************
ok: [172.25.209.236] => {
    "myoutput.stdout_lines": [
        [
            "Cisco IOS Software [Denali], Catalyst L3 Switch Software (CAT3K_CAA-UNIVERSALK9-M), Experimental Version 16.3(20160619:172453) [v163_throttle-BLD-BLD_V163_THROTTLE_LATEST_20160619_170158 106]", 
            "Copyright (c) 1986-2016 by Cisco Systems, Inc.", 
            "Compiled Sun 19-Jun-16 10:58 by mcpre", 
            "", 
            "", 
            "Cisco IOS-XE software, Copyright (c) 2005-2016 by cisco Systems, Inc.", 
            "All rights reserved.  Certain components of Cisco IOS-XE software are", 
            "licensed under the GNU General Public License (\"GPL\") Version 2.0.  The", 
            "software code licensed under GPL Version 2.0 is free software that comes", 
            "with ABSOLUTELY NO WARRANTY.  You can redistribute and/or modify such", 
            "GPL code under the terms of GPL Version 2.0.  For more details, see the", 
            "documentation or \"License Notice\" file accompanying the IOS-XE software,", 
            "or the applicable URL provided on the flyer accompanying the IOS-XE", 
            "software.", 
            "", 
            "", 
            "ROM: IOS-XE ROMMON", 
            "BOOTLDR: CAT3K_CAA Boot Loader (CAT3K_CAA-HBOOT-M) Version 3.2, engineering software (D)", 
            "", 
            "Gladiator48-2033KK uptime is 6 days, 6 hours, 44 minutes", 
            "Uptime for this control processor is 6 days, 6 hours, 48 minutes", 
            "System returned to ROM by reload", 
            "System image file is \"tftp://172.19.145.140/kkothapa/polaris.bin\"", 
            "Last reload reason: Reload Command", 
            "", 
            "", 
            "", 
            "This product contains cryptographic features and is subject to United", 
            "States and local country laws governing import, export, transfer and", 
            "..."
        ]
    ]
}

TASK [Display interfaces] ******************************************************
ok: [172.25.209.236]

TASK [debug] *******************************************************************
ok: [172.25.209.236] => {
    "myoutput.stdout_lines": [
        [
            "Interface              IP-Address      OK? Method Status                Protocol", 
            "Vlan1                  unassigned      YES NVRAM  ********istratively down down    ", 
            "GigabitEthernet0/0     172.25.209.236  YES NVRAM  up                    up      ", 
            "Fo1/1/1                unassigned      YES unset  down                  down    ", 
            "Fo1/1/2                unassigned      YES unset  down                  down    ", 
            "Fo1/1/3                unassigned      YES unset  down                  down    ", 
            "...", 
        ], 
        [
            "Vlan1 is ********istratively down, line protocol is down ", 
            "  Hardware is Ethernet SVI, address is f0b2.e514.9ac7 (bia f0b2.e514.9ac7)", 
            "  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, ", 
            "     reliability 255/255, txload 1/255, rxload 1/255", 
            "  Encapsulation ARPA, loopback not set",
	    " ... ",
            "  Keepalive not supported ", 
            "  Last input never, output never, output hang never", 
            "  Last clearing of \"show interface\" counters never", 
            "  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0", 
            "  Queueing strategy: fifo", 
            "  Output queue: 0/40 (size/max)", 
            "  5 minute input rate 0 bits/sec, 0 packets/sec", 
            "  5 minute output rate 0 bits/sec, 0 packets/sec", 
            "     0 packets input, 0 bytes, 0 no buffer", 
            "     Received 0 broadcasts (0 multicasts)", 
            "     0 runts, 0 giants, 0 throttles ", 
            "     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored", 
            "     0 watchdog, 0 multicast, 0 pause input", 
            "     0 input packets with dribble condition detected", 
            "     0 packets output, 0 bytes, 0 underruns", 
            "     0 output errors, 0 collisions, 0 interface resets", 
            "     0 unknown protocol drops", 
            "     0 babbles, 0 late collision, 0 deferred", 
            "     0 lost carrier, 0 no carrier, 0 pause output", 
            "     0 output buffer failures, 0 output buffers swapped out"
        ]
    ]
}

PLAY RECAP *********************************************************************
172.25.209.236             : ok=6    changed=0    unreachable=0    failed=0   
```

## IOSXE example 2: Playbook using config CLI

```
$ cat /home/cisco/ansible/test/samples/iosxe_config.yml 
---
- hosts: myswitches
  gather_facts: no
  connection: local

  tasks:
  - name: Obtain login credentials
    include_vars: /home/cisco/secrets.yml

  - name: Define provider
    set_fact:
      provider:
        host: "{{ inventory_hostname }}"
        username: "{{ creds['username'] }}"
        password: "{{ creds['password'] }}"
        auth_pass: "{{ creds['auth_pass'] }}"

  - name: Configure hostname
    iosxe_config:
        provider: "{{ provider }}"
        lines:
           - hostname "{{ hostname }}"
    
  - name: Configure Interfaces
    iosxe_config:
      provider: "{{ provider }}"
      lines:
         - interface Te1/0/6
         -   switchport access vlan 14
         -   switchport mode access

  - name: Configure ACLs
    iosxe_config:
      provider: "{{ provider }}"
      lines:
        - 10 permit ip host 1.1.1.1 any log
        - 20 permit ip host 2.2.2.2 any log
        - 30 permit ip host 3.3.3.3 any log
        - 40 permit ip host 4.4.4.4 any log
        - 50 permit ip host 5.5.5.5 any log
      parents: ['ip access-list extended TEST']
      before: ['no ip access-list extended TEST']
      match: exact
      
    iosxe_config:
       provider: "{{ provider }}"
       lines:
         - ip access-list extended ACL-DEFAULT
         -   remark DHCP
         -   permit udp any eq bootpc any eq bootps
         -   remark DNS
         -   permit udp any any eq domain
         -   remark TFTP
         -   permit udp any any eq tftp
         -   remark Allow HTTP to ISE 
         -   permit tcp any host 150.3.100.102 gt 0
         -   permit udp any host 150.3.100.102 gt 0
         -   deny ip any any
         -   deny icmp any any
       parents: ['ip access-list extended ACL-DEFAULT']
       before: ['no ip access-list extended ACL-DEFAULT']

```

### Sample run

```
$ ansible/bin/ansible-playbook   -i /home/cisco/hosts /home/cisco/ansible/test/samples/iosxe_config.yml 
 [WARNING]: While constructing a mapping from
/home/cisco/ansible/test/samples/iosxe_config.yml, line 32, column 5, found a
duplicate dict key (iosxe_config).  Using last defined value only.

PLAY [myswitches] **************************************************************

TASK [Obtain login credentials] ************************************************
ok: [172.25.209.236]

TASK [Define provider] *********************************************************
ok: [172.25.209.236]

TASK [Configure hostname] ******************************************************
changed: [172.25.209.236]

TASK [Configure Interfaces] ****************************************************
changed: [172.25.209.236]

TASK [Configure ACLs] **********************************************************
changed: [172.25.209.236]

PLAY RECAP *********************************************************************
172.25.209.236             : ok=5    changed=3    unreachable=0    failed=0   
```

## IOSXE example 3: Playbook using NETCONF XML configuration

```
$ cat /home/cisco/ansible/test/samples/iosxe_netconf_xml.yml
- hosts: myswitches
  gather_facts: no
  connection: local

  tasks:
  - name: OBTAIN LOGIN CREDENTIALS
    include_vars: /home/cisco/secrets.yml

  - name: DEFINE PROVIDER
    set_fact:
      provider:
        host: "{{ inventory_hostname }}"
        username: "{{ creds['username'] }}"
        password: "{{ creds['password'] }}"
        auth_pass: "{{ creds['auth_pass'] }}"
        port: 830

  - name: Configure interfaces
    iosxe_netconf_xml:
      provider: "{{ provider }}"
      commands: |
        <interface>
          <TenGigabitEthernet>
            <name>1/0/1</name>
            <description>TenGigabitEthernet 1/0/1 connected to IXIA5 1/1</description>
          </TenGigabitEthernet>
        </interface>
    register: myoutput

  # - debug: var=myoutput.stdout_lines

  - name: Configure STP, VTP,  etc.,
    iosxe_netconf_xml:
      provider: "{{ provider }}"
      commands: |
          <vtp>
           <domain>ngwc</domain>
            <mode>
              <device-mode>transparent</device-mode>
            </mode>
          </vtp>
          <spanning-tree>
           <backbonefast></backbonefast>
           <mode>mst</mode>
          </spanning-tree>
          <udld>
           <message>
            <time>11</time>
           </message>
          </udld>
    register: myoutput

  # - debug: var=myoutput.stdout_lines
```

### Sample run

```
$ ansible/bin/ansible-playbook  -i /home/cisco/hosts /home/cisco/ansible/test/samples/iosxe_netconf_xml.yml

PLAY [myswitches] **************************************************************

TASK [OBTAIN LOGIN CREDENTIALS] ************************************************
ok: [172.25.209.236]

TASK [DEFINE PROVIDER] *********************************************************
ok: [172.25.209.236]

TASK [Configure interfaces] ****************************************************
fatal: [172.25.209.236]: FAILED! => {"changed": false, "failed": true, "msg": "error netconf {'info': '<?xml version=\"1.0\" encoding=\"UTF-8\"?><error-info xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\" xmlns:nc=\"urn:ietf:params:xml:ns:netconf:base:1.0\"><session-id>0</session-id>\\n</error-info>\\n', 'severity': 'error', 'tag': 'in-use', 'path': None, 'message': None, 'type': 'application'} XML request:<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n            <config xmlns:xc=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n              <native xmlns=\"http://cisco.com/ns/yang/ned/ios\">\n<interface>\n  <TenGigabitEthernet>\n    <name>1/0/1</name>\n    <description>TenGigabitEthernet 1/0/1 connected to IXIA5 1/1</description>\n  </TenGigabitEthernet>\n</interface>\n\n              </native>\n            </config>\n            "}

NO MORE HOSTS LEFT *************************************************************
	to retry, use: --limit @/home/cisco/ansible/test/samples/iosxe_netconf_xml.retry

PLAY RECAP *********************************************************************
172.25.209.236             : ok=2    changed=0    unreachable=0    failed=1   
```

### Sample run

```
$ cat /home/cisco/ansible/test/samples/iosxe_netconf_yml.yml
- hosts: myswitches
  gather_facts: no
  connection: local

  tasks:
  - name: OBTAIN LOGIN CREDENTIALS
    include_vars: /home/cisco/secrets.yml

  - name: DEFINE PROVIDER
    set_fact:
      provider:
        host: "{{ inventory_hostname }}"
        username: "{{ creds['username'] }}"
        password: "{{ creds['password'] }}"
        auth_pass: "{{ creds['auth_pass'] }}"
        port: 830

  - name: Configure interfaces
    iosxe_netconf_yml:
      provider: "{{ provider }}"
      commands:
       interface:
         TenGigabitEthernet:
           name: "1/0/1"
           description: TenGigabitEthernet 1/0/1 connected to IXIA5 1/1
    register: myoutput

  - debug: var=myoutput.stdout_lines

  - name: Configure STP, VTP,  etc.,
    iosxe_netconf_yml:
      provider: "{{ provider }}"
      commands:
        "spanning-tree":
          backbonefast: ""
          mode: mst
        udld:
          message:
            time: 11
        vtp:
          domain: ngwc
          mode:
           "device-mode": transparent
    register: myoutput

  - debug: var=myoutput.stdout_lines

```

## IOSXE example 4: Playbook using NETCONF YML configuration

```
$ ansible/bin/ansible-playbook   -i /home/cisco/hosts /home/cisco/ansible/test/samples/iosxe_netconf_yml.yml

PLAY [myswitches] **************************************************************

TASK [OBTAIN LOGIN CREDENTIALS] ************************************************
ok: [172.25.209.236]

TASK [DEFINE PROVIDER] *********************************************************
ok: [172.25.209.236]

TASK [Configure interfaces] ****************************************************
changed: [172.25.209.236]

TASK [debug] *******************************************************************
ok: [172.25.209.236] => {
    "myoutput.stdout_lines": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n            <config xmlns:xc=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n              <native xmlns=\"http://cisco.com/ns/yang/ned/ios\">\n<interface><TenGigabitEthernet><name>1/0/1</name><description>TenGigabitEthernet 1/0/1 connected to IXIA5 1/1</description></TenGigabitEthernet></interface>\n              </native>\n            </config>\n            "
}

TASK [Configure STP, VTP,  etc.,] **********************************************
changed: [172.25.209.236]

TASK [debug] *******************************************************************
ok: [172.25.209.236] => {
    "myoutput.stdout_lines": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n            <config xmlns:xc=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n              <native xmlns=\"http://cisco.com/ns/yang/ned/ios\">\n<vtp><domain>ngwc</domain><mode><device-mode>transparent</device-mode></mode></vtp><spanning-tree><backbonefast></backbonefast><mode>mst</mode></spanning-tree><udld><message><time>11</time></message></udld>\n              </native>\n            </config>\n            "
}

PLAY RECAP *********************************************************************
172.25.209.236             : ok=6    changed=2    unreachable=0    failed=0   

```

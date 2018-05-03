# mr-provisioner-kea-dhcp4-role
Ansible role for installing/upgrading a kea-dhcp4 server, which is required for a fully functional Mr. Provisioner installation.

# Overview
This Ansible allows you to install kea-dhpc4, and the mr-provisioner-kea hook in an automated way.
You have two choices : either you install kea and the hook, and build them on the target machine,
Or you give a .deb package to this role to install instead.
We recommend you go for the former option, as kea takes 192/20 minutes to build (depending on how many cores and memory you throw at it). Sadly, no official deb package is distributed.

# Variables Example
 ```yaml
    - kea_version: "1.3.0"  #The verion of kea you'd like to install
    - kea_services:
        - kea-dhcp4		#The kea service to install, dhcp4 being the only one supported by MrP

    - kea_aptinstall: True	#Boolean to switch the deb install on or off (off being compiling) Default: False
    - kea_deb_pkg_name: "kea_1.3.0-RTDEPS_amd64.deb"	#The name of the kea deb package
    - kea_deb_pkg_html: "http://my.packages.com/{{ kea_deb_pkg_name}}" #the url to fetch the deb from

    - kea_hook_aptinstall: True 	#Boolean to switch the hook install on or off Default: False
    - kea_hook_deb_pkg_name: "mr-provisioner-kea-hook_0.1-SNAPSHOT_amd64.deb"   
    - kea_hook_repo: "http://my.packages.com/{{ kea_hook_deb_pkg_name }}"	

    - kea_validlifetime: "1728000" Default: 3600
    - kea_renewtimer: "86900"	Default: 1000
    - kea_rebindtimer: "172800"	Default: 2000

    - kea_dhcp_listen_ifaces: ["eth0", "eth1"]	#The interfaces that kea should listen on
    - kea_config: "/opt/kea/etc/kea/kea.conf"	#The path to the config, for the systemd file
    - kea_path: "/opt/kea"		#Root path to kea install
    - kea_build_path: "{{kea_path}}/build"
    - kea_log_level: "DEBUG"
    - kea_nameservers: "192.90.0.1, 8.8.8.8, 8.8.4.4"
    - kea_default_gateway: "192.90.0.1"
    - kea_subnets:
        - subnet: "192.90.0.0/16"
          interface: "ens2"
          pools:
            - "192.90.16.1-192.90.19.254"
            - "192.90.24.1-192.90.27.254"
        - subnet: "192.41.0.0/16"
          interface: "ens3"
          pools:
            - "192.168.1.2-192.168.1.255"
          nameserver: "8.8.8.8, 8.8.4.4"
          default_gateway: "192.168.1.1"
          reservations:
                  - macaddress: "01:02:03:04:05:06"
                    ipaddress: "192.168.1.123"
                  - macaddress: "02:03:04:05:06:07"
                    ipaddress: "192.168.1.234"
```
### For additional variables, please refer to defaults/main.yml and the kea.conf.j2 template.

# Usage:
```yaml
- hosts: mr-provisioner
  vars:
    #Define your variables here

  pre_tasks:
    - apt:
        update_cache: yes
    - file:
        path: "{{item}}"
        state: directory
      with_items:
        - "{{kea_build_path}}"
        - "{{kea_path}}/bin"
        - "{{kea_path}}/logs"
  roles:
    - kea-dhcp4
```


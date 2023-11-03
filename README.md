# Role Name
`na_ots_cluster`: Install an ONTAP Select Cluster
# Requirements
This role requires Ansible 2.7 release.
# Role Variables
```yaml
# -------------------------------------------------------------------
# Passwords
# - Place these in a separate file and implement encryption if needed
# -------------------------------------------------------------------
deploy_pwd: ""
vcenter_password: ""
ontap_pwd: ""
host_esx_password: ""
host_kvm_password: ""
```
```yaml
# -------------------------------------------------------------------
# state:
# - present : create cluster
# - absent  : delete cluster
# - hosts_present : registers hosts with deploy without building a cluster
# - hosts_absent  : deletes hosts from deploy without building a cluster
# - power_on  : powers on existing cluster
# - power_off : shut down running cluster
# - constructing : creates cluster but does not deploy it
# -------------------------------------------------------------------
state: present
```
```yaml
# ----------------------------------------------------------------
# Configuration Settings
# - Place these variables in a separate .yml file and reference in
#   the playbook along with the include_role statement
# ----------------------------------------------------------------
node_count: <Number of nodes in the cluster - 1,2,4,6,8>
hypervisor: <Hypervisor Type - ESX or KVM>
```
```yaml
# ------------------------------------------------------------------------
# true = Authenticate through vCenter | false = Authenticate host directly
#        if esxi host is managed by vCenter then you MUST use vCenter
# ------------------------------------------------------------------------
use_vcenter: <whether to authenticate through vCenter - true or false>
```
```yaml
# -------------------------------------------------
# Different parameters to indicate preferences to monitor a job until "success" or "failure"
# -------------------------------------------------
monitor_deploy_job: <true or false>
monitor_deploy_retries: <Number of retries until cluster creation succeeds or fails>
monitor_deploy_delay: <Delay in seconds between monitor retries>
monitor_netcheck_retries: <Number of retries until network check succeeds or fails>
monitor_netcheck_delay: <Delay in seconds between netcheck retries>
monitor_host_retries: <Number of retries until host add/delete succeeds or fails>
monitor_host_delay: <Delay in seconds between host retries>
monitor_availability_retries: <Number of retries until cluster availability succeeds or fails>
monitor_availability_delay: <Delay in seconds between availability retries>
```
```yaml
# ----------------------------------------------------------------------
# Network Connectivity Check
# - Set to true to run the network connectivity check
# - Valid modes: quick, extended, cleanup
# - Valid switch types: StandardvSwitch, DistributedvSwitch, OpenvSwitch
# - cluster_nodes referenced so make sure item count = node count !!!
# ----------------------------------------------------------------------
run_net_check: <flag to indicate if network connectivity check should be run or not - true, false>
net_mode: <modes of network checking - quick, extended, cleanup>
net_mtu: <MTU size>
net_switch_type: <types of switch - StandardvSwitch, DistributedvSwitch, OpenvSwitch>
```

```yaml
# -----------
# Deploy Info
# -----------
deploy_ip: <your deploy vm ip address>
```

```yaml
# ------------
# vCenter Info
# ------------
vcenter_login: <your v-center login name>
vcenter_name: <your v-center name or IP>
```

```yaml
# -----
# Hosts
# -----
esxi_hosts:
  - host_name:
    host_login:
  - host_name:
    host_login:

kvm_hosts:
  - host_name:
    host_login:
  - host_name:
    host_login:
```

```yaml
# ------------
# Cluster Info
# ------------
cluster_name:
cluster_ip:
cluster_netmask:
cluster_gateway:
cluster_ontap_image: "9.14.1"
cluster_ntp:
  -
cluster_dns_ips:
  -
cluster_dns_domains:
  -
```
```yaml
# --------
# Networks
# --------
mgt_network: Management
mgt_network_vlan:
data_network: Data
data_network_vlan:
internal_network: Internal
internal_network_vlan: 
```
```yaml
# --------
# Instance
# --------
instance_type: <small or medium>
```
```yaml
# --------------------------------------------------
# Node Info
# - cluster_nodes # of items should equal node_count
# --------------------------------------------------
cluster_nodes:
  - node_name: "{{ cluster_name }}-01"
    ipAddress:
    storage_pool:
    capacityTB: 3
    host_name:
  - node_name: "{{ cluster_name }}-02"
    ipAddress:
    storage_pool:
    capacityTB: 3
    host_name:
# --------------------------------------------------
# Node Info
# - options for software raid
# --------------------------------------------------
  - node_name: "{{ cluster_name }}-03"
    ipAddress:
    storage_pool:
    host_name:
    software_raid: true
  - node_name: "{{ cluster_name }}-04"
    ipAddress:
    storage_pool:
    host_name:
    software_raid: true
    host_disks:   # optionally specify which disks to use
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201172"
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201489"
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201513"
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201701"
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201401"
      - "/dev/disk/by-id/ata-SAMSUNG_MZ7LM960HMJP-00005_S2TZNB0J201472"

```
# Dependencies
This role assumes that the `na_ots_deploy` role (or the manual equivalent) has already been run and the deploy VM is running.

# Example Playbook
```yaml
---
- name: Create ONTAP Select cluster (ESXi)
  hosts: "localhost"
  gather_facts: false
  # -------------------
  # Read variable files
  # -------------------
  vars_files:
  - vars_cluster.yml
  - vars_cluster_pwd.yml
  roles:
    - na_ots_cluster
```
I use global files to hold variables.
```yaml
node_count: 2
monitor_deploy_job: true
deploy_api_url: "https://xx.xxx.xx.xx/api/v3"
deploy_login: "admin"
vcenter_login: "yourvclogin@yourlab.local"
vcenter_name: "xx.xxx.xx.xx"
esxi_hosts:
  - host_name: xx.xxx.xx.xx

cluster_name: "onenodecluster"
cluster_ip: "10.193.xx.xx"
cluster_netmask: "255.255.255.0"
cluster_gateway: "10.193.xx.xx"
cluster_ontap_image: "9.5P2X1"
cluster_ntp:
  - "ntpxx.your.ntp.com"
cluster_dns_ips:
  - "10.193.x.xxx"
cluster_dns_domains:
  - "thisis.your.dns.com"

mgt_network: "your-mgmt-network"
data_network: "your-data-network"
internal_network: "your-select-internal-network"
instance_type: "small"
cluster_nodes:
  - node_name: "{{ cluster_name }}-01"
    ipAddress: 10.193.xx.xx
    storage_pool: yourstopool
    capacityTB: 1.2
    host_name: 10.193.xx.xx
  - node_name: "{{ cluster_name }}-02"
    ipAddress: 10.0.10.xx
    storage_pool: dsONTAP2
    capacityTB: 1.2
```
# License
BSD
# Author Information
NetApp

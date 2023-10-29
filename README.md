# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - eUPF(eBPF/XDP UPF)
This describes a simple configuration for working Open5GS 5GC and eUPF(eBPF/XDP UPF).
In particular, see [here](https://github.com/s5uishida/install_eupf) for eUPF.

---

<a id="conf_list"></a>

## List of Sample Configurations

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. [One SMF, one UPF and one DNN](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
4. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
5. [Select nearby UPF(PGW-U) according to the connected eNodeB](https://github.com/s5uishida/open5gs_epc_srsran_nearby_upf_sample_config)
6. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
7. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
8. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
9. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
10. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
11. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
12. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
13. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
14. [eUPF(eBPF/XDP UPF(PGW-U))](https://github.com/s5uishida/open5gs_epc_srsran_eupf_sample_config)
15. eUPF(eBPF/XDP UPF) (this article)

---

<a id="misc"></a>

## Miscellaneous Notes

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [Install eUPF(eBPF/XDP UPF) on Host](https://github.com/s5uishida/install_eupf)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC, eUPF and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of eUPF](#changes_up)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE (IMSI-001010000000000)](#changes_ue)
- [Network settings of Open5GS 5GC, eUPF and UERANSIM UE / RAN](#network_settings)
  - [Network settings of eUPF and Data Network Gateway](#network_settings_up)
- [Build Open5GS, eUPF and UERANSIM](#build)
- [Run Open5GS 5GC, eUPF and UERANSIM UE / RAN](#run)
  - [Run eUPF](#run_up)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run UERANSIM](#run_ueran)
    - [Start gNB](#start_gnb)
    - [Start UE](#start_ue)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)

---

<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

This describes a simple configuration of C-Plane, eBPF/XDP UPF and Data Network Gateway for Open5GS 5GC.
**Note that this configuration is implemented with Virtualbox VMs.**

The following minimum configuration was set as a condition.
- One UPF and Data Network Gateway
- One UE and one DNN

The built simulation environment is as follows.
**Note. According to [this](https://github.com/open5gs/open5gs/discussions/1964#discussioncomment-4492394), eUPF uses User Plane IP Resource Information, but Open5GS does not support this since it was removed in 3GPP Release 16.3.
Open5GS SMF uses the PFCP IP address instead because the User Plain IP resource is not available.
So make N3 and N4 interfaces of eUPF same and set the same IP address.**

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / eBPF/XDP UPF / UE / RAN used are as follows.
- 5GC - Open5GS v2.6.6 (2023.10.28) - https://github.com/open5gs/open5gs
- eBPF/XDP UPF - eUPF v0.5.1 (2023.10.28) - https://github.com/edgecomllc/eupf
- UE / RAN - UERANSIM v3.2.6 (2023.06.14) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU<br>(Min) | Memory<br>(Min) | HDD<br>(Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 22.04 | 1 | 1GB | 20GB |
| VM-UP | eUPF U-Plane | 192.168.0.151/24 | Ubuntu 22.04 | 1 | 2GB | 20GB |
| VM-DN | Data Network Gateway  | 192.168.0.152/24 | Ubuntu 22.04 | 1 | 1GB | 10GB |
| VM2 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 22.04 | 1 | 1GB | 10GB |
| VM3 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 22.04 | 1 | 1GB | 10GB |

The network interfaces of each VM are as follows.
| VM | Device | Network Adapter | IP address | Interface | XDP |
| --- | --- | --- | --- | --- | --- |
| VM1 | enp0s3 | NAT(default) | 10.0.2.15/24 | (VM default NW) | -- |
| | enp0s8 | Bridged Adapter | 192.168.0.111/24 | (Mgmt NW) | -- |
| | enp0s9 | NAT Network | 192.168.14.111/24 | N4 | -- |
| VM-UP | ~~enp0s3~~ | ~~NAT(default)~~ | ~~10.0.2.15/24~~ | ~~(VM default NW)~~ ***down*** | -- |
| | enp0s8 | Bridged Adapter | 192.168.0.151/24 | (Mgmt NW) | -- |
| | ~~enp0s9~~ | ~~NAT Network~~ | ~~192.168.13.151/24~~ | ~~N3~~ | ~~x~~ |
| | **enp0s10** | **NAT Network** | **192.168.14.151/24** | **N3, N4** | **x** |
| | enp0s16 | NAT Network | 192.168.16.151/24 | N6 | x |
| VM-DN | enp0s3 | NAT(default) | 10.0.2.15/24 | (VM default NW) | -- |
| | enp0s8 | Bridged Adapter | 192.168.0.152/24 | (Mgmt NW) | -- |
| | enp0s9 | NAT Network | 192.168.16.152/24 | N6, ***default GW for VM-UP*** | -- |
| VM2 | enp0s3 | NAT(default) | 10.0.2.15/24 | (VM default NW) | -- |
| | enp0s8 | Bridged Adapter | 192.168.0.131/24 | (Mgmt NW) | -- |
| | ~~enp0s9~~ | ~~NAT Network~~ | ~~192.168.13.131/24~~ | ~~N3~~ | -- |
| | **enp0s10** | **NAT Network** | **192.168.14.131/24** | **N3** | -- |
| VM3 | enp0s3 | NAT(default) | 10.0.2.15/24 | (VM default NW) | -- |
| | enp0s8 | Bridged Adapter | 192.168.0.132/24 | (Mgmt NW) | -- |

NAT networks of Virtualbox  are as follows.
| Network Name | Network CIDR |
| --- | --- |
| N3 | 192.168.13.0/24 |
| N4 | 192.168.14.0/24 |
| N6 | 192.168.16.0/24 |


Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration file.**
| UE | IMSI | DNN | OP/OPc |
| --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

The DN is as follows.
| DN | DNN | TUNnel interface of UE |
| --- | --- | --- |
| 10.45.0.0/16 | internet | uesimtun0 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC, eUPF and UERANSIM UE / RAN

Please refer to the following for building Open5GS, eUPF and UERANSIM respectively.
- Open5GS v2.6.6 (2023.10.28) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- eUPF v0.5.1 (2023.10.28) - https://github.com/s5uishida/install_eupf
- UERANSIM v3.2.6 (2023.06.14) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

The following parameters including DNN can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time. Please refer to [here](https://github.com/open5gs/open5gs/pull/560#issue-483001043) for the logic to select UPF.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2023-10-28 22:33:42.000000000 +0900
+++ amf.yaml    2023-10-28 22:38:40.059200024 +0900
@@ -474,26 +474,26 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.5
         port: 9090
     guami:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
     security:
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2023-10-28 22:33:42.000000000 +0900
+++ smf.yaml    2023-10-28 22:38:44.076164900 +0900
@@ -602,25 +602,20 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.14.111
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.14.111
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
-      - 2001:4860:4860::8888
-      - 2001:4860:4860::8844
     mtu: 1400
     ctf:
       enabled: auto
@@ -808,7 +803,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.14.151
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```

<a id="changes_up"></a>

### Changes in configuration files of eUPF

See [here](https://github.com/s5uishida/install_eupf#create-configuration-file) for the original file.

**Note. According to [this](https://github.com/open5gs/open5gs/discussions/1964#discussioncomment-4492394), eUPF uses User Plane IP Resource Information, but Open5GS does not support this since it was removed in 3GPP Release 16.3.
Open5GS SMF uses the PFCP IP address instead because the User Plain IP resource is not available.
So make N3 and N4 interfaces of eUPF same and set the same IP address. And the `config.yml` file will need to be changed as follows.**

- `eupf/config.yml`  
```diff
--- config.yml.orig     2023-10-21 20:32:17.000000000 +0900
+++ config.yml  2023-10-28 19:33:14.000000000 +0900
@@ -1,10 +1,10 @@
-interface_name: [enp0s9, enp0s16]
+interface_name: [enp0s10, enp0s16]
 xdp_attach_mode: generic
 api_address: :8080
 pfcp_address: 192.168.14.151:8805
 pfcp_node_id: 192.168.14.151
 metrics_address: :9090
-n3_address: 192.168.13.151
+n3_address: 192.168.14.151
 qer_map_size: 1024
 far_map_size: 1024
 pdr_map_size: 1024
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:44.000000000 +0900
+++ open5gs-gnb.yaml    2023-10-28 18:57:12.680487533 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.14.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```


<a id="changes_ue"></a>

#### Changes in configuration files of UE (IMSI-001010000000000)

- `UERANSIM/config/open5gs-ue.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-05-10 19:00:38.000000000 +0900
+++ open5gs-ue.yaml     2023-06-15 21:42:14.363706123 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC, eUPF and UERANSIM UE / RAN

<a id="network_settings_up"></a>

### Network settings of eUPF and Data Network Gateway

See [this1](https://github.com/s5uishida/install_eupf#setup-eupf-on-vm-up) and [this2](https://github.com/s5uishida/install_eupf#setup-data-network-gateway-on-vm-dn).

<a id="build"></a>

## Build Open5GS, eUPF and UERANSIM

Please refer to the following for building Open5GS, eUPF and UERANSIM respectively.
- Open5GS v2.6.6 (2023.10.28) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- eUPF v0.5.1 (2023.10.28) - https://github.com/s5uishida/install_eupf
- UERANSIM v3.2.6 (2023.06.14) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<a id="run"></a>

## Run Open5GS 5GC, eUPF and UERANSIM UE / RAN

First run eUPF, then the 5GC and UERANSIM (UE & RAN implementation).

<a id="run_up"></a>

### Run eUPF

See [this](https://github.com/s5uishida/install_eupf#run-eupf-on-vm-up).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```
The PFCP association log between eUPF and Open5GS SMF is as follows.
```
2023/10/29 14:05:53 INF Received 30 bytes from 192.168.14.111:8805
2023/10/29 14:05:53 INF Handling PFCP message from 192.168.14.111:8805
2023/10/29 14:05:53 INF Got Association Setup Request from: 192.168.14.111. 

2023/10/29 14:05:53 INF 
Association Setup Request:
  Node ID: 192.168.14.111
  Recovery Time: 2023-10-29 14:05:53 +0900 JST

2023/10/29 14:05:53 INF Saving new association: &{ID:192.168.14.111 Addr:192.168.14.111 NextSessionID:1 NextSequenceID:1 Sessions:map[] HeartbeatRetries:0 cancelRetries:<nil>}
```

<a id="run_ueran"></a>

### Run UERANSIM

Here, the case of UE (IMSI-001010000000000) & RAN is described.
First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="start_gnb"></a>

#### Start gNB

Start gNB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2023-10-29 14:08:14.747] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2023-10-29 14:08:14.750] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2023-10-29 14:08:14.750] [sctp] [debug] SCTP association setup ascId[16]
[2023-10-29 14:08:14.750] [ngap] [debug] Sending NG Setup Request
[2023-10-29 14:08:14.753] [ngap] [debug] NG Setup Response received
[2023-10-29 14:08:14.753] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
10/29 14:08:14.775: [amf] INFO: gNB-N2 accepted[192.168.0.131]:56102 in ng-path module (../src/amf/ngap-sctp.c:113)
10/29 14:08:14.775: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:741)
10/29 14:08:14.775: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1203)
10/29 14:08:14.775: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:780)
```


<a id="start_ue"></a>

#### Start UE

Start UE as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue.yaml
UERANSIM v3.2.6
[2023-10-29 14:09:20.243] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-10-29 14:09:20.244] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-10-29 14:09:20.244] [nas] [info] Selected plmn[001/01]
[2023-10-29 14:09:20.244] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-10-29 14:09:20.244] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-10-29 14:09:20.245] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-10-29 14:09:20.245] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-10-29 14:09:20.245] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-10-29 14:09:20.245] [nas] [debug] Sending Initial Registration
[2023-10-29 14:09:20.245] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-10-29 14:09:20.246] [rrc] [debug] Sending RRC Setup Request
[2023-10-29 14:09:20.246] [rrc] [info] RRC connection established
[2023-10-29 14:09:20.246] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-10-29 14:09:20.246] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-10-29 14:09:20.274] [nas] [debug] Authentication Request received
[2023-10-29 14:09:20.280] [nas] [debug] Security Mode Command received
[2023-10-29 14:09:20.280] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-10-29 14:09:20.308] [nas] [debug] Registration accept received
[2023-10-29 14:09:20.308] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-10-29 14:09:20.309] [nas] [debug] Sending Registration Complete
[2023-10-29 14:09:20.309] [nas] [info] Initial Registration is successful
[2023-10-29 14:09:20.309] [nas] [debug] Sending PDU Session Establishment Request
[2023-10-29 14:09:20.309] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-10-29 14:09:20.519] [nas] [debug] Configuration Update Command received
[2023-10-29 14:09:20.548] [nas] [debug] PDU Session Establishment Accept received
[2023-10-29 14:09:20.552] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-10-29 14:09:20.573] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
10/29 14:09:20.248: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
10/29 14:09:20.248: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2522)
10/29 14:09:20.248: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
10/29 14:09:20.254: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1807)
10/29 14:09:20.254: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1588)
10/29 14:09:20.254: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1165)
10/29 14:09:20.254: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:157)
10/29 14:09:20.260: [sbi] WARNING: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.261: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.261: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.261: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.261: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.262: [sbi] INFO: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.302: [sbi] WARNING: [d5db7744-7618-41ee-8980-fbcaf15368ad] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.303: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.303: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.304: [sbi] INFO: [d5db7744-7618-41ee-8980-fbcaf15368ad] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.518: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2097)
10/29 14:09:20.518: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:612)
10/29 14:09:20.519: [gmm] INFO:     UTC [2023-10-29T05:09:20] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
10/29 14:09:20.519: [gmm] INFO:     LOCAL [2023-10-29T14:09:20] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
10/29 14:09:20.520: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2543)
10/29 14:09:20.520: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1247)
10/29 14:09:20.521: [gmm] INFO: SMF Instance [d5e70ff0-7618-41ee-99b3-2d1d4eb67607] (../src/amf/gmm-handler.c:1286)
10/29 14:09:20.524: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1010)
10/29 14:09:20.525: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3057)
10/29 14:09:20.526: [sbi] WARNING: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.527: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.527: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.527: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.527: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.527: [sbi] INFO: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.534: [sbi] WARNING: [d5db3edc-7618-41ee-a613-f96cceebf2b6] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.534: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.534: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.534: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.534: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.535: [sbi] INFO: [d5db3edc-7618-41ee-a613-f96cceebf2b6] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.536: [sbi] WARNING: [d5db7744-7618-41ee-8980-fbcaf15368ad] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.536: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.537: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.537: [sbi] INFO: [d5db7744-7618-41ee-8980-fbcaf15368ad] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.539: [sbi] WARNING: [d5d5620a-7618-41ee-9b87-8f01ca991c1b] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.539: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.539: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.539: [sbi] INFO: [d5d5620a-7618-41ee-9b87-8f01ca991c1b] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.542: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:539)
10/29 14:09:20.543: [gtp] INFO: gtp_connect() [192.168.14.151]:2152 (../lib/gtp/path.c:60)
10/29 14:09:20.555: [sbi] WARNING: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/29 14:09:20.555: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1732)
10/29 14:09:20.555: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.555: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.556: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1615)
10/29 14:09:20.556: [sbi] INFO: [d5d4fbd0-7618-41ee-8a65-7765dcdd9845] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/29 14:09:20.559: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:837)
```
The PDU session establishment log of eUPF is as follows.
```
2023/10/29 14:09:20 INF Got Session Establishment Request from: 192.168.14.111.
2023/10/29 14:09:20 INF 
Session Establishment Request:
  CreatePDR ID: 1 
    FAR ID: 1 
    QER ID: 1 
    URR ID: 1 
    Source Interface: 1 
    UE IPv4 Address: 10.45.0.2 
  CreatePDR ID: 2 
    Outer Header Removal: 0 
    FAR ID: 2 
    QER ID: 1 
    Source Interface: 0 
    TEID: 59898 
    Ipv4: 192.168.14.151 
    Ipv6: <nil> 
    UE IPv4 Address: 10.45.0.2 
  CreatePDR ID: 3 
    Outer Header Removal: 0 
    FAR ID: 1 
    QER ID: 1 
    Source Interface: 3 
    TEID: 24413 
    Ipv4: 192.168.14.151 
    Ipv6: <nil> 
  CreatePDR ID: 4 
    Outer Header Removal: 0 
    FAR ID: 3 
    Source Interface: 0 
    TEID: 59898 
    Ipv4: 192.168.14.151 
    Ipv6: <nil> 
    SDF Filter: permit out 58 from ff02::2/128 to assigned 
  CreateFAR ID: 1 
    Apply Action: [12 0] 
    BAR ID: 1 
  CreateFAR ID: 2 
    Apply Action: [2 0] 
    Forwarding Parameters:
      Network Instance:internet 
  CreateFAR ID: 3 
    Apply Action: [2 0] 
    Forwarding Parameters:
      Network Instance:internet 
      Outer Header Creation: &{OuterHeaderCreationDescription:256 TEID:1 IPv4Address:192.168.14.111 IPv6Address:<nil> PortNumber:0 CTag:0 STag:0} 
  CreateQER ID: 1 
    Gate Status DL: 0 
    Gate Status UL: 0 
    Max Bitrate DL: 10000000 
    Max Bitrate UL: 10000000 
    QFI: 1 
  CreateURR ID: 1 
    Measurement Method: 2 
    Volume Threshold: &{Flags:1 TotalVolume:104857600 UplinkVolume:0 DownlinkVolume:0} 
  CreateBAR ID: 1

2023/10/29 14:09:20 INF Saving FAR info to session: 1, {Action:12 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF EBPF: Put FAR: internalId=0, qerInfo={Action:12 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF WARN: No OuterHeaderCreation
2023/10/29 14:09:20 INF Saving FAR info to session: 2, {Action:2 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF EBPF: Put FAR: internalId=1, qerInfo={Action:2 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF Saving FAR info to session: 3, {Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:1863231680 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF EBPF: Put FAR: internalId=2, qerInfo={Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:1863231680 LocalIP:2534320320 TransportLevelMarking:0}
2023/10/29 14:09:20 INF Saving QER info to session: 1, {GateStatusUL:0 GateStatusDL:0 Qfi:1 MaxBitrateUL:1410065408 MaxBitrateDL:1410065408 StartUL:0 StartDL:0}
2023/10/29 14:09:20 INF EBPF: Put QER: internalId=0, qerInfo={GateStatusUL:0 GateStatusDL:0 Qfi:1 MaxBitrateUL:1410065408 MaxBitrateDL:1410065408 StartUL:0 StartDL:0}
2023/10/29 14:09:20 INF EBPF: Put PDR Downlink: ipv4=10.45.0.2, pdrInfo={OuterHeaderRemoval:0 FarId:0 QerId:0 SdfFilter:<nil>}
2023/10/29 14:09:20 INF EBPF: Put PDR Uplink: teid=59898, pdrInfo={OuterHeaderRemoval:0 FarId:1 QerId:0 SdfFilter:<nil>}
2023/10/29 14:09:20 INF EBPF: Put PDR Uplink: teid=24413, pdrInfo={OuterHeaderRemoval:0 FarId:0 QerId:0 SdfFilter:<nil>}
2023/10/29 14:09:20 INF Session Establishment Request from 192.168.14.111 accepted.
2023/10/29 14:09:20 INF Received 70 bytes from 192.168.14.111:8805
2023/10/29 14:09:20 INF Handling PFCP message from 192.168.14.111:8805
2023/10/29 14:09:20 INF Got Session Modification Request from: 192.168.14.111. 

2023/10/29 14:09:20 INF Finding association for 192.168.14.111
2023/10/29 14:09:20 INF Finding session 2
2023/10/29 14:09:20 INF 
Session Modification Request:
  UpdateFAR ID: 1 
    Apply Action: [2 0] 
    Update forwarding Parameters:
      Network Instance:internet 
      Outer Header Creation: &{OuterHeaderCreationDescription:256 TEID:1 IPv4Address:192.168.14.131 IPv6Address:<nil> PortNumber:0 CTag:0 STag:0} 

2023/10/29 14:09:20 INF Updating FAR info: 1, {FarInfo:{Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:2198776000 LocalIP:2534320320 TransportLevelMarking:0} GlobalId:0}
2023/10/29 14:09:20 INF EBPF: Update FAR: internalId=0, farInfo={Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:2198776000 LocalIP:2534320320 TransportLevelMarking:0}
```
Looking at the console log of the `nr-ue` command, UE has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2023-10-29 14:09:20.573] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE's TUNnel interface.
```
# ip addr show
...
16: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::fe17:db49:79ec:3362/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping"></a>

## Ping google.com

Specify the UE's TUNnel interface and try ping.

Please refer to the following for usage of TUNnel interface.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="ping_1"></a>

### Case for going through DN 10.45.0.0/16

Run `tcpdump` on VM-DN and check that the packet goes through N6 (enp0s9).
- `ping google.com` on VM3 (UE)
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.198.14) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.198.14: icmp_seq=1 ttl=61 time=24.3 ms
64 bytes from 142.250.198.14: icmp_seq=2 ttl=61 time=18.8 ms
64 bytes from 142.250.198.14: icmp_seq=3 ttl=61 time=18.8 ms
```
- Run `tcpdump` on VM-DN
```
# tcpdump -i enp0s9 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:17:32.672066 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 17, seq 1, length 64
14:17:32.694054 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 17, seq 1, length 64
14:17:33.672949 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 17, seq 2, length 64
14:17:33.689430 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 17, seq 2, length 64
14:17:34.673908 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 17, seq 3, length 64
14:17:34.690576 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 17, seq 3, length 64
```
- See `/sys/kernel/debug/tracing/trace_pipe` on VM-UP
```
# cat /sys/kernel/debug/tracing/trace_pipe
...
          <idle>-0       [000] d.s31 30214.961340: bpf_trace_printk: upf: gtp-u received
          <idle>-0       [000] dNs31 30214.961457: bpf_trace_printk: upf: far:1 action:2 outer_header_creation:0
          <idle>-0       [000] dNs31 30214.961459: bpf_trace_printk: upf: qer:0 gate_status:0 mbr:1410065408
          <idle>-0       [000] dNs31 30214.961462: bpf_trace_printk: upf: session for teid:4599 far:1 outer_header_removal:0
          <idle>-0       [000] dNs31 30214.961475: bpf_trace_printk: upf: bpf_fib_lookup 10.45.0.2 -> 142.250.198.14: nexthop: 192.168.16.152
          <idle>-0       [000] d.s31 30214.980291: bpf_trace_printk: upf: downlink session for ip:10.45.0.2  far:0 action:2
          <idle>-0       [000] dNs31 30214.980320: bpf_trace_printk: upf: qer:0 gate_status:0 mbr:1410065408
          <idle>-0       [000] dNs31 30214.980323: bpf_trace_printk: upf: use mapping 10.45.0.2 -> TEID:1
          <idle>-0       [000] dNs31 30214.980326: bpf_trace_printk: upf: send gtp pdu 192.168.14.151 -> 192.168.14.131
          <idle>-0       [000] dNs31 30214.980336: bpf_trace_printk: upf: bpf_fib_lookup 192.168.14.151 -> 192.168.14.131: nexthop: 192.168.14.131
...
```
You could specify the IP address assigned to the TUNnel interface to run almost any applications as in the following example using `nr-binder` tool.

- Run `curl google.com` on VM3 (UE)
```
# sh nr-binder 10.45.0.2 curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM-DN
```
14:19:46.016728 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [S], seq 3713281153, win 65280, options [mss 1360,sackOK,TS val 4274341564 ecr 0,nop,wscale 7], length 0
14:19:46.032561 IP 142.250.198.14.80 > 10.45.0.2.46831: Flags [S.], seq 7360001, ack 3713281154, win 65535, options [mss 1460], length 0
14:19:46.034932 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [.], ack 1, win 65280, length 0
14:19:46.035287 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [P.], seq 1:75, ack 1, win 65280, length 74: HTTP: GET / HTTP/1.1
14:19:46.035495 IP 142.250.198.14.80 > 10.45.0.2.46831: Flags [.], ack 75, win 65535, length 0
14:19:46.283274 IP 142.250.198.14.80 > 10.45.0.2.46831: Flags [P.], seq 1:774, ack 75, win 65535, length 773: HTTP: HTTP/1.1 301 Moved Permanently
14:19:46.285353 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [.], ack 774, win 64507, length 0
14:19:46.287861 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [F.], seq 75, ack 774, win 64507, length 0
14:19:46.288095 IP 142.250.198.14.80 > 10.45.0.2.46831: Flags [.], ack 76, win 65535, length 0
14:19:46.304499 IP 142.250.198.14.80 > 10.45.0.2.46831: Flags [F.], seq 774, ack 76, win 65535, length 0
14:19:46.306430 IP 10.45.0.2.46831 > 142.250.198.14.80: Flags [.], ack 775, win 64507, length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.
You could now connect to the DN and send any packets on the network using eUPF.

---

Now you could work Open5GS 5GC with eUPF.
I would like to thank the excellent developers and all the contributors of Open5GS, eUPF and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2023.10.29] Initial release.

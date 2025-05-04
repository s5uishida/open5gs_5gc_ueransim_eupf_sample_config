# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - eUPF(eBPF/XDP UPF)
This describes a simple configuration for working Open5GS 5GC and eUPF(eBPF/XDP UPF).
In particular, see [here](https://github.com/s5uishida/install_eupf) for eUPF.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

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
**Note that this configuration is implemented with Proxmox VE VMs.**

The following minimum configuration was set as a condition.
- One UPF and Data Network Gateway
- One UE and one DNN

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / eBPF/XDP UPF / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.5 (2025.04.25) - https://github.com/open5gs/open5gs
- eBPF/XDP UPF - eUPF v0.7.1 (2025.04.16) - https://github.com/edgecomllc/eupf
- UE / RAN - UERANSIM v3.2.7 (2025.04.28) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU<br>(Min) | Mem<br>(Min) | HDD<br>(Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM-UP | eUPF U-Plane | 192.168.0.151/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM-DN | Data Network Gateway  | 192.168.0.152/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |
| VM2 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |
| VM3 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |

The network interfaces of each VM are as follows.
| VM | Device | Model | Linux Bridge | IP address | Interface | XDP |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | ens18 | VirtIO | vmbr1 | 10.0.0.111/24 | (NAPT NW) | -- |
| | ens19 | VirtIO | mgbr0 | 192.168.0.111/24 | (Mgmt NW) | -- |
| | ens20 | VirtIO | vmbr4 | 192.168.14.111/24 | N4 | -- |
| VM-UP | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** | -- |
| | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) | -- |
| | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 | x |
| | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 | -- |
| | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 | x |
| VM-DN | ens18 | VirtIO | vmbr1 | 10.0.0.152/24 | (NAPT NW) | -- |
| | ens19 | VirtIO | mgbr0 | 192.168.0.152/24 | (Mgmt NW) | -- |
| | ens20 | VirtIO | vmbr6 | 192.168.16.152/24 | N6, ***default GW for VM-UP*** | -- |
| VM2 | ens18 | VirtIO | vmbr1 | 10.0.0.131/24 | (NAPT NW) | -- |
| | ens19 | VirtIO | mgbr0 | 192.168.0.131/24 | (Mgmt NW) | -- |
| | ens20 | VirtIO | vmbr3 | 192.168.13.131/24 | N3 | -- |
| VM3 | ens18 | VirtIO | vmbr1 | 10.0.0.132/24 | (NAPT NW) | -- |
| | ens19 | VirtIO | mgbr0 | 192.168.0.132/24 | (Mgmt NW) | -- |

Linux Bridges of Proxmox VE are as follows.
| Linux Bridge | Network CIDR | Interface |
| --- | --- | --- |
| vmbr1 | 10.0.0.0/24 | NAPT NW |
| mgbr0 | 192.168.0.0/24 | Mgmt NW |
| vmbr3 | 192.168.13.0/24 | N3 |
| vmbr4 | 192.168.14.0/24 | N4 |
| vmbr6 | 192.168.16.0/24 | N6 |

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
- Open5GS v2.7.5 (2025.04.25) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- eUPF v0.7.1 (2025.04.16) - https://github.com/s5uishida/install_eupf
- UERANSIM v3.2.7 (2025.04.28) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

The following parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ amf.yaml    2025-05-04 08:12:34.196824268 +0900
@@ -20,27 +20,27 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       tac: 1
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
   security:
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ nrf.yaml    2025-05-04 08:13:05.973154453 +0900
@@ -11,8 +11,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ smf.yaml    2025-05-04 13:40:51.181590101 +0900
@@ -20,16 +20,14 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.14.151
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
   metrics:
     server:
       - address: 127.0.0.4
@@ -37,20 +35,17 @@
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
 
 ################################################################################
 # SMF Info
```

<a id="changes_up"></a>

### Changes in configuration files of eUPF

See [here](https://github.com/s5uishida/install_eupf#create-configuration-file) for the original file.

- `eupf/config.yml`  
There is no change.

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2023-12-02 06:14:20.000000000 +0900
+++ open5gs-gnb.yaml    2025-05-04 08:59:07.242339870 +0900
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
+gtpIp: 192.168.13.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
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
--- open5gs-ue.yaml.orig        2025-03-16 15:49:12.000000000 +0900
+++ open5gs-ue.yaml     2025-05-04 09:24:31.352554298 +0900
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
@@ -31,7 +31,7 @@
 
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
- Open5GS v2.7.5 (2025.04.25) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- eUPF v0.7.1 (2025.04.16) - https://github.com/s5uishida/install_eupf
- UERANSIM v3.2.7 (2025.04.28) - https://github.com/aligungr/UERANSIM/wiki/Installation

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
2025/05/04 13:41:52 INF Got Association Setup Request from: 192.168.14.111
2025/05/04 13:41:52 INF 
Association Setup Request:
  Node ID: 192.168.14.111
  Recovery Time: 2025-05-04 13:41:52 +0900 JST

2025/05/04 13:41:52 INF Saving new association: &{ID:192.168.14.111 Addr:192.168.14.111 NextSessionID:1 NextSequenceID:1 Sessions:map[] HeartbeatChannel:0xc0006a6540 HeartbeatsActive:false Mutex:{_:{} mu:{state:0 sema:0}}}
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
UERANSIM v3.2.7
[2025-05-04 13:42:17.111] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2025-05-04 13:42:17.114] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2025-05-04 13:42:17.115] [sctp] [debug] SCTP association setup ascId[4]
[2025-05-04 13:42:17.115] [ngap] [debug] Sending NG Setup Request
[2025-05-04 13:42:17.121] [ngap] [debug] NG Setup Response received
[2025-05-04 13:42:17.121] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
05/04 13:42:17.104: [amf] INFO: gNB-N2 accepted[192.168.0.131]:40899 in ng-path module (../src/amf/ngap-sctp.c:113)
05/04 13:42:17.104: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:813)
05/04 13:42:17.110: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
05/04 13:42:17.110: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:860)
```

<a id="start_ue"></a>

#### Start UE

Start UE as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue.yaml
UERANSIM v3.2.7
[2025-05-04 13:42:45.348] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2025-05-04 13:42:45.349] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2025-05-04 13:42:45.349] [nas] [info] Selected plmn[001/01]
[2025-05-04 13:42:45.349] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2025-05-04 13:42:45.349] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2025-05-04 13:42:45.350] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2025-05-04 13:42:45.350] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2025-05-04 13:42:45.351] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-05-04 13:42:45.351] [nas] [debug] Sending Initial Registration
[2025-05-04 13:42:45.351] [rrc] [debug] Sending RRC Setup Request
[2025-05-04 13:42:45.351] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2025-05-04 13:42:45.351] [rrc] [info] RRC connection established
[2025-05-04 13:42:45.351] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2025-05-04 13:42:45.351] [nas] [info] UE switches to state [CM-CONNECTED]
[2025-05-04 13:42:45.357] [nas] [debug] Authentication Request received
[2025-05-04 13:42:45.357] [nas] [debug] Received SQN [000000000140]
[2025-05-04 13:42:45.357] [nas] [debug] SQN-MS [000000000000]
[2025-05-04 13:42:45.361] [nas] [debug] Security Mode Command received
[2025-05-04 13:42:45.362] [nas] [debug] Selected integrity[2] ciphering[0]
[2025-05-04 13:42:45.374] [nas] [debug] Registration accept received
[2025-05-04 13:42:45.374] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2025-05-04 13:42:45.374] [nas] [debug] Sending Registration Complete
[2025-05-04 13:42:45.374] [nas] [info] Initial Registration is successful
[2025-05-04 13:42:45.374] [nas] [debug] Sending PDU Session Establishment Request
[2025-05-04 13:42:45.374] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-05-04 13:42:45.576] [nas] [debug] Configuration Update Command received
[2025-05-04 13:42:45.590] [nas] [debug] PDU Session Establishment Accept received
[2025-05-04 13:42:45.590] [nas] [info] PDU Session establishment is successful PSI[1]
[2025-05-04 13:42:45.610] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
05/04 13:42:45.347: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:437)
05/04 13:42:45.347: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2789)
05/04 13:42:45.347: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:598)
05/04 13:42:45.347: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1906)
05/04 13:42:45.347: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1682)
05/04 13:42:45.347: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1323)
05/04 13:42:45.347: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:174)
05/04 13:42:45.347: [sbi] INFO: [199ec8ea-28a2-41f0-bca8-6b4276578d8f] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
05/04 13:42:45.347: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.348: [sbi] INFO: [199dd4bc-28a2-41f0-977a-6d3293fe1ce0] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
05/04 13:42:45.348: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.348: [sbi] INFO: [19a068ee-28a2-41f0-b13b-89b2b6a5af7f] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
05/04 13:42:45.349: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.351: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
05/04 13:42:45.352: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.353: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.353: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.355: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
05/04 13:42:45.357: [sbi] INFO: [199dd4bc-28a2-41f0-977a-6d3293fe1ce0] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
05/04 13:42:45.357: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.357: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.359: [sbi] INFO: [199dd4bc-28a2-41f0-977a-6d3293fe1ce0] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
05/04 13:42:45.359: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.359: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.361: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.361: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.362: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.363: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.363: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:355)
05/04 13:42:45.364: [sbi] INFO: [19a08f68-28a2-41f0-a306-f71a3337ffac] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
05/04 13:42:45.364: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.365: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
05/04 13:42:45.365: [sbi] INFO: [19a068ee-28a2-41f0-b13b-89b2b6a5af7f] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
05/04 13:42:45.365: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.367: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
05/04 13:42:45.570: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2658)
05/04 13:42:45.570: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:607)
05/04 13:42:45.570: [gmm] INFO:     UTC [2025-05-04T04:42:45] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
05/04 13:42:45.570: [gmm] INFO:     LOCAL [2025-05-04T13:42:45] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
05/04 13:42:45.571: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2810)
05/04 13:42:45.571: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1374)
05/04 13:42:45.571: [amf] INFO: [19b0641a-28a2-41f0-a5cb-f1c1e2fe6dbc] Setup NF Instance [type:SMF] (../src/amf/context.c:2433)
05/04 13:42:45.571: [gmm] INFO: SMF Instance [19b0641a-28a2-41f0-a5cb-f1c1e2fe6dbc] (../src/amf/gmm-handler.c:1415)
05/04 13:42:45.571: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.572: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1033)
05/04 13:42:45.572: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3190)
05/04 13:42:45.572: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:270)
05/04 13:42:45.572: [sbi] INFO: [199dd4bc-28a2-41f0-977a-6d3293fe1ce0] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
05/04 13:42:45.572: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.573: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.575: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.575: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:461)
05/04 13:42:45.576: [sbi] INFO: [19a08f68-28a2-41f0-a306-f71a3337ffac] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
05/04 13:42:45.576: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.576: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
05/04 13:42:45.577: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:442)
05/04 13:42:45.577: [sbi] INFO: [19a068ee-28a2-41f0-b13b-89b2b6a5af7f] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
05/04 13:42:45.577: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.578: [sbi] INFO: [199e1774-28a2-41f0-aee6-6b7b13bacaed] Setup NF Instance [type:BSF] (../lib/sbi/path.c:307)
05/04 13:42:45.578: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.579: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:121)
05/04 13:42:45.580: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:367)
05/04 13:42:45.580: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:580)
05/04 13:42:45.581: [gtp] INFO: gtp_connect() [192.168.13.151]:2152 (../lib/gtp/path.c:60)
05/04 13:42:45.582: [sbi] INFO: [18670bfe-28a2-41f0-9796-1fb118da8db1] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
05/04 13:42:45.582: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.585: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.586: [sbi] INFO: [199dd4bc-28a2-41f0-977a-6d3293fe1ce0] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
05/04 13:42:45.586: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.586: [sbi] INFO: [19a068ee-28a2-41f0-b13b-89b2b6a5af7f] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
05/04 13:42:45.587: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:461)
05/04 13:42:45.587: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:942)
```
The PDU session establishment log of eUPF is as follows.
```
2025/05/04 13:42:45 INF Got Session Establishment Request from: 192.168.14.111.
2025/05/04 13:42:45 INF 
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
    TEID: 0 
    Ipv4: <nil> 
    Ipv6: <nil> 
  CreatePDR ID: 3 
    Outer Header Removal: 0 
    FAR ID: 1 
    Source Interface: 3 
    TEID: 0 
    Ipv4: <nil> 
    Ipv6: <nil> 
  CreatePDR ID: 4 
    Outer Header Removal: 0 
    FAR ID: 3 
    Source Interface: 0 
    TEID: 0 
    Ipv4: <nil> 
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
      Outer Header Creation: &{OuterHeaderCreationDescription:256 TEID:1 IPv4Address:192.168.14.111 IPv6Address:<nil> PortNumber:0 CTag:0 STag:0} 
  CreateQER ID: 1 
    Gate Status DL: 0 
    Gate Status UL: 0 
    Max Bitrate DL: 1000000 
    Max Bitrate UL: 1000000 
    QFI: 1 
  CreateURR ID: 1 
    Measurement Method: 2 
    Volume Threshold: &{Flags:1 TotalVolume:104857600 UplinkVolume:0 DownlinkVolume:0} 
  CreateBAR ID: 1

2025/05/04 13:42:45 INF Saving FAR info to session: 1, {Action:12 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534254784 TransportLevelMarking:0}
2025/05/04 13:42:45 WRN No OuterHeaderCreation
2025/05/04 13:42:45 INF Saving FAR info to session: 2, {Action:2 OuterHeaderCreation:0 Teid:0 RemoteIP:0 LocalIP:2534254784 TransportLevelMarking:0}
2025/05/04 13:42:45 INF Saving FAR info to session: 3, {Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:1863231680 LocalIP:2534254784 TransportLevelMarking:0}
2025/05/04 13:42:45 INF Saving QER info to session: 1, {GateStatusUL:0 GateStatusDL:0 Qfi:1 MaxBitrateUL:1000000000 MaxBitrateDL:1000000000 StartUL:0 StartDL:0}
2025/05/04 13:42:45 INF Saving URR info to session: 1, {UplinkVolume:0 DownlinkVolume:0}
2025/05/04 13:42:45 Matched groups: ["permit out 58 from ff02::2/128 to assigned" "58" "ff02::2" "128" "" "assigned" "" ""]
2025/05/04 13:42:45 INF Session Establishment Request from 192.168.14.111 accepted.
2025/05/04 13:42:45 INF Got Session Modification Request from: 192.168.14.111. 

2025/05/04 13:42:45 INF Finding association for 192.168.14.111
2025/05/04 13:42:45 INF Finding session 2
2025/05/04 13:42:45 INF 
Session Modification Request:
  UpdateFAR ID: 1 
    Apply Action: [2 0] 
    Update forwarding Parameters:
      Network Instance:internet 
      Outer Header Creation: &{OuterHeaderCreationDescription:256 TEID:1 IPv4Address:192.168.13.131 IPv6Address:<nil> PortNumber:0 CTag:0 STag:0} 

2025/05/04 13:42:45 INF Updating FAR info: 1, {FarInfo:{Action:2 OuterHeaderCreation:1 Teid:1 RemoteIP:2198710464 LocalIP:2534254784 TransportLevelMarking:0} GlobalId:0}
```
Looking at the console log of the `nr-ue` command, UE has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2025-05-04 13:42:45.610] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE's TUNnel interface.
```
# ip addr show
...
4: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::90fd:28bd:ea5b:7e52/64 scope link stable-privacy 
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

Run `tcpdump` on VM-DN and check that the packet goes through N6 (ens20).
- `ping google.com` on VM3 (UE)
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.196.142) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.196.142: icmp_seq=1 ttl=110 time=17.4 ms
64 bytes from 142.250.196.142: icmp_seq=2 ttl=110 time=17.3 ms
64 bytes from 142.250.196.142: icmp_seq=3 ttl=110 time=17.2 ms
```
- Run `tcpdump` on VM-DN
```
# tcpdump -i ens20 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens20, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:46:59.706830 IP 10.45.0.2 > 142.250.196.142: ICMP echo request, id 1078, seq 1, length 64
13:46:59.723303 IP 142.250.196.142 > 10.45.0.2: ICMP echo reply, id 1078, seq 1, length 64
13:47:00.708305 IP 10.45.0.2 > 142.250.196.142: ICMP echo request, id 1078, seq 2, length 64
13:47:00.724708 IP 142.250.196.142 > 10.45.0.2: ICMP echo reply, id 1078, seq 2, length 64
13:47:01.709724 IP 10.45.0.2 > 142.250.196.142: ICMP echo request, id 1078, seq 3, length 64
13:47:01.725941 IP 142.250.196.142 > 10.45.0.2: ICMP echo reply, id 1078, seq 3, length 64
```
If you have built eUPF to output the kernel log for debugging as described [here](https://github.com/s5uishida/install_eupf#generate_codes), you can see the kernel log as follows.
```
# cat /sys/kernel/debug/tracing/trace_pipe
```
You could specify the IP address assigned to the TUNnel interface to run almost any applications (iperf3 etc.) as in the following example using `nr-binder` tool.

- `curl google.com` on VM3 (UE)
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
13:58:52.983887 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [S], seq 1466640255, win 65280, options [mss 1360,sackOK,TS val 1322752831 ecr 0,nop,wscale 7], length 0
13:58:53.000520 IP 142.250.199.110.80 > 10.45.0.2.38289: Flags [S.], seq 4149231434, ack 1466640256, win 65535, options [mss 1412,sackOK,TS val 374433140 ecr 1322752831,nop,wscale 8], length 0
13:58:53.001554 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [.], ack 1, win 510, options [nop,nop,TS val 1322752848 ecr 374433140], length 0
13:58:53.001554 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [P.], seq 1:74, ack 1, win 510, options [nop,nop,TS val 1322752849 ecr 374433140], length 73: HTTP: GET / HTTP/1.1
13:58:53.018213 IP 142.250.199.110.80 > 10.45.0.2.38289: Flags [.], ack 74, win 1050, options [nop,nop,TS val 374433158 ecr 1322752849], length 0
13:58:53.111632 IP 142.250.199.110.80 > 10.45.0.2.38289: Flags [P.], seq 1:774, ack 74, win 1050, options [nop,nop,TS val 374433252 ecr 1322752849], length 773: HTTP: HTTP/1.1 301 Moved Permanently
13:58:53.112496 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [.], ack 774, win 504, options [nop,nop,TS val 1322752960 ecr 374433252], length 0
13:58:53.112850 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [F.], seq 74, ack 774, win 504, options [nop,nop,TS val 1322752960 ecr 374433252], length 0
13:58:53.129346 IP 142.250.199.110.80 > 10.45.0.2.38289: Flags [F.], seq 774, ack 75, win 1050, options [nop,nop,TS val 374433269 ecr 1322752960], length 0
13:58:53.130165 IP 10.45.0.2.38289 > 142.250.199.110.80: Flags [.], ack 775, win 504, options [nop,nop,TS val 1322752977 ecr 374433269], length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.
You could now connect to the DN and send any packets on the network using eUPF.

---

Now you could work Open5GS 5GC with eUPF.
I would like to thank the excellent developers and all the contributors of Open5GS, eUPF and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2025.05.04] Updated to eUPF `v0.7.1 (2025.04.16)`, Open5GS `v2.7.5 (2025.04.25)` and UERANSIM `v3.2.7 (2025.04.28)`. Changed the VM environment from Virtualbox to Proxmox VE.
- [2024.03.31] [This commit](https://github.com/open5gs/open5gs/commit/e8a3b76af395a9986234b7d339a7a96dc5bb537f) fixed the issue where SMF crashes without `gtpc` section in `smf.yaml`. So deleted the `gtpc` section in `smf.yaml` for 5G use.
- [2024.03.24] Updated to eUPF v0.6.1.
- [2023.12.05] The eUPF version confirmed to work in the changelog on 2023.12.04 has been tagged as `v0.6.0`.
- [2023.12.05] Updated to Open5GS v2.7.0.
- [2023.12.04] Updated as eUPF FTUP feature has been merged into `main` branch.
- [2023.11.24] Updated to eUPF `120-upf-ftup-fteid` branch that supports FTUP.
- [2023.10.29] Initial release.

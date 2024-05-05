# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration
UERANSIM (5G UE & RAN (gNodeB) implementation) supports IPv4 of PDU Session Type from 2020.11.17 version, and the Data Plane facility has been enabled.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE0 (IMSI-001010000000000)](#changes_ue0)
    - [Changes in configuration files of UE1 (IMSI-001010000000001)](#changes_ue1)
    - [Changes in configuration files of UE2 (IMSI-001010000000002)](#changes_ue2)
    - [Changes in configuration files of UE3 (IMSI-001010000000003)](#changes_ue3)
    - [Changes in configuration files of UE4 (IMSI-001010000000004)](#changes_ue4)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS 5GC U-Plane2](#network_settings_up2)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run UERANSIM](#run_ueran)
    - [Start gNB](#start_gnb)
    - [Start UE (UE0)](#start_ue)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with different DNs for each DNN.

The following minimum configuration was set as a condition.
- C-Plane have multiple U-Planes.
- U-Plane have multiple DNs.
- Multiple UEs connect to same DN.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.0 (2024.03.24) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.112/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.113/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE # | IMSI | DNN | OP/OPc |
| --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OPc |
| UE1 | 001010000000001 | internet2 | OPc |
| UE2 | 001010000000002 | internet2 | OPc |
| UE3 | 001010000000003 | ims | OPc |
| UE4 | 001010000000004 | ims | OPc |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 | U-Plane1 |
| 10.46.0.0/16 | ogstun2 | internet2 | uesimtun1, uesimtun2 | U-Plane1 |
| 10.47.0.0/16 | ogstun3 | ims | uesimtun3, uesimtun4 | U-Plane2 |

Additional information.

Open5GS 5GC U-Plane worked fine on Raspberry Pi 4 Model B. I used [Ubuntu 20.04 (64bit) for Raspberry Pi 4](https://ubuntu.com/download/raspberry-pi) as the OS. I think it would be convenient to place a compact U-Plane in the edge environment and use it as an end-point for DN.

In addition, I have not confirmed the communication performance.

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

The following parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ amf.yaml    2024-03-24 20:19:58.000000000 +0900
@@ -19,27 +19,27 @@
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
--- nrf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ nrf.yaml    2024-03-24 20:58:02.000000000 +0900
@@ -10,8 +10,8 @@
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
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf.yaml    2024-03-31 22:47:39.617010062 +0900
@@ -19,35 +19,37 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.0.112
+          dnn: [internet, internet2]
+        - address: 192.168.0.113
+          dnn: ims
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.4
         port: 9090
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
+    - subnet: 10.46.0.1/16
+      dnn: internet2
+    - subnet: 10.47.0.1/16
+      dnn: ims
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

<a id="changes_up1"></a>

### Changes in configuration files of Open5GS 5GC U-Plane1

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-29 18:40:33.139948345 +0900
@@ -10,16 +10,20 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
+      dev: ogstun
+    - subnet: 10.46.0.1/16
+      dnn: internet2
+      dev: ogstun2
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_up2"></a>

### Changes in configuration files of Open5GS 5GC U-Plane2

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-29 18:42:31.755931554 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.113
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.113
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.47.0.1/16
+      dnn: ims
+      dev: ogstun3
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:44.000000000 +0900
+++ open5gs-gnb.yaml    2024-03-29 16:25:09.415183762 +0900
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
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<a id="changes_ue0"></a>

#### Changes in configuration files of UE0 (IMSI-001010000000000)

First, copy `open5gs-ue0.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue0.yaml
```
Next, edit `open5gs-ue0.yaml`.
- `UERANSIM/config/open5gs-ue0.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue0.yaml    2024-03-29 16:27:36.390313563 +0900
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

<a id="changes_ue1"></a>

#### Changes in configuration files of UE1 (IMSI-001010000000001)

First, copy `open5gs-ue1.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue1.yaml
```
Next, edit `open5gs-ue1.yaml`.
- `UERANSIM/config/open5gs-ue1.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue1.yaml    2024-03-29 18:52:49.604939075 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000001'
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
@@ -47,7 +47,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
 
```

<a id="changes_ue2"></a>

#### Changes in configuration files of UE2 (IMSI-001010000000002)

First, copy `open5gs-ue2.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue2.yaml
```
Next, edit `open5gs-ue2.yaml`.
- `UERANSIM/config/open5gs-ue2.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue2.yaml    2024-03-29 18:53:06.317952751 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000002'
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
@@ -47,7 +47,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
 
```

<a id="changes_ue3"></a>

#### Changes in configuration files of UE3 (IMSI-001010000000003)

First, copy `open5gs-ue3.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue3.yaml
```
Next, edit `open5gs-ue3.yaml`.
- `UERANSIM/config/open5gs-ue3.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue3.yaml    2024-03-29 18:53:41.885991489 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000003'
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
@@ -47,7 +47,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
 
```

<a id="changes_ue4"></a>

#### Changes in configuration files of UE4 (IMSI-001010000000004)

First, copy `open5gs-ue4.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue4.yaml
```
Next, edit `open5gs-ue4.yaml`.
- `UERANSIM/config/open5gs-ue4.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue4.yaml    2024-03-29 18:54:02.414019541 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000004'
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
@@ -47,7 +47,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
 
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC and UERANSIM UE / RAN

<a id="network_settings_up1"></a>

### Network settings of Open5GS 5GC U-Plane1

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE

ip tuntap add name ogstun2 mode tun
ip addr add 10.46.0.1/16 dev ogstun2
ip link set ogstun2 up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun2 -j MASQUERADE
```

<a id="network_settings_up2"></a>

### Network settings of Open5GS 5GC U-Plane2

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun3 mode tun
ip addr add 10.47.0.1/16 dev ogstun3
ip link set ogstun3 up

iptables -t nat -A POSTROUTING -s 10.47.0.0/16 ! -o ogstun3 -j MASQUERADE
```

<a id="build"></a>

## Build Open5GS and UERANSIM

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<a id="run"></a>

## Run Open5GS 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
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

<a id="run_up"></a>

### Run Open5GS 5GC U-Plane1 & U-Plane2

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane1
```
./install/bin/open5gs-upfd &
```
- Open5GS 5GC U-Plane2
```
./install/bin/open5gs-upfd &
```

<a id="run_ueran"></a>

### Run UERANSIM

Here, the case of UE0 (IMSI-001010000000000) & RAN is described.
First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="start_gnb"></a>

#### Start gNB

Start gNB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2024-03-29 19:14:15.643] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2024-03-29 19:14:15.656] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2024-03-29 19:14:15.657] [sctp] [debug] SCTP association setup ascId[20]
[2024-03-29 19:14:15.657] [ngap] [debug] Sending NG Setup Request
[2024-03-29 19:14:15.675] [ngap] [debug] NG Setup Response received
[2024-03-29 19:14:15.675] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/29 19:14:15.672: [amf] INFO: gNB-N2 accepted[192.168.0.131]:42377 in ng-path module (../src/amf/ngap-sctp.c:113)
03/29 19:14:15.672: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:754)
03/29 19:14:15.688: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1236)
03/29 19:14:15.689: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:793)
```

<a id="start_ue"></a>

#### Start UE (UE0)

Start UE (UE0) as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml 
UERANSIM v3.2.6
[2024-03-29 19:15:04.380] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-29 19:15:04.381] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-29 19:15:04.382] [nas] [info] Selected plmn[001/01]
[2024-03-29 19:15:04.383] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-29 19:15:04.384] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-29 19:15:04.384] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-29 19:15:04.385] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-29 19:15:04.389] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 19:15:04.390] [nas] [debug] Sending Initial Registration
[2024-03-29 19:15:04.390] [rrc] [debug] Sending RRC Setup Request
[2024-03-29 19:15:04.391] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-29 19:15:04.393] [rrc] [info] RRC connection established
[2024-03-29 19:15:04.393] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-29 19:15:04.394] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-29 19:15:04.422] [nas] [debug] Authentication Request received
[2024-03-29 19:15:04.422] [nas] [debug] Received SQN [000000000341]
[2024-03-29 19:15:04.423] [nas] [debug] SQN-MS [000000000000]
[2024-03-29 19:15:04.440] [nas] [debug] Security Mode Command received
[2024-03-29 19:15:04.440] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-29 19:15:04.485] [nas] [debug] Registration accept received
[2024-03-29 19:15:04.485] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-29 19:15:04.485] [nas] [debug] Sending Registration Complete
[2024-03-29 19:15:04.486] [nas] [info] Initial Registration is successful
[2024-03-29 19:15:04.486] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-29 19:15:04.487] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 19:15:04.693] [nas] [debug] Configuration Update Command received
[2024-03-29 19:15:04.755] [nas] [debug] PDU Session Establishment Accept received
[2024-03-29 19:15:04.761] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-29 19:15:04.809] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/29 19:15:04.397: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/29 19:15:04.397: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2656)
03/29 19:15:04.397: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/29 19:15:04.398: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1840)
03/29 19:15:04.398: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1621)
03/29 19:15:04.398: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/29 19:15:04.398: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/29 19:15:04.405: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.406: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.406: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.407: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.408: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.408: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.418: [sbi] INFO: [UDM] (SCP-discover) NF registered [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/path.c:211)
03/29 19:15:04.477: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [1071b454-edb5-41ee-acea-637fa974d3e9:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.478: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.478: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.478: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [1071b454-edb5-41ee-acea-637fa974d3e9:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.481: [sbi] INFO: [UDR] (SCP-discover) NF registered [1071b454-edb5-41ee-acea-637fa974d3e9:1] (../lib/sbi/path.c:211)
03/29 19:15:04.691: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/29 19:15:04.692: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/29 19:15:04.693: [gmm] INFO:     UTC [2024-03-29T10:15:04] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/29 19:15:04.693: [gmm] INFO:     LOCAL [2024-03-29T19:15:04] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/29 19:15:04.695: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/29 19:15:04.695: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/29 19:15:04.696: [gmm] INFO: SMF Instance [107d70f0-edb5-41ee-8d98-11ef17677feb] (../src/amf/gmm-handler.c:1324)
03/29 19:15:04.700: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/29 19:15:04.701: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/29 19:15:04.706: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.707: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.708: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.709: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.709: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.709: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.719: [sbi] INFO: [UDM] (SCP-discover) NF registered [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/path.c:211)
03/29 19:15:04.724: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [1071741c-edb5-41ee-9bbc-47729d50ab25:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.725: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.726: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.726: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.727: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.727: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [1071741c-edb5-41ee-9bbc-47729d50ab25:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.732: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [1071b454-edb5-41ee-acea-637fa974d3e9:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.733: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.733: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.734: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [1071b454-edb5-41ee-acea-637fa974d3e9:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.737: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [1071b454-edb5-41ee-acea-637fa974d3e9:2] (../lib/sbi/path.c:216)
03/29 19:15:04.740: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [106ac766-edb5-41ee-ac23-695413178af1:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.741: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.741: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.741: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [106ac766-edb5-41ee-ac23-695413178af1:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.744: [sbi] INFO: [BSF] (SCP-discover) NF registered [106ac766-edb5-41ee-ac23-695413178af1:1] (../lib/sbi/path.c:211)
03/29 19:15:04.747: [sbi] INFO: [PCF] (SCP-discover) NF registered [1071741c-edb5-41ee-9bbc-47729d50ab25:1] (../lib/sbi/path.c:211)
03/29 19:15:04.748: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/29 19:15:04.750: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
03/29 19:15:04.762: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 19:15:04.763: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 19:15:04.763: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.764: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.764: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 19:15:04.764: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [106a8792-edb5-41ee-901e-57e68da538eb:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 19:15:04.768: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [106a8792-edb5-41ee-901e-57e68da538eb:2] (../lib/sbi/path.c:216)
03/29 19:15:04.770: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane1 log when executed is as follows.
```
03/29 19:15:04.743: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/29 19:15:04.744: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
03/29 19:15:04.744: [upf] INFO: UE F-SEID[UP:0x57e CP:0x6bd] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/29 19:15:04.744: [upf] INFO: UE F-SEID[UP:0x57e CP:0x6bd] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/29 19:15:04.754: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2024-03-29 19:15:04.809] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
21: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::3d90:4d95:c73e:89da/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping"></a>

## Ping google.com

Specify the UE0's TUNnel interface and try ping.

Please refer to the following for usage of TUNnel interface.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="ping_1"></a>

### Case for going through DN 10.45.0.0/16

Execute `tcpdump` on VM2 (U-Plane1) and check that the packet goes through `if=ogstun`.
- `ping google.com` on VM5 (UE0)
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.26.238) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.26.238: icmp_seq=1 ttl=61 time=24.8 ms
64 bytes from 172.217.26.238: icmp_seq=2 ttl=61 time=29.5 ms
64 bytes from 172.217.26.238: icmp_seq=3 ttl=61 time=20.8 ms
```
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
19:17:14.619358 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 13, seq 1, length 64
19:17:14.641298 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 13, seq 1, length 64
19:17:15.620458 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 13, seq 2, length 64
19:17:15.647406 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 13, seq 2, length 64
19:17:16.621402 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 13, seq 3, length 64
19:17:16.639589 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 13, seq 3, length 64
```

You could specify the IP address assigned to the TUNnel interface to run almost any applications (iperf3 etc.) as in the following example using `nr-binder` tool.

- `curl google.com` on VM5 (UE0)
```
# sh nr-binder 10.45.0.2 curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM2 (U-Plane1)
```
19:18:10.546698 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [S], seq 3582922701, win 65280, options [mss 1360,sackOK,TS val 3395679140 ecr 0,nop,wscale 7], length 0
19:18:10.563086 IP 172.217.26.238.80 > 10.45.0.2.44451: Flags [S.], seq 13760001, ack 3582922702, win 65535, options [mss 1460], length 0
19:18:10.565440 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [.], ack 1, win 65280, length 0
19:18:10.565708 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [P.], seq 1:75, ack 1, win 65280, length 74: HTTP: GET / HTTP/1.1
19:18:10.565876 IP 172.217.26.238.80 > 10.45.0.2.44451: Flags [.], ack 75, win 65535, length 0
19:18:10.626697 IP 172.217.26.238.80 > 10.45.0.2.44451: Flags [P.], seq 1:774, ack 75, win 65535, length 773: HTTP: HTTP/1.1 301 Moved Permanently
19:18:10.629617 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [.], ack 774, win 64507, length 0
19:18:10.632064 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [F.], seq 75, ack 774, win 64507, length 0
19:18:10.632358 IP 172.217.26.238.80 > 10.45.0.2.44451: Flags [.], ack 76, win 65535, length 0
19:18:10.649379 IP 172.217.26.238.80 > 10.45.0.2.44451: Flags [F.], seq 774, ack 76, win 65535, length 0
19:18:10.651794 IP 10.45.0.2.44451 > 172.217.26.238.80: Flags [.], ack 775, win 64507, length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.

For `UE1`-`UE4` as well, execute `tcpdump` on each U-Plane and check the packets flowing through `ogstunX`.

You could now create the end-to-end TUN interfaces on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.31] [This commit](https://github.com/open5gs/open5gs/commit/e8a3b76af395a9986234b7d339a7a96dc5bb537f) fixed the issue where SMF crashes without `gtpc` section in `smf.yaml`. So deleted the `gtpc` section in `smf.yaml` for 5G use.
- [2024.03.29] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.01.12] Updated to Open5GS v2.5.6.
- [2022.06.05] Updated to Open5GS v2.4.7 and UERANSIM v3.2.6.
- [2021.08.29] Updated to Open5GS v2.3.3 and added a little about BSF. And updated to UERANSIM v3.2.3.
- [2021.03.09] Updated to Open5GS v2.2.0 and added a little about NSSF. And updated to UERANSIM v3.1.3.
- [2021.01.28] Updated to UERANSIM v3.0.1 and updated the operation procedure.
- [2020.12.23] Updated to UERANSIM v2.2.1 and updated the operation procedure.
- [2020.12.20] Updated to UERANSIM v2.1.1.
- [2020.12.14] Updated to UERANSIM v2.0.1 and updated the operation procedure.
- [2020.12.13] Updated to Open5GS v2.1.0 and added a little about PCF.
- [2020.11.29] Initial release.

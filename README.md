# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration
UERANSIM (5G UE & RAN (gNodeB) implementation) supports IPv4 of PDU Session Type from 2020.11.17 version, and the Data Plane facility has been enabled.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files.

---

<h2 id="toc">Table of Contents</h2>

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
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with different DNs for each DNN.

The following minimum configuration was set as a condition.
- C-Plane have multiple U-Planes.
- U-Plane have multiple DNs.
- Multiple UEs connect to same DN.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.0.22 or later (v2.2.0 used) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v1.0.9 or later (v3.1.3 used) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.112/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.113/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 20.04 | 1GB | 10GB |

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

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.0.22 or later (v2.2.0 used) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v1.0.9 or later (v3.1.3 used) - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

The following parameters including DNN can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time. Please refer to [here](https://github.com/open5gs/open5gs/pull/560#issue-483001043) for the logic to select UPF.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2021-03-08 14:42:42.060073927 +0000
+++ amf.yaml    2021-03-08 15:01:23.481611115 +0000
@@ -169,25 +169,26 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     guami:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
+            sd: 1
     security:
         integrity_order : [ NIA2, NIA1, NIA0 ]
         ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2021-03-08 14:42:42.038073647 +0000
+++ smf.yaml    2021-03-08 15:07:04.084034999 +0000
@@ -298,11 +298,15 @@
       - addr: 127.0.0.4
       - addr: ::1
     pfcp:
-      - addr: 127.0.0.4
+      - addr: 192.168.0.111
       - addr: ::1
     subnet:
       - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+        dnn: internet
+      - addr: 10.46.0.1/16
+        dnn: internet2
+      - addr: 10.47.0.1/16
+        dnn: ims
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -398,7 +402,10 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
+        dnn: [internet, internet2]
+      - addr: 192.168.0.113
+        dnn: ims
 
 #
 # parameter:
```

<h3 id="changes_up1">Changes in configuration files of Open5GS 5GC U-Plane1</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2021-03-08 14:50:24.998625885 +0000
+++ upf.yaml    2021-03-08 15:36:49.312532254 +0000
@@ -139,12 +139,16 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     subnet:
       - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+        dnn: internet
+        dev: ogstun
+      - addr: 10.46.0.1/16
+        dnn: internet2
+        dev: ogstun2
 
 #
 # smf:
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2021-03-08 14:56:07.466518288 +0000
+++ upf.yaml    2021-03-08 15:35:40.917727488 +0000
@@ -139,12 +139,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+      - addr: 10.47.0.1/16
+        dnn: ims
+        dev: ogstun3
 
 #
 # smf:
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2021-02-11 11:03:28.000000000 +0000
+++ open5gs-gnb.yaml    2021-02-23 14:59:10.000000000 +0000
@@ -1,17 +1,17 @@
-mcc: '901'          # Mobile Country Code value
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

<h4 id="changes_ue0">Changes in configuration files of UE0 (IMSI-001010000000000)</h4>

First, copy `open5gs-ue0.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue0.yaml
```
Next, edit `open5gs-ue0.yaml`.
- `UERANSIM/config/open5gs-ue0.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-02-28 06:32:38.000000000 +0000
+++ open5gs-ue0.yaml    2021-03-08 14:31:41.086067568 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # Initial PDU sessions to be established
 sessions:
```

<h4 id="changes_ue1">Changes in configuration files of UE1 (IMSI-001010000000001)</h4>

First, copy `open5gs-ue1.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue1.yaml
```
Next, edit `open5gs-ue1.yaml`.
- `UERANSIM/config/open5gs-ue1.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-02-28 06:32:38.000000000 +0000
+++ open5gs-ue1.yaml    2021-03-08 14:32:37.447088979 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000001'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,12 +20,12 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
       sd: 1
```

<h4 id="changes_ue2">Changes in configuration files of UE2 (IMSI-001010000000002)</h4>

First, copy `open5gs-ue2.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue2.yaml
```
Next, edit `open5gs-ue2.yaml`.
- `UERANSIM/config/open5gs-ue2.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-02-28 06:32:38.000000000 +0000
+++ open5gs-ue2.yaml    2021-03-08 14:33:21.961521226 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000002'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,12 +20,12 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
       sd: 1
```

<h4 id="changes_ue3">Changes in configuration files of UE3 (IMSI-001010000000003)</h4>

First, copy `open5gs-ue3.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue3.yaml
```
Next, edit `open5gs-ue3.yaml`.
- `UERANSIM/config/open5gs-ue3.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-02-28 06:32:38.000000000 +0000
+++ open5gs-ue3.yaml    2021-03-08 14:34:23.795294255 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000003'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,12 +20,12 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
       sd: 1
```

<h4 id="changes_ue4">Changes in configuration files of UE4 (IMSI-001010000000004)</h4>

First, copy `open5gs-ue4.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue4.yaml
```
Next, edit `open5gs-ue4.yaml`.
- `UERANSIM/config/open5gs-ue4.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-02-28 06:32:38.000000000 +0000
+++ open5gs-ue4.yaml    2021-03-08 14:35:01.301258088 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000004'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,12 +20,12 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
       sd: 1
```

<h2 id="network_settings">Network settings of Open5GS 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_up1">Network settings of Open5GS 5GC U-Plane1</h3>

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
ip addr add cafe::1/64 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE

ip tuntap add name ogstun2 mode tun
ip addr add 10.46.0.1/16 dev ogstun2
ip addr add cafe::2/64 dev ogstun2
ip link set ogstun2 up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun2 -j MASQUERADE
```

<h3 id="network_settings_up2">Network settings of Open5GS 5GC U-Plane2</h3>

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
ip addr add cafe::3/64 dev ogstun3
ip link set ogstun3 up

iptables -t nat -A POSTROUTING -s 10.47.0.0/16 ! -o ogstun3 -j MASQUERADE
```

<h2 id="build">Build Open5GS and UERANSIM</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.0.22 or later (v2.2.0 used) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v1.0.9 or later (v3.1.3 used) - https://github.com/aligungr/UERANSIM/wiki/Installation

Note. Install MongoDB with package manager on Open5GS 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
```
# apt update
# apt install mongodb
# systemctl start mongodb
# systemctl enable mongodb
```
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 5
./install/bin/open5gs-smfd &
./install/bin/open5gs-amfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
```
Additional information.

PCF was added in Open5GS v2.1.0 released on 2020.12.11. Since PCF connects only to SBI, `pcf.yaml` is used as it is in this configuration example.
And NSSF was added in Open5GS v2.2.0 released on 2021.03.08. Since NSSF also connects only to SBI, `nssf.yaml` is used as it is in this configuration example.

<h3 id="run_up">Run Open5GS 5GC U-Plane1 & U-Plane2</h3>

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane1
```
./install/bin/open5gs-upfd &
```
- Open5GS 5GC U-Plane2
```
./install/bin/open5gs-upfd &
```

<h3 id="run_ueran">Run UERANSIM</h3>

Here, the case of UE0 (IMSI-001010000000000) & RAN is described.
First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h4 id="start_gnb">Start gNB</h4>

Start gNB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.1.3
[2021-03-08 16:25:26.103] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2021-03-08 16:25:26.107] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2021-03-08 16:25:26.107] [sctp] [debug] SCTP association setup ascId[5]
[2021-03-08 16:25:26.107] [ngap] [debug] Sending NG Setup Request
[2021-03-08 16:25:26.108] [ngap] [debug] NG Setup Response received
[2021-03-08 16:25:26.108] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/08 16:25:26.106: [amf] INFO: gNB-S1 accepted[192.168.0.131]:35930 in ng-path module (../src/amf/ngap-sctp.c:107)
03/08 16:25:26.106: [amf] INFO: gNB-N1 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:592)
03/08 16:25:26.106: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:865)
```

<h4 id="start_ue">Start UE (UE0)</h4>

Start UE (UE0) as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml 
UERANSIM v3.1.3
[2021-03-08 16:26:56.206] [nas] [debug] NAS layer started
[2021-03-08 16:26:56.206] [rrc] [debug] RRC layer started
[2021-03-08 16:26:56.207] [nas] [info] UE switches to state: MM-DEREGISTERED/PLMN-SEARCH
[2021-03-08 16:26:56.208] [nas] [info] UE connected to gNB
[2021-03-08 16:26:56.208] [nas] [info] UE switches to state: MM-DEREGISTERED/NORMAL-SERVICE
[2021-03-08 16:26:56.208] [nas] [debug] Sending Initial Registration
[2021-03-08 16:26:56.209] [nas] [info] UE switches to state: MM-REGISTERED-INITIATED/NA
[2021-03-08 16:26:56.209] [rrc] [debug] Sending RRC Setup Request
[2021-03-08 16:26:56.210] [rrc] [info] RRC connection established
[2021-03-08 16:26:56.210] [nas] [info] UE switches to state: CM-CONNECTED
[2021-03-08 16:26:56.221] [nas] [debug] Security Mode Command received
[2021-03-08 16:26:56.221] [nas] [debug] Derived kNasEnc[3A8D65964FA2676984494AB12D071169] kNasInt[4ED0D87052564AE91A80BD1D4D9F2889]
[2021-03-08 16:26:56.222] [nas] [debug] Selected integrity[2] ciphering[0]
[2021-03-08 16:26:56.236] [nas] [debug] Registration accept received
[2021-03-08 16:26:56.236] [nas] [info] UE switches to state: MM-REGISTERED/NORMAL-SERVICE
[2021-03-08 16:26:56.236] [nas] [info] Initial Registration is successful
[2021-03-08 16:26:56.237] [nas] [info] Initial PDU sessions are establishing [1#]
[2021-03-08 16:26:56.237] [nas] [debug] Sending PDU session establishment request
[2021-03-08 16:26:56.625] [nas] [info] PDU Session establishment is successful PSI[1]
[2021-03-08 16:26:56.646] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/08 16:26:56.207: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:349)
03/08 16:26:56.207: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:1834)
03/08 16:26:56.207: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:480)
03/08 16:26:56.207: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1305)
03/08 16:26:56.207: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1122)
03/08 16:26:56.207: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:131)
03/08 16:26:56.207: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:72)
03/08 16:26:56.207: [app] WARNING: Try to discover [AUSF] (../lib/sbi/path.c:108)
03/08 16:26:56.207: [amf] INFO: [db15559a-802a-41eb-ae85-1f753cb18135] (NF-discover) NF registered (../src/amf/nnrf-handler.c:332)
03/08 16:26:56.208: [amf] INFO: [db15559a-802a-41eb-ae85-1f753cb18135] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:391)
03/08 16:26:56.218: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:108)
03/08 16:26:56.220: [amf] INFO: [db159654-802a-41eb-9f76-090b823c358d] (NF-discover) NF registered (../src/amf/nnrf-handler.c:332)
03/08 16:26:56.220: [amf] INFO: [db159654-802a-41eb-9f76-090b823c358d] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:391)
03/08 16:26:56.224: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:108)
03/08 16:26:56.225: [amf] INFO: [db17b588-802a-41eb-81f0-53362eff3f45] (NF-discover) NF registered (../src/amf/nnrf-handler.c:332)
03/08 16:26:56.226: [amf] INFO: [db17b588-802a-41eb-81f0-53362eff3f45] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:391)
03/08 16:26:56.227: [app] WARNING: Try to discover [UDR] (../lib/sbi/path.c:108)
03/08 16:26:56.228: [pcf] INFO: [db16e90a-802a-41eb-92d5-adb2599cb2db] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:280)
03/08 16:26:56.228: [pcf] INFO: [db16e90a-802a-41eb-92d5-adb2599cb2db] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:339)
03/08 16:26:56.433: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:969)
03/08 16:26:56.434: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:349)
03/08 16:26:56.435: [gmm] INFO:     UTC [2021-03-08T16:26:56] Timezone[0]/DST[0] (../src/amf/gmm-build.c:509)
03/08 16:26:56.435: [gmm] INFO:     LOCAL [2021-03-08T16:26:56] Timezone[0]/DST[0] (../src/amf/gmm-build.c:514)
03/08 16:26:56.436: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:1846)
03/08 16:26:56.437: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:812)
03/08 16:26:56.439: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:899)
03/08 16:26:56.440: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2518)
03/08 16:26:56.441: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:108)
03/08 16:26:56.443: [smf] INFO: [db159654-802a-41eb-9f76-090b823c358d] (NF-discover) NF registered (../src/smf/nnrf-handler.c:278)
03/08 16:26:56.444: [smf] INFO: [db159654-802a-41eb-9f76-090b823c358d] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:323)
03/08 16:26:56.446: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:108)
03/08 16:26:56.447: [smf] INFO: [db17b588-802a-41eb-81f0-53362eff3f45] (NF-discover) NF registered (../src/smf/nnrf-handler.c:278)
03/08 16:26:56.447: [smf] INFO: [db17b588-802a-41eb-81f0-53362eff3f45] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:323)
03/08 16:26:56.448: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:400)
03/08 16:26:56.597: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:108)
03/08 16:26:56.600: [smf] INFO: [db2f2d1c-802a-41eb-bb75-272aaf517570] (NF-discover) NF registered (../src/smf/nnrf-handler.c:278)
03/08 16:26:56.600: [smf] INFO: [db2f2d1c-802a-41eb-bb75-272aaf517570] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:323)
```
The Open5GS U-Plane1 log when executed is as follows.
```
03/08 16:26:56.522: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:448)
03/08 16:26:56.522: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:610)
03/08 16:26:56.609: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:58)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2021-03-08 16:26:56.646] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
16: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::498e:251a:5a70:7495/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h2 id="ping">Ping google.com</h2>

Specify the UE0's TUNnel interface and try ping.

Please refer to the following for usage of TUNnel interface.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h3 id="ping_1">Case for going through DN 10.45.0.0/16</h3>

Execute `tcpdump` on VM2 (U-Plane1) and check that the packet goes through `if=ogstun`.
- `ping google.com` on VM5 (UE0)
```
# ping google.com -I uesimtun0 -n
PING google.com (216.58.197.14) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 216.58.197.14: icmp_seq=1 ttl=61 time=13.6 ms
64 bytes from 216.58.197.14: icmp_seq=2 ttl=61 time=13.3 ms
64 bytes from 216.58.197.14: icmp_seq=3 ttl=61 time=13.5 ms
```
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
16:33:29.049467 IP 10.45.0.2 > 216.58.197.14: ICMP echo request, id 12, seq 1, length 64
16:33:29.061813 IP 216.58.197.14 > 10.45.0.2: ICMP echo reply, id 12, seq 1, length 64
16:33:30.051349 IP 10.45.0.2 > 216.58.197.14: ICMP echo request, id 12, seq 2, length 64
16:33:30.063491 IP 216.58.197.14 > 10.45.0.2: ICMP echo reply, id 12, seq 2, length 64
16:33:31.052332 IP 10.45.0.2 > 216.58.197.14: ICMP echo request, id 12, seq 3, length 64
16:33:31.064416 IP 216.58.197.14 > 10.45.0.2: ICMP echo reply, id 12, seq 3, length 64
```

You could specify the IP address assigned to the TUNnel interface to run almost any applications as in the following example using `nr-binder` tool.

- Run `curl google.com` on VM5 (UE0)
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
16:34:32.578882 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [S], seq 2401622356, win 64240, options [mss 1460,sackOK,TS val 1081719107 ecr 0,nop,wscale 7], length 0
16:34:32.590745 IP 216.58.197.14.80 > 10.45.0.2.52665: Flags [S.], seq 69440001, ack 2401622357, win 65535, options [mss 1460], length 0
16:34:32.592629 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [.], ack 1, win 64240, length 0
16:34:32.593150 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [P.], seq 1:75, ack 1, win 64240, length 74: HTTP: GET / HTTP/1.1
16:34:32.593431 IP 216.58.197.14.80 > 10.45.0.2.52665: Flags [.], ack 75, win 65535, length 0
16:34:32.643434 IP 216.58.197.14.80 > 10.45.0.2.52665: Flags [P.], seq 1:529, ack 75, win 65535, length 528: HTTP: HTTP/1.1 301 Moved Permanently
16:34:32.645872 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [.], ack 529, win 63784, length 0
16:34:32.646590 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [F.], seq 75, ack 529, win 63784, length 0
16:34:32.646901 IP 216.58.197.14.80 > 10.45.0.2.52665: Flags [.], ack 76, win 65535, length 0
16:34:32.658931 IP 216.58.197.14.80 > 10.45.0.2.52665: Flags [F.], seq 529, ack 76, win 65535, length 0
16:34:32.660612 IP 10.45.0.2.52665 > 216.58.197.14.80: Flags [.], ack 530, win 63784, length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.

For `UE1`-`UE4` as well, execute `tcpdump` on each U-Plane and check the packets flowing through `ogstunX`.

You could now create the end-to-end TUN interfaces on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2021.03.09] Updated to Open5GS v2.2.0 and added a little about NSSF. And updated to UERANSIM v3.1.3.
- [2021.01.28] Updated to UERANSIM v3.0.1 and updated the operation procedure.
- [2020.12.23] Updated to UERANSIM v2.2.1 and updated the operation procedure.
- [2020.12.20] Updated to UERANSIM v2.1.1.
- [2020.12.14] Updated to UERANSIM v2.0.1 and updated the operation procedure.
- [2020.12.13] Updated to Open5GS v2.1.0 and added a little about PCF.
- [2020.11.29] Initial release.

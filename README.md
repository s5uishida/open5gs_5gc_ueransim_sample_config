# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration
UERANSIM (5G UE & RAN (gNodeB) simulator) supports IPv4 of PDU Session Type from 2020.11.17 version, and the Data Plane facility has been enabled.
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
    - [Start UERANSIM agent](#start_nr_agent)
    - [Create a new gNB](#create_new_gnb)
    - [Create a new UE (UE0)](#create_new_ue)
    - [Create a new PDU session for UE0](#create_new_pdu_session)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with different DNs for each APN.

The following minimum configuration was set as a condition.
- C-Plane have multiple U-Planes.
- U-Plane have multiple DNs.
- Multiple UEs connect to same DN.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.0.22 or later - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v1.0.9 or later - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.112/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.113/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM UE / RAN | 192.168.0.130/24 | Ubuntu 20.04 | 1GB | 10GB |

Subscriber Information (other information is the same) is as follows.  
| UE # | IMSI | APN | OP/OPc |
| --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OP |
| UE1 | 001010000000001 | internet2 | OP |
| UE2 | 001010000000002 | internet2 | OP |
| UE3 | 001010000000003 | ims | OP |
| UE4 | 001010000000004 | ims | OP |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | TUNnel interface of DN | APN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 | U-Plane1 |
| 10.46.0.0/16 | ogstun2 | internet2 | uesimtun1, uesimtun2 | U-Plane1 |
| 10.47.0.0/16 | ogstun3 | ims | uesimtun3, uesimtun4 | U-Plane2 |

Additional information.

Open5GS 5GC U-Plane worked fine on Raspberry Pi 4 Model B. I used [Ubuntu 20.04 (64bit) for Raspberry Pi 4](https://ubuntu.com/download/raspberry-pi) as the OS. I think it would be convenient to place a compact U-Plane in the edge environment and use it as an end-point for DN.

In addition, I have not confirmed the communication performance.

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.0.22 or later - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v1.0.9 or later - https://github.com/aligungr/UERANSIM/wiki/Installation-and-Usage

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

The following parameters including APN can be used in the logic that selects UPF as the connection destination by PFCP.

- APN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only APN this time. Please refer to [here](https://github.com/open5gs/open5gs/pull/560#issue-483001043) for the logic to select UPF.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2020-11-27 13:44:15.729869224 +0000
+++ amf.yaml    2020-11-24 17:12:56.000000000 +0000
@@ -165,25 +165,26 @@
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
+            sd: 010203
     security:
         integrity_order : [ NIA2, NIA1, NIA0 ]
         ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2020-11-27 13:44:15.697869141 +0000
+++ smf.yaml    2020-11-27 13:55:44.000000000 +0000
@@ -175,11 +175,18 @@
       - addr: 127.0.0.4
       - addr: ::1
     pfcp:
-      - addr: 127.0.0.4
+      - addr: 192.168.0.111
       - addr: ::1
     pdn:
       - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+        apn: internet
+        dev: ogstun
+      - addr: 10.46.0.1/16
+        apn: internet2
+        dev: ogstun2
+      - addr: 10.47.0.1/16
+        apn: ims
+        dev: ogstun3
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -275,7 +282,10 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
+        apn: [internet, internet2]
+      - addr: 192.168.0.113
+        apn: ims
 
 #
 # parameter:
```

<h3 id="changes_up1">Changes in configuration files of Open5GS 5GC U-Plane1</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2020-11-27 13:53:56.993456263 +0000
+++ upf.yaml    2020-11-24 16:44:30.000000000 +0000
@@ -163,12 +163,16 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     pdn:
       - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+        apn: internet
+        dev: ogstun
+      - addr: 10.46.0.1/16
+        apn: internet2
+        dev: ogstun2
 
 #
 # smf:
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2020-11-23 08:20:22.000000000 +0000
+++ upf.yaml    2020-11-24 16:45:48.000000000 +0000
@@ -163,12 +163,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     pdn:
-      - addr: 10.45.0.1/16
-      - addr: cafe::1/64
+      - addr: 10.47.0.1/16
+        apn: ims
+        dev: ogstun3
 
 #
 # smf:
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN</h4>

- `UERANSIM/config/open5gs/gnb.yaml`
```diff
--- gnb.yaml.orig       2020-12-23 12:53:50.837835670 +0000
+++ gnb.yaml    2020-12-23 13:07:57.730567363 +0000
@@ -1,7 +1,7 @@
 # MCC and MNC values of the gNB
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # ID of the gNB inside of its PLMN
 gnbId: 1
@@ -13,14 +13,14 @@
 nci: '0000000100'
 
 # gNB's local IP address for N2 interface.
-ngapIp: 127.0.0.1
+ngapIp: 192.168.0.130
 
 # gNB's local IP address for N3 interface.
-gtpIp: 127.0.0.1
+gtpIp: 192.168.0.130
 
 # List of AMF address information
 amfConfigs:
-  - ngapIp: 127.0.0.5
+  - ngapIp: 192.168.0.111
     ngapPort: 38412
 
 # List of NSSAIs supported by this gNB
```

<h4 id="changes_ue0">Changes in configuration files of UE0 (IMSI-001010000000000)</h4>

First, copy `ue0.yaml` from `ue.yaml`.
```
# cd UERANSIM/config/open5gs
# cp ue.yaml ue0.yaml
```
Next, edit `ue0.yaml`.
- `UERANSIM/config/open5gs/ue0.yaml`
```diff
--- ue.yaml.orig        2020-12-23 12:53:50.837835670 +0000
+++ ue0.yaml    2020-12-23 12:57:29.719399423 +0000
@@ -1,10 +1,10 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000003'
+supi: 'imsi-001010000000000'
 
 # MCC and MNC values of the UE.
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # Permanent subscription key.
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
```

<h4 id="changes_ue1">Changes in configuration files of UE1 (IMSI-001010000000001)</h4>

First, copy `ue1.yaml` from `ue.yaml`.
```
# cd UERANSIM/config/open5gs
# cp ue.yaml ue1.yaml
```
Next, edit `ue1.yaml`.
- `UERANSIM/config/open5gs/ue1.yaml`
```diff
--- ue.yaml.orig        2020-12-23 12:53:50.837835670 +0000
+++ ue1.yaml    2020-12-23 12:58:16.498696487 +0000
@@ -1,10 +1,10 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000003'
+supi: 'imsi-001010000000001'
 
 # MCC and MNC values of the UE.
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # Permanent subscription key.
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -19,7 +19,7 @@
 imei: '356938035643803'
 
 # Domain Network Name (DNN)
-dnn: 'internet'
+dnn: 'internet2'
 
 # List of NSSAIs for the UE
 requestedNssai:
```

<h4 id="changes_ue2">Changes in configuration files of UE2 (IMSI-001010000000002)</h4>

First, copy `ue2.yaml` from `ue.yaml`.
```
# cd UERANSIM/config/open5gs
# cp ue.yaml ue2.yaml
```
Next, edit `ue2.yaml`.
- `UERANSIM/config/open5gs/ue2.yaml`
```diff
--- ue.yaml.orig        2020-12-23 12:53:50.837835670 +0000
+++ ue2.yaml    2020-12-23 12:58:46.350828102 +0000
@@ -1,10 +1,10 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000003'
+supi: 'imsi-001010000000002'
 
 # MCC and MNC values of the UE.
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # Permanent subscription key.
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -19,7 +19,7 @@
 imei: '356938035643803'
 
 # Domain Network Name (DNN)
-dnn: 'internet'
+dnn: 'internet2'
 
 # List of NSSAIs for the UE
 requestedNssai:
```

<h4 id="changes_ue3">Changes in configuration files of UE3 (IMSI-001010000000003)</h4>

First, copy `ue3.yaml` from `ue.yaml`.
```
# cd UERANSIM/config/open5gs
# cp ue.yaml ue3.yaml
```
Next, edit `ue3.yaml`.
- `UERANSIM/config/open5gs/ue3.yaml`
```diff
--- ue.yaml.orig        2020-12-23 12:53:50.837835670 +0000
+++ ue3.yaml    2020-12-23 12:59:23.532052903 +0000
@@ -1,10 +1,10 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000003'
+supi: 'imsi-001010000000003'
 
 # MCC and MNC values of the UE.
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # Permanent subscription key.
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -19,7 +19,7 @@
 imei: '356938035643803'
 
 # Domain Network Name (DNN)
-dnn: 'internet'
+dnn: 'ims'
 
 # List of NSSAIs for the UE
 requestedNssai:
```

<h4 id="changes_ue4">Changes in configuration files of UE4 (IMSI-001010000000004)</h4>

First, copy `ue4.yaml` from `ue.yaml`.
```
# cd UERANSIM/config/open5gs
# cp ue.yaml ue4.yaml
```
Next, edit `ue4.yaml`.
- `UERANSIM/config/open5gs/ue4.yaml`
```diff
--- ue.yaml.orig        2020-12-23 12:53:50.837835670 +0000
+++ ue4.yaml    2020-12-23 12:59:55.524785803 +0000
@@ -1,10 +1,10 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 or 16 digits)
-supi: 'imsi-901700000000003'
+supi: 'imsi-001010000000004'
 
 # MCC and MNC values of the UE.
 plmn:
-  mcc: 901
-  mnc: 70
+  mcc: 001
+  mnc: 01
 
 # Permanent subscription key.
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -19,7 +19,7 @@
 imei: '356938035643803'
 
 # Domain Network Name (DNN)
-dnn: 'internet'
+dnn: 'ims'
 
 # List of NSSAIs for the UE
 requestedNssai:
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
- Open5GS v2.0.22 or later - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v1.0.9 or later - https://github.com/aligungr/UERANSIM/wiki/Installation-and-Usage

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

First run the 5GC, then UERANSIM (UE & RAN integrated simulator).

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
```
Additional information.

PCF was added in Open5GS v2.1.0 released on 2020.12.11. Since PCF connects only to SBI, `pcf.yaml` is used as it is in this configuration example.

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
First, do an NG Setup between UE / gNodeB and 5GC, then register UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Installation-and-Usage

<h4 id="start_nr_agent">Start UERANSIM agent</h4>

Start UERANSIM agent as follows. After this, when one gNB and multiple UEs start, but this agent only needs to be started once.
```
# ./nr-agent 
UERANSIM v2.2.1
Selected profile: "open5gs"
2020-12-23 13:09:17.629 [INFO] [SYS] UERANSIM agent has been started.
```

<h4 id="create_new_gnb">Create a new gNB</h4>

Create a new gNB as follows.
```
# ./nr-cli gnb-create -c config/open5gs/gnb.yaml
gNB created with id: 1.
```
The UERANSIM agent log when executed is as follows.
```
2020-12-23 13:09:36.140 [INFO] [CONN] [gnb-1] Trying to establish SCTP connection... (192.168.0.111:38412)
2020-12-23 13:09:36.147 [INFO] [CONN] [gnb-1] SCTP connection established
2020-12-23 13:09:36.483 [SUCC] [PROC] [gnb-1] NGSetup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
12/23 13:09:36.184: [amf] INFO: gNB-S1 accepted[192.168.0.130]:59360 in ng-path module (../src/amf/ngap-sctp.c:107)
12/23 13:09:36.184: [amf] INFO: gNB-N1 accepted[192.168.0.130] in master_sm module (../src/amf/amf-sm.c:538)
12/23 13:09:36.184: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:869)
```

<h4 id="create_new_ue">Create a new UE (UE0)</h4>

Create a new UE (UE0) as follows.
```
# ./nr-cli ue-create -c config/open5gs/ue0.yaml 
UE created: imsi-001010000000000.
```
The UERANSIM agent log when executed is as follows.
```
2020-12-23 13:10:11.377 [INFO] [STATE] [ue-001010000000000] UE switches to state: MM_DEREGISTERED/MM_DEREGISTERED__PLMN_SEARCH
2020-12-23 13:10:11.591 [INFO] [CONN] [ue-001010000000000] UE connected to gNB.
2020-12-23 13:10:11.596 [INFO] [STATE] [ue-001010000000000] UE switches to state: MM_DEREGISTERED/MM_DEREGISTERED__NORMAL_SERVICE
2020-12-23 13:10:11.691 [INFO] [STATE] [ue-001010000000000] UE switches to state: MM_REGISTERED_INITIATED/MM_REGISTERED_INITIATED__NA
2020-12-23 13:10:12.957 [INFO] [STATE] [ue-001010000000000] UE switches to state: RM_REGISTERED
2020-12-23 13:10:12.957 [INFO] [STATE] [ue-001010000000000] UE switches to state: MM_REGISTERED/MM_REGISTERED__NORMAL_SERVICE
2020-12-23 13:10:12.959 [SUCC] [PROC] [ue-001010000000000] Registration is successful
```
The Open5GS C-Plane log when executed is as follows.
```
12/23 13:10:11.880: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:1722)
12/23 13:10:11.899: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1289)
12/23 13:10:11.899: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1121)
12/23 13:10:11.899: [app] WARNING: Try to discover [AUSF] (../lib/sbi/path.c:56)
12/23 13:10:11.900: [amf] INFO: [f3f225f2-451f-41eb-b669-edef1695e8ec] (NF-discover) NF registered (../src/amf/nnrf-handler.c:250)
12/23 13:10:11.900: [amf] INFO: [f3f225f2-451f-41eb-b669-edef1695e8ec] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:294)
12/23 13:10:11.916: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:56)
12/23 13:10:11.916: [ausf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF registered (../src/ausf/nnrf-handler.c:248)
12/23 13:10:11.917: [ausf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF Profile updated (../src/ausf/nnrf-handler.c:292)
12/23 13:10:12.860: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:56)
12/23 13:10:12.860: [amf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF registered (../src/amf/nnrf-handler.c:250)
12/23 13:10:12.860: [amf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:294)
12/23 13:10:12.888: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:56)
12/23 13:10:12.890: [amf] INFO: [f3f1753a-451f-41eb-8dec-ed8b88468d80] (NF-discover) NF registered (../src/amf/nnrf-handler.c:250)
12/23 13:10:12.891: [amf] INFO: [f3f1753a-451f-41eb-8dec-ed8b88468d80] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:294)
```

<h4 id="create_new_pdu_session">Create a new PDU session for UE0</h4>

Create a new PDU session for UE0 as follows.
```
# ./nr-cli session-create 001010000000000
PDU session establishment has been triggered.
```
The UERANSIM agent log when executed is as follows.
```
2020-12-23 13:12:29.046 [SUCC] [PROC] [gnb-1] PDU Session Establishment is successful
2020-12-23 13:12:29.076 [INFO] [UEAPP] [ue-001010000000000] IPv4 connection setup with local IP: 10.45.0.2
2020-12-23 13:12:29.092 [INFO] [FLOW] [ue-001010000000000] PDU session established: PDU session identity value 1
2020-12-23 13:12:29.095 [SUCC] [PROC] [ue-001010000000000] PDU Session Establishment is successful
```
The Open5GS C-Plane log when executed is as follows.
```
12/23 13:12:28.745: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:1734)
12/23 13:12:28.780: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:532)
12/23 13:12:28.780: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:1873)
12/23 13:12:28.838: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:56)
12/23 13:12:28.839: [smf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF registered (../src/smf/nnrf-handler.c:248)
12/23 13:12:28.839: [smf] INFO: [f3f0c3ec-451f-41eb-948f-d1f596d40d7b] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:292)
12/23 13:12:28.842: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:56)
12/23 13:12:28.843: [smf] INFO: [f3f1753a-451f-41eb-8dec-ed8b88468d80] (NF-discover) NF registered (../src/smf/nnrf-handler.c:248)
12/23 13:12:28.843: [smf] INFO: [f3f1753a-451f-41eb-8dec-ed8b88468d80] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:292)
12/23 13:12:28.861: [smf] INFO: UE SUPI:[imsi-001010000000000] DNN:[internet] IPv4:[10.45.0.2] IPv6:[] (../src/smf/npcf-handler.c:131)
12/23 13:12:29.022: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:56)
12/23 13:12:29.023: [smf] INFO: [f3f1b73e-451f-41eb-894a-cde0e4a59926] (NF-discover) NF registered (../src/smf/nnrf-handler.c:248)
12/23 13:12:29.023: [smf] INFO: [f3f1b73e-451f-41eb-894a-cde0e4a59926] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:292)
```
The Open5GS U-Plane1 log when executed is as follows.
```
12/23 13:12:28.899: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:445)
12/23 13:12:28.899: [upf] INFO: UE F-SEID[CP:0x1,UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:603)
12/23 13:12:29.052: [gtp] INFO: gtp_connect() [192.168.0.130]:2152 (../lib/gtp/path.c:58)
```
On UERANSIM agent log, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC. Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
4: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::d84b:d5fd:e5f1:c6c2/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
...
```

<h2 id="ping">Ping google.com</h2>

Specify the UE0's TUNnel interface and try ping.

Please refer to the following for usage of TUNnel interface.

https://github.com/aligungr/UERANSIM/wiki/Using-the-TUN-interface

<h3 id="ping_1">Case for going through DN 10.45.0.0/16</h3>

Execute `tcpdump` on VM2 (U-Plane1) and check that the packet goes through `if=ogstun`.
- `ping google.com` on VM4 (UE0)
```
# ping google.com -I uesimtun0
PING google.com (172.217.161.238) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from kix06s05-in-f14.1e100.net (172.217.161.238): icmp_seq=1 ttl=61 time=54.7 ms
64 bytes from kix06s05-in-f14.1e100.net (172.217.161.238): icmp_seq=2 ttl=61 time=10.7 ms
64 bytes from kix06s05-in-f14.1e100.net (172.217.161.238): icmp_seq=3 ttl=61 time=22.9 ms
```
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
13:14:00.938862 IP 10.45.0.2 > kix06s05-in-f14.1e100.net: ICMP echo request, id 1, seq 1, length 64
13:14:00.982798 IP kix06s05-in-f14.1e100.net > 10.45.0.2: ICMP echo reply, id 1, seq 1, length 64
13:14:01.934917 IP 10.45.0.2 > kix06s05-in-f14.1e100.net: ICMP echo request, id 1, seq 2, length 64
13:14:01.943514 IP kix06s05-in-f14.1e100.net > 10.45.0.2: ICMP echo reply, id 1, seq 2, length 64
13:14:02.936778 IP 10.45.0.2 > kix06s05-in-f14.1e100.net: ICMP echo request, id 1, seq 3, length 64
13:14:02.957501 IP kix06s05-in-f14.1e100.net > 10.45.0.2: ICMP echo reply, id 1, seq 3, length 64
```

You could specify the IP address assigned to the TUNnel interface to run almost any applications as in the following example using `nr-binder` tool.

- Run `curl google.com` on VM4 (UE0)
```
# ./nr-binder 10.45.0.2 curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM2 (U-Plane1)
```
13:20:54.284631 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [S], seq 2144490152, win 64240, options [mss 1460,sackOK,TS val 3785445157 ecr 0,nop,wscale 7], length 0
13:20:54.305650 IP kix06s05-in-f14.1e100.net.http > 10.45.0.2.55315: Flags [S.], seq 2816001, ack 2144490153, win 65535, options [mss 1460], length 0
13:20:54.308665 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [.], ack 1, win 64240, length 0
13:20:54.309295 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [P.], seq 1:75, ack 1, win 64240, length 74: HTTP: GET / HTTP/1.1
13:20:54.309603 IP kix06s05-in-f14.1e100.net.http > 10.45.0.2.55315: Flags [.], ack 75, win 65535, length 0
13:20:54.479191 IP kix06s05-in-f14.1e100.net.http > 10.45.0.2.55315: Flags [P.], seq 1:529, ack 75, win 65535, length 528: HTTP: HTTP/1.1 301 Moved Permanently
13:20:54.488089 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [.], ack 529, win 63784, length 0
13:20:54.488162 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [F.], seq 75, ack 529, win 63784, length 0
13:20:54.488310 IP kix06s05-in-f14.1e100.net.http > 10.45.0.2.55315: Flags [.], ack 76, win 65535, length 0
13:20:54.505975 IP kix06s05-in-f14.1e100.net.http > 10.45.0.2.55315: Flags [F.], seq 529, ack 76, win 65535, length 0
13:20:54.506554 IP 10.45.0.2.55315 > kix06s05-in-f14.1e100.net.http: Flags [.], ack 530, win 63784, length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.

For `UE1`-`UE4` as well, execute `tcpdump` on each U-Plane and check the packets flowing through `ogstunX`.

You could now create the end-to-end TUN interfaces on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2020.12.23] Updated to UERANSIM v2.2.1 and updated the operation procedure.
- [2020.12.20] Updated to UERANSIM v2.1.1.
- [2020.12.14] Updated to UERANSIM v2.0.1 and updated the operation procedure.
- [2020.12.13] Updated to Open5GS v2.1.0 and added a little about PCF.
- [2020.11.29] Initial release.

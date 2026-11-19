# Test catalog

Every test case in Kaldrivon SMO Tester 0.1.0: 288 cases in 49 use cases on 11 interfaces. The id is shown in the app next to each case's title.

Kinds: **request** — the tester sends a request to the SMO and checks the answer; **wait** — the tester waits for the SMO to send something (a notification, a request to the tester's O-DU, O-RU or O-Cloud, a file download); **steps** — the tester does several requests itself (for example finding what the SMO manages); **info** — an explanation, nothing is sent.

## Contents

- [O1](#o1)
- [Open Fronthaul M-Plane](#open-fronthaul-m-plane)
- [3GPP Management Services](#3gpp-management-services)
- [R1](#r1)
- [A1](#a1)
- [O2](#o2)
- [TMF Open APIs](#tmf-open-apis)
- [Intent-driven management](#intent-driven-management)
- [Management data analytics](#management-data-analytics)
- [Interface security](#interface-security)
- [Other interfaces](#other-interfaces)

## O1

The tester acts as an NF sending VES events to the SMO's collector (alarms, heartbeats, measurements, 3GPP notifications) and a file-ready event for a measurement file it serves.

- **Between:** SMO and O-RAN NFs (O-CU, O-DU, Near-RT RIC)
- **The tester plays:** the network function
- **Standards:** VES 7.2.1, TS 28.532 (stndDefined), TS 32.435
- **Defined by:** O-RAN WG10

### O1 › Connection check

Is the SMO reachable, and can the SMO reach the tester?

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o1-conn-01` | The SMO's ports are reachable from this PC | steps | Opens a TCP connection to each SMO port. Fails if the SMO is not listening or a firewall blocks it. |  |  |
| `o1-conn-02` | The tester's own listeners work | steps | Checks the tester's HTTPS listener runs and that its advertised address works from this PC. |  |  |
| `o1-conn-03` | The SMO accepts the tester's HTTPS certificate | steps | The SMO sends notifications and fetches files from the tester over HTTPS, so it has to accept the tester's self-signed certificate. The tester makes the SMO call it (an immediate heartbeat on a subscription of its own, removed afterwards) and watches the TLS handshake: PASS when the request gets through, FAIL when the SMO breaks off with a certificate alert (for example unknown CA). Without it, every case that waits for the SMO to call the tester stays inconclusive. | RFC 8446 (TLS 1.3) §4.4.2, §6.2; RFC 5280; TS 28.623 HeartbeatControl (trigger) |  |

### O1 › VES event collection

The NF sends events to the SMO's VES collector (tester → SMO).

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o1-ves-01` | PNF registration (pnfRegistration) | request | The NF announces itself. The collector must accept it; the tester does not check what the SMO does next. | ONAP VES Event Listener 7.2.1, pnfRegistration domain; O-RAN WG10 O1 Interface §6 (PNF start-up) |  |
| `o1-ves-02` | Fault raised (fault domain, MAJOR) | request | The SMO should show a new alarm SMOTESTER_LINK_DOWN from the tester's source name. | VES 7.2.1 faultFields |  |
| `o1-ves-03` | Fault cleared (eventSeverity NORMAL) | request | The alarm raised by the previous case should clear in the SMO. | VES 7.2.1 faultFields |  |
| `o1-ves-04` | Heartbeat | request | `POST /eventListener/v7` | VES 7.2.1 heartbeatFields |  |
| `o1-ves-05` | Measurement (measurement domain) | request | Synthetic value 42 for the metric on the measured class=1. | VES 7.2.1 measurementFields |  |
| `o1-ves-06` | Notification (notification domain) | request | `POST /eventListener/v7` | VES 7.2.1 notificationFields |  |
| `o1-ves-07` | 3GPP notifyNewAlarm (stndDefined, 3GPP-FaultSupervision) | request | `POST /eventListener/v7` | O-RAN WG10 O1 (stndDefined VES), 3GPP TS 28.532 FaultMnS NotifyNewAlarm, VES 7.2.1 stndDefinedFields |  |
| `o1-ves-08` | 3GPP notifyMOIAttributeValueChanges (stndDefined, 3GPP-Provisioning) | request | `POST /eventListener/v7` | 3GPP TS 28.532 ProvMnS notifications over VES stndDefined |  |
| `o1-ves-09` | 3GPP notifyHeartbeat (stndDefined, 3GPP-Heartbeat) | request | `POST /eventListener/v7` | 3GPP TS 28.623 HeartbeatNtf over VES stndDefined |  |
| `o1-ves-10` | Event batch (eventBatch, two events) | request | `POST /eventListener/v7/eventBatch` | VES 7.2.1 eventBatch resource |  |
| `o1-ves-11` | Refuses an event without commonEventHeader | request | Negative test: the collector must validate events. | VES 7.2.1 (400 Bad Request, requestError) |  |
| `o1-ves-12` | Refuses malformed JSON | request | Negative test. | VES 7.2.1 |  |
| `o1-ves-13` | Refuses an event without credentials | request | Negative test. | VES 7.2.1 security (HTTP Basic or token over TLS) |  |
| `o1-ves-14` | Refuses an event with wrong credentials | request | Negative test. | VES 7.2.1 security |  |

### O1 › File-based PM

The NF tells the SMO a measurement file is ready; the SMO fetches it (both ways).

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o1-file-01` | File ready notification (VES notification, PM_MEAS_FILES) | request | Tells the SMO a measurement file is ready on the tester's HTTPS server (synthetic values). | O-RAN WG10 O1 file-based PM; VES 7.2.1 notificationFields; 3GPP TS 32.435 |  |
| `o1-file-02` | The SMO fetches the measurement file | wait | The file is on the tester's HTTPS server at the address the SMO uses to reach this PC. An SMO usually fetches only from servers it allows for that network function and whose certificate it trusts: if no download arrives, allow this PC and trust the tester's HTTPS certificate in the SMO's file PM settings for the O-DU. | O-RAN WG10 O1 file-based PM (HTTPS file transfer) | Waits up to 300 s; The tester first sends the file-ready event, then waits for the SMO to download the file. |

### O1 › Integration: the SMO manages the tester's O-DU

Add the tester's O-DU to the SMO first (Integration page). Without it these cases are not applicable (NONE).

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o1-int-01` | The SMO connects over NETCONF | steps | Waits until the SMO opens a NETCONF session to the tester's O-DU with the user and password from the Integration page. | O-RAN WG10 O1 §5 (NETCONF); RFC 6241, RFC 6242 |  |
| `o1-int-02` | The SMO reads the O-DU's YANG library | steps | The SMO learns which models the O-DU implements (3GPP TS 28.541 / 28.623 Rel-18). | RFC 8525, RFC 8526 |  |
| `o1-int-03` | The SMO subscribes to the O-DU's notifications | steps | Needed to hear about configuration changes (netconf-config-change, RFC 6470). | RFC 5277, RFC 6470 |  |
| `o1-int-04` | The SMO sets the O-DU's VES destination | steps | The standard way to tell an NF where to send its notifications. From then on the O-DU sends its VES events (faults, heartbeats at the period asked) to that address. | 3GPP TS 28.623 NtfSubscriptionControl, HeartbeatControl; O-RAN WG10 O1 |  |
| `o1-int-05` | The O-DU raises an alarm and the SMO lists it | steps | The O-DU adds the alarm to its AlarmList, tells the SMO over NETCONF (netconf-config-change) and sends a VES fault to the destination the SMO set. The SMO's alarm list must then show it. | VES 7.2.1 faultFields; 3GPP TS 28.532 getAlarmList; _3gpp-common-fm AlarmList |  |
| `o1-int-06` | The O-DU clears its alarm | steps | The O-DU removes the alarm from its AlarmList and sends the clearing VES fault. | VES 7.2.1 faultFields (eventSeverity NORMAL) |  |

## Open Fronthaul M-Plane

The tester acts as an O-RU the SMO manages directly over the Open Fronthaul M-Plane (hybrid model): start-up and call home, supervision, accounts, inventory, U-Plane configuration, synchronisation, faults, measurements, files, software and reset. Add the tester's O-RU to the SMO first (Integration page). Cases marked manual wait for a step you do in the SMO.

- **Between:** SMO (hybrid model) and O-RUs
- **The tester plays:** the O-RU (NETCONF server, O-RAN WG4 M-Plane YANG models)
- **Standards:** O-RAN.WG4.MP (M-Plane); O-RAN WG4 M-Plane YANG; RFC 6241, 6242, 5277, 8071, 8525, 8341
- **Defined by:** O-RAN WG4

### Open Fronthaul M-Plane › Start-up and connection

The SMO connects to the O-RU (or takes its call home) and learns its models.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-su-01` | The SMO opens a NETCONF session to the O-RU | steps | Once the O-RU is added, the SMO opens a NETCONF session over SSH with an account of the O-RU and exchanges hello messages. The M-Plane uses NETCONF 1.1, so the SMO's hello has to announce base:1.1. | O-RAN.WG4.MP (M-Plane) start-up; RFC 6241; RFC 6242 |  |
| `ofh-su-02` | The O-RU calls home and the SMO takes the session | steps | At start-up an O-RU calls its NETCONF clients (NETCONF call home over SSH, RFC 8071, port 4334): it opens the TCP connection, then acts as the SSH server on it. The SMO has to accept the connection, log in and open the NETCONF session. An O-RU is added to the SMO once, either to be connected to or to call home; not applicable while the O-RU is set to be connected to. | O-RAN.WG4.MP (M-Plane) start-up (NETCONF call home); RFC 8071 |  |
| `ofh-su-03` | The SMO reads the O-RU's YANG library | steps | To know which O-RAN WG4 modules, revisions and features the O-RU implements, the SMO reads its YANG library. | RFC 8525 (RFC 7895); RFC 6022 |  |
| `ofh-su-04` | The SMO fetches modules it does not have (get-schema) | steps | An SMO that lacks a module the O-RU lists downloads it with get-schema. Not needed when it already has them. | RFC 6022 get-schema |  |
| `ofh-su-05` | The SMO subscribes to the O-RU's notifications | steps | Alarms, supervision, measurement results, transfer results and state changes reach the SMO only as NETCONF notifications, so the SMO subscribes on its session. | O-RAN.WG4.MP (M-Plane); RFC 5277 |  |

### Open Fronthaul M-Plane › Supervision

The SMO supervises the O-RU with the watchdog.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-sup-01` | The SMO starts supervising the O-RU | steps | In the hybrid model the NETCONF client supervises the O-RU: it calls supervision-watchdog-reset with its notification interval and guard timer, and receives supervision-notification on a subscribed session. | O-RAN.WG4.MP (M-Plane) supervision; o-ran-supervision supervision-watchdog-reset |  |
| `ofh-sup-02` | The SMO keeps the watchdog from expiring | steps | The SMO has to reset the watchdog again before interval + guard timer runs out; otherwise the O-RU treats it as a supervision failure. | O-RAN.WG4.MP (M-Plane) supervision failure; o-ran-supervision |  |

### Open Fronthaul M-Plane › User accounts

The SMO manages the O-RU's accounts.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-um-01` | The SMO creates an account on the O-RU | steps | Checks the new account against o-ran-usermgmt: a password of 8 to 128 characters from the allowed set, with at least two lower case, upper case, digits and special characters. | O-RAN.WG4.MP (M-Plane) user accounts; o-ran-usermgmt | Manual step in the SMO |
| `ofh-um-02` | The SMO changes a password (chg-password) | steps | The SMO changes the password of the account it logged in with. The input must conform to o-ran-usermgmt and the O-RU answers Successful. | O-RAN.WG4.MP (M-Plane); o-ran-usermgmt chg-password | Manual step in the SMO |
| `ofh-um-03` | The SMO removes the account | steps | Removes the account made in the first case, so the O-RU keeps only its own. | o-ran-usermgmt; RFC 8040 DELETE | Manual step in the SMO |

### Open Fronthaul M-Plane › Inventory and capabilities

The SMO reads what the O-RU is and can do.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-inv-01` | The SMO reads the hardware inventory | steps | The O-RU's components (module, port, antenna connectors) with product code, serial number and o-ran-name. | O-RAN.WG4.MP (M-Plane) inventory; ietf-hardware, o-ran-hardware |  |
| `ofh-inv-02` | The SMO reads the interfaces | steps | The fronthaul Ethernet interface with its MAC address, port reference and IP address. | ietf-interfaces, o-ran-interfaces, ietf-ip |  |
| `ofh-inv-03` | The SMO reads the O-RU's capabilities | steps | Category, number of ports, compression and the band capabilities (NR band n78). | o-ran-module-cap |  |
| `ofh-inv-04` | The SMO reads the U-Plane capabilities | steps | Transmit and receive arrays, static low-level endpoints and endpoint types: what the SMO configures carriers and endpoints on. | o-ran-uplane-conf |  |
| `ofh-inv-05` | The SMO reads the software inventory | steps | The software slots with their state (valid, active, running). | o-ran-software-management |  |
| `ofh-inv-06` | The SMO reads operational information | steps | Declarations (supported M-Plane version, header mechanism), the operational state and restart cause. | o-ran-operations |  |

### Open Fronthaul M-Plane › U-Plane configuration

The SMO configures transport, carriers and endpoints.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-up-01` | The SMO configures a processing element | steps | The transport flow the O-RU uses for C/U-Plane: an interface of the O-RU, its MAC as ru-mac-address, a VLAN and the O-DU's MAC. | O-RAN.WG4.MP (M-Plane) U-Plane transport; o-ran-processing-element | Manual step in the SMO |
| `ofh-up-02` | The SMO creates a transmit carrier | steps | The carrier must fit the band and bandwidth the O-RU declares (NR band n78, 3300–3800 MHz, up to 100 MHz). | O-RAN.WG4.MP (M-Plane) U-Plane configuration; o-ran-uplane-conf tx-array-carriers | Manual step in the SMO |
| `ofh-up-03` | The SMO creates a receive carrier | steps | The same checks for the receive direction. | O-RAN.WG4.MP (M-Plane) U-Plane configuration; o-ran-uplane-conf rx-array-carriers | Manual step in the SMO |
| `ofh-up-04` | The SMO links endpoints, carriers and processing elements | steps | Low-level endpoints take the names of the O-RU's static endpoints; each link joins an endpoint, a carrier and a processing element that exist. | O-RAN.WG4.MP (M-Plane) U-Plane configuration; o-ran-uplane-conf low-level endpoints and links | Manual step in the SMO |
| `ofh-up-05` | The O-RU reports the carrier state to the SMO | steps | When the SMO activates or deactivates a carrier, the O-RU changes its state (READY / DISABLED) and notifies it (tx-array-carriers-state-change). | o-ran-uplane-conf tx/rx-array-carriers-state-change |  |

### Open Fronthaul M-Plane › Synchronisation and delay

Sync state and delay profile.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-sy-01` | The SMO reads the synchronisation state | steps | Sync state, PTP configuration and status (G.8275.1, clock class) of the O-RU. | O-RAN.WG4.MP (M-Plane) synchronisation; o-ran-sync |  |
| `ofh-sy-02` | The SMO gets the O-RU's synchronisation change | steps | The O-RU loses lock (FREERUN) and regains it (LOCKED), notifying both; the SMO's session must receive them and stay open. | o-ran-sync synchronization-state-change |  |
| `ofh-dl-01` | The SMO reads the O-RU's delay profile | steps | The O-RU's transmission and reception windows (t2a, ta3, tcp-adv) the O-DU needs to place its timing. | O-RAN.WG4.MP (M-Plane) delay management; o-ran-delay-management |  |

### Open Fronthaul M-Plane › Fault management

The O-RU's alarms reach the SMO.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-fm-01` | The O-RU raises an alarm and the SMO shows it | steps | The O-RU raises fault 1 (unit temperature high, synthetic) with alarm-notif. The SMO should list it; the tester looks for it in the SMO's 3GPP alarm list (advisory: an SMO may show O-RU alarms only in its own views). | O-RAN.WG4.MP (M-Plane) fault management; o-ran-fm alarm-notif |  |
| `ofh-fm-02` | The SMO reads the O-RU's active alarm list | steps | After connecting (or a notification), an SMO reads the active alarms to be in step with the O-RU. | o-ran-fm active-alarm-list |  |
| `ofh-fm-03` | The O-RU clears the alarm and the SMO clears it too | steps | The O-RU clears the fault (alarm-notif with is-cleared true); it should leave the SMO's active alarms. | o-ran-fm alarm-notif (is-cleared) |  |

### Open Fronthaul M-Plane › Performance management

The SMO activates measurements and receives results and files.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-pm-01` | The SMO activates measurements | steps | Only measurements the O-RU lists as capable are activated; a measurement interval and a notification interval are noted when they are missing. | O-RAN.WG4.MP (M-Plane) performance management; o-ran-performance-management | Manual step in the SMO |
| `ofh-pm-02` | The O-RU reports measurement results | steps | At each notification interval the O-RU sends measurement-result-stats (synthetic counts); the SMO's session must receive them and stay open. | o-ran-performance-management measurement-result-stats |  |
| `ofh-pm-03` | The SMO sets up measurement file upload | steps | Where the O-RU puts its measurement files: an sftp:// address with credentials, file upload on and an interval. | o-ran-performance-management remote-file-uploads; o-ran-file-management | Manual step in the SMO |

### Open Fronthaul M-Plane › Files, troubleshooting and trace

The SMO retrieves files and logs from the O-RU.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-fl-01` | The SMO lists the O-RU's files | steps | A logical path the O-RU supports (o-ran/log, o-ran/pm, …) and the answer with the file list. | o-ran-file-management retrieve-file-list | Manual step in the SMO |
| `ofh-fl-02` | The SMO collects troubleshooting logs | steps | The SMO starts and stops troubleshooting log collection; the O-RU then reports the log file. | o-ran-troubleshooting start/stop-troubleshooting-logs, troubleshooting-log-generated | Manual step in the SMO |
| `ofh-fl-03` | The SMO has the O-RU upload a file to it | steps | The O-RU pushes the file to the SMO's SFTP server with the credentials in the request; the transfer must work. | o-ran-file-management file-upload, file-upload-notification | Manual step in the SMO |
| `ofh-fl-04` | The SMO collects trace logs | steps | The SMO starts and stops trace collection; the O-RU reports the trace file. | o-ran-trace start/stop-trace-logs, trace-log-generated | Manual step in the SMO |

### Open Fronthaul M-Plane › Software management

Download, install, activate, reset.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-sw-01` | The SMO has the O-RU download software | steps | The O-RU fetches the file from the SMO's SFTP server; the URI, the credentials and the transfer are checked. Any file works: the O-RU only stores it (synthetic software). | O-RAN.WG4.MP (M-Plane) software management; o-ran-software-management software-download, download-event | Manual step in the SMO |
| `ofh-sw-02` | The SMO installs it into the inactive slot | steps | Installation goes into a slot that is not active or running (swSlot2), with the downloaded file. | o-ran-software-management software-install, install-event | Manual step in the SMO |
| `ofh-sw-03` | The SMO activates the new software | steps | The slot becomes the active one; it runs after the next reset. | o-ran-software-management software-activate, activation-event | Manual step in the SMO |
| `ofh-sw-04` | The SMO resets the O-RU onto the new software | steps | After the reset the O-RU comes back on the activated slot; the SMO has to connect again and find it running. | o-ran-operations reset; o-ran-software-management | Manual step in the SMO |

### Open Fronthaul M-Plane › Operations

Reset of the O-RU.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `ofh-op-01` | The SMO resets the O-RU and connects again | steps | The O-RU ends its sessions, restarts within a few seconds (restart-cause MPLANE-TRIGGERED-RESTART) and the SMO connects again (or takes the O-RU's call home). | O-RAN.WG4.MP (M-Plane) O-RU reset; o-ran-operations reset | Manual step in the SMO |

## 3GPP Management Services

The tester acts as the OSS: reads and changes managed objects, handles alarms, sets up measurement jobs and receives notifications at its callback URLs.

- **Between:** Operator OSS and the SMO
- **The tester plays:** the OSS (MnS consumer)
- **Standards:** TS 28.532, TS 28.623, TS 28.550, TS 32.158, TS 32.435, TS 32.422
- **Defined by:** 3GPP SA5

### 3GPP Management Services › Find what the SMO manages

Reads the DNs and alarms the other cases use from the SMO itself, so no made-up names are sent. Runs by itself when a case needs one of these values.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-disc-01` | Find a managed element the SMO has | steps | Reads the SMO's alarm list and resource inventory for managed element DNs and keeps the first one the SMO lets the tester read (the tester's own O-DU first, when it is integrated). The other cases use it (the O-DU's managed element) instead of a made-up name. Runs by itself when a case needs it. | TS 28.532 getAlarmList, getMOIAttributes; TS 32.300 DN |  |
| `mns-disc-02` | Find an object below it to change | steps | Reads the managed element's subtree and keeps an NR cell (or the first object) for the change cases (the O-DU's cell, attribute the attribute). | TS 28.532 getMOIAttributes (BASE_SUBTREE) |  |
| `mns-disc-03` | Find an alarm to work with | steps | With the tester's O-DU integrated, the O-DU raises its own alarm and the tester waits until the SMO lists it: acknowledging and clearing it then changes nothing else. Otherwise the first alarm in the list is used. | TS 28.532 getAlarmList |  |

### 3GPP Management Services › Fault Supervision

Alarm list, acknowledge, clear, comment, subscribe.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-fm-01` | Read the alarm list | request | Reads the whole alarm list. The alarm the next cases act on is found by 'Find an alarm to work with'. | TS 28.532 §11.2.1.1 (getAlarmList), TS28532_FaultMnS.yaml GET /alarms |  |
| `mns-fm-02` | Filter: unacknowledged alarms only | request | `GET {fm}/alarms` | TS 28.532 getAlarmList, alarmAckState filter |  |
| `mns-fm-03` | Filter: alarms below one object (baseObjectInstance) | request | `GET {fm}/alarms` | TS 28.532 getAlarmList, baseObjectInstance |  |
| `mns-fm-04` | Alarm count | request | `GET {fm}/alarms/alarmCount` | TS 28.532 getAlarmCount |  |
| `mns-fm-05` | Acknowledge one alarm | request | `PATCH {fm}/alarms/{alarm_id}` | TS 28.532 acknowledgeAlarms (merge patch) | Changes only the tester's own O-DU / requests |
| `mns-fm-06` | Unacknowledge the alarm | request | `PATCH {fm}/alarms/{alarm_id}` | TS 28.532 unacknowledgeAlarms | Changes only the tester's own O-DU / requests |
| `mns-fm-07` | Comment the alarm | request | `POST {fm}/alarms/{alarm_id}/comments` | TS 28.532 setComment | Changes only the tester's own O-DU / requests |
| `mns-fm-08` | Acknowledge several alarms at once | request | `PATCH {fm}/alarms` | TS 28.532 acknowledgeAlarms (multiple) | Changes only the tester's own O-DU / requests |
| `mns-fm-09` | Clear the alarm | request | Clears the alarm in the SMO (and on the NF, if the SMO passes it on). | TS 28.532 clearAlarms | Changes only the tester's own O-DU / requests |
| `mns-fm-10` | Unknown alarm is refused (404) | request | `PATCH {fm}/alarms/smotester-no-such-alarm` | TS 28.532 error handling (ErrorResponse) |  |
| `mns-fm-10b` | Remove the tester's earlier alarm subscription (if there is one) | request | The tester keeps the id of its last alarm subscription between runs and removes it before it subscribes again, so the SMO is not left with an old subscription to the tester. Not applicable when the tester has none. | TS 28.532 unsubscribe (FaultMnS) |  |
| `mns-fm-11` | Subscribe to alarm notifications | request | The SMO will POST alarm notifications to the tester's callback address/fm. | TS 28.532 subscribe (FaultMnS) |  |
| `mns-fm-12` | notifyNewAlarm arrives | wait | The tester first clears its test alarm (it may still be active from an earlier case), then raises it again by VES. | TS 28.532 notifyNewAlarm | Waits up to 180 s |
| `mns-fm-13` | notifyClearedAlarm arrives | wait | The tester first clears the alarm by VES. | TS 28.532 notifyClearedAlarm | Waits up to 180 s |
| `mns-fm-14` | notifyAckStateChanged arrives | wait | The tester first acknowledges an alarm. | TS 28.532 notifyAckStateChanged | Waits up to 180 s; Changes only the tester's own O-DU / requests |
| `mns-fm-15` | Remove the alarm subscription | request | `DELETE {fm}/subscriptions/{fm_sub_id}` | TS 28.532 unsubscribe |  |

### 3GPP Management Services › Provisioning (read)

Read managed objects with scope, filter and selection.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-prov-01` | Read a managed element (scope BASE_ONLY) | request | `GET {prov}/{me_dn}` | TS 28.532 §12.1.1.3 getMOIAttributes; TS 32.158 REST SS |  |
| `mns-prov-02` | Read a subtree (BASE_SUBTREE) | request | `GET {prov}/{me_dn}` | TS 28.532 getMOIAttributes, scope |  |
| `mns-prov-03` | Read one level below (BASE_NTH_LEVEL, 1) | request | `GET {prov}/{me_dn}` | TS 28.532 scope |  |
| `mns-prov-04` | Read selected attributes | request | `GET {prov}/{me_dn}` | TS 28.532 attributes selection |  |
| `mns-prov-05` | Read with fields selection | request | `GET {prov}/{me_dn}` | TS 28.532 fields selection |  |
| `mns-prov-06` | Read with an XPath filter | request | `GET {prov}/{me_dn}` | TS 28.532 filter (XPath 1.0 subset, TS 32.158) |  |
| `mns-prov-07` | Unknown object is refused (404) | request | `GET {prov}/{me_dn}/SmoTesterNoSuchClass=99` | TS 28.532 error handling |  |
| `mns-prov-08` | Unsupported MnS version is refused | request | `GET {mns_root}/ProvMnS/v0/{me_dn}` | TS 32.158 URI versioning |  |

### 3GPP Management Services › Provisioning (change)

Change, create and delete managed objects. Lab use only.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-provw-01` | Change an attribute (JSON merge patch) | request | Changes the attribute of the O-DU's cell on the NF. Lab use only. | TS 28.532 modifyMOIAttributes (application/merge-patch+json) | Changes only the tester's own O-DU / requests |
| `mns-provw-02` | Read back the changed attribute | request | `GET {prov}/{write_dn}` | TS 28.532 getMOIAttributes |  |
| `mns-provw-03` | Change an attribute (3GPP JSON patch) | request | `PATCH {prov}/{write_dn}` | TS 28.532 modifyMOIAttributes (application/3gpp-json-patch+json) | Changes only the tester's own O-DU / requests |
| `mns-provw-04` | Create an object (PUT) | request | Creates a harmless NtfSubscriptionControl object; deleted by the next case. | TS 28.532 createMOI (PUT) | Changes only the tester's own O-DU / requests |
| `mns-provw-05` | Delete the object | request | `DELETE {prov}/{me_dn}/NtfSubscriptionControl=smotester-create` | TS 28.532 deleteMOI | Changes only the tester's own O-DU / requests |
| `mns-provw-06` | Wrong value is refused | request | Negative test: must be refused, nothing changes. | TS 28.532 error handling | Changes only the tester's own O-DU / requests |
| `mns-provw-07` | Unsupported media type is refused | request | `PATCH {prov}/{write_dn}` | TS 28.532 / RFC 7231 (415) |  |

### 3GPP Management Services › Configuration notifications

Be told about object changes.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-cm-01` | Subscribe to configuration notifications (NtfSubscriptionControl) | request | `PUT {prov}/{me_dn}/NtfSubscriptionControl=smotester-cm` | TS 28.623 NtfSubscriptionControl; TS 28.532 subscribe via ProvMnS |  |
| `mns-cm-02` | notifyMOIAttributeValueChanges arrives | wait | The tester first changes an attribute over ProvMnS. | TS 28.532 notifyMOIAttributeValueChanges / notifyMOIChanges | Waits up to 180 s; Changes only the tester's own O-DU / requests |
| `mns-cm-03` | Read the subscription back | request | `GET {prov}/{me_dn}/NtfSubscriptionControl=smotester-cm` | TS 28.623 NtfSubscriptionControl |  |
| `mns-cm-04` | Remove the subscription | request | `DELETE {prov}/{me_dn}/NtfSubscriptionControl=smotester-cm` | TS 28.532 unsubscribe (deleteMOI) |  |

### 3GPP Management Services › Heartbeat

Periodic and triggered heartbeats.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-hb-01` | Subscribe with heartbeat (NtfSubscriptionControl + HeartbeatControl) | request | `PUT {prov}/{me_dn}/NtfSubscriptionControl=smotester-hb` | TS 28.623 HeartbeatControl; TS 28.532 heartbeat |  |
| `mns-hb-02` | notifyHeartbeat arrives | wait | Wait up to one heartbeat period (60 s). | TS 28.532 notifyHeartbeat | Waits up to 150 s |
| `mns-hb-03` | Ask for an immediate heartbeat (triggerHeartbeatNtf) | request | `PATCH {prov}/{me_dn}/NtfSubscriptionControl=smotester-hb/HeartbeatControl=1` | TS 28.623 triggerHeartbeatNtf |  |
| `mns-hb-04` | The triggered heartbeat arrives | wait | The tester first asks for an immediate heartbeat. | TS 28.532 notifyHeartbeat | Waits up to 30 s |
| `mns-hb-05` | Change the period | request | `PATCH {prov}/{me_dn}/NtfSubscriptionControl=smotester-hb/HeartbeatControl=1` | TS 28.623 |  |
| `mns-hb-06` | Remove the heartbeat subscription | request | `DELETE {prov}/{me_dn}/NtfSubscriptionControl=smotester-hb` | TS 28.532 |  |

### 3GPP Management Services › Performance jobs and files

PerfMetricJob, file-ready notifications, file download.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-pm-01` | Create a measurement job (PerfMetricJob, file reporting) | request | Asks for the metric every 15 minutes, delivered as files; notifyFileReady goes to the tester's callback address/pm. | TS 28.550 measurement job control (PerfMetricJob, TS 28.622 NRM) |  |
| `mns-pm-02` | Read the job | request | `GET {prov}/{me_dn}/PerfMetricJob=smotester-pm1` | TS 28.550 |  |
| `mns-pm-03` | Subscribe to file-ready notifications (FileDataReportingMnS) | request | `POST {fdr}/subscriptions` | TS 28.532 File data reporting MnS, subscribe |  |
| `mns-pm-04` | notifyFileReady arrives | wait | Wait for the end of the reporting period (up to 15 minutes). | TS 28.532 notifyFileReady | Waits up to 1200 s |
| `mns-pm-05` | List available files | request | `GET {fdr}/files` | TS 28.532 listAvailableFiles |  |
| `mns-pm-06` | Download the reported file | request | Uses the fileLocation from the notification (absolute or relative). | TS 32.435 / TS 32.432 |  |
| `mns-pm-07` | Lock (pause) the job | request | `PATCH {prov}/{me_dn}/PerfMetricJob=smotester-pm1` | TS 28.550 |  |
| `mns-pm-08` | Delete the job | request | `DELETE {prov}/{me_dn}/PerfMetricJob=smotester-pm1` | TS 28.550 |  |
| `mns-pm-09` | Remove the file-ready subscription | request | `DELETE {fdr}/subscriptions/{fdr_sub_id}` | TS 28.532 |  |

### 3GPP Management Services › Streaming of measurements

The SMO connects to the tester as streaming consumer and streams values.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-st-00` | Find the granularity periods supported for streaming | steps | Reads the object the job is created under and takes a granularity period of a group whose reportingMethods include STREAM_BASED. | TS 28.622 supportedPerfMetricGroups (TS 28.623 SupportedPerfMetricGroup) |  |
| `mns-st-01` | Create a streaming measurement job | request | Tells the SMO to stream the metric to the tester's streaming consumer at the tester's streaming address. The job covers the whole managed element (rootObjectInstances: the element and everything under it, TS 28.622), because the O-DU's measurements are on its cell, not on the managed element itself. | TS 28.550 streaming; TS 28.622 reportingCtrl streamTarget |  |
| `mns-st-02` | The SMO requests a streaming connection | wait | The tester first creates the streaming job. | TS 28.532 streaming data reporting: establish connection (POST /connections) | Waits up to 180 s |
| `mns-st-03` | The SMO opens the WebSocket | wait | Follows the connection request. | TS 28.532 streaming: GET /connections/{connectionId} (101 Switching Protocols) | Waits up to 120 s |
| `mns-st-04` | Measurement data arrives on the stream | wait | Wait one granularity period (the supported period). | TS 28.532 streaming data (serialisation per TS 28.550) | Waits up to 300 s (or the supported period + 120 s when longer) |
| `mns-st-05` | Delete the streaming job | request | `DELETE {prov}/{me_dn}/PerfMetricJob=smotester-st1` | TS 28.550 |  |

### 3GPP Management Services › Trace and MDT activation

TraceJob for trace and MDT.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mns-tr-01` | Start trace (create a TraceJob) | request | The trace records go to the address in the job; the tester does not collect them. | TS 32.422 trace control; TS 28.622 / TS 28.623 TraceJob | Changes only the tester's own O-DU / requests |
| `mns-tr-02` | Read the trace job | request | `GET {prov}/{me_dn}/TraceJob=smotester-tr1` | TS 28.622 |  |
| `mns-tr-03` | Stop trace (delete the TraceJob) | request | `DELETE {prov}/{me_dn}/TraceJob=smotester-tr1` | TS 32.422 | Changes only the tester's own O-DU / requests |
| `mns-tr-04` | Start MDT (immediate MDT job) | request | `PUT {prov}/{me_dn}/TraceJob=smotester-mdt1` | TS 32.422 MDT; TS 28.622 / TS 28.623 TraceJob (mdtConfig) | Changes only the tester's own O-DU / requests |
| `mns-tr-05` | Stop MDT | request | `DELETE {prov}/{me_dn}/TraceJob=smotester-mdt1` | TS 32.422 | Changes only the tester's own O-DU / requests |

## R1

The tester acts as an rApp: onboards, discovers and publishes services, gets CAPIF tokens, creates data jobs, reads configuration and alarms, and manages A1 policies through the SMO.

- **Between:** rApps and the SMO (Non-RT RIC framework)
- **The tester plays:** an rApp
- **Standards:** O-RAN R1AP (ETSI TS 104 231), 3GPP TS 29.222 (CAPIF), TS 28.111
- **Defined by:** O-RAN WG2

### R1 › Service management and exposure (CAPIF)

Onboarding, discovery, provider registration, publishing.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-sme-00` | Offboard the tester's earlier rApp (if there is one) | request | The tester keeps its rApp's invoker id between runs. Before onboarding again it removes the earlier one, so the SMO is not left with old invokers. Not applicable when the tester has none (first run). | TS 29.222 §8.4 (offboard) |  |
| `r1-sme-01` | Onboard the rApp (API invoker) | request | `POST /api-invoker-management/v1/onboardedInvokers` | TS 29.222 §8.4 CAPIF_API_Invoker_Management_API (onboardedInvokers) |  |
| `r1-sme-02` | Discover all service APIs | request | `GET /service-apis/v1/allServiceAPIs` | TS 29.222 §8.1 CAPIF_Discover_Service_API |  |
| `r1-sme-03` | Discover by API name | request | `GET /service-apis/v1/allServiceAPIs` | TS 29.222 discovery query parameters |  |
| `r1-sme-04` | Discovery without an invoker id is refused | request | `GET /service-apis/v1/allServiceAPIs` | TS 29.222 (api-invoker-id is mandatory) |  |
| `r1-sme-05` | Update the invoker | request | `PUT /api-invoker-management/v1/onboardedInvokers/{invoker_id}` | TS 29.222 §8.4 (PUT individual onboarded invoker) |  |
| `r1-sme-06` | Register as API provider (rApp offering an API) | request | `POST /api-provider-management/v1/registrations` | TS 29.222 §8.9 CAPIF_API_Provider_Management_API |  |
| `r1-sme-07` | Publish a service API | request | `POST /published-apis/v1/{apf_id}/service-apis` | TS 29.222 §8.2 CAPIF_Publish_Service_API |  |
| `r1-sme-08` | Read the published API | request | `GET /published-apis/v1/{apf_id}/service-apis/{api_id}` | TS 29.222 §8.2 |  |
| `r1-sme-09` | The invoker discovers the new API | request | `GET /service-apis/v1/allServiceAPIs` | TS 29.222 §8.1 |  |
| `r1-sme-10` | Unpublish the API | request | `DELETE /published-apis/v1/{apf_id}/service-apis/{api_id}` | TS 29.222 §8.2 |  |
| `r1-sme-11` | Deregister the provider | request | `DELETE /api-provider-management/v1/registrations/{prov_dom_id}` | TS 29.222 §8.9 |  |

### R1 › CAPIF events

Notifications about APIs appearing and going away.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-evt-01` | Subscribe to CAPIF events | request | `POST /capif-events/v1/{invoker_id}/subscriptions` | TS 29.222 §8.3 CAPIF_Events_API |  |
| `r1-evt-02` | Register as API provider (for the events) | request | The tester's own provider registration, made after subscribing. | TS 29.222 §8.9 |  |
| `r1-evt-03` | Publish a service API (after subscribing) | request | `POST /published-apis/v1/{evt_apf_id}/service-apis` | TS 29.222 §8.2 |  |
| `r1-evt-04` | SERVICE_API_AVAILABLE arrives | wait | The tester published its API in the previous step; the SMO notifies the subscriber. | TS 29.222 §8.3 EventNotification | Waits up to 120 s |
| `r1-evt-05` | Unpublish the API | request | `DELETE /published-apis/v1/{evt_apf_id}/service-apis/{evt_api_id}` | TS 29.222 §8.2 |  |
| `r1-evt-06` | SERVICE_API_UNAVAILABLE arrives | wait | The tester unpublished its API in the previous step. | TS 29.222 §8.3 EventNotification | Waits up to 120 s |
| `r1-evt-07` | Deregister the provider | request | `DELETE /api-provider-management/v1/registrations/{evt_prov_dom_id}` | TS 29.222 §8.9 |  |
| `r1-evt-08` | Remove the event subscription | request | `DELETE /capif-events/v1/{invoker_id}/subscriptions/{capif_evt_sub}` | TS 29.222 §8.3 |  |

### R1 › R1 security (CAPIF access tokens)

OAuth 2.0 tokens for discovered APIs.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-sec-01` | Create the invoker's security context | request | `PUT /capif-security/v1/trustedInvokers/{invoker_id}` | TS 29.222 §8.5 CAPIF_Security_API (trustedInvokers) |  |
| `r1-sec-02` | Get an access token (client credentials) | request | `POST /capif-security/v1/securities/{invoker_id}/token` | TS 29.222 §8.5 obtain access token (AccessTokenReq) |  |
| `r1-sec-03` | Wrong secret is refused | request | `POST /capif-security/v1/securities/{invoker_id}/token` | TS 29.222 AccessTokenErr (RFC 6749 §5.2) |  |
| `r1-sec-04` | Call a discovered API with the access token | request | 200 if the token's scope covers the data discovery API; 403 if it names another API (both correct). | TS 29.222 / R1AP security (OAuth 2.0 bearer) |  |
| `r1-sec-05` | Revoke the security context | request | `DELETE /capif-security/v1/trustedInvokers/{invoker_id}` | TS 29.222 §8.5 |  |

### R1 › Data management and exposure (R1AP)

Data discovery, data jobs, push delivery, data registration.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-dme-01` | Discover data types (DME types) | request | `GET /data-discovery/{r1_dme_version}/dme-types` | R1AP §7.2 Data discovery API |  |
| `r1-dme-02` | Read one DME type | request | `GET /data-discovery/{r1_dme_version}/dme-types/{dme_ns}:{dme_name}:{dme_ver}` | R1AP §7.2 |  |
| `r1-dme-03` | Filter DME types by data category | request | `GET /data-discovery/{r1_dme_version}/dme-types` | R1AP §7.2 query parameters |  |
| `r1-dme-04` | Create a data job (continuous push) | request | An empty productionJobDefinition asks for everything; the SMO may need a filter (see the DME type's schema). | R1AP §7.3 Data access API (DataJobInfo) |  |
| `r1-dme-05` | Data is pushed to the rApp | wait | The tester first clears its test alarm and raises it again by VES (data for alarm-type jobs). | R1AP §7.4 Push data | Waits up to 180 s |
| `r1-dme-07` | Read the data job status | request | `GET /data-access/{r1_dme_version}/data-jobs/{data_job_id}/status` | R1AP §7.3 DataJobStatusInfo |  |
| `r1-dme-08` | Unsupported delivery method is refused | request | 400 if the SMO has no Kafka delivery; 201 if it has (then delete the job). | R1AP §7.3 |  |
| `r1-dme-09` | Delete the data job | request | `DELETE /data-access/{r1_dme_version}/data-jobs/{data_job_id}` | R1AP §7.3 |  |
| `r1-dme-09b` | Remove the tester's earlier data registration (if there is one) | request | The tester keeps the id of its last data registration between runs and removes it before registering again, so the SMO is not left with an old registration (an SMO may refuse a second registration of the same data type with 409). Not applicable when the tester has none. | R1AP §7.1 Data registration API (deregister) |  |
| `r1-dme-10` | Register the rApp as data producer (data registration) | request | The rApp registers a synthetic data type it can produce (smotester:synthetic-kpi, version dr type ver). The SMO answers 201 with the new registration's address (Location). The tester first removes the registration it made in an earlier run (r1-dme-09b); if the SMO still holds one the tester has no address for (HTTP 409), it registers the next version of its data type. | R1AP §7.1 Data registration API |  |
| `r1-dme-11` | Deregister the data producer | request | Removes the registration made in the previous case, so nothing is left in the SMO. | R1AP §7.1 Data registration API (deregister) |  |

### R1 › Data delivery (O-RAN SC ICS form)

The open-source consumer API many rApps use.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-ics-00` | About this form | info | The O-RAN Software Community's Information Coordinator Service (ICS) consumer API. It is open-source reference code, not an O-RAN specification, but many SMOs and rApps use it. | O-RAN SC ICS data-consumer API v1 |  |
| `r1-ics-01` | List information types | request | `GET /data-consumer/v1/info-types` | O-RAN SC ICS |  |
| `r1-ics-02` | Read an information type | request | `GET /data-consumer/v1/info-types/{info_type}` | O-RAN SC ICS |  |
| `r1-ics-03` | Create an information job | request | `PUT /data-consumer/v1/info-jobs/smotester-job-1` | O-RAN SC ICS |  |
| `r1-ics-04` | Job results arrive | wait | The tester first clears its test alarm (it may still be active from an earlier case, and a fault that is already active is only a repeat, not a new alarm), then raises it again by VES. | O-RAN SC ICS | Waits up to 180 s |
| `r1-ics-05` | Job status | request | `GET /data-consumer/v1/info-jobs/smotester-job-1/status` | O-RAN SC ICS |  |
| `r1-ics-06` | Delete the job | request | `DELETE /data-consumer/v1/info-jobs/smotester-job-1` | O-RAN SC ICS |  |

### R1 › RAN OAM services

Configuration, alarm list and schemas through R1.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-oam-01` | Read configuration through R1 (ProvMnS) | request | `GET {mns_root}/ProvMnS/v1/{me_dn}` | R1AP §8.1 Configuration management API (TS 28.532) |  |
| `r1-oam-02` | Read the alarm list through R1 (AlarmList) | request | `GET {mns_root}/ProvMnS/v1/{me_dn}/AlarmList=1` | R1AP §8.2 Fault management API (TS 28.111 AlarmList) |  |
| `r1-oam-03` | Acknowledge an alarm through R1 (JSON patch) | request | `PATCH {mns_root}/ProvMnS/v1/{me_dn}/AlarmList=1` | R1AP §8.2, TS 28.111 Annex A | Changes only the tester's own O-DU / requests |
| `r1-oam-04` | List configuration schemas | request | `GET /ran-oam-cm-schema-info/v1/schemas` | R1AP §8.3 Configuration schema information API |  |
| `r1-oam-05` | Read one schema | request | `GET /ran-oam-cm-schema-info/v1/schemas/{schema_id}` | R1AP §8.3 |  |

### R1 › A1 policy management

rApp manages A1 policies through the SMO.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-a1-01` | List policy types | request | Reads only. The list may be empty: the policy types come from the Near-RT RICs the SMO manages. | R1AP §9.1 A1 policy management API |  |
| `r1-a1-02` | Policies on a Near-RT RIC are not tested | info | Reading a policy type, creating, reading and deleting a policy and reading its status all need a Near-RT RIC that the SMO manages over A1. The tester has no Near-RT RIC of its own, and it never changes a RIC it does not own, so these steps are left out for now. | R1AP §9.1 |  |
| `r1-a1-06` | Subscribe to policy status changes | request | The tester's own subscription in the SMO; no RIC is needed and nothing in the network changes. | R1AP §9.1 PolicyStatusSubscription |  |
| `r1-a1-08` | Remove the status subscription | request | `DELETE /a1-policy-management/v1/policies/subscriptions/{r1_a1_sub}` | R1AP §9.1 |  |

### R1 › Offboarding

Leave cleanly.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `r1-off-01` | Offboard the rApp | request | `DELETE /api-invoker-management/v1/onboardedInvokers/{invoker_id}` | TS 29.222 §8.4 |  |
| `r1-off-02` | The offboarded invoker can no longer discover | request | `GET /service-apis/v1/allServiceAPIs` | TS 29.222 §8.1 |  |

## A1

The tester is a Near-RT RIC: it takes enrichment information from the SMO (A1-EI) and asks the SMO's Non-RT RIC to train AI/ML models (A1-ML). A1-P is not tested (the SMO would have to be configured to call the tester).

- **Between:** Non-RT RIC (in the SMO) and Near-RT RIC
- **The tester plays:** the Near-RT RIC
- **Standards:** O-RAN A1AP R005 v07.00 (A1-EI 1.3.1, A1-ML 1.0.0), A1TD
- **Defined by:** O-RAN WG2

### A1 › A1-P policy management (not tested)

Needs the tester set as a RIC in the SMO.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `a1p-00` | Why A1-P is not tested here | info | For A1 policies the SMO is the client: it calls a Near-RT RIC whose A1 address has been set in the SMO. Testing it would need the tester configured in the SMO as a RIC, which is a change on the SMO side. Policies an rApp manages through the SMO are tested under R1 > A1 policy management. | A1AP §6.2 A1-P |  |

### A1 › A1-EI enrichment information (Near-RT RIC → SMO)

The tester creates EI jobs at the SMO.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `a1ei-01` | List EI types | request | `GET /A1-EI/v1/eitypes` | A1AP §6.3 A1-EI: query EI type identifiers |  |
| `a1ei-02` | Read an EI type | request | `GET /A1-EI/v1/eitypes/{ei_type}` | A1AP A1-EI: query EI type |  |
| `a1ei-03` | Unknown EI type is refused | request | `GET /A1-EI/v1/eitypes/smotester-no-such-type` | A1AP error handling |  |
| `a1ei-04` | Create an EI job | request | `PUT /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI: create EI job |  |
| `a1ei-05` | Read the EI job | request | `GET /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI: query EI job |  |
| `a1ei-06` | List EI jobs of the type | request | `GET /A1-EI/v1/eijobs` | A1AP A1-EI: query EI job identifiers |  |
| `a1ei-07` | EI job status | request | `GET /A1-EI/v1/eijobs/smotester-ei-1/status` | A1AP A1-EI: query EI job status |  |
| `a1ei-08` | Enrichment information is delivered | wait | The tester first clears its test alarm, then sends it again by VES, so alarm-type EI jobs get data. | A1AP A1-EI: deliver EI job result | Waits up to 180 s |
| `a1ei-09` | Update the EI job | request | `PUT /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI: update EI job |  |
| `a1ei-10` | Changing the type of a job is refused | request | `PUT /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI |  |
| `a1ei-11` | Delete the EI job | request | `DELETE /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI: delete EI job |  |
| `a1ei-12` | The deleted job is gone | request | `GET /A1-EI/v1/eijobs/smotester-ei-1` | A1AP A1-EI |  |

### A1 › A1-ML model training (Near-RT RIC → SMO)

The tester asks the SMO's Non-RT RIC to train a model.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `a1ml-01` | List AI/ML training capabilities | request | `GET /a1-ml/v1/training-capabilities` | A1AP R005 §6.4 A1-ML: query training capabilities (SMO's Non-RT RIC as A1-ML producer) |  |
| `a1ml-02` | Filter capabilities by environment type | request | `GET /a1-ml/v1/training-capabilities` | A1AP A1-ML query parameters |  |
| `a1ml-03` | Create a training job | request | Asks the SMO to train a (synthetic) model; status notifications go to the tester's callback address/a1ml. | A1AP A1-ML: create training job | Changes only the tester's own O-DU / requests |
| `a1ml-04` | A training job status notification arrives | wait | The producer notifies status changes of the job created in the previous case. | A1AP A1-ML: training job status notification (callback) | Waits up to 300 s |
| `a1ml-05` | Read the training job status | request | `GET /a1-ml/v1/training-jobs/{training_job_id}/status` | A1AP A1-ML: query training job status |  |
| `a1ml-06` | A job without modelId is refused | request | `POST /a1-ml/v1/training-jobs` | A1AP A1-ML (modelId is mandatory) |  |
| `a1ml-07` | Delete the training job | request | `DELETE /a1-ml/v1/training-jobs/{training_job_id}` | A1AP A1-ML: delete training job | Changes only the tester's own O-DU / requests |
| `a1ml-08` | The deleted job is gone | request | `DELETE /a1-ml/v1/training-jobs/{training_job_id}` | A1AP A1-ML |  |

## O2

The tester acts as an O-Cloud the SMO manages over O2ims: the SMO reads its inventory and alarms over mutual TLS with an OAuth 2.0 token, subscribes to its notifications and acts on its alarms. Add the tester's O-Cloud to the SMO first (Integration page); without it these cases are not applicable.

- **Between:** SMO and the O-Cloud
- **The tester plays:** the O-Cloud (O2ims producer)
- **Standards:** O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00; RFC 6749; O-RAN WG11 SecProtSpec
- **Defined by:** O-RAN WG6

### O2 › Security: token and mutual TLS

How the SMO logs in to the O-Cloud.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o2-00` | The O-Cloud ends its earlier access tokens | steps | Preparation for o2-01: the tester's O-Cloud ends the tokens it issued earlier (they are kept across tester restarts), so the SMO cannot keep using a token it already has and must get a new one. | RFC 6749 4.4; RFC 6750 3.1 | Needs the tester's O-Cloud in the SMO |
| `o2-01` | The SMO gets an access token | wait | The SMO fetches a token from the O-Cloud's authorization server with the client credentials grant. | RFC 6749 4.4; O-RAN WG11 SecProtSpec 4.7; O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.1.7 | Waits up to 360 s; The tester's O-Cloud first ends the tokens it issued earlier (o2-00). In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. The SMO's request with its old token gets 401 invalid_token, and it has to get a new token.; Needs the tester's O-Cloud in the SMO |

### O2 › Infrastructure inventory

The SMO reads the O-Cloud's inventory and subscribes to its changes.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o2-02` | The SMO reads the O-Cloud description (CloudInfo) | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.10 (O-Cloud description) | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-03` | The SMO reads the resource types | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.2 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-04` | The SMO reads the resource pools | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.4 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-05` | The SMO reads the resources of a pool | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.6 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-06` | The SMO reads the deployment managers | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.8 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-07` | The SMO reads the O-Cloud sites | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.18 (added in 2.0.0) | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-08` | The SMO subscribes to inventory changes | wait | In the SMO, turn on notifications from this O-Cloud (MiniSMO: CHANGE… > Receive notifications from this O-Cloud). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.4.11, table 3.2.6.2.7-1 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-09` | The SMO accepts an inventory change notification | steps | The O-Cloud changes a resource and POSTs the notification (MODIFY, prior and post state) to the SMO's callback. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.2.5.1 Inventory Change Notification | Needs the tester's O-Cloud in the SMO |

### O2 › Infrastructure monitoring

Alarm list, alarm notifications, acknowledging and clearing.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o2-10` | The SMO reads the alarm list | wait | In the SMO, read the O-Cloud now (MiniSMO: NETWORK > O-Cloud > READ NOW), or wait for its next periodic read. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.4.2 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-11` | The SMO subscribes to alarm events | wait | In the SMO, turn on notifications from this O-Cloud (MiniSMO: CHANGE… > Receive notifications from this O-Cloud). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.4.4, table 3.3.6.2.3-1 | Waits up to 360 s; Needs the tester's O-Cloud in the SMO |
| `o2-12` | The SMO accepts a new-alarm notification | steps | The O-Cloud raises a MAJOR alarm on a server and POSTs the notification to the SMO's alarm callback. The SMO should then show it (MiniSMO: Alarm Console, category O-Cloud). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.5.1 Alarm Change Notification (NEW) | Needs the tester's O-Cloud in the SMO |
| `o2-13` | The SMO acknowledges the O-Cloud alarm | wait | In the SMO, acknowledge the O-Cloud alarm (MiniSMO: NETWORK > O-Cloud > Alarms > ACKNOWLEDGE…; the O-Cloud must have 'Allow acknowledging and clearing' on). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.4.3.3.4, table 3.3.6.2.4-1 | Waits up to 600 s; Manual step in the SMO; Needs the tester's O-Cloud in the SMO |
| `o2-14` | The SMO clears the O-Cloud alarm | wait | In the SMO, clear the O-Cloud alarm (MiniSMO: NETWORK > O-Cloud > Alarms > CLEAR…). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.4.3.3.4 | Waits up to 600 s; Manual step in the SMO; Needs the tester's O-Cloud in the SMO |
| `o2-15` | The SMO accepts a cleared-alarm notification | steps | The O-Cloud clears an active alarm itself and POSTs the CLEAR notification to the SMO's alarm callback. When the SMO has already cleared the alarm (previous case), the O-Cloud first raises a new one, so this case does not depend on the manual steps. | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.3.5.1 Alarm Change Notification (CLEAR) | Needs the tester's O-Cloud in the SMO |

### O2 › Infrastructure provisioning

Provisioning requests from the SMO (optional).

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `o2-16` | The SMO sends a provisioning request | wait | Optional: only SMOs that provision O-Cloud infrastructure send this. The tester's O-Cloud moves it to FULFILLED after a few seconds (SYNTHETIC). | O-RAN.WG6.TS.O2IMS-INTERFACE-R005-v13.00 3.4.4, table 3.4.6.2.2-1 | Waits up to 600 s; In the SMO, create a provisioning request on this O-Cloud with these values (the O-Cloud accepts only this template): name smotester-provisioning-1, description "SMO Tester provisioning test (synthetic)", templateName smotester-node-cluster, templateVersion 1.0.0, template parameters {} (empty; the template takes none). Let the SMO make the provisioningRequestId (a UUID); if it asks, use any new UUID.; Manual step in the SMO; Needs the tester's O-Cloud in the SMO |

## TMF Open APIs

The tester acts as an OSS/BSS using TM Forum Open APIs for alarms, inventory and performance.

- **Between:** Operator OSS/BSS and the SMO
- **The tester plays:** the OSS/BSS
- **Standards:** TMF642 v4, TMF639 v4, TMF628 v5, TMF630 guidelines
- **Defined by:** TM Forum

### TMF Open APIs › TMF642 Alarm Management

List, filter, acknowledge, clear, comment, notifications.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `tmf642-01` | List alarms | request | `GET {tmf_alarm}/alarm` | TMF642 v4 GET /alarm |  |
| `tmf642-01b` | Find the alarm of the tester's O-DU | steps | The acknowledge, comment and clear cases act only on an alarm the tester's own O-DU raised. Needs the O-DU integrated (Integration page). | TMF642 v4 GET /alarm |  |
| `tmf642-02` | Paging and totals (offset, limit) | request | `GET {tmf_alarm}/alarm` | TMF630 API design guidelines (paging, X-Total-Count) |  |
| `tmf642-03` | Filter by severity | request | `GET {tmf_alarm}/alarm` | TMF642 filtering |  |
| `tmf642-04` | Attribute selection (fields) | request | `GET {tmf_alarm}/alarm` | TMF630 attribute selection |  |
| `tmf642-05` | Read one alarm | request | `GET {tmf_alarm}/alarm/{tmf_alarm_id}` | TMF642 GET /alarm/{id} |  |
| `tmf642-06` | Acknowledge (PATCH ackState) | request | `PATCH {tmf_alarm}/alarm/{tmf_own_alarm_id}` | TMF642 PATCH /alarm/{id} | Changes only the tester's own O-DU / requests |
| `tmf642-07` | Add a comment | request | `PATCH {tmf_alarm}/alarm/{tmf_own_alarm_id}` | TMF642 comments | Changes only the tester's own O-DU / requests |
| `tmf642-08` | Unacknowledge | request | `PATCH {tmf_alarm}/alarm/{tmf_own_alarm_id}` | TMF642 | Changes only the tester's own O-DU / requests |
| `tmf642-09` | Acknowledge by pattern (ackAlarms task) | request | The pattern matches only alarms of the tester's own O-DU (its alarmed object). | TMF642 task resource ackAlarms | Changes only the tester's own O-DU / requests |
| `tmf642-10` | Clear by pattern (clearAlarms task) | request | The pattern matches only alarms of the tester's own O-DU (its alarmed object). | TMF642 task resource clearAlarms | Changes only the tester's own O-DU / requests |
| `tmf642-11` | Unknown alarm is refused (404) | request | `GET {tmf_alarm}/alarm/smotester-no-such-alarm` | TMF630 error representation |  |
| `tmf642-12` | Subscribe (hub) | request | `POST {tmf_alarm}/hub` | TMF642 / TMF630 notifications (hub) |  |
| `tmf642-13` | AlarmCreateEvent arrives | wait | The tester first clears its test alarm (it may still be active from an earlier case), then raises it again by VES. | TMF642 AlarmCreateEvent | Waits up to 180 s |
| `tmf642-14` | Unsubscribe | request | `DELETE {tmf_alarm}/hub/{tmf642_hub}` | TMF630 hub |  |

### TMF Open APIs › TMF639 Resource Inventory

Read resources, subscribe and unsubscribe.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `tmf639-01` | List resources | request | `GET {tmf_ri}/resource` | TMF639 v4 GET /resource |  |
| `tmf639-02` | Filter logical resources | request | `GET {tmf_ri}/resource` | TMF639 filtering |  |
| `tmf639-03` | Read one resource | request | `GET {tmf_ri}/resource/{tmf_res_id}` | TMF639 GET /resource/{id} |  |
| `tmf639-04` | Subscribe (hub) | request | `POST {tmf_ri}/hub` | TMF639 hub |  |
| `tmf639-06` | Unsubscribe | request | `DELETE {tmf_ri}/hub/{tmf639_hub}` | TMF639 |  |

### TMF Open APIs › TMF628 Performance Management

Indicators, collection jobs, measurements.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `tmf628-01` | Define an indicator | request | `POST {tmf_pm}/performanceIndicatorSpecification` | TMF628 v5 PerformanceIndicatorSpecification |  |
| `tmf628-02` | Create a collection job | request | `POST {tmf_pm}/measurementCollectionJob` | TMF628 MeasurementCollectionJob |  |
| `tmf628-03` | Read the job | request | `GET {tmf_pm}/measurementCollectionJob/{tmf_job_id}` | TMF628 |  |
| `tmf628-04` | Read measurements of the job | request | Empty until the first reporting period ends. | TMF628 PerformanceMeasurement |  |
| `tmf628-05` | Change the reporting period (PATCH) | request | TMF628 v5 has no pause attribute; this changes a standard attribute instead. | TMF628 v5 PATCH /measurementCollectionJob/{id} (MeasurementCollectionJob_MVO) |  |
| `tmf628-06` | Delete the job | request | `DELETE {tmf_pm}/measurementCollectionJob/{tmf_job_id}` | TMF628 |  |

## Intent-driven management

The tester states radio network intents about its own cells and reads or receives the SMO's fulfilment reports.

- **Between:** OSS and the SMO
- **The tester plays:** the intent owner (OSS)
- **Standards:** TS 28.312
- **Defined by:** 3GPP SA5

### Intent-driven management › Radio network intents

Create, read, report, change, delete.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `int-01` | Read intent handling capabilities | request | `GET {prov}/{subnet_dn}/IntentHandlingFunction=1` | TS 28.312 IntentHandlingFunction (intentHandlingCapabilityList) |  |
| `int-02` | Create an intent (DL throughput above 10 Mbit/s) | request | The intent concerns only the tester's own cells (PLMN 999-70 on its downlink frequency). | TS 28.312 §7.1 createMOI Intent | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-03` | Read the intent | request | `GET {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i1` | TS 28.312 |  |
| `int-04` | Read the intent report | request | The report the SMO named in the intent (previous case). | TS 28.312 IntentReport (intentFulfilmentReport) |  |
| `int-05` | Subscribe to intent report changes | request | `PUT {prov}/{subnet_dn}/NtfSubscriptionControl=smotester-int` | TS 28.312 table 7.1-2 (report notifications) |  |
| `int-06` | An intent report notification arrives | wait | The tester first puts its report subscription (int-05), then its intent (int-02) again (both are PUTs, so this also brings back ones removed in an earlier run; the subscription goes first so a report about the intent finds it), then waits up to the end of an observation period (15 minutes). | TS 28.312 IntentReport notifications | Waits up to 1200 s |
| `int-07` | Change the target (merge patch) | request | `PATCH {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i1` | TS 28.312 modifyMOIAttributes | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-08` | Energy intent (RANEnergyConsumption below a limit) | request | `PUT {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i2` | TS 28.312 | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-09` | Unknown target is refused | request | `PUT {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i3` | TS 28.312 (feasibility / validation) | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-10` | Delete the intents | request | `DELETE {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i1` | TS 28.312 | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-11` | Delete the energy intent | request | `DELETE {prov}/{subnet_dn}/IntentHandlingFunction=1/Intent=smotester-i2` | TS 28.312 | Changes only the tester's own O-DU / requests; Needs the tester's O-DU in the SMO |
| `int-12` | Remove the report subscription | request | `DELETE {prov}/{subnet_dn}/NtfSubscriptionControl=smotester-int` | TS 28.532 |  |

## Management data analytics

The tester asks the SMO for analytics (MDARequest) and receives the reports.

- **Between:** Analytics consumers and the SMO
- **The tester plays:** the MDA consumer
- **Standards:** TS 28.104
- **Defined by:** 3GPP SA5

### Management data analytics › Resource utilization analysis

One-time and periodic requests, reports, refusals.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `mda-01` | Read MDA capabilities | request | `GET {prov}/{subnet_dn}/MDAFunction=1` | TS 28.104 §9 MDAFunction (supportedMDACapabilities) |  |
| `mda-02` | Request a one-time analysis | request | The analysis is scoped to the tester's own O-DU (managedEntitiesScope). | TS 28.104 MDARequest (resource utilization analysis) | Needs the tester's O-DU in the SMO |
| `mda-03` | The MDA report arrives | wait | The report follows the request (previous case). | TS 28.104 §11.1 reporting by notification | Waits up to 300 s |
| `mda-04` | Read the report | request | The report named in the notification (previous case). | TS 28.104 MDAReport |  |
| `mda-05` | Request a periodic analysis (every 15 minutes) | request | `PUT {prov}/{subnet_dn}/MDAFunction=1/MDARequest=smotester-m2` | TS 28.104 | Needs the tester's O-DU in the SMO |
| `mda-06` | File reporting is refused if not supported | request | 400 if the SMO has no file reporting; 201 if it has. | TS 28.104 reportingMethod |  |
| `mda-07` | Unknown MDA type is refused | request | `PUT {prov}/{subnet_dn}/MDAFunction=1/MDARequest=smotester-m4` | TS 28.104 |  |
| `mda-08` | Delete the requests | request | `DELETE {prov}/{subnet_dn}/MDAFunction=1/MDARequest=smotester-m1` | TS 28.104 |  |
| `mda-09` | Delete the periodic request | request | `DELETE {prov}/{subnet_dn}/MDAFunction=1/MDARequest=smotester-m2` | TS 28.104 |  |

## Interface security

Checks the SMO refuses requests without or with wrong credentials, plain HTTP and old TLS versions.

- **Between:** Every HTTP interface
- **The tester plays:** an unauthorised or outdated client
- **Standards:** O-RAN WG11 security requirements, TS 33.310, RFC 6750
- **Defined by:** O-RAN WG11, 3GPP SA3

### Interface security › 3GPP MnS endpoint

Northbound listener.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `sec-northbound-01` | 3GPP MnS: request without credentials is refused | request | `GET {fm}/alarms` | O-RAN WG11 (authentication of API consumers); RFC 6750 §3 |  |
| `sec-northbound-02` | 3GPP MnS: invalid credentials are refused | request | `GET {fm}/alarms` | RFC 6750 §3.1 |  |
| `sec-northbound-03` | 3GPP MnS: plain HTTP is not served | request | `GET {fm}/alarms` | O-RAN WG11: TLS for all management interfaces |  |
| `sec-northbound-04` | 3GPP MnS: TLS 1.1 is refused | request | `GET {fm}/alarms` | O-RAN WG11 / TS 33.310 TLS profile (TLS 1.2 or later) |  |
| `sec-northbound-05` | 3GPP MnS: TLS 1.2 works | request | Any HTTP answer means the TLS 1.2 handshake worked. | TS 33.310 (TLS 1.2 support) |  |
| `sec-northbound-06` | 3GPP MnS: security headers | request | `GET {fm}/alarms` | OWASP API security (advisory) |  |

### Interface security › VES collector

Event collector.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `sec-ves-01` | VES collector: request without credentials is refused | request | `POST /eventListener/v7` | O-RAN WG11 (authentication of API consumers); RFC 6750 §3 |  |
| `sec-ves-02` | VES collector: invalid credentials are refused | request | `POST /eventListener/v7` | RFC 6750 §3.1 |  |
| `sec-ves-03` | VES collector: plain HTTP is not served | request | `POST /eventListener/v7` | O-RAN WG11: TLS for all management interfaces |  |
| `sec-ves-04` | VES collector: TLS 1.1 is refused | request | `POST /eventListener/v7` | O-RAN WG11 / TS 33.310 TLS profile (TLS 1.2 or later) |  |
| `sec-ves-05` | VES collector: TLS 1.2 works | request | Any HTTP answer means the TLS 1.2 handshake worked. | TS 33.310 (TLS 1.2 support) |  |
| `sec-ves-06` | VES collector: security headers | request | `POST /eventListener/v7` | OWASP API security (advisory) |  |

### Interface security › R1 endpoint

R1 services.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `sec-r1-01` | R1: request without credentials is refused | request | `GET /service-apis/v1/allServiceAPIs?api-invoker-id=smotester-unknown` | O-RAN WG11 (authentication of API consumers); RFC 6750 §3 |  |
| `sec-r1-02` | R1: invalid credentials are refused | request | `GET /service-apis/v1/allServiceAPIs?api-invoker-id=smotester-unknown` | RFC 6750 §3.1 |  |
| `sec-r1-03` | R1: plain HTTP is not served | request | `GET /service-apis/v1/allServiceAPIs?api-invoker-id=smotester-unknown` | O-RAN WG11: TLS for all management interfaces |  |
| `sec-r1-04` | R1: TLS 1.1 is refused | request | `GET /service-apis/v1/allServiceAPIs?api-invoker-id=smotester-unknown` | O-RAN WG11 / TS 33.310 TLS profile (TLS 1.2 or later) |  |
| `sec-r1-05` | R1: TLS 1.2 works | request | Any HTTP answer means the TLS 1.2 handshake worked. | TS 33.310 (TLS 1.2 support) |  |
| `sec-r1-06` | R1: security headers | request | `GET /service-apis/v1/allServiceAPIs?api-invoker-id=smotester-unknown` | OWASP API security (advisory) |  |

### Interface security › TMF endpoint

TM Forum APIs.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `sec-tmf-01` | TMF: request without credentials is refused | request | `GET {tmf_alarm}/alarm` | O-RAN WG11 (authentication of API consumers); RFC 6750 §3 |  |
| `sec-tmf-02` | TMF: invalid credentials are refused | request | `GET {tmf_alarm}/alarm` | RFC 6750 §3.1 |  |
| `sec-tmf-03` | TMF: plain HTTP is not served | request | `GET {tmf_alarm}/alarm` | O-RAN WG11: TLS for all management interfaces |  |
| `sec-tmf-04` | TMF: TLS 1.1 is refused | request | `GET {tmf_alarm}/alarm` | O-RAN WG11 / TS 33.310 TLS profile (TLS 1.2 or later) |  |
| `sec-tmf-05` | TMF: TLS 1.2 works | request | Any HTTP answer means the TLS 1.2 handshake worked. | TS 33.310 (TLS 1.2 support) |  |
| `sec-tmf-06` | TMF: security headers | request | `GET {tmf_alarm}/alarm` | OWASP API security (advisory) |  |

## Other interfaces

Interfaces between other systems. The SMO does not take part, so there are no test cases.

- **Between:** Other systems
- **The tester plays:** -
- **Standards:** O-RAN WG3 E2AP, Y1; WG4 C/U/S-Plane
- **Defined by:** O-RAN WG3, WG4

### Other interfaces › Not SMO interfaces

Why these are not tested.

| Id | Test | Kind | What it does | Standard | Notes |
|---|---|---|---|---|---|
| `oth-e2` | E2 | info | Between the Near-RT RIC and the radio nodes (O-CU, O-DU). The SMO does not take part. | O-RAN.WG3.E2AP |  |
| `oth-y1` | Y1 | info | The Near-RT RIC offers RAN analytics to consumers. Not an SMO management interface. | O-RAN WG3 Y1 |  |
| `oth-ofh` | Open Fronthaul C/U/S-Plane | info | Real-time control, user data and synchronisation between O-DU and O-RU. Not management. | O-RAN WG4 CUS-Plane |  |

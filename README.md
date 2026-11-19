# Kaldrivon SMO Tester — User guide

Kaldrivon SMO Tester checks whether an SMO (Service Management and Orchestration) speaks its standard interfaces the
way the standards say it should. You tell it where the SMO is and how to log in, open a few ports on your PC, and run
prepared test cases interface by interface. To test how the SMO manages network functions and O-Clouds, you add the
tester to the SMO as an O-DU and as an O-Cloud: the tester plays both itself.

It is meant for people who build, buy or operate an SMO. Every test explains in plain words what it does and why it
passed or failed, so you do not need to be an interface specialist to follow it, and every test also shows the exact
messages for those who are.

**Download:** [latest release](https://github.com/yoroyarell/KaldrivonSMOTester/releases/latest) ·
**Website:** [kaldrivon.com](https://kaldrivon.com)

## Contents

1. [How the tester works](#1-how-the-tester-works)
2. [What you need](#2-what-you-need)
3. [Install, update and remove](#3-install-update-and-remove)
4. [Quick start](#4-quick-start)
5. [The window](#5-the-window)
6. [Connection page](#6-connection-page)
7. [Integration & ports page](#7-integration--ports-page)
8. [Running tests](#8-running-tests)
9. [Reading a result](#9-reading-a-result)
10. [Variables page](#10-variables-page)
11. [Transaction log, traffic and exports](#11-transaction-log-traffic-and-exports)
12. [Report page](#12-report-page)
13. [Clean up after testing](#13-clean-up-after-testing)
14. [Reset to defaults](#14-reset-to-defaults)
15. [Custom tests](#15-custom-tests)
16. [The interfaces](#16-the-interfaces)
17. [The tester's O-DU](#17-the-testers-o-du)
18. [The tester's O-RU](#18-the-testers-o-ru)
19. [The tester's O-Cloud](#19-the-testers-o-cloud)
20. [Certificates and passwords](#20-certificates-and-passwords)
21. [Command line](#21-command-line)
22. [Your data and security](#22-your-data-and-security)
23. [Troubleshooting](#23-troubleshooting)
24. [Support and legal](#24-support-and-legal)

Every single test case is listed in the [test catalog](docs/TEST-CATALOG.md).

## 1. How the tester works

The tester talks to the SMO in the roles the standards define for each interface:

- as the **operator's OSS** for the 3GPP management services, intents and analytics;
- as an **rApp** for R1;
- as a **Near-RT RIC** for A1;
- as a **network function (an O-DU)** for O1, sending events to the SMO's VES collector and, once added to the SMO,
  letting the SMO manage it over NETCONF;
- as an **O-Cloud** for O2, once added to the SMO;
- as the **OSS/BSS** for the TM Forum Open APIs;
- as an **unauthorised or outdated client** for the security checks.

Every answer and notification from the SMO is compared with the published definition of that interface (see
[Checked against the standards](#checked-against-the-standards)).

### What the tester changes in the SMO

The tester never changes anything the SMO manages except its own things:

- It **reads** whatever the SMO already has, to use real names instead of made-up ones.
- It **changes** only its own O-DU, its own O-Cloud and its own requests in the SMO (its subscriptions, jobs,
  intents, registrations). A test that would change anything else is shown as *not applicable*.
- Everything it creates is recorded, and the [Clean up](#13-clean-up-after-testing) page removes it again.

Cases that change something are marked **tester's O-DU only** or **own request** in the list. They run without
asking, because they never touch anything else.

Cases marked **(manual)** wait for you to do a step in the SMO yourself, for example acknowledging or clearing the
O-Cloud's alarm, or creating a provisioning request. The case says what to do and which values to use, and waits up
to 10 minutes. They run with the others in **Run** and **Run all**; use **Step** if you want to do them one at a time.

### Checked against the standards

- **3GPP Release 18:** TS 28.532 (management services), TS 28.623 (common definitions), TS 28.111 (fault
  management), TS 28.312 (intents), TS 28.104 (management data analytics), TS 28.550 (performance assurance),
  TS 29.222 (CAPIF).
- **O-RAN:** R1AP R005 v12.00, A1AP R005 v07.00 (A1-P, A1-EI, A1-ML), O2IMS R005 v13.00.
- **TM Forum:** TMF642 v4 (alarms), TMF639 v4 (resource inventory), TMF628 v5 (performance).
- **VES** event listener 7.2.1.
- The tester's own O-DU uses the 3GPP Rel-18 YANG models (TS 28.541, TS 28.623).

The exact versions are listed on the **About** page.

A missing mandatory field, a wrong type, a value that is not allowed, or an HTTP status or content type the standard
does not give for that operation is a **deviation** and fails the test. A field the standard does not define is an
**extension**: it is reported, and fails the test only when **Strict standard conformance** is on (Connection page).
Where a published definition is known to contradict itself, the tester accepts both readings and says so.

## 2. What you need

**On your PC**

- Windows 10 or 11 (64-bit) with the Microsoft Edge WebView2 runtime (normally already there), or Linux x86_64 with
  glibc 2.28 or newer (Debian/Ubuntu, RHEL/Rocky/AlmaLinux, Fedora, openSUSE…).
- A network path from your PC to the SMO, and from the SMO back to your PC.
- These TCP ports free and open in the PC's firewall, so the SMO can reach the tester:

  | Port | What uses it |
  |---|---|
  | 9643 | HTTPS: notifications, file downloads and streaming from the SMO to the tester |
  | 1830 | NETCONF: the SMO managing the tester's O-DU |
  | 1831 | NETCONF: the SMO managing the tester's O-RU (Open Fronthaul M-Plane) |
  | 9644 | O2ims: the SMO talking to the tester's O-Cloud |
  | 9680 | plain HTTP, only if you switch it on |

  The window itself uses 127.0.0.1:8790, which is reachable only from your own PC.

**On the SMO**

- Its northbound REST APIs (3GPP management services, and R1, TMF, A1 where it has them) reachable from your PC,
  and an API token or other login for them. For the change tests the token must allow changes.
- Its VES collector reachable from your PC.
- The right to add a network function (NETCONF) and an O-Cloud (O2ims), if you want to run the tests that need them.

You can start with only the SMO address and a token: every reading test works without the O-DU and O-Cloud, and the
tests that need them say so.

## 3. Install, update and remove

### Windows

1. Download `Kaldrivon-SMOTester-<version>-Setup.exe` from the release.
2. Run it. The installer and the program are signed by Kaldrivon.
3. Leave **Allow the SMO to reach the tester through Windows Firewall** ticked. It adds one inbound rule for
   TCP 9643, 1830, 1831 and 9644.
4. Start **Kaldrivon SMO Tester** from the Start menu or the desktop.

To remove it: **Settings › Apps › Kaldrivon SMO Tester › Uninstall**. Your settings, certificates and custom tests
stay in `%APPDATA%\Kaldrivon\SMOTester` until you delete that folder (or use
[Reset to defaults](#14-reset-to-defaults) before uninstalling).

### Linux

Debian, Ubuntu and similar:

```
sudo apt install ./kaldrivon-smotester_<version>-1_amd64.deb
```

RHEL, Rocky, AlmaLinux, Fedora, openSUSE and similar:

```
sudo dnf install ./kaldrivon-smotester-<version>-1.x86_64.rpm
```

Start **Kaldrivon SMO Tester** from the applications menu, or run `kaldrivon-smotester`. It opens in your default web
browser. The package brings its own Python; nothing else has to be installed. Open the ports above in the firewall
(for example `sudo firewall-cmd --add-port=9643/tcp --add-port=1830/tcp --add-port=1831/tcp --add-port=9644/tcp --permanent` and
`sudo firewall-cmd --reload`, or the matching `ufw allow` commands; the Integration & ports page shows them).

To remove it: `sudo apt remove kaldrivon-smotester` or `sudo dnf remove kaldrivon-smotester`. Your data stays in
`~/.config/kaldrivon-smotester`.

### Check the download

Every release has `.sha256` / `SHA256SUMS` files:

```
# Windows (PowerShell)
Get-FileHash .\Kaldrivon-SMOTester-<version>-Setup.exe -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS --ignore-missing
```

### Updates

**About › Check now** looks for a newer release on GitHub. Install the new version over the old one; your settings,
certificates, passwords and custom tests are kept, so the SMO does not need to integrate the O-DU and O-Cloud again.

### Starting twice

Only one tester can run on a PC, because the second would find all its ports taken. If you start it while it is
already running, it tells you so and stops. If no window is visible, a closed window may have left its process
running: end it in Task Manager (`KaldrivonSMOTester.exe`) and start again.

## 4. Quick start

1. **Connection** page: enter the **SMO address**, the northbound port and the API token, and the VES collector's
   port (and its login if it has one). Press **Save and check the connection**: the first O1 tests step through
   whether the SMO's ports answer and whether the tester's own listeners work.
2. **Integration & ports** page: the box at the top lists the minimum still missing. Copy the O-DU's details into the
   SMO (add a network function: host, NETCONF port, user name, password, managed element id) and the O-Cloud's
   details (O2ims address, token address, client id and secret, CA and client certificate). Open the ports with the
   firewall commands on the same page if the installer did not. Download **the tester's HTTPS certificate** there and
   add it to the SMO's trusted certificates, so the SMO can send its notifications to the tester.
3. Wait until the box shows ✓ for the O-DU (the SMO has opened a NETCONF session) and, after the SMO has contacted
   it, for the O-Cloud.
4. Choose an interface on the left, then **Run** a use case, or **Step** through it one test at a time.
   **Run all** / **Step through all** at the top run every use case of the interface.
5. Open a test to see what happened, then **Export** the results or open the **Report**.
6. When you have finished, open **Clean up** and press **Remove all**.

## 5. The window

- **Top menu:** Connection, Integration & ports, Variables, Custom tests, Transaction log, Traffic from SMO,
  Clean up, Glossary, Report, About.
- **Plain / Technical** (top right): Plain shows each test in everyday words and opens on the Overview; Technical opens
  on the request and shows the standard's terms. Underlined words explain themselves when you point at them; all of
  them are on the **Glossary** page.
- **SMO pill** (top right): the SMO address in use.
- **Interfaces** (left): each interface with how many of its tests passed.
- **Use cases and tests** (middle): the use cases of the chosen interface, each with **Step**, **Run** and **Export**,
  and its tests with their verdicts. This list scrolls separately from the test you look at.
- **Test view** (right): the chosen test, with **Run** / **Run again** and the tabs described in
  [Reading a result](#9-reading-a-result).
- **Run bar** (top, while something runs): what is running, and **Stop all**.
- **Step bar** (bottom, while stepping): **Run next ▶**, **Skip**, **Stop**, and *keep going without waiting*.

In the app window, files are saved through a Save dialog and HTML pages open in your web browser. Text anywhere in the
window can be selected and copied.

## 6. Connection page

Settings marked with a red **\*** are needed by the tests. Each section also shows, as an example only, where
Kaldrivon MiniSMO keeps the same setting; other SMOs have their own menus.

### The SMO

| Setting | Meaning |
|---|---|
| SMO address \* | IP address or host name of the SMO. |
| Which SMO is it? | *Another SMO* or *Kaldrivon MiniSMO*. With Kaldrivon MiniSMO, the Open Fronthaul cases with a step in the SMO also say where that step is in MiniSMO (see *The tester's O-RU*). The checks and verdicts are the same for every SMO. |
| Northbound scheme / port | How the SMO serves its REST APIs (3GPP management services, R1, TMF, A1-EI/ML), usually HTTPS. |
| Base path | Only if the SMO serves its APIs under a prefix (for example behind a gateway). |
| Northbound: how to log in \* | **bearer** (an API token), **basic** (user and password), **oauth2** (client credentials: token address, client id, secret, scope) or **none**. This is the *general login*, used for every interface unless that interface has its own (below). |
| Verify the SMO's certificate | Leave off for self-signed lab certificates. With it on, give a CA file if the SMO's CA is not a public one. |
| Client certificate / key | For an SMO that asks for mutual TLS. |

### VES collector

Scheme, port and login of the SMO's VES collector, where the tester sends O1 events. If the collector asks for a login
(as it should), enter it here; the tests that send wrong credentials check that it refuses them.

### A login for each interface

An SMO may give each API its own token: OAuth 2.0 tokens carry scopes (RFC 6749), CAPIF gives an rApp a token per
service API (3GPP TS 33.122, TS 29.222), and O-RAN security asks for authorisation per API. Tick **Own login** for
3GPP management services, R1, TMF or A1 to give that interface its own login; the others keep the general one.
**Same address as northbound** is on by default for R1, TMF and A1. Untick it when the SMO serves that API on its
own port or path: the page then shows the scheme, port, base path and certificate check for that interface (the
SMO address stays the one above).

### This PC (the tester)

| Setting | Meaning |
|---|---|
| Address the SMO uses to reach this PC \* | Filled in automatically with this PC's address toward the SMO. Type another one if the SMO reaches the PC through NAT, a VPN or a host name; clear it to go back to the automatic one. It goes into every callback address, the file download address, the O-DU's and O-Cloud's details and the tester's certificates. |
| HTTPS listener port | Where the SMO sends notifications, downloads the measurement file and streams data (9643). |
| Also listen on plain HTTP / HTTP port | Only for an SMO that calls `http://` callbacks (9680). |
| Strict standard conformance | Extensions (fields the standard does not define) also fail a test. |

**Save** stores the settings; **Save and check the connection** also steps through the O1 connection tests.

## 7. Integration & ports page

### The minimum to set before testing

A box at the top lists what the tests need and whether it is there:

| Item | Shows ✓ when |
|---|---|
| SMO address | it is set |
| The SMO answers at its address | the tester can open a connection to the SMO's northbound port (checked every few seconds) |
| Login for the northbound / VES collector | it is filled in and the SMO accepted it. The tester tries each login once a minute, and at once after you change it: for the northbound it reads the alarm count (it changes nothing); for the VES collector it sends an empty object, which is not an event, so the collector stores nothing and only says whether it accepts the login. ✗ *refused* when the SMO answers 401 or 403, **?** while it has not been tried (for example while the SMO does not answer) |
| Address the SMO uses to reach this PC | it is set and is not 127.0.0.1 while the SMO is elsewhere |
| The SMO accepts the tester's HTTPS certificate | an HTTPS request from the SMO got through with the current certificate. It shows ✗ when the SMO broke off the connection because it does not trust the certificate (naming the part of the SMO), and **?** until the SMO has called the tester at least once. In a test result, the row is about the part of the SMO that test waits for |
| The tester's O-DU added in the SMO | the SMO has an open NETCONF session to the O-DU right now |
| The tester's O-RU added in the SMO | the SMO has an open NETCONF session to the O-RU right now |
| The SMO takes the O-RU's call home | shown while call home is on for the O-RU: the SMO accepted the O-RU's last call home. When it did not, it says why: nothing listens on the call home port, no answer, or the SMO closed the connection during the SSH key exchange (it accepts call home only from NFs added, or approved, in call home mode, with the O-RU's host key) |
| The tester's O-Cloud added in the SMO | the SMO has contacted the O-Cloud and still answers |

The box refreshes itself every 5 seconds without disturbing the rest of the page. Each missing item says why, for
example *the SMO had a NETCONF session to it but has none now: is the SMO running?*

### The tester's O-DU (O1)

- **Turn on**, **managed element id**, **NETCONF port**, **user name** and **password** (made at random on first
  start; you can change it), and **VES measurements every (s)** (default 60; 0 turns them off). **Save and apply**
  restarts the O-DU with the new values.
- **Add it to the SMO with:** host, NETCONF port, NF type, user name and password, the managed object id
  (`ManagedElement=<id>`) and the SSH host key fingerprint — compare it with what the SMO shows before you accept it.
- **What the SMO did:** its NETCONF sessions, the VES destination it configured in the O-DU (and the heartbeat
  period), and the O-DU's active alarms.
- **Raise a test alarm** / **Clear the last alarm**, **Show the NETCONF messages**, and the VES events the O-DU sent.

See [The tester's O-DU](#17-the-testers-o-du) for what it is made of.

### The tester's O-RU (Open Fronthaul M-Plane)

- **Turn on**, **O-RU id**, **NETCONF port** (1831; the M-Plane default 830 also works where the system allows it),
  **user name** and **password** (made at random on first start, in the form o-ran-usermgmt asks for), **Call home**,
  **Call home to (host)** (empty: the SMO address), **Call home port** (4334), **RESTCONF path of this O-RU in the SMO**
  and **SFTP server port** (2222). **Save and apply** restarts the O-RU.
- **RESTCONF path:** where the SMO offers this O-RU's data and operations over RESTCONF (RFC 8040, the O-RU's models
  mounted per RFC 8528), relative to the SMO's RESTCONF root on its northbound address, or a full URL. For SMOs built
  on OpenDaylight it is `data/network-topology:network-topology/topology=topology-netconf/node=<O-RU name>/yang-ext:mount`.
  With it set, the Open Fronthaul cases that need an action in the SMO ask the SMO themselves, with the northbound
  login, and are no longer marked manual. Empty: they wait for you to do the step in the SMO. RESTCONF is an IETF
  standard; where an SMO places a managed O-RU in its RESTCONF tree is not standardised, hence the setting.
- **The tester's SFTP server:** the automatic software download, file upload and measurement file cases give the SMO
  this server (with its user name and password, shown on the card) as the file server; the O-RU then transfers to and
  from it. It holds a synthetic software file and keeps what is uploaded.
- **Password:** made at random on first start and kept between runs and updates. When the SMO changes it with
  chg-password (*The SMO changes a password*, ofh-um-02), the tester keeps the new one, so the SMO does not need it again.
  **Save** on the Integration page stores only the fields you changed, and restarts only the NF whose settings
  changed (a restart drops the SMO's sessions with it).
- **Add it to the SMO with:** host, NETCONF port, NF type O-RU, user name and password, and the SSH host key
  fingerprint. With call home on, add it in call home mode instead: the SMO listens on its call home port and the O-RU
  connects to it, and calls again whenever it has no call home session.
- **Add the O-RU only once.** Connected to and calling home are two ways to reach the same O-RU, with the same serial
  number; a second entry in the SMO duplicates it. The call home test runs only with Call home on (otherwise it is
  *not applicable*). To test both ways, test with one, then change the entry in the SMO to the other mode and switch
  **Call home** on the card to match.
- **Values that fit the tester's O-RU:** the U-Plane cases (processing element, transmit and receive carriers,
  endpoints and links) show a table of the fields to fill in and values the tester's O-RU accepts, for any SMO:
  the processing element on the O-RU's VLAN 100 sub-interface (fh-eth0.100), a 100 MHz n78 carrier at 3619.2 MHz
  (NR-ARFCN 641280), and endpoints named after the O-RU's static endpoints. Fields not in the table stay empty;
  parts that apply only to another transport type or to LTE (udpip-flow, aliasmac-flow, lte-tdd-frame) do not apply.
- **How to do it in Kaldrivon MiniSMO:** with *Kaldrivon MiniSMO* chosen on the Connection page, the cases for user
  accounts, U-Plane configuration, performance management, files, troubleshooting and trace, software management and
  operations show a box under the test purpose with the path in the MiniSMO desktop app, and the O-RU write permission
  the step needs in brackets. MiniSMO has no O-RU-specific screens: it offers general functions from each NF's own YANG
  models. U-Plane objects are created with FCAPS > Configuration > Configuration tab > NEW ENTRY… in the order
  processing element, endpoints, carriers, links. Measurements, with their measurement and notification intervals,
  are turned on in FCAPS > Performance > Measurements > Measurement control, and file uploads are pointed at MiniSMO
  from the O-RU's Monitoring > File-based PM (MiniSMO's PM file receiver must be on). *Configuration changes* is turned
  on in FCAPS > Configuration > Configuration tab > choose the O-RU > ENABLE CHANGES…, the other permissions in NF
  details > Write permissions. MiniSMO previews every change with the exact NETCONF payload, sends it only after you
  confirm Write, and shows the O-RU's rpc-error when it refuses one. The O-RU has to be integrated in MiniSMO
  (NETWORK > Network Functions > ADD EXTERNAL NF, or approved on its first call home) and connected. The box is help
  only: whether a case passes still depends only on what reaches the O-RU.
- **What the SMO did:** its NETCONF sessions (and which came by call home), supervision (interval, guard timer,
  resets, time left), the running software slot, the O-RU's accounts and active alarms, and the last operations the
  SMO sent.
- **Raise a test alarm** / **Clear the last alarm**, **Call home now** and **Show the NETCONF messages**. **Call home
  now** (only with Call home on) says at once whether the SMO took the connection and, if not, why; an SMO usually
  takes call home only from an O-RU added (or approved) in call home mode with its SSH host key. While the SMO already
  has the O-RU's call home session, it says so and opens no second connection.

See [The tester's O-RU](#18-the-testers-o-ru).

### The tester's O-Cloud (O2)

- **Turn on**, **O-Cloud name**, **O2ims port**, **OAuth client id** and **secret** (made at random), and **Require the
  SMO's client certificate** (mutual TLS, on by default).
- **Add it to the SMO with:** the O2ims address (apiRoot), the token address, the client id and secret, the
  provisioning template, and the certificates below.
- **CA certificate** (the SMO trusts the O-Cloud with it), **server certificate** of the O-Cloud (for information),
  **client certificate** and **client private key** for the SMO. Each has a button to make it again; see
  [Certificates and passwords](#20-certificates-and-passwords).
- **Raise an O-Cloud alarm** and **Change the inventory**, to make the O-Cloud notify the SMO.
- **Tokens:** an access token the O-Cloud issues is valid for an hour and is kept when the tester restarts. A token the
  O-Cloud does not know (or that has expired) gets 401 with `invalid_token`; the SMO is expected to get a new token and
  ask again. An O2 reading case waits 20 s for that retry and grades it, noting that the SMO renewed its token. Because the SMO may keep using a token it already has,
  *The SMO gets an access token* (o2-01) first has the O-Cloud end the tokens it issued earlier (*The O-Cloud ends its
  earlier access tokens*, o2-00); the SMO's next request then gets 401 and it has to get a new token.

See [The tester's O-Cloud](#19-the-testers-o-cloud).

### Firewall

The commands to open the tester's ports, for Windows (PowerShell, as administrator) and Linux (firewalld and ufw).
The Windows commands first remove an earlier rule of the tester and then add it again, so running them twice does no
harm.

Below them: **The tester's HTTPS certificate**. The SMO sends notifications, callbacks and streams to the tester, and
fetches measurement files from it, over HTTPS. The certificate is self-signed, so the SMO has to trust it:
**Download the tester's HTTPS certificate** and add it to the SMO's trusted certificates (an SMO on Windows may use the
Windows certificate store, *Trusted Root Certification Authorities*). The section shows whether the SMO accepts it
(✓ with the time of the last request that got through, ✗ with the reason the SMO gave, or **?** not confirmed yet), the
certificate's SHA-256 fingerprint, and **New HTTPS certificate**.

An SMO may keep one trust store for everything, or one for each interface or client (for example one for its 3GPP
notifications and another for its R1 deliveries). The section therefore lists each part of the SMO the tester has
seen: 3GPP MnS notifications, PM file download, streaming, R1 callbacks, A1 callbacks, TMF hubs, intent reports, MDA
reports. A request that gets through counts for the part its address belongs to. A refusal happens before any request
is sent, so it counts for the tests that were waiting at that moment; a refusal while no test was waiting is listed as
*not known which part* until a later request gets through. Neither design is a deviation: the standards ask for
certificate validation on these interfaces, not for one trust store.

The tester uses this one certificate for everything the SMO fetches or sends over HTTPS; the O-RU and the SFTP server
use SSH keys, not this certificate. Some SMOs check a file server first against their trust store and, when that
fails, against a fingerprint confirmed for that NF in the SMO. The tester then sees a refusal followed by the download:
the part shows ✓ once the download gets through. A case that sees a refusal waits 20 s for such a second connection
before it fails.

The tester does not check this on its own: a part's state changes only when that part of the SMO next connects to the
tester. After you trust the certificate or confirm a fingerprint in the SMO, run a case that makes that part connect
(for the PM file download: *The SMO fetches the measurement file*, o1-file-02). When it stays ✗, add the certificate to the trust store that part of
the SMO uses, or confirm the fingerprint shown here where the SMO asks for it.

Example, Kaldrivon MiniSMO (the 3GPP PM file download; the confirmation is on the O-DU, not the O-RU):

1. Go to *Network Functions*, open the O-DU, then the *Monitoring* tab.
2. Scroll to the *File-based PM* section. Under the toolbar (**SETTINGS…**, **POINT UPLOADS AT MINISMO…**, …) there is
   a row that reads "TLS server host:port: identity not confirmed, nothing was sent to it", with the fingerprint and
   the time it was seen.
3. Before you click **TRUST…**, check that the fingerprint matches the one the SMO Tester shows for its HTTPS
   certificate. Then confirm.
4. From then on, MiniSMO accepts that server for this NF's file downloads. It refuses the server if its identity ever
   changes.

### Export the integration messages

The O-DU's NETCONF messages and VES events, and the O-Cloud's requests from the SMO and its notifications, as HTML
or JSON.

## 8. Running tests

### Run, Step and Stop

- **Run** (on a use case) runs its tests one after the other; **Run all** runs every use case of the interface.
- **Step** / **Step through all** runs one test at a time and waits for you: look at the result, then **Run next ▶**.
  **Skip** passes over a test; tick *keep going without waiting* to let it continue on its own.
- **Run** / **Run again** in the test view runs just that test.
- **Stop all** in the run bar stops everything.

When you run a use case again, its tests show *queued* until their turn comes, so you see how far the run is; the
test you look at also shows its result from last time until it runs.

Use cases that do not share anything in the SMO can run at the same time (for example two interfaces). Ones that would
disturb each other (for example two that raise and clear alarms) wait for each other, and some security tests run on
their own.

### Tests that wait for the SMO

Some tests wait for the SMO to do something: send a notification, download the measurement file, open a NETCONF
session, send a provisioning request. Most of them make the SMO do it themselves first, for example by sending it an
alarm. A few need an action by you in the SMO; the test says exactly which (for example *In the SMO, acknowledge the
O-Cloud alarm*), and also counts an action you did a little early, while the step before was still running.

### Values found in the SMO

Tests that need a name (the managed element, an object to change, an alarm, an rApp id) get it first: from the
tester's own O-DU, or by running the test that finds it in the SMO. You do not need to type DNs; the
[Variables](#10-variables-page) page shows every value and where it came from.

### Tests that depend on another test

A test that works on an object another test creates (for example reading back a subscription) runs that test first
if needed. If it did not work, the test is *not applicable* and says which test it needed and how that one ended.

The tester also removes its alarm subscription (3GPP fault management) from an earlier run before it subscribes again, and the intent case puts its report subscription in place before it waits. The tester's rApp removes the data registration it made in an earlier run before it registers again. If the SMO still
holds one the tester has no address for (the SMO answers 409, and R1AP has no way to list registrations), the tester
registers the next version of its synthetic data type instead, keeps that version, and says so in the result; the old
registration can be removed in the SMO.

If something a test needs was removed in the SMO in the meantime (a subscription, a hub, a registration), the test
that gets *404 Not Found* makes it again with the test that creates it and repeats once. Its result starts with
*The object was missing in the SMO (HTTP 404), so the tester made it again…*

### When a setting or integration is missing

A test that does not pass while something from the minimum list is missing starts its result with **Check first:**
and names it, for example *Check first: The tester's O-DU added in the SMO (the SMO has not connected to it yet) —
Integration & ports page*, or *Check first: The SMO answers at its address (no answer on 192.0.2.10:8770): is the SMO
running?*

A test that waits for the SMO to call the tester and gets nothing says *Check first: The SMO accepts the tester's
HTTPS certificate* while that is not confirmed yet. When the SMO does try and breaks off because it does not trust the
certificate, the test fails at once, with the SMO's reason (for example *TLSV1_ALERT_UNKNOWN_CA*), instead of waiting
until it times out.

### Other situations a result explains

- **429 Too Many Requests:** the SMO asked the tester to slow down, often just after the wrong-credential tests. The
  tester waits as long as the SMO asks (at most 20 seconds) and tries again, up to twice, before the test is
  inconclusive.
- **An optional service the SMO does not offer** (for example A1-ML answering 404 on the whole service) makes its
  tests *not applicable*, not failed.
- **403 on a change** (the SMO's own settings or the token do not allow it) makes the test inconclusive, with what to
  allow in the SMO.
- **A test that would change something other than the tester's own objects** is *not applicable*.

## 9. Reading a result

### Verdicts

| Verdict | Meaning |
|---|---|
| **PASS** | The SMO did what the standard says. |
| **FAIL** | It did not: a wrong answer, a deviation from the standard, or something that never came. |
| **INCONC** | Inconclusive: the test could not decide (for example the SMO asked to slow down, or did not allow a change). |
| **ERROR** | The test could not run (for example the SMO could not be reached). |
| **NONE** | Not run, or not applicable (for example it needs the tester's O-DU, which is not in the SMO). |
| *queued* | Waiting for its turn in the current run. |

### The tabs

- **Overview:** what happens, in plain words and as arrows (*the tester, acting as the operator's OSS, asks the SMO
  for its alarm list → the SMO answers OK, here are 3 alarms*), the verdict and what it means.
- **Request:** the exact `curl` command and the raw HTTP request; copy the command to repeat it yourself (the token
  is read from `$SMO_TOKEN`). For a test that does several steps, every message it exchanged.
- **Response:** the SMO's answer, formatted and raw (for a waiting test: what the SMO sent to the tester).
- **Result:** the verdict and every check, each with what was expected and what came back.
- **Analysis:** expected against actual, the likely cause, and every deviation from the standard with the field and
  the rule.

Checks marked *advisory* are reported but do not change the verdict. **Add to my tests** turns the test into a custom
test you can change (see [Custom tests](#15-custom-tests)).

## 10. Variables page

- **Values in use now:** every value the prepared requests use (for example the managed element's DN), with where it
  comes from: the tester's own O-DU, found in the SMO by an earlier test, typed by you, or the default.
- **Your values (override):** type a value only to override it; clear the field to go back.
- **Found in the SMO by earlier cases:** values tests captured (ids, addresses). You can correct one, or **Forget all
  found values**.

## 11. Transaction log, traffic and exports

- **Transaction log:** every run, newest first; click a row to open that test with its request, response and result.
  Export it as HTML or JSON (passwords and tokens are hidden).
- **Traffic from SMO:** everything the SMO sent to the tester (notifications, file downloads, streaming), newest
  first, with the raw message. Export it as HTML or JSON, or **Clear** it.
- **Export** next to a use case or at the top of an interface: that part's results as HTML or JSON, or open it as an
  HTML report in the browser.
- **Integration & ports › Export the integration messages:** the O-DU's NETCONF messages and VES events and the
  O-Cloud's requests and notifications.

## 12. Report page

The report lists every test with its verdict, what happened in plain words, the checks, and for tests that did not
pass the request, response and messages exchanged. **Open the full HTML report** opens it in the browser;
**Download HTML** / **Download JSON** save it. **Clear all results** empties the results and the transaction log.

## 13. Clean up after testing

The **Clean up** page lists what the tests left in the SMO.

- **Objects the tester created in the SMO:** subscriptions, notification hubs, measurement and trace jobs, intents,
  analytics requests, the rApp's and the API provider's registrations — everything the SMO accepted from the tester,
  recorded with its address. **Remove** deletes one through the SMO's own API; **Remove all** deletes them all, newest
  first. One that is already gone counts as removed. A test that deletes its own object takes it off the list.
- **Alarms raised by the tester's O-DU and O-Cloud:** **Clear the O-DU's alarms** and **Clear the O-Cloud's alarms**
  send the standard clear (a VES fault event with severity NORMAL; an O2ims alarm notification with severity
  CLEARED), so they also clear in the SMO.
- **To do by hand in the SMO:** what the tester cannot or must not do itself —
  - jobs the SMO created on the O-DU on its own;
  - provisioning requests the SMO sent to the O-Cloud (they belong to the SMO);
  - settings you turned on only for testing (allowed changes on the O-DU, alarm actions on the O-Cloud, trusted
    certificates and file servers for the O-DU's measurement files);
  - when you have finished testing for good: the tester's O-DU, O-Cloud and the API token you made for it.
- **Forget the list:** when the SMO's data was wiped, so the objects no longer exist; nothing is sent.

## 14. Reset to defaults

**About › Reset to defaults…** puts the tester back to how a new installation starts.

1. With **Clean up the SMO first** (ticked by default) it removes the objects it created in the SMO and clears its
   O-DU's and O-Cloud's alarms.
2. It deletes its settings, the O-DU's and O-Cloud's passwords and certificates, the tester's HTTPS certificate,
   results and logs. With **Keep my custom tests** (ticked by default) the custom tests stay; export them first if you
   untick it.
3. It starts again with defaults: new passwords, new certificates, the O-DU and O-Cloud running.
4. It lists what is left to do by hand in the SMO — including that the SMO still holds the O-DU's and O-Cloud's old
   details: remove them there, or add them again with the new details from Integration & ports.

## 15. Custom tests

On the **Custom tests** page you write your own tests in [Robot Framework](https://robotframework.org/) keyword
language. They use the same connection, the same standards checks and the same listener as the prepared tests, and
each step shows its `curl` command, raw request and response in Robot's log. Three examples are created the first
time.

- **New** makes a file; click a keyword to insert it at the cursor; **Save**, **Save and run** or **Delete**. The
  result shows Robot's own log and report.
- **Add to my tests** on any prepared test makes a custom test from it.
- **Export all** saves every custom test file in one zip; **Import…** takes a `.robot` file or such a zip. A file
  whose name exists already is kept as *name (2).robot*, so nothing is overwritten.

All keywords and templates: [Custom tests](docs/CUSTOM-TESTS.md).

## 16. The interfaces

| Interface | The tester plays | What is tested |
|---|---|---|
| O1 | a network function (an O-DU) | connection; VES events of every domain; 3GPP notifications sent as VES; batches; malformed events; file-ready notification and file download; once integrated: the SMO's NETCONF session, YANG library read, notification subscription, VES destination (3GPP NtfSubscriptionControl) and the O-DU's alarms |
| 3GPP management services | the operator's OSS | finding what the SMO manages; fault supervision (alarm list, acknowledge, comment, subscriptions, notifications); provisioning read and change; configuration change notifications; heartbeat; performance jobs and files; streaming; trace and MDT |
| R1 | an rApp | CAPIF onboarding, discovery, publishing, events and security tokens; R1AP data discovery, access and registration; RAN OAM through R1; A1 policy types and status subscriptions |
| A1 | a Near-RT RIC | A1-EI enrichment jobs and data delivery; A1-ML model training |
| Open Fronthaul M-Plane | an O-RU (hybrid model) | once integrated: NETCONF session and call home; YANG library and get-schema; notification subscription; supervision watchdog; user accounts and password change; reads of inventory, interfaces, capabilities, U-Plane, software, operational information, sync and delay; processing element, carriers, endpoints and links; carrier state; synchronisation change; alarms; measurements and measurement files; file list, troubleshooting and trace logs, file upload; software download, install, activate; reset |
| O2 | an O-Cloud | once integrated: OAuth 2.0 token and mutual TLS; inventory reads; inventory and alarm subscriptions; alarm notifications; acknowledging and clearing alarms; provisioning requests |
| TM Forum Open APIs | the OSS/BSS | TMF642 alarms, TMF639 resource inventory, TMF628 performance |
| Intent-driven management | the operator's OSS | intents about the tester's own cells, intent reports (TS 28.312) |
| Management data analytics | an analytics consumer | analytics requests about the tester's O-DU and their reports (TS 28.104) |
| Interface security | an unauthorised or outdated client | missing and wrong credentials, plain HTTP, old TLS versions, security headers, on every endpoint |

Not tested, and shown in the app with the reason: A1 policies on a Near-RT RIC (they need a Near-RT RIC the SMO
manages), and the interfaces the SMO does not take part in.

Some notes on what the tests use:

- **Intents** concern only the tester's own cells: the intent names their PLMN (999-70, a test value no real network
  uses) and downlink frequency as object contexts (TS 28.312). An SMO that does not support object contexts refuses
  the intent; that is reported as the SMO's finding.
- **Analytics** requests are about the tester's O-DU only, and the report is read from the name the SMO gives in its
  notification.
- **Streaming** measurement jobs use a granularity period the SMO states as supported for streaming
  (`supportedPerfMetricGroups`); with none listed, they are not applicable.
- **R1 onboarding** uses a new key each run, and first removes the rApp the tester onboarded in an earlier run.
- **Provisioning requests** on the O-Cloud use the template shown on Integration & ports (O2IMS R005 v13.00 has no API
  that lists templates, so you enter it in the SMO).

The complete list, with every test's standard clause: [test catalog](docs/TEST-CATALOG.md).

## 17. The tester's O-DU

What the SMO sees when it manages the tester as a network function. All data is synthetic.

- **NETCONF** over SSH (RFC 6241, RFC 6242), with notifications (RFC 5277), YANG library (RFC 8525), monitoring
  (RFC 6022) and a datastore that validates every change against the models.
- **Models:** 3GPP Rel-18 NRM YANG, unmodified: the managed element, the gNB-DU function, an NR cell (`NRCellDU=1`)
  with its sector carrier, alarm list, notification subscription control with heartbeat control, measurement jobs,
  trace jobs and files under the managed element.
- **Identity:** `ManagedElement=SMOTESTER-ODU-1` (changeable), PLMN 999-70, one cell on a downlink ARFCN of 632628.
- **Supported measurements:** RRU.PrbTotDl/Ul and DRB.UEThpDl/Ul, granularity periods 60, 300 and 900 s, file-based
  and stream-based reporting (stated in `SupportedPerfMetricGroups`).
- **VES:** once the SMO has written its VES destination into the O-DU (3GPP NtfSubscriptionControl), the O-DU sends
  its events and heartbeats there; before that, the tests send to the collector on the Connection page.
- **Measurements by VES:** every 60 s (setting on Integration & ports) the O-DU reports its cell's measurements
  (RRU.PrbTotDl/Ul, DRB.UEThpDl/Ul, synthetic values) as a VES perf3gpp event. An SMO that serves streaming jobs from
  the measurements it collects needs them: without measurement values it has nothing to stream and does not open the
  streaming connection.
- **Alarms** it raises are listed in its alarm list and sent as VES fault events; clearing them sends severity NORMAL.

## 18. The tester's O-RU

What the SMO sees when it manages the tester as an O-RU over the Open Fronthaul M-Plane, in the hybrid model (the SMO
manages the O-RU directly). All data is synthetic; file transfers are real.

- **NETCONF** over SSH (RFC 6241, RFC 6242) with notifications (RFC 5277), YANG library (RFC 8525) and get-schema
  (RFC 6022), and **NETCONF call home** over SSH (RFC 8071): the O-RU connects to the SMO's call home port and acts as
  the SSH server on that connection. With call home on, it calls again every re-call-home-no-ssh-timer (60 s) while
  the SMO has no call home session.
- **Models:** the O-RAN WG4 M-Plane YANG models, unmodified, with the IETF modules they use. Every change the SMO
  writes is checked against them; what the SMO sends to the O-RU's operations (watchdog reset, software, files,
  logs, password change) is checked against the operation's input, and an unknown or missing element is refused.
- **Identity:** a category A O-RU, 4T4R, NR band n78 (3300–3800 MHz), carriers up to 100 MHz at 30 kHz, eCPRI over
  Ethernet (interface `fh-eth0`), PTP G.8275.1, locked.
- **Accounts:** the user on the Integration page, plus any the SMO adds (o-ran-usermgmt); passwords are never
  returned on a read (NACM default-deny-all). Password accounts only: SSH public-key accounts are not offered (the
  O-RU does not announce that feature). NETCONF access control (ietf-netconf-acm) holds the M-Plane groups sudo, nms,
  fm-pm, swm, smo and hybrid-odu, with the tester's account in sudo; the groups are kept but access is not restricted.
- **Supervision:** after each supervision-watchdog-reset the O-RU sends supervision-notification at the requested
  interval; when interval + guard timer pass without a reset, it records a supervision failure.
- **Configuration:** the SMO's processing elements, carriers, endpoints and links are kept. A carrier set ACTIVE
  becomes READY, an inactive one DISABLED, and the O-RU notifies the change.
- **Alarms:** fault 1 (unit temperature is high) and fault 9 (synchronisation error), synthetic, sent as alarm-notif
  and listed in the active alarm list.
- **Measurements:** rx-window measurements (RX_ON_TIME, RX_EARLY, RX_LATE, RX_TOTAL); while active, results are sent
  at the notification interval (measurement-result-stats) and, with file upload on, uploaded as files to the SMO's
  SFTP server at the file upload interval.
- **Files and logs:** logical paths o-ran/log, o-ran/pm, o-ran/transceiver, o-ran/uplane-conf, o-ran/beamforming.
  Troubleshooting and trace logs are reported when collection stops (troubleshooting logs also 30 s after start).
  Files are uploaded to, and downloaded from, the SMO's SFTP server (sftp:// with a password; certificate login is
  not offered).
- **Software:** three slots (swSlot1 running, swSlot2 empty, a read-only recovery slot). The SMO downloads a file
  (any file; it is only stored), installs it into the inactive slot, activates it, and after a reset the O-RU runs it.
- **Reset:** the sessions end, the O-RU is away for a few seconds and comes back with restart cause
  MPLANE-TRIGGERED-RESTART (and calls home if that is on).

Most Open Fronthaul cases wait for something the SMO does. With the RESTCONF path set (Integration & ports), the
tester asks the SMO to do each step over RESTCONF: add, and later remove, an account; change the password and set it
back; configure a processing element, carriers, endpoints and links; activate measurements and file upload; list
files; collect troubleshooting and trace logs; upload a file; download, install and activate software; reset. It
then checks what reaches the O-RU. Without the path, those cases are marked **(manual)** and say what to do in the
SMO; the others pass as soon as the SMO has done it (reading the inventory, for example, usually happens when it
connects). A RESTCONF request the SMO refuses fails the case with the SMO's answer.

## 19. The tester's O-Cloud

What the SMO sees when it adds the tester as an O-Cloud (O2IMS R005 v13.00). All data is synthetic.

- **Security:** HTTPS with mutual TLS (the SMO presents the client certificate from Integration & ports) and OAuth 2.0
  client credentials at the token address.
- **Inventory** (v2): the O-Cloud, resource types, a resource pool with a server resource, deployment managers and
  sites; inventory change notifications to the SMO's subscriptions.
- **Monitoring** (v2): alarm list, alarm dictionary, alarm subscriptions and notifications (new, changed, cleared);
  acknowledging and clearing alarms by the SMO.
- **Provisioning** (v1): provisioning requests with the template `smotester-node-cluster` version `1.0.0`; other
  templates are refused with 422 as the standard asks. Values to enter in the SMO (also on Integration & ports):
  name `smotester-provisioning-1`, description `SMO Tester provisioning test (synthetic)`, templateName
  `smotester-node-cluster`, templateVersion `1.0.0`, template parameters `{}`; the SMO makes the
  provisioningRequestId. A request goes from PENDING to PROGRESSING to FULFILLED within
  a few seconds.

## 20. Certificates and passwords

Made once and kept, also across updates:

- the O-DU's NETCONF password and SSH host key;
- the O-RU's NETCONF password and SSH host key;
- the O-Cloud's client id and secret, its CA, its server certificate and the client certificate and key for the SMO;
- the tester's HTTPS certificate (self-signed, for the address the SMO uses to reach this PC).

When this PC's address changes (another network, a VPN) while **Address the SMO uses to reach this PC** is filled in
automatically, the tester's HTTPS certificate and the O-Cloud's server certificate are made again for the new
address. The server certificate is signed by the same CA and the SMO's client certificate is kept, so an SMO that
trusts the CA needs no change; an SMO that trusts the tester's HTTPS certificate itself needs the new one. To avoid
this, type a fixed address or a host name in that setting.

To make one again on purpose, use the button under it on Integration & ports:

| Button | Changes | Update in the SMO |
|---|---|---|
| New CA (and all O-Cloud certificates) | CA, server certificate, client certificate and key | CA certificate, client certificate and private key |
| New server certificate (same CA) | the O-Cloud's server certificate | nothing |
| New client certificate and key | the SMO's client certificate and key | client certificate and private key |
| New HTTPS certificate | the tester's HTTPS certificate | the certificate (or fingerprint) the SMO trusts for the tester |

Each asks first, and the listener restarts with the new certificate at once. Passwords and secrets can be changed on
Integration & ports and take effect with **Save and apply**.

## 21. Command line

```
kaldrivon-smotester                         # open the app
kaldrivon-smotester --no-window             # start without a window and print the link
kaldrivon-smotester --port 8790             # the window's port on 127.0.0.1
kaldrivon-smotester list                    # interfaces and number of tests
kaldrivon-smotester run --interface 3gpp-mns [--use-case "Fault supervision"] [--include-waits] \
                        [--max-wait 120] [--report report.html]
kaldrivon-smotester robot my-tests/ --outputdir out    # custom tests; exit code = failed tests
kaldrivon-smotester --version
```

`run` uses the settings made in the app. Without `--include-waits` it skips the tests that wait for the SMO;
`--max-wait` limits how long each of those waits. On Windows, use the app's pages for the same things.

## 22. Your data and security

- The window is served on 127.0.0.1 only, with a key made at each start, so other computers and other users' browser
  tabs cannot use it.
- Settings, passwords, tokens, certificates, results and custom tests stay on your PC, in
  `%APPDATA%\Kaldrivon\SMOTester` (Windows) or `~/.config/kaldrivon-smotester` (Linux). Passwords and tokens are
  stored there in readable form so the tester can use them: protect the folder like any file holding credentials.
- The `curl` commands, raw requests, exports and reports show the token as `$SMO_TOKEN` (or `$SMO_BASIC`), not its
  value; set that variable in your shell to run a copied command, or use **show it in the command** above the command when you
  need it. The integration details on screen show the O-DU password and O-Cloud secret on purpose, so you can copy
  them into the SMO.
- The tester contacts only the SMO, and GitHub when you press **Check now** for updates. It sends no usage data.
- The tester's listeners (9643, 1830, 1831, 9644) accept connections from the network, as the SMO must reach them.
  Open them only toward the SMO where you can, and turn off the O-DU, O-RU or O-Cloud on Integration & ports when you
  do not need them. With call home on, the O-RU also connects out to the SMO (port 4334).

## 23. Troubleshooting

| What you see | What to do |
|---|---|
| *Kaldrivon SMO Tester is already running* | Use the open window, or end the leftover process in Task Manager (`KaldrivonSMOTester.exe`) and start again. |
| A listener shows *failed … only one usage of each socket address* | Another program (or a second tester) uses that port. Close it, or change the port on Connection / Integration & ports and in the firewall rule. |
| *Check first: The SMO answers at its address* | The SMO is not running, the address or port is wrong, or a firewall blocks the way from your PC to the SMO. |
| 401 or 403 on every test | The token is missing, expired, or lacks the rights; for change tests the token must allow changes. Check **A login for each interface** if the SMO uses a token per API. |
| A waiting test fails with *the SMO … refused the tester's certificate* | The SMO tried to call the tester but does not trust its HTTPS certificate. Download it on Integration & ports, add it to the SMO's trusted certificates, then run O1 › Connection check › *The SMO accepts the tester's HTTPS certificate*. |
| Waiting tests time out, no notification arrives | The SMO cannot reach your PC: check **Address the SMO uses to reach this PC** (not 127.0.0.1, not a VPN address the SMO cannot route to), the firewall rule for 9643, and whether the SMO trusts the tester's HTTPS certificate. **Traffic from SMO** shows whether anything arrived. |
| The O-DU stays ✗ | In the SMO, check host, port 1830, user name and password, and accept the SSH host key shown on Integration & ports. Check the firewall. The O-DU shows ✓ only while the SMO keeps its NETCONF session open. |
| The SMO keeps asking for the O-RU's password | Compare the password on the O-RU card with the one in the SMO. The SMO's own chg-password changes it and the tester keeps the new one; if the SMO did not keep it, type the card's password in the SMO. |
| The O-RU stays ✗ | As for the O-DU, with port 1831 and the O-RU's user name and password. With call home on, the SMO must listen on the call home port shown on the O-RU card (4334); **Call home now** shows the result at once. |
| The SMO shows the O-RU twice, or complains about a duplicate serial number | The O-RU was added once to be connected to and once in call home mode. Keep one entry, in the mode that matches **Call home** on the O-RU card. |
| An Open Fronthaul case waits | Most wait for something the SMO does. A **(manual)** case says which step to do in the SMO; the others pass as soon as the SMO has done it (the reads usually happen when it connects). |
| The O-Cloud stays ✗ or O2 tests fail at the token | Check the O2ims address, token address, client id and secret, and that the SMO uses the client certificate and key and trusts the CA from Integration & ports. |
| The SMO stops trusting the tester after a network change | The HTTPS or O-Cloud server certificate was made again for the new address; see [Certificates and passwords](#20-certificates-and-passwords). |
| Many tests suddenly INCONC with 429 | The SMO's rate limit after the wrong-credential tests. Wait a minute and run again; the tester already waits as asked. |
| A change test is *not applicable* | It would change something other than the tester's own O-DU: add the O-DU to the SMO. |
| A change test is INCONC with 403 | The SMO does not allow changes to the O-DU (its own settings or the token). Allow it in the SMO to test writing. |
| The measurement file is not downloaded | The SMO usually fetches only from servers it allows for that network function and whose certificate it trusts: allow this PC and trust the tester's HTTPS certificate in the SMO's file settings for the O-DU. |
| The app window does not open on Windows | The WebView2 runtime is missing; install it from Microsoft. The tester then opens in the window; until then it opens in your browser. |
| *Loading…* stays in the middle of the window | Close all testers (Task Manager) and start again; if it stays, contact support. |

## 24. Support and legal

[kaldrivon.com](https://kaldrivon.com) · info@kaldrivon.com

- [Release notes](docs/RELEASE-NOTES-0.1.0.md)
- [Test catalog](docs/TEST-CATALOG.md)
- [Custom tests](docs/CUSTOM-TESTS.md)
- End-user licence agreement (EULA.txt) and third-party notices (THIRD-PARTY-NOTICES.txt), in the installation folder
  and with each release.

© 2026 Kaldrivon. Kaldrivon SMO Tester is not affiliated with or endorsed by 3GPP, O-RAN ALLIANCE, TM Forum or the
Linux Foundation. The test cases follow the published specifications; passing them is not a certification.

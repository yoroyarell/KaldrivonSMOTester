# Kaldrivon SMO Tester

Kaldrivon SMO Tester checks whether an SMO (Service Management and Orchestration) speaks its standard interfaces the
way the standards say it should. You enter the SMO's address and a token, open a few ports on your PC, and run
prepared test cases interface by interface. To test how the SMO manages network functions and O-Clouds, you add the
tester to the SMO as an O-DU and as an O-Cloud; the tester plays both itself.

The tester never changes anything else the SMO manages. It reads what the SMO already has (to find real names
instead of made-up ones), and the cases that change something act only on the tester's own O-DU, its own O-Cloud and
its own requests in the SMO. A case that would touch anything else is shown as not applicable.

It is meant for people who build, buy or operate an SMO, and it is written so that someone who is not an interface
specialist can still follow what each test does and why it passed or failed.

**Download:** [latest release](https://github.com/yoroyarell/KaldrivonSMOTester/releases/latest) ·
**Website:** [kaldrivon.com](https://kaldrivon.com)

## What you see for every test

- **Overview**: what happens, in plain words and as arrows ("the tester, acting as the operator's OSS, asks the SMO
  for its alarm list → the SMO answers *OK, here are 3 alarms*"), the verdict and what it means.
- **Request**: the exact `curl` command and the raw HTTP request. You can copy the command and run it yourself.
- **Response**: the SMO's answer, formatted and raw.
- **Result**: the verdict and every check that was made.
- **Analysis**: what was expected against what came back, the likely cause, and any deviation from the standard.

Verdicts: **PASS**, **FAIL**, **INCONC** (inconclusive: the test could not decide, for example because the SMO asked
the tester to slow down), **ERROR** (the test could not run, for example the SMO could not be reached), **NONE** (not
run, or not applicable: for example a case that needs the tester's O-DU when it is not added to the SMO).

A **Plain / Technical** switch keeps jargon out of the way. Underlined words explain themselves when you point at them.
**Step** runs a use case one test at a time and waits for you after each one.

## Checked against the standards

Every answer and notification from the SMO is compared with the published definitions:

- 3GPP Release 18: TS 28.532, TS 28.623, TS 28.111, TS 28.312, TS 28.104, TS 28.550, TS 29.222 (CAPIF)
- O-RAN: R1AP R005 v12.00, A1AP R005 v07.00, O2IMS R005 v13.00
- the tester's own O-DU uses the 3GPP Rel-18 YANG models (TS 28.541, TS 28.623)
- TM Forum: TMF642 v4 (alarms), TMF639 v4 (resource inventory), TMF628 v5 (performance)
- VES event listener 7.2.1

A missing mandatory field, a wrong type, a value that is not allowed, or an HTTP status or content type the standard
does not give for that operation is a **deviation** and fails the test. A field the standard does not define is
reported as an **extension**; it fails the test only if you switch on Strict mode on the Connection page.

## Interfaces

| Interface | The tester plays | What is tested |
|---|---|---|
| O1 | a network function (an O-DU) | connection, VES events of every domain, 3GPP notifications sent as VES, batches, malformed events, file-ready notification and file download; once integrated: the SMO's NETCONF session, YANG library read, subscription, VES destination (3GPP NtfSubscriptionControl) and the O-DU's alarms |
| 3GPP management services | the operator's OSS | finding what the SMO manages, fault supervision, provisioning (read and change), configuration change notifications, heartbeat, performance jobs and files, streaming, trace and MDT |
| R1 | an rApp | CAPIF onboarding, discovery, publishing, events and security tokens; R1AP data discovery, access and registration; RAN OAM; A1 policy management |
| A1 | a Near-RT RIC | A1-EI enrichment jobs and data delivery, A1-ML model training |
| O2 | an O-Cloud | once integrated: OAuth 2.0 token and mutual TLS, inventory reads, inventory and alarm subscriptions, alarm notifications, acknowledging and clearing alarms, provisioning requests |
| TM Forum Open APIs | the OSS/BSS | TMF642, TMF639, TMF628 |
| Intent | the operator's OSS | intents and intent reports (TS 28.312) |
| Management data analytics | an analytics consumer | analytics requests and reports (TS 28.104) |
| Interface security | an unauthorised or outdated client | missing and wrong credentials, plain HTTP, old TLS versions, security headers, on every endpoint |

The Open Fronthaul M-Plane (the SMO managing O-RUs) and the interfaces the SMO does not take part in are listed in the
app with the reason they are not tested.

Tests that wait for the SMO to call back (notifications, files, streaming) first make the SMO do something themselves,
for example by sending it an alarm. A few O2 cases wait for an action in the SMO (for example acknowledging an O-Cloud
alarm); they say which.

Values the tests need (the DN of a managed element, an object to change, an alarm) are read from the SMO first by the
cases under **3GPP management services › Find what the SMO manages**. They run by themselves when a case needs them.

Cases that change something are marked **tester's O-DU only**, or **own request** for the tester's own subscriptions
and requests in the SMO. They run without asking, because they never change anything else.

Use cases that do not share anything in the SMO can run at the same time. **Stop all**, in the bar that appears at the
top while something runs, stops every run.

## Install

### Windows 10 / 11 (64-bit)

1. Download `Kaldrivon-SMOTester-0.1.0-Setup.exe` from the release.
2. Run it. The installer and the program are signed by Kaldrivon.
3. Leave **Allow the SMO to reach the tester through Windows Firewall (TCP 9643, 1830, 9644)** ticked so the SMO can
   send notifications to the tester and manage its O-DU and O-Cloud.

Needs the Microsoft Edge WebView2 runtime, which Windows 10 and 11 normally already have.

### Linux (x86_64)

Debian, Ubuntu and similar:

```
sudo apt install ./kaldrivon-smotester_0.1.0-1_amd64.deb
```

RHEL, Rocky, AlmaLinux, Fedora, openSUSE and similar:

```
sudo dnf install ./kaldrivon-smotester-0.1.0-1.x86_64.rpm
```

Start **Kaldrivon SMO Tester** from the applications menu, or run `kaldrivon-smotester`. On Linux it opens in your
default web browser. The package brings its own Python, so nothing else has to be installed (glibc 2.28 or newer).

To remove it: `sudo apt remove kaldrivon-smotester` or `sudo dnf remove kaldrivon-smotester`. Your settings and
custom tests stay in `~/.config/kaldrivon-smotester`.

### Check the download

Each release has `.sha256` / `SHA256SUMS` files. Compare them with your download:

```
# Windows (PowerShell)
Get-FileHash .\Kaldrivon-SMOTester-0.1.0-Setup.exe -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS --ignore-missing
```

## First steps

1. **Connection**: enter the SMO's address and its API token, then press **Save and check the connection**. This
   steps through the first O1 tests: can the SMO's ports be reached, and do the tester's own listeners work. Each
   section shows, as an example, where Kaldrivon MiniSMO keeps the same setting. **Address the SMO uses to reach this
   PC** is filled in with this PC's own address; change it if the SMO reaches the PC another way (NAT, a name).
2. **Integration & ports**: turn on the tester's O-DU and O-Cloud and add them to the SMO with the details shown there
   (address, ports, user name and password, certificates). Open the ports in this PC's firewall with the commands on
   the page: **TCP 9643** (HTTPS: notifications, files, streaming), **1830** (NETCONF, the O-DU) and **9644** (O2ims,
   the O-Cloud); optionally 9680 (plain HTTP).
3. Pick an interface in the menu and press **Run** on a use case (or **Step** to go one test at a time). The list of
   cases and the case you look at scroll separately, so **Run** stays in view.
4. **Export** next to a use case (or at the top of an interface) saves its results as HTML or JSON; **Report** does the
   same for everything. The **Transaction log** and **Traffic** pages show every message.

Settings are kept in `%APPDATA%\Kaldrivon\SMOTester` (Windows) or `~/.config/kaldrivon-smotester` (Linux).

## Your own tests

On the **Custom tests** page you can write your own tests in [Robot Framework](https://robotframework.org/) keyword
language. Three examples are created the first time the app starts. Every prepared test can be turned into a Robot
test with **Add to my tests**, so you can start from it and change it. Your tests use the same connection, the same
standards checks and the same listener as the prepared ones.

```robotframework
*** Settings ***
Library    smotester.robotlib.SMOTester

*** Test Cases ***
New alarm is notified
    ${cb}=    Callback URL    robot-fm
    Send Request    POST    {{fm}}/subscriptions    {"consumerReference": "${cb}", "timeTick": 60}
    Status Should Be    201
    Send VES Event    fault    MAJOR
    ${ntf}=    Wait For Notification    /cb/robot-fm    timeout=60    notification_type=notifyNewAlarm
    Response Should Conform To Standard
```

All keywords: [docs/CUSTOM-TESTS.md](docs/CUSTOM-TESTS.md).

## Command line (Linux)

```
kaldrivon-smotester --no-window              # print the link instead of opening a window
kaldrivon-smotester list                     # interfaces and number of tests
kaldrivon-smotester run --interface 3gpp-mns --include-waits --report report.html
kaldrivon-smotester robot my-tests/ --outputdir out
```

On Windows, use the app's pages for the same things.

## Updates

**About** shows the installed version; **Check now** looks here for a newer release.

## Documents

- [Release notes](docs/RELEASE-NOTES-0.1.0.md)
- [Custom tests (Robot Framework)](docs/CUSTOM-TESTS.md)
- [End-user licence agreement](EULA.txt)
- [Third-party notices](THIRD-PARTY-NOTICES.txt)

## Support

[kaldrivon.com](https://kaldrivon.com) · info@kaldrivon.com

© 2026 Kaldrivon. Kaldrivon SMO Tester is not affiliated with or endorsed by 3GPP, O-RAN ALLIANCE, TM Forum or the
Linux Foundation. The test cases follow the published specifications; passing them is not a certification.

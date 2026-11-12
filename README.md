# Kaldrivon SMO Tester

Kaldrivon SMO Tester checks whether an SMO (Service Management and Orchestration) speaks its standard interfaces the
way the standards say it should. It tests the SMO **from the outside**: you enter the SMO's address and a token, open
one port on your PC, and run prepared test cases interface by interface. Nothing has to be set up on the SMO: no
network function to add, no O-Cloud or RIC to configure.

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
run yet).

A **Plain / Technical** switch keeps jargon out of the way. Underlined words explain themselves when you point at them.
**Step** runs a use case one test at a time and waits for you after each one.

## Checked against the standards

Every answer and notification from the SMO is compared with the published definitions:

- 3GPP Release 18: TS 28.532, TS 28.623, TS 28.111, TS 28.312, TS 28.104, TS 28.550, TS 29.222 (CAPIF)
- O-RAN: R1AP R005 v12.00, A1AP R005 v07.00
- TM Forum: TMF642 v4 (alarms), TMF639 v4 (resource inventory), TMF628 v5 (performance)
- VES event listener 7.2.1

A missing mandatory field, a wrong type, a value that is not allowed, or an HTTP status or content type the standard
does not give for that operation is a **deviation** and fails the test. A field the standard does not define is
reported as an **extension**; it fails the test only if you switch on Strict mode on the Connection page.

## Interfaces

| Interface | The tester plays | What is tested |
|---|---|---|
| O1 | a network function | connection, VES events of every domain, 3GPP notifications sent as VES, batches, malformed events, file-ready notification and file download |
| 3GPP management services | the operator's OSS | fault supervision, provisioning (read and change), configuration change notifications, heartbeat, performance jobs and files, streaming, trace and MDT |
| R1 | an rApp | CAPIF onboarding, discovery, publishing, events and security tokens; R1AP data discovery, access and registration; RAN OAM; A1 policy management |
| A1 | a Near-RT RIC | A1-EI enrichment jobs and data delivery, A1-ML model training |
| TM Forum Open APIs | the OSS/BSS | TMF642, TMF639, TMF628 |
| Intent | the operator's OSS | intents and intent reports (TS 28.312) |
| Management data analytics | an analytics consumer | analytics requests and reports (TS 28.104) |
| Interface security | an unauthorised or outdated client | missing and wrong credentials, plain HTTP, old TLS versions, security headers, on every endpoint |

Interfaces where the SMO is the one that starts the conversation (NETCONF towards network functions, the M-Plane,
A1-P towards a RIC, O2 towards an O-Cloud) are listed in the app with the reason they cannot be tested from outside.

Tests that wait for the SMO to call back (notifications, files, streaming) first make the SMO do something themselves,
for example by sending it an alarm, so you never have to act on the SMO by hand.

Tests that change something on the SMO (create, modify or delete) are marked and ask before they run.

## Install

### Windows 10 / 11 (64-bit)

1. Download `Kaldrivon-SMOTester-0.1.0-Setup.exe` from the release.
2. Run it. The installer and the program are signed by Kaldrivon.
3. Leave **Allow the SMO to reach the tester through Windows Firewall (TCP 9443)** ticked so the SMO can send notifications back to the tester.

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

1. **Connection**: enter the SMO's address (for example `https://192.0.2.10:8443`) and its API token, then press
   **Save and check the connection**. This steps through the first O1 tests: can the SMO's ports be reached, and do
   the tester's own listeners work.
2. **Ports**: the SMO has to reach your PC on **TCP 9443** (HTTPS) for notifications, files and streaming, and
   optionally TCP 9080 (HTTP). The page shows the address the SMO will be given and the firewall commands for Windows
   and Linux.
3. Pick an interface in the menu and press **Run** on a use case (or **Step** to go one test at a time).
4. **Report** opens everything as an HTML report or downloads it as JSON. The **Transaction log** and **Traffic** pages show every message.

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
kaldrivon-smotester run --interface 3gpp-mns --report report.html
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

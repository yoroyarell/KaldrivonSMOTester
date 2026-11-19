# Kaldrivon SMO Tester 0.1.0

The first release of Kaldrivon SMO Tester: a desktop application for Windows and Linux that checks whether an SMO
speaks its standard interfaces the way the standards say it should.

## What it does

- **288 prepared test cases** in 49 use cases on 11 interfaces: O1, Open Fronthaul M-Plane, 3GPP management services, R1, A1, O2, TM Forum
  Open APIs, intent-driven management, management data analytics and interface security, plus the interfaces the SMO
  does not take part in, listed with the reason. The [test catalog](TEST-CATALOG.md) lists every one.
- **Checked against the published definitions:** 3GPP Rel-18 (TS 28.532, TS 28.623, TS 28.111, TS 28.312, TS 28.104,
  TS 28.550, TS 29.222), O-RAN R1AP R005 v12.00, A1AP R005 v07.00, O2IMS R005 v13.00, TMF642 v4, TMF639 v4,
  TMF628 v5, VES 7.2.1 and the O-RAN WG4 M-Plane YANG models. Deviations fail a test; fields the standard does not define are reported (and fail in
  strict mode).
- **The tester as an O-DU, an O-RU and an O-Cloud:** add it to the SMO and it is managed like a real network
  function (NETCONF with the 3GPP Rel-18 YANG models, VES events, alarms, heartbeats, measurements reported by VES,
  measurement and trace jobs), a real O-RU over the Open Fronthaul M-Plane (NETCONF and call home with the O-RAN WG4
  YANG models: supervision, accounts, carriers and endpoints, alarms, measurements, files and logs over SFTP,
  software download, install and activation, reset) and a real O-Cloud (O2ims inventory, monitoring and provisioning,
  OAuth 2.0 and mutual TLS). All data is synthetic. With the O-RU's RESTCONF path in the SMO set, the Open
  Fronthaul cases ask the SMO to act on the O-RU themselves (RFC 8040), using the tester's own SFTP server for transfers.
- **Only its own objects change:** the tester reads what the SMO has, and changes only its own O-DU, O-RU and
  O-Cloud and its own requests in the SMO.
- **Every test explained:** plain-language overview, the exact `curl` command, the raw request and response, every
  check, and an analysis of what was expected against what came back.
  Cases where you do a step in the SMO yourself are marked **(manual)** and say which values to use; the O-RU's
  U-Plane cases (processing element, carriers, endpoints and links) show a table of values the tester's O-RU accepts.
- **SMO-specific help:** choose *Kaldrivon MiniSMO* as the SMO on the Connection page and the Open Fronthaul cases
  with a step in the SMO also say where that step is in MiniSMO, with the write permission it needs. The checks are
  the same for every SMO.
- **Clear prerequisites:** the settings the tests need are marked on the Connection page and checked live at the top
  of the Integration page (is the SMO answering, does it accept the northbound and VES logins, are the O-DU and O-RU connected, has the O-Cloud been contacted, does the SMO
  accept the tester's HTTPS certificate, for each part of the SMO that calls the tester); a test that fails because one is missing says so first. A test waiting for
  the SMO fails with the SMO's reason when the SMO refuses the tester's certificate; an SMO that checks its trust store
  first and then connects again with a confirmed fingerprint is given time to do so.
- **The O-RU in either start-up mode:** the SMO connects to the O-RU (port 1831), or the O-RU calls home to the SMO
  (RFC 8071, port 4334 by default); add the O-RU to the SMO once, in the matching mode. **Call home now** says at once
  whether the SMO took the connection and, if not, why.
- **Logins and addresses per interface:** a general token for every interface, or a separate one for 3GPP management
  services, R1, TMF and A1; R1, TMF and A1 can also have their own port and base path on the Connection page.
- **Running:** run or step through a use case or a whole interface, runs that do not clash in parallel, a run bar
  with Stop all, queued status while a run progresses, tests that need another test's object run it first, and
  objects removed in the SMO in the meantime are made again. Before it registers or subscribes again, the tester
  removes what it registered or subscribed in an earlier run, and a case waiting for a notification first puts its
  subscription back in place.
- **Results:** transaction log, traffic from the SMO and integration messages, each exportable; HTML and JSON reports
  per use case, per interface or for everything.
- **Clean up:** removes everything the tests created in the SMO, clears the tester's alarms, and lists what is left
  to do by hand.
- **Custom tests** in Robot Framework, using the same connection, checks and listener; any prepared test can be
  turned into one; export and import as a zip.
- **Certificates and passwords** made at first start and kept across updates, each renewable with one button. A
  password the SMO changes with chg-password is kept, and the O-Cloud's access tokens stay valid when the tester
  restarts.
- **Reset to defaults**, like a new installation, cleaning up the SMO first and optionally keeping custom tests.
- **Command line** for running test cases and custom tests without the window.

## Install

- **Windows 10/11 (64-bit):** `Kaldrivon-SMOTester-0.1.0-Setup.exe` (signed). The installer can add the firewall rule
  for TCP 9643, 1830, 1831 and 9644.
- **Linux x86_64:** `kaldrivon-smotester_0.1.0-1_amd64.deb` (Debian, Ubuntu) or
  `kaldrivon-smotester-0.1.0-1.x86_64.rpm` (RHEL, Rocky, AlmaLinux, Fedora, openSUSE). Opens in the web browser.
- Check the downloads against `SHA256SUMS`, which is signed by Kaldrivon (`SHA256SUMS.sig`, public key
  `kaldrivon-linux-signing.pub`; how to check it is in the user guide, *Check the download*), and the `.sha256` file
  next to the Windows installer.
- The user guide is also attached as one HTML page and as a PDF.

See the [user guide](../README.md) to get started.

## Known limits

- Not tested: A1 policies on a Near-RT RIC, which need a Near-RT RIC the SMO manages.
- The tester plays an O-DU and an O-RU; an O-CU role is not offered.
- Some O2 and Open Fronthaul tests need an action by you in the SMO (acknowledging or clearing an O-Cloud alarm,
  creating a provisioning request, configuring a carrier, starting a software download, resetting the O-RU); they are
  marked (manual) and say which.
- Passing the tests is not a certification.

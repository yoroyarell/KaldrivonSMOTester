# Kaldrivon SMO Tester 0.1.0

First release.

- Tests an SMO's standard interfaces: O1 (VES events, file-based PM, NETCONF management of the tester's own O-DU),
  3GPP management services (fault supervision, provisioning, notifications, heartbeat, performance jobs and files,
  streaming, trace/MDT), R1 (CAPIF, R1AP data management and exposure, RAN OAM, A1 policy management), A1 (A1-EI,
  A1-ML), O2 (O2ims, the tester's own O-Cloud), TMF642 / TMF639 / TMF628, intents (TS 28.312), management data
  analytics (TS 28.104) and interface security.
- The tester can be added to the SMO as an O-DU (NETCONF with the 3GPP Rel-18 YANG models; it sends its VES events
  where the SMO tells it with the standard NtfSubscriptionControl) and as an O-Cloud (O2ims with mutual TLS and
  OAuth 2.0). The Integration page gives everything to enter in the SMO and the firewall commands.
- Test cases read the names they need from the SMO (managed elements, objects, alarms) instead of using made-up ones,
  and change only the tester's own O-DU, O-Cloud and requests. Nothing else the SMO manages is changed.
- Every answer and notification is checked against the published definitions (3GPP Rel-18 OpenAPI, O-RAN R1AP
  R005 v12.00, A1AP R005 v07.00 and O2IMS R005 v13.00, TMF, VES 7.2.1); deviations are listed and fail the test.
- Each test case shows its curl command, the raw request and response, a verdict, every check, and a plain-language
  explanation. Plain and Technical views, step-by-step runs, parallel runs of independent use cases, Stop all,
  transaction log, HTML and JSON export per use case, per interface or for everything.
- Custom tests in Robot Framework, with a keyword library that uses the same connection, checks and listener.
- Windows installer (signed) and Linux .deb / .rpm packages.

# Kaldrivon SMO Tester 0.1.0

First release.

- Tests an SMO's standard interfaces from the outside: O1 (VES events, file-based PM), 3GPP management services
  (fault supervision, provisioning, notifications, heartbeat, performance jobs and files, streaming, trace/MDT),
  R1 (CAPIF, R1AP data management and exposure, RAN OAM, A1 policy management), A1 (A1-EI, A1-ML), TMF642 / TMF639 /
  TMF628, intents (TS 28.312), management data analytics (TS 28.104) and interface security. 216 prepared test
  cases. Nothing has to be set up on the SMO: enter its address and token, open TCP 9443 on your PC.
- Every answer and notification is checked against the published definitions (3GPP Rel-18 OpenAPI, O-RAN R1AP
  R005 v12.00 and A1AP R005 v07.00, TMF, VES 7.2.1); deviations are listed and fail the test.
- Each test case shows its curl command, the raw request and response, an ETSI-style verdict, every check, and a
  plain-language explanation. Plain and Technical views, step-by-step runs, transaction log, HTML and JSON reports.
- Custom tests in Robot Framework, with a keyword library that uses the same connection, checks and listener.
- Windows installer (signed) and Linux .deb / .rpm packages.

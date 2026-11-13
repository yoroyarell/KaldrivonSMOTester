# Custom tests with Robot Framework

Custom tests live in the data folder, under `robot/` (Windows: `%APPDATA%\Kaldrivon\SMOTester\robot`, Linux:
`~/.config/kaldrivon-smotester/robot`). Edit and run them on the **Custom tests** page, or run them from a shell with
`kaldrivon-smotester robot <file or folder> --outputdir <folder>` on Linux (exit code = number of failed tests, as with
`robot`).

Start every file with:

```robotframework
*** Settings ***
Library    smotester.robotlib.SMOTester
```

Robot separates columns with at least two spaces (the editor's Tab key inserts four).

## Templates

Paths and bodies take the tester's templates, filled in at run time:

| Template | Value |
|---|---|
| `{{fm}}` `{{prov}}` `{{fdr}}` | 3GPP Fault Supervision, Provisioning and File Data Reporting MnS roots (MnS root and version from the Variables page) |
| `{{tmf_alarm}}` `{{tmf_ri}}` `{{tmf_pm}}` | TMF642, TMF639, TMF628 roots |
| `{{me_dn}}` `{{write_dn}}` | DNs found in the SMO by the discovery cases (or typed on the Variables page); `{{me_dn\|dn}}` gives the comma form |
| `{{subnet_dn}}` | the SubNetwork DN from the Variables page |
| `{{cb}}` | the tester's callback root, e.g. `https://<this PC>:9643/cb` |
| `{{$uuid}}` `{{$now}}` `{{$now_us}}` `{{$iso:+3600}}` `{{$rand}}` | made fresh for each step |
| anything a prepared case captured, or set with `Set Template Variable` | e.g. `{{alarm_id}}`, `{{invoker_id}}` |

Robot's own variables (`${cb}`) work as usual and can be mixed with templates.

## Keywords

| Keyword | Arguments | What it does |
|---|---|---|
| Send Request | method, path, body=, target=northbound, content_type=json, auth=default, headers=, query=, timeout=20 | Sends a request to the SMO, returns the HTTP status. `target`: northbound, ves, r1, tmf, a1ei. `content_type`: json, merge, 3gpp_patch, json_patch, form, xml, text. `auth`: default, none, invalid. A full URL (e.g. a Location header) works as path. |
| Send VES Event | domain=heartbeat, severity=MAJOR, event= | Sends a VES 7.2.1 event (heartbeat, fault, measurement, notification, or your own JSON). |
| Run Catalog Case | case_id, expected_verdict=PASS | Runs a prepared case by its id and checks the verdict; its captured values become templates. Cases that change something run only on the tester's own O-DU (NONE otherwise). |
| Status Should Be | *codes | Fails unless the last HTTP status is one of the codes. |
| Response Should Conform To Standard | strict=False | Fails if the last answer or notification deviates from the published definition (strict: extensions also fail). |
| Get Response Field | path | Returns `body.x`, `body[0].id`, `headers.location`, `body.*firstkey`, `status`. |
| Response Field Should Be | path, expected | Compares one field. |
| Response Should Contain | text | Checks the raw body. |
| Callback URL | name=custom | The tester's address for notifications, `{{cb}}/<name>`. |
| Wait For Notification | path=/cb, timeout=60, notification_type=, answer_status=204 | Waits for the SMO to call the tester (also accepts one that arrived right after the previous request), returns its body, checks it against the standard. |
| Set Template Variable / Get Template Variable | name, value / name | Templates for later steps. |
| Render Template | text | Returns text with templates filled in. |
| Explain Last Result | | Logs the plain-language explanation. |

Any other Robot Framework library (BuiltIn, Collections, String …) can be used next to these.

## What custom tests may change

The same rule as for the prepared cases: a request that changes a managed object is sent only when it is the
tester's own O-DU (or an alarm it raised). Read anything; change only what is yours. `Run Catalog Case` applies
this on its own; with `Send Request` you are responsible for the paths you write.

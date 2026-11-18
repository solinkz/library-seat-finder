# Technical history

This history follows the major changes in Git from the imported IFI7386 course example to the current prototype. To inspect a change locally, run `git show <commit>` with the short commit ID below. See the [code walkthrough](code-walkthrough.md) for how the parts fit together today.

## Baseline · The course example

| Commits | What the import already included |
| --- | --- |
| `791f17b`, `7669064` | The Arduino Wi-Fi and serial calibration sketches. They already separated two sensor channels, smoothed readings, and used distinct on and off thresholds. |
| `2c6a43f`, `a44dd2e` | The Node.js host and package scripts. The host already validated batches, tracked each seat independently, and returned Unknown for expired readings. |
| `47b51a8` | A browser interface with a single SVG floor plan, status polling, and clickable table areas. |
| `148c394`, `a346692` | A simulator for hardware-free testing and an integration test for host behavior, including validation, expiry, and endpoint isolation. |

## Major improvements

| Commit or sequence | Technical change | What it enabled |
| --- | --- | --- |
| `51aeb99` | Replaced the SVG floor plan with HTML table cards. Added `GET /api/version` and a script that compares the HTML file’s modification time between requests. | Each table has its own card. During development, editing the HTML can trigger a browser reload on the next version check. Live seat data still comes from the existing `/api/state` endpoint. |
| `36b29c0`, `4fe3e02`, `816c2a8`, `882e01a`, `6fc224d` | Iterated on separate seat and table details, then consolidated the interaction around the table card and added a visible More details button. | Visitors can open one dialog per table. Table 1’s dialog shows the two live seat statuses together, alongside example table features. |
| `e808e3a` | Lowered the Wi-Fi sketch’s ON threshold from 550 to 350 and its OFF threshold from 400 to 250 for both chairs. The serial calibration sketch retained 550 and 400. | Lower filtered readings can trigger the occupied flag, and the reading must drop below the new OFF value to clear it. The change supplies different example defaults. Calibration and real-world accuracy still depend on the mounted sensors. |
| `186c344`, `6cfe4b2`, `c4207f2` | Added an in-person check explanation for tables without sensors, moved the status guide below the table cards, and clarified the monitored table footer. | The interface explains which availability claims come from sensors and what to do when a table’s seats have no readings at all. |
| `94d8249`, `e7aaf87` | Replaced displayed identifiers with friendlier seat labels, then restarted numbering at Seat 1 for each table. | Visitors can refer to seats by table and number. The firmware and host still exchange the stable IDs `L1` and `L2`. |

## Where the prototype stands

The later commits changed how visitors browse the tables, how the page reloads during development, and how the Wi-Fi sketch classifies weight. The imported host still holds two monitored seats in memory. Git shows the threshold change, but contains no measured accuracy result for it. See the [current-state PRD](Prd.md) for the shipped behavior and the [improvement tickets](future-features.md) for proposed reservations and interface changes.

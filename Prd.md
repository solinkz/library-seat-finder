# Library Seat Finder product requirements and technical specification

This document records the current Group 1 course prototype. Library visitors can check the latest estimated availability of monitored seats before walking to a table. Course operators can calibrate the sensors and diagnose the data path using Serial Monitor, the host API, and a simulator. See the [setup guide](docs/setup.md) for operating instructions and [future features](docs/future-features.md) for proposals from user feedback.

## Product behavior

| Surface | Current behavior |
| --- | --- |
| Table cards | Table 1 shows two monitored seats. Tables 2 and 3 show four and six seats respectively without sensors. Seat labels restart at Seat 1 on each table. |
| Availability summary | Counts Available seats out of two monitored seats. |
| Status guide | Explains Available, Occupied, Unknown, and Not monitored using labels and symbols. |
| Table details | Selecting a table card or its More details button opens a dialog. Table 1 lists both live statuses. Tables 2 and 3 direct visitors to check in person. The dialog shows example locations and amenities. It closes via either close control, backdrop click, or native dialog dismissal. |
| Connection indicator | Starts at Connecting to host. A successful state poll shows Host connected. A failed poll shows Host unavailable and sets both monitored seats to Unknown. |
| Availability estimate | A fresh reading indicates whether the sensor detects weight under its current calibration. The interface warns that availability can change before arrival. |

## Architecture and firmware

```text
Pressure sensors on A0 and A1
      -> Arduino UNO R4 WiFi
      -> POST /api/seats
      -> Node.js latest readings in memory
      -> GET /api/state
      -> Browser table cards, count, and details

Simulator -> POST /api/seats (alternative reading source)
Serial calibration sketch -> Serial Monitor
```

- A0 maps to `L1` and Table 1 Seat 1. A1 maps to `L2` and Table 1 Seat 2. Both sketches use 10-bit analog readings and integer smoothing, `(3 * previous + sample) / 4`. The reported `raw` field contains this filtered reading. Each seat starts with a zero reading and a false occupied flag.
- A filtered reading strictly above ON sets occupied. A reading strictly below OFF clears occupied. Readings at or between the thresholds retain the previous state. Defaults are independent constants for each seat and intended for calibration after mounting.

| Sketch | Default ON / OFF per seat | Timing and output |
| --- | --- | --- |
| Serial calibration | 550 / 400 | Reads and logs both seats, then waits 100 ms. |
| Wi-Fi reporting | 350 / 250 | Targets sampling every 50 ms and sends both seats every 2 s. Wi-Fi and HTTP calls can delay the loop. |

Both sketches log at 115200 baud. The Wi-Fi sketch joins the open `IFI7386-G1` network and sends HTTP to `10.42.0.1:3000`. It sets a 10-second Wi-Fi connection timeout and waits at least 10 seconds after an attempt completes before retrying on a disconnected board. After sending a request it waits up to 1.5 seconds for response data and considers a first response line starting with `HTTP/1.1 200` successful. A failed send is followed by a later reading report.

## Host configuration and state

The host runs on Node.js 18+ with built-in modules and binds to `0.0.0.0` when started directly.

| Setting | Default | Applies to |
| --- | --- | --- |
| `PORT` | `3000` | Server listen port |
| `STALE_MS` | `10000` | Maximum age of a fresh reading in milliseconds, inclusive |
| `HOST_URL` | `http://localhost:3000` | Simulator destination origin |

The host stores the latest accepted reading per seat in process memory. Each starts with null `occupied` and `raw`, and `seen: 0`. On update, `seen` is the host receipt time in Unix epoch milliseconds. A reading stays fresh through `STALE_MS` and resolves to `free` or `occupied` from its boolean flag. Missing or expired readings resolve to `unknown`. Expiry retains the last measurement and timestamp in the snapshot. Restarting the process resets both seats.

## HTTP API

| Method and path | Success response |
| --- | --- |
| `GET /` | 200 HTML page with a live-reload script injected by the host |
| `GET /api/state` | 200 JSON object with a top-level `seats` field containing the `L1` and `L2` snapshots |
| `GET /api/version` | 200 plain-text modification time of the HTML file in epoch milliseconds |
| `POST /api/seats` | 200 JSON `{"ok":true}` |

The host sends `Cache-Control: no-store` on these responses. Example ingestion request:

```sh
curl -i http://localhost:3000/api/seats \
  -H 'Content-Type: application/json' \
  --data '{"v":1,"seats":[{"id":"L1","occupied":true,"raw":700}]}'
```

`v` must be the number `1`. The `seats` array must contain one or two entries, each with a unique `id` of `L1` or `L2`, a Boolean `occupied`, and an integer `raw` from 0 through 1023. The host validates the entire batch before updating either seat. A one-seat update leaves the other seat’s data and freshness untouched. The server rejects the request if the accumulated body string exceeds 2,048 UTF-16 code units.

Example state snapshot after the request above reaches a fresh host. The timestamp is illustrative.

```json
{
  "seats": {
    "L1": {"occupied": true, "raw": 700, "seen": 1795000000000, "state": "occupied"},
    "L2": {"occupied": null, "raw": null, "seen": 0, "state": "unknown"}
  }
}
```

| HTTP status | JSON error code | Condition |
| --- | --- | --- |
| 400 | `BAD_JSON` | The body cannot be parsed as JSON. |
| 400 | `BAD_VERSION` | The parsed body is falsy or its version differs from numeric `1`. |
| 400 | `BAD_SEATS` | The readings fail validation. |
| 400 | `REQUEST_FAILED` | An unexpected error occurs while handling a write before headers are sent. |
| 404 | `NOT_FOUND` | The method and path do not match a route. |
| 413 | `BODY_TOO_LARGE` | The body exceeds the accumulated string limit. |

## Browser data flow and limits

The browser requests `/api/state` immediately, with a 2.5-second request timeout, and schedules another poll one second after each attempt finishes. It maps `free` to Available, `occupied` to Occupied, and missing or other state values to Unknown for monitored seats. Each attempt updates the cards, count, connection indicator, and any open details dialog. A subsequent successful poll restores live statuses according to freshness. A separate loop checks `/api/version` about every 700 ms after the previous check finishes and reloads the page when the file version changes.

The layout and monitored seat IDs are hard-coded. The host retains one reading per seat, with no history or persistent storage. The demo uses plain HTTP without authentication. Pressure can come from a person or an object, and the software has no way to identify a visitor. Confirm the physical layout and amenities before using those details with library visitors.

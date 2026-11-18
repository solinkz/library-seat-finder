# How the seat finder works

Start with one question. How does sitting on Table 1, Seat 1 change what a visitor sees in the browser? This walkthrough follows that reading through the project. You can try the browser and server with the [simulator](setup.md#try-it-with-simulated-readings), without an Arduino.

```text
Chair pressure -> Arduino -> Node.js host -> Browser
                              ^
                              |
                          Simulator
```

## 1. The Arduino turns pressure into a seat status

Read [library.ino](../group-1-library/library/library.ino), starting with `setup()`, `loop()`, and `readChairs()`. A sketch is a program that runs on the Arduino. `setup()` runs once when the board starts, and `loop()` runs again and again.

- `analogRead()` turns the voltage from a sensor into a number from 0 through 1023. A0 belongs to Table 1, Seat 1 (`L1`), and A1 belongs to Seat 2 (`L2`).
- `readChairs()` smooths each reading by giving the previous filtered value three times the weight of the new sample. This softens short spikes.
- The Wi-Fi sketch’s example thresholds set a seat to occupied above 350 and free below 250. At either threshold or between them, the seat keeps its previous state. This gap reduces flicker near the boundary. The [serial sketch](../group-1-library/library_serial/library_serial.ino) prints sensor values and flags for calibration and uses its own example thresholds.

About every two seconds, `makeSeatMessage()` creates a JSON message with both seat IDs, their occupied flags, and their filtered numbers. JSON is the text format the Arduino and host use to exchange data. `sendToPi()` sends the message over Wi-Fi to `POST /api/seats`. The `raw` field contains the filtered number.

## 2. The host remembers the most recent reading

Read [server.js](../group-1-library/host/server.js), starting with `handleRequest()`. The host runs on Node.js and answers HTTP requests. An endpoint is a path the host handles, such as `/api/seats`.

- A device sends new data with `POST /api/seats`. `validSeats()` checks all entries before the server changes any seat. The host remembers each accepted reading and when it arrived.
- The browser asks for data with `GET /api/state`. `seatState()` checks each reading’s age. A fresh occupied flag becomes `occupied` or `free`. A missing or expired reading becomes `unknown`. The default freshness window is ten seconds.
- Restarting the host clears its readings because they are held in memory. The [PRD](Prd.md#http-api) gives the exact request and response format.

For example, a fresh `L1` report with `occupied: true` and `raw: 700` produces an `occupied` state. If reports stop for more than ten seconds, the next state response reports `unknown` for that seat and still includes the previous value and timestamp.

## 3. The browser displays the status

Read the script near the bottom of [index.html](../group-1-library/host/index.html). HTML describes the cards and details dialog, CSS controls how they look, and JavaScript changes what they show as new readings arrive.

- `refresh()` asks the host for the latest state and schedules another poll about a second after the request finishes. It maps the host’s `free` state to the visitor-facing label Available.
- `updateSeats()` applies the labels and counts Available seats. Table 1 has two monitored seats. Tables 2 and 3 have no sensors, so they stay Not monitored.
- `renderTable()` shows details for the selected card and refreshes them when new data arrives. Host connection and seat status are separate. A reachable host can still report Unknown if the last sensor reading is too old.

## Try following a reading

1. Start the host and simulator using the [setup guide](setup.md#try-it-with-simulated-readings). [simulate.js](../group-1-library/host/simulate.js) plays the Arduino’s role by sending test readings to the same endpoint. Seat 1 alternates between Available and Occupied. Seat 2 stays Available.
2. Open `http://localhost:3000/api/state` in a second tab. Refresh it to compare `seats.L1.state` with the label on the Table 1 card.
3. Stop the simulator and wait for both seats to become Unknown while the host stays connected. [test.js](../group-1-library/host/test.js) checks expiry and other host behavior automatically. Run it with `npm test --prefix group-1-library/host` from the repository root.

# Run and troubleshoot Library Seat Finder

## Try it with simulated readings

Use Node.js 18 or later and a browser that supports native dialogs and `AbortSignal.timeout()`. The host uses built-in Node.js modules and has no dependencies to install. Run commands from the repository root.

1. Start the host.

   ```sh
   npm start --prefix group-1-library/host
   ```

2. Open `http://localhost:3000`. With a fresh host, Table 1 starts with both seats marked Unknown. Tables 2 and 3 show seats without sensors.
3. In a second terminal, start the simulator.

   ```sh
   node group-1-library/host/simulate.js
   ```

   The simulator sends both readings every two seconds and prints the HTTP result. Seat 1 alternates between Available and Occupied, Seat 2 stays Available, and the monitored availability count alternates between two and one. Open Table 1’s details to see the same live changes there. Stop the simulator with Ctrl+C. With the default settings, the first browser poll after the last readings are more than ten seconds old shows both seats as Unknown. The host still shows as connected.

To use another port or a remote host, set `PORT` for the server and point the simulator’s `HOST_URL` at the same origin. Use the actual host address in the browser when testing from another device. Stop the simulator before connecting the real sensors, since both write the same seat IDs.

## Calibrate and connect the hardware

### Prerequisites

- An Arduino UNO R4 WiFi and two wired, mounted force-sensitive resistor circuits. The analog outputs go to A0 for Table 1 Seat 1 and A1 for Table 1 Seat 2.
- Arduino IDE with the UNO R4 WiFi board package and its bundled WiFiS3 library.
- A Raspberry Pi or equivalent Node.js host and a browser device on a configured local demo network. The Wi-Fi sketch defaults to the open SSID `IFI7386-G1` and expects the host at `10.42.0.1:3000`.

This walkthrough starts with assembled sensors and a configured network. The repository supplies no circuit schematic, resistor values, mounting procedure, or hotspot provisioning instructions.

### Calibrate and run

1. Select the UNO R4 WiFi board in Arduino IDE and upload the [serial sketch](../group-1-library/library_serial/library_serial.ino). Open Serial Monitor at 115200 baud. Record stable filtered readings for each mounted seat unloaded and loaded. Adjust each seat’s thresholds so the unloaded reading falls below OFF and the loaded reading rises above ON. Verify that loading and releasing each seat changes its occupied flag independently.
2. Start the Node.js host on the Pi using the command above. If the configured network differs from the defaults, update the Wi-Fi sketch’s SSID, destination IP, HTTP Host header, and port as applicable.
3. Copy the calibrated values into the [Wi-Fi sketch](../group-1-library/library/library.ino), which has different example defaults. Upload it and look for `WIFI_CONNECTED` and `HTTP/1.1 200` in Serial Monitor.
4. Open `http://10.42.0.1:3000` from the demo network, or use the host address and port you configured. Load and release each seat and verify the card, count, and Table 1 details. Check `/api/state` on the same origin to inspect the latest values and receipt timestamps.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `WIFI_NO_MODULE` | Check the board selection and Arduino connectivity firmware. |
| `WIFI_NOT_CONNECTED` | Verify that the expected open SSID is reachable. The sketch retries after ten seconds from the end of a failed attempt. |
| `HOST_UNREACHABLE` or `HTTP_TIMEOUT` | Confirm the Pi is serving the page and state endpoint at the address and port compiled into the sketch. Check local network reachability from a device on the same network. |
| HTTP 400 | Compare the printed JSON with the [API contract](Prd.md#http-api). |
| Host connected, seats Unknown | Check whether the Arduino or simulator is sending successfully, then inspect `seen` in `/api/state`. A successful browser poll can return stale or missing sensor data. |
| Host unavailable | Start or restart the server and check the browser’s network connection. The page retries automatically. |
| A seat stays Occupied or Available | Recheck that seat’s filtered readings, mounting, and thresholds using Serial Monitor. Confirm the simulator is stopped. |

## Verification

Run the host integration test with `npm test --prefix group-1-library/host`. The test binds a temporary loopback port.

The host integration test and simulator HTTP checks passed on Node.js 24.12.0. The HTTP checks covered initial state, alternating readings, expiry, and host restart. Hands-on verification is pending for browser interactions, including the details dialog and disconnect recovery, and for the physical Arduino and Raspberry Pi setup.

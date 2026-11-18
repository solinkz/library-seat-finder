# Library Seat Finder

Group 1’s IFI7386 Interactive Systems course prototype shows library visitors whether either of two monitored seats is available. The browser displays three tables and marks seats without sensors separately.

Two pressure sensors feed an Arduino UNO R4 WiFi. The Arduino classifies occupancy and posts readings to a Node.js host, typically a Raspberry Pi. The browser polls that host for the latest status.

- [Setup and troubleshooting](docs/setup.md) covers the simulator, calibration, and the local hardware demo.
- [Hardware record](docs/hardware.md) lists the bench-test parts, known connections, and observed results.
- [Current-state PRD](docs/Prd.md) specifies the product behavior, architecture, firmware, and HTTP API.
- [Code walkthrough](docs/code-walkthrough.md) follows a reading from the sensor to the browser.
- [Technical history](docs/technical-history.md) traces the major changes in Git.
- [Future features](docs/future-features.md) records user feedback, proposed behavior, and what remains to deliver it.

## Source layout

| Path | Contents |
| --- | --- |
| [group-1-library/host](group-1-library/host) | Node.js server, browser interface, simulator, and integration test |
| [group-1-library/library](group-1-library/library) | Arduino Wi-Fi sketch for reporting live readings |
| [group-1-library/library_serial](group-1-library/library_serial) | Arduino serial sketch for checking sensor readings and calibrating thresholds |

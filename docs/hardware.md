# Seat finder hardware record

The team tested two force-sensitive resistors (FSRs) on the bench by pressing them by hand to simulate pressure on a seat. An Arduino UNO R4 WiFi read the sensors and sent updates to the Node.js host on a Raspberry Pi 400. The red box visible in the bench photo was unrelated to the project.

## Bill of materials

| ID | Item | Quantity | Role and evidence |
| --- | --- | --- | --- |
| U1 | Arduino UNO R4 WiFi | 1 | Reads both sensors and sends seat updates over Wi-Fi. |
| FSR1, FSR2 | Force-sensitive resistors | 2 | One per monitored seat. Exact models are unknown. |
| R1, R2 | Fixed resistors | 2 | Used in the sensor setup. Resistance in ohms and exact connections are unconfirmed. |
| B1 | Breadboard | 1 | Used for the bench circuit. |
| W1 | Wire | At least 2 m total | Used for electrical connections. Individual lengths are unknown. |
| C1 | USB cable | 1 | Connected the Arduino to a PC for power during the test. |
| H1 | Raspberry Pi 400 | 1 | Ran the Node.js host for the demo. |
| DEV1 | PC | 1 | Supplied USB power to the Arduino. |
| TOOL1 | Soldering tools | Quantity unknown | Used to join wires. The team also insulated the joints. Insulation material is unconfirmed. |

## Replacement cost estimate

Prices checked on 18 November 2026 for delivery to Estonia. These are advertised prices in euros, including VAT as displayed by the retailers and before delivery. The table records the lowest priced offers I could verify for this comparison that did not show a business-account restriction. Stock, discounts, and checkout totals can change.

| ID | Priced item | Purchase quantity | Unit price | Line total | Supplier |
| --- | --- | ---: | ---: | ---: | --- |
| U1 | Arduino UNO R4 WiFi, ABX00087 | 1 | €33.85 | €33.85 | [Lemona Arduino boards](https://www.lemona.ee/en/robotics-open-source-electronics/arduino/arduino-boards?page=1) |
| FSR1, FSR2 | Round FSR-402, Pololu 1696, reference part | 2 | €16.50 | €33.00 | [Anodas FSR-402](https://www.anodas.lt/en/force-sensor-fsr-402-2kg-round-18mm-pololu-1696) |
| R1, R2 | 10 kΩ axial resistors, price reference only | 1 pack of 10 | €0.39 | €0.39 | [Lemona resistor pack](https://www.lemona.ee/takisti-0-25w-10k-5-10tk-pk.html) |
| B1 | 830-point solderless breadboard, ST1184 | 1 | €4.17 | €4.17 | [Lemona breadboards](https://www.lemona.ee/en/robotics-open-source-electronics/prototyping-breadboard-accessories/breadboards) |
| W1 | Insulated hookup wire, K/MOWM | 1 kit of 60 m | €14.45 | €14.45 | [Lemona wire kit listing](https://www.lemona.ee/en/search.html?query=PROTOTYPING+KIT) |
| C1 | USB-A to USB-C cable, 1 m, CATKLF-B91 | 1 | €5.72 | €5.72 | [Lemona USB cable listings](https://www.lemona.ee/en/wires-and-cables/cable-assemblies/usb-cables-and-adapters) |
| H1 | Raspberry Pi 400, US keyboard layout | 1 | €77.20 | €77.20 | [Lemona Raspberry Pi 400](https://www.lemona.ee/en/raspberry-pi-400-us.html) |

- **Bench electronics with an existing host and tools:** €91.58 before delivery.
- **Bench electronics plus the Raspberry Pi 400:** €168.78 before delivery. This prices the computer only. Budget separately for a Pi power supply, storage, and display accessories if needed.
- **Lower FSR price for eligible buyers:** Botland lists the same Pololu part at €12.50 each and shows a registered-business restriction. At that price, the pre-delivery subtotals would be €83.58 and €160.78. [Botland FSR-402](https://botland.store/pressure-sensors/753-force-sensor-fsr-402-2kg-round-18mm-pololu-1696-5904422366896.html)

The estimate assumes access to the PC and soldering tools. It excludes solder, insulation materials, and other unconfirmed accessories. The full wire kit is included at its purchase price because the notes record only a minimum of 2 m used. The FSR model and resistor value in the original build are still unconfirmed. Verify both before ordering replacements.

## Connections and network

| Connection | What is known |
| --- | --- |
| FSR1 circuit to U1 | The firmware reads Table 1 Seat 1 (`L1`) on `A0`. |
| FSR2 circuit to U1 | The firmware reads Table 1 Seat 2 (`L2`) on `A1`. |
| DEV1 to U1 | USB supplied Arduino power during the bench test. |
| U1 to H1 | The demo used the firmware defaults. The Arduino joined `IFI7386-G1` and sent readings to `http://10.42.0.1:3000/api/seats`. The device providing the hotspot is unconfirmed. |

The team’s notes record “1 V” beside the resistor value. Volts measure voltage, and resistor values are specified in ohms. Confirm what “1 V” referred to, the resistance of R1 and R2, and where each FSR and resistor connects to the supply, ground, and analog input. A powered-off wiring photo with labels or a hand-drawn sketch would complete the connection table and support a circuit diagram.

## Bench test results

The team reported that pressing the sensors by hand with sufficient force caused the seat display to show **Occupied**. This confirmed that hand pressure could trigger the prototype’s occupancy display during the demo. The repository’s [Wi-Fi sketch](../group-1-library/library/library.ino) contains example ON and OFF thresholds of 350 and 250 for both seats. The team’s test notes give no measured sensor values or independent record of the thresholds uploaded for the bench test.

For a repeatable verification record, test each sensor separately and log both seats’ displayed status before pressing, while pressed, and after release. Record the uploaded sketch version and filtered readings. Chair-mounted calibration and performance under seated use remain unverified. See the [setup guide](setup.md#calibrate-and-connect-the-hardware) for the firmware and calibration workflow.

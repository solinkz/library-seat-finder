# Feature improvement tickets

These mini tickets describe proposed improvements from user feedback. See the [current-state PRD](Prd.md) for the existing behavior.

## 1. Reserve a seat for five minutes

**User story**

As a visitor, I want to hold an available seat while I walk to it, so another visitor can see that it is reserved.

**Implementation scope**

Add a Reserve action for monitored seats with a fresh Available reading. The host grants a single in-memory hold per seat and sets expiry five minutes after granting it. Use an anonymous browser-scoped request token that the browser reuses when retrying the same request. Show Reserved and the time remaining for an active hold. Early cancellation is outside the local demo scope. Physical access remains unrestricted.

**Acceptance criteria**

- [ ] A fresh Available seat can be reserved. Occupied, Unknown, unmonitored, and already reserved seats are ineligible.
- [ ] If two visitors try to reserve the same seat at once, the host confirms one request and rejects the other as a conflict. After refreshing, both see Reserved with the same expiry time.
- [ ] If the sensor detects weight or its reading goes stale during a hold, the seat still shows Reserved and stays out of the availability count.
- [ ] The hold expires five minutes after the host grants it, even if the visitor closes the page. The next successful refresh displays the latest sensor-derived status, including Unknown if the reading has expired.
- [ ] If the host grants the hold but the response is lost, retrying with the same request token while the hold is active returns the original expiry. The retry leaves that expiry intact.
- [ ] While disconnected, the browser indicates that it cannot verify a hold and disables new reservations. After reconnecting, it refreshes from the host and shows any active hold with its remaining time.
- [ ] Restarting the host clears holds and readings. On reconnect, a previously reserved seat shows Unknown until a new reading arrives, or the status from that reading if it has already arrived.

## 2. Make tables and seats easier to recognize

**User story**

As a visitor, I want to distinguish a table from its seats at a glance, so I can find the seat I’m looking for.

**Implementation scope**

Use distinct table and chair silhouettes on the cards and keep seat numbers visible. Repeat the numbered chair icons in Table 1’s existing detail rows. Preserve the seat layout on the cards and the single availability explanation in the details for Tables 2 and 3.

**Acceptance criteria**

- [ ] At a 320 CSS-pixel viewport, seat numbers remain legible without overlap or horizontal scrolling. Table 1’s seat numbers match between its card and details.
- [ ] Table and seat shapes are distinguishable without color, and seat status is still readable using text or symbols.
- [ ] Given the task “Find Table 2, Seat 3,” visitors point to Table 2’s table shape and Seat 3’s chair shape separately without hints.

## 3. Simplify seat status explanations

**User story**

As a visitor, I want to understand seat status without consulting a legend, so I can quickly decide where to go.

**Implementation scope**

Use the four existing labels consistently on cards and in details. Keep Available and Occupied visible beside the seat. Where clarification is needed, use “No recent reading” for Unknown and “No sensor. Check in person.” for Not monitored.

**Acceptance criteria**

- [ ] The same status uses the same label on the seat card and in table details.
- [ ] With one monitored seat Available and the other Occupied, visitors choose the Available seat without consulting the legend.
- [ ] With the host disconnected, visitors identify an Unknown seat as monitored with no recent reading available to the browser. They identify a Not monitored seat as having no sensor and recognize that neither seat is confirmed available.

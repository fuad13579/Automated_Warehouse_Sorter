# Automated Warehouse Sorter

A digital logic simulation built in **Proteus 8.13** that captures package information, moves packages through conveyor stages, and routes them to three storage zones.

Each zone holds a maximum of **3 packages**. The system rejects packages with invalid destinations or full target zones.

## Circuit Preview

<!-- Upload your circuit screenshot to this path. -->
![Integrated circuit](images/full-circuit.png)

## Features

- Registered package destination and priority information
- Four clocked conveyor stages
- Routing to storage zones A, B and C
- Independent storage counters and remaining-capacity displays
- FULL feedback to prevent overfilling
- Rejection of invalid destinations and full-zone requests
- BUSY lockout during package processing
- DONE indication for both accepted and rejected packages
- Global reset

## System Architecture

| Module | Responsibility |
|---|---|
| **M1 — Input & Conveyor** | Captures package information, advances conveyor stages and manages BUSY |
| **M2 — Classification & Routing** | Decodes the destination, checks capacity and registers the route or rejection |
| **M3 — Storage & Capacity** | Updates storage counts, calculates remaining space, generates FULL feedback and completion |

All sequential components use one shared rising-edge clock.

## Inputs

| Input | Function |
|---|---|
| `D1`, `D0` | Two-bit destination code |
| `P` | Priority flag |
| `LOAD` | Requests capture of a new package |
| `CLK` | Advances the circuit on each rising edge |
| `RESET` | Active-high reset that clears stored state and counts |

### Destination Encoding

| D1 | D0 | Destination |
|---|---|---|
| 0 | 0 | Storage A |
| 0 | 1 | Storage B |
| 1 | 0 | Storage C |
| 1 | 1 | Reject |

A valid destination is also rejected if its storage zone is full.

## Outputs

| Output | Meaning |
|---|---|
| `S1–S4` | Conveyor-stage indicators |
| `ROUTE A/B/C` | Registered routing decision |
| `REJECT` | Package cannot be accepted |
| `BUSY` | A package is being processed |
| `DONE` | Processing has completed |
| `PRIORITY` | An accepted package has its priority flag set |
| `COUNT` | Packages stored in a zone |
| `FREE` | Remaining spaces in a zone |
| `FULL` | Zone has reached capacity 3 |

## Main Components

- **74HC157:** selects new package data or stored-data feedback
- **74HC175:** stores package data, conveyor stages and routing decisions
- **74HC74:** stores BUSY and DONE
- **74HC139:** decodes the destination
- **74HC161:** counts accepted packages
- **74HC283:** supports remaining-capacity arithmetic
- AND, OR, NOR and NOT gates
- Logic inputs, logic probes and decoded seven-segment displays

## How to Run

1. Open the `.pdsprj` file in Proteus 8.13.
2. Set `CLK = 0` and `LOAD = 0`.
3. Start the simulation.
4. Toggle `RESET` from `0 → 1 → 0`.
5. Confirm all COUNT displays show `0` and FREE displays show `3`.
6. Set `D1`, `D0` and `P` for the package.
7. Set `LOAD = 1`.
8. Give one clock pulse, then return `LOAD` to `0`.
9. Give five additional pulses to finish processing.

One manual clock pulse means `CLK: 0 → 1 → 0`.

Keep RESET low during normal operation. Return LOAD low after capture to prevent another package from being loaded when the system becomes idle.

## Processing Sequence

| Pulse | Cycle | Expected action |
|---|---|---|
| 1 | C0 | Capture package, activate S1 and set BUSY |
| 2 | C1 | Advance to S2 |
| 3 | C2 | Advance to S3 / SORT |
| 4 | C3 | Activate S4 and the selected route or REJECT |
| 5 | C4 | Increment the selected count if accepted; assert DONE |
| 6 | C5 | Clear DONE and BUSY; return to idle |

Rejected packages follow the completion sequence without increasing any storage count.

## Capacity Behaviour

Each zone has a capacity of 3:

| COUNT | FREE | FULL |
|---:|---:|---:|
| 0 | 3 | 0 |
| 1 | 2 | 0 |
| 2 | 1 | 0 |
| 3 | 0 | 1 |

Further requests to a full zone are rejected. Other zones can continue accepting packages.

## Priority Behaviour

The priority flag is captured with the package.

For an accepted package, `P = 1` activates PRIORITY alongside its route signal.

**Priority is currently an indicator, not a scheduling mechanism.** It does not change the destination, shorten processing time, interrupt a package already being processed, or bypass capacity checks.

## Verification Checklist

Unchecked items are not claimed as verified.

- [ ] Reset produces COUNT = 0 and FREE = 3 in every zone
- [ ] Destination 00 routes to A
- [ ] Destination 01 routes to B
- [ ] Destination 10 routes to C
- [ ] Destination 11 is rejected without changing counts
- [ ] Each accepted package increments exactly one count once
- [ ] Each zone accepts three packages and rejects the fourth
- [ ] A full zone does not block acceptance into another zone
- [ ] DONE occurs for accepted and rejected packages
- [ ] BUSY prevents new package capture during processing
- [ ] Changing input switches after capture does not alter the stored package
- [ ] PRIORITY activates only for accepted priority packages
- [ ] Extra clocks with LOAD = 0 do not change counts
- [ ] Reset during processing clears the pending transaction

## Repository Contents

```text
proteus/    Proteus project
docs/       Schematic PDF, design notes and test results
images/     Circuit and simulation screenshots
```

## Limitations and Future Improvements

- One package is processed at a time.
- There is no waiting queue or priority scheduling.
- Stored packages remain counted until reset; unloading is not implemented.
- The design is a simulation and has not been validated as physical hardware.

Possible extensions include input queues, priority-based scheduling, controlled unloading and sensor-driven package capture.

## Team

# Version 1.0

- [Fuad BIN Sattar] — [STORAGE & CAPACITY]
- [Tahmeed Ahmed] — [CLASSIFICATION & ROUTING]
- [Addin Naim Robin] — [INPUT & CONVEYOR]

## Testing


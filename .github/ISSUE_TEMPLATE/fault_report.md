---
name: Fault report
about: A heater runaway, fire cut, welded switch or failed self-check, sent from the hub's dashboard
title: "Fault report: <fault> on <role>, firmware <version>"
labels: ["fault-report", "safety"]
---

<!-- The dashboard fills this in (doc 07 §8, D106). Check it before you send:
     passwords, tokens, plug credentials and notification URLs are stripped,
     but remove anything else you don't want public. -->

## What happened

- **Fault:** <!-- runaway RW-a / RW-b, fire, welded low side, welded high side, self-check failed -->
- **When:** <!-- date and time, and what the printer was doing -->
- **Self-check after the fault:** <!-- passed / failed, with the failing step -->

## System

- **Node role and board revision:**
- **Firmware and pack versions:** <!-- node, hub, printer pack, fan pack -->
- **Printer pack:** <!-- id and status -->
- **Heater supply:** <!-- tap, upgraded supply, brick -->
- **Mains-cut backend:**

## Attached by the dashboard

- [ ] Fault report (configuration, profile, self-check results)
- [ ] Event log
- [ ] Sensor history around the event

## Anything else

<!-- What you found when you inspected the printer, heater and wiring; photos help. -->

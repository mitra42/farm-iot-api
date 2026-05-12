# Farm IoT Interoperability Standard — CLAUDE.md

## Project Overview

This repo contains the **Farm IoT Interoperability Standard** (`API.md`), an early-draft specification defining a platform-to-platform API between:

- **Device-Platforms** (e.g. Frugal IoT, OurSci) — manage IoT sensors in the field
- **Farm-Platforms** (e.g. LiteFarm, FarmOS) — provide UX to farmers

The goal is a single shared API so any farm platform works with any device platform, avoiding lock-in and integration sprawl.

## Repository Contents

- `API.md` — the specification (primary document, all work centres here)
- `README.md` — one-line project description
- Two informational PDFs (needs assessment and shared API overview)

## API Architecture

**Protocol:** HTTP/HTTPS for all requests; MQTT optionally for Device→Farm push.  
**Data format:** SenML ([RFC 8428](https://www.rfc-editor.org/rfc/rfc8428)), JSON encoding only (`application/senml+json`).  
**Auth:** Cookie-based token agreed out-of-band during platform registration.  
**Device Schema:** W3C Web of Things (WoT) Thing Description (JSON-LD), fetched via `GET /devices/schema?device={id}`.

### Endpoints

| Direction | Endpoint | Purpose |
|---|---|---|
| Farm → Device | `GET /data?device=&from=&to=` | Fetch historical sensor data |
| Farm → Device | `POST /users/register` | Register a Farm-Platform user |
| Farm → Device | `POST /devices/register` | Register a device to a user |
| Farm → Device | `POST /devices/action` | Send actuation / setpoint change |
| Device → Farm | `POST /data` | Push sensor readings (SenML body) |
| Device → Farm | `POST /notification` | Push alert/event notification |
| Either | `GET /devices/schema?device=` | Fetch WoT Thing Description |

### SenML Packet Rules (narrower than RFC 8428)

- First record MUST contain `bn` (base name = device id) and `bt` (Unix timestamp ≥ 2^28), with **no value fields** in that record.
- Field names (`n`) use `module/field` form (e.g. `sht/temperature`).
- One pack = one device only.
- No CBOR, XML, CoAP, or `s` (sum) field.

### Device ID Format

`org/project/device` — e.g. `dev/lotus/esp32-123456`

### Error Response Shape

```json
{ "error": "device_not_found", "message": "Human readable." }
```

## Key Terminology

| Term | Meaning |
|---|---|
| Device | Physical sensor/actuator (e.g. ESP32 node) |
| Device-Platform | Backend managing devices (e.g. frugaliot.naturalinnovation.org) |
| Farm-Platform | Farmer-facing app (e.g. LiteFarm, FarmOS) |
| Farm | Logical group of devices on a Farm-Platform |
| Actuation | Sending a control command to a device |
| Device Schema | WoT Thing Description describing a device's properties and actions |
| `frugal-iot:` | Vendor namespace for Frugal IoT extensions in JSON-LD |

## Document Status and Style

- **Status:** Early draft (v0.1, 2026-03-09). Not reviewed by any standards body.
- Uses RFC 2119 normative language (MUST, SHOULD, MAY, etc.) — preserve this precisely when editing.
- Sections marked `[TBD]` (Security §8, Privacy §9, Annex E) are intentional placeholders.
- User stories in Annex C are informative, not normative.

## Known Open Issues (§11)

- Auth scheme not specified (bearer tokens / OAuth2 / API keys — TBD)
- No mechanism for Farm-Platform to discover available devices
- MQTT binding details not specified
- Multi-device packs not yet supported
- Notification subscription/configuration not defined
- Device status / diagnostics query not defined

## Contribution Notes

- This is a **documentation-only** repo — no build system, no tests, no CI.
- All changes go in `API.md`. Preserve the existing section numbering and heading style.
- The primary author is Mitra Ardron (Natural Innovation). Target audiences: Frugal IoT, OurSci, LiteFarm, FarmOS developers.
- Vendor extensions belong under a namespaced prefix in `@context` (e.g. `frugal-iot:`, `litefarm:`).

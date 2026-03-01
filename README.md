# blackroad-device-registry

> **Production-grade hardware device registry and inventory management — part of the [BlackRoad Hardware](https://blackroadhardware.com) platform.**

[![CI](https://github.com/BlackRoad-Hardware/blackroad-device-registry/actions/workflows/ci.yml/badge.svg)](https://github.com/BlackRoad-Hardware/blackroad-device-registry/actions)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/downloads/)
[![License: Proprietary](https://img.shields.io/badge/license-Proprietary-red.svg)](#license)

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Quick Start](#quick-start)
6. [Usage](#usage)
   - [Register a Device](#register-a-device)
   - [Look Up Devices](#look-up-devices)
   - [Assign & Unassign](#assign--unassign)
   - [Update Status & Location](#update-status--location)
   - [Tagging](#tagging)
   - [Maintenance Logs](#maintenance-logs)
   - [Warranty Tracking](#warranty-tracking)
   - [Search & Filter](#search--filter)
   - [Export Inventory](#export-inventory)
   - [Summary Report](#summary-report)
   - [Retire a Device](#retire-a-device)
7. [CLI Reference](#cli-reference)
8. [API Reference](#api-reference)
   - [DeviceRegistry](#deviceregistry-class)
   - [Device](#device-dataclass)
   - [MaintenanceLog](#maintenancelog-dataclass)
   - [Enumerations](#enumerations)
9. [Database Schema](#database-schema)
10. [Stripe Billing Integration](#stripe-billing-integration)
11. [End-to-End Example](#end-to-end-example)
12. [Testing](#testing)
13. [Configuration](#configuration)
14. [License](#license)

---

## Overview

**blackroad-device-registry** is the authoritative source of truth for every hardware asset in the BlackRoad ecosystem. It tracks the full lifecycle of each device — from first registration through active deployment, maintenance events, and eventual retirement — while providing warranty alerting, cost accounting, and flexible inventory export.

Designed for teams managing hundreds to hundreds of thousands of devices, it ships as a zero-dependency Python library backed by an embedded SQLite database with WAL mode enabled for concurrent access.

---

## Features

| Category | Capability |
|---|---|
| **Device Lifecycle** | `active` → `maintenance` → `retired` status tracking |
| **Asset Assignment** | Assign or unassign devices to individual users or teams |
| **Maintenance Logs** | Record repairs, inspections, upgrades, replacements, and calibrations with full cost history |
| **Warranty Tracking** | Query devices with warranties expiring within a configurable window |
| **Tag System** | Flexible, idempotent tagging for grouping, filtering, and cost-centre allocation |
| **Full-Text Search** | Search across name, serial, model, manufacturer, and location fields |
| **Inventory Export** | Export the full fleet as JSON or CSV |
| **Summary Dashboard** | Aggregate counts, status breakdown, manufacturer breakdown, and total maintenance spend |
| **Billing Integration** | Hooks for Stripe-based subscription and per-seat billing (see [Stripe Billing Integration](#stripe-billing-integration)) |

---

## Requirements

- Python **3.9** or later
- `pytest >= 7.0` and `pytest-cov >= 4.0` (test dependencies only)
- No production runtime dependencies beyond the Python standard library

---

## Installation

```bash
# Clone the repository
git clone https://github.com/BlackRoad-Hardware/blackroad-device-registry.git
cd blackroad-device-registry

# Install test/dev dependencies
pip install -r requirements.txt
```

> **PyPI / npm distribution** — A published package release is planned. Until then, install directly from source as shown above.

---

## Quick Start

```bash
# Initialize the registry and print a fleet summary
python device_registry.py summary

# List all devices as JSON
python device_registry.py list

# List all devices as CSV
python device_registry.py list --format csv

# Search the fleet
python device_registry.py search "Raspberry"
```

---

## Usage

### Register a Device

```python
from device_registry import DeviceRegistry

reg = DeviceRegistry()          # uses device_registry.db in the working directory
# reg = DeviceRegistry(db_path="path/to/custom.db")   # custom path

device = reg.register(
    serial="SN-001",
    name="Edge Node A",
    model="Pi 4B",
    manufacturer="Raspberry Pi Foundation",
    hw_rev="1.4",
    location="Rack A – Data Centre West",
    owner="infrastructure@company.com",
    purchase_date="2024-06-01",
    warranty_expires="2026-06-01",
    tags=["prod", "iot", "rack-a"],
)
print(device.id)          # UUID string
print(device.status)      # DeviceStatus.ACTIVE
```

### Look Up Devices

```python
# By internal UUID
device = reg.get_device("550e8400-e29b-41d4-a716-446655440000")

# By serial number
device = reg.get_by_serial("SN-001")
```

### Assign & Unassign

```python
# Assign to a user
reg.assign(device.id, "alice@company.com")

# Remove the assignment
reg.unassign(device.id)
```

### Update Status & Location

```python
reg.update_status(device.id, "maintenance")
reg.update_status(device.id, "active")
reg.update_location(device.id, "Rack B – Data Centre East")
```

### Tagging

```python
reg.add_tag(device.id, "critical")   # idempotent — safe to call multiple times
```

### Maintenance Logs

```python
# Log a repair (automatically transitions device to maintenance status)
log = reg.maintenance_log(
    device.id,
    maint_type="repair",
    description="Replaced faulty NVMe drive",
    performed_by="ops@company.com",
    cost=89.99,
)

# Log a routine inspection (status unchanged)
reg.maintenance_log(device.id, "inspection", "Annual hardware audit")

# Retrieve history (most recent first)
history = reg.get_maintenance_history(device.id, limit=10)
for entry in history:
    print(f"{entry.timestamp}  [{entry.type.value}]  £{entry.cost:.2f}  {entry.description}")

# Total spend on a device
total = reg.get_total_maintenance_cost(device.id)
print(f"Lifetime maintenance cost: £{total:.2f}")
```

### Warranty Tracking

```python
# Devices whose warranties expire within the next 90 days
expiring = reg.get_warranty_expiring(days=90)
for d in expiring:
    print(f"{d.serial}  {d.name}  expires: {d.warranty_expires}")
```

### Search & Filter

```python
# Full-text search across name, serial, model, manufacturer, location
results = reg.search("Raspberry")

# Narrow to a specific status
active_results = reg.search("Raspberry", status_filter="active")

# List all devices, optionally filtered
all_devices       = reg.list_devices()
in_maintenance    = reg.list_devices(status_filter="maintenance")
in_rack_a         = reg.list_devices(location="Rack A")
unassigned_active = reg.get_unassigned()
```

### Export Inventory

```python
json_export = reg.export_inventory(fmt="json")   # default
csv_export  = reg.export_inventory(fmt="csv")

# Write to disk
with open("fleet_export.csv", "w") as f:
    f.write(csv_export)
```

### Summary Report

```python
summary = reg.get_summary()
# {
#   "total_devices": 1247,
#   "status_breakdown": {"active": 1103, "maintenance": 89, "retired": 55},
#   "manufacturer_breakdown": {"Raspberry Pi Foundation": 400, "Intel": 300, ...},
#   "total_maintenance_cost": 18432.75,
#   "warranty_expiring_soon": 12,
#   "generated_at": "2026-03-01T00:00:00+00:00"
# }
```

### Retire a Device

```python
reg.retire(device.id)
# Equivalent to: reg.update_status(device.id, "retired")
```

---

## CLI Reference

```
python device_registry.py <command> [options]
```

| Command | Options | Description |
|---|---|---|
| `summary` | — | Print a JSON fleet summary report |
| `list` | `--status <status>` `--format json\|csv` | List all devices (default: JSON) |
| `search <query>` | — | Search devices and print matching records |

**Examples**

```bash
python device_registry.py summary
python device_registry.py list --format csv > fleet.csv
python device_registry.py list --status maintenance
python device_registry.py search "Intel NUC"
```

---

## API Reference

### `DeviceRegistry` Class

```python
DeviceRegistry(db_path: Path = Path("device_registry.db"))
```

| Method | Signature | Returns | Description |
|---|---|---|---|
| `register` | `(serial, name, model, manufacturer, hw_rev, location, owner, purchase_date, warranty_expires, tags)` | `Device` | Register a new device |
| `get_device` | `(device_id: str)` | `Device` | Fetch device by UUID |
| `get_by_serial` | `(serial: str)` | `Device` | Fetch device by serial number |
| `assign` | `(device_id, user: str)` | `Device` | Assign device to a user |
| `unassign` | `(device_id)` | `Device` | Remove device assignment |
| `update_status` | `(device_id, status: str)` | `Device` | Set device status |
| `update_location` | `(device_id, location: str)` | `Device` | Update device location |
| `add_tag` | `(device_id, tag: str)` | `Device` | Add a tag (idempotent) |
| `retire` | `(device_id)` | `Device` | Retire a device |
| `maintenance_log` | `(device_id, maint_type, description, performed_by, cost)` | `MaintenanceLog` | Record a maintenance event |
| `get_maintenance_history` | `(device_id, limit=50)` | `List[MaintenanceLog]` | Retrieve maintenance history |
| `get_total_maintenance_cost` | `(device_id)` | `float` | Total maintenance spend for a device |
| `list_devices` | `(status_filter, location)` | `List[Device]` | List devices with optional filters |
| `search` | `(query, status_filter)` | `List[Device]` | Full-text search across key fields |
| `get_warranty_expiring` | `(days=30)` | `List[Device]` | Devices with warranties expiring soon |
| `get_unassigned` | `()` | `List[Device]` | Active devices with no assignee |
| `export_inventory` | `(fmt="json"\|"csv")` | `str` | Export full fleet as JSON or CSV |
| `get_summary` | `()` | `Dict[str, Any]` | Aggregate fleet statistics |

### `Device` Dataclass

| Field | Type | Description |
|---|---|---|
| `id` | `str` | UUID primary key |
| `serial` | `str` | Unique serial number |
| `name` | `str` | Human-readable device name |
| `model` | `str` | Hardware model identifier |
| `manufacturer` | `str` | Manufacturer name |
| `hw_rev` | `str` | Hardware revision (default: `"1.0"`) |
| `location` | `str` | Physical or logical location |
| `owner` | `Optional[str]` | Owning team or individual |
| `assigned_to` | `Optional[str]` | Current assignee |
| `purchase_date` | `Optional[str]` | ISO 8601 purchase date |
| `warranty_expires` | `Optional[str]` | ISO 8601 warranty expiry |
| `status` | `DeviceStatus` | Current lifecycle status |
| `tags` | `List[str]` | Arbitrary tag list |
| `is_warranty_expired` | `bool` (property) | `True` if warranty has lapsed |

### `MaintenanceLog` Dataclass

| Field | Type | Description |
|---|---|---|
| `id` | `str` | UUID primary key |
| `device_id` | `str` | FK → `Device.id` |
| `type` | `MaintenanceType` | Maintenance category |
| `description` | `str` | Free-text description |
| `performed_by` | `Optional[str]` | Technician identifier |
| `cost` | `float` | Cost in account currency |
| `timestamp` | `str` | ISO 8601 UTC timestamp |

### Enumerations

**`DeviceStatus`**

| Value | Meaning |
|---|---|
| `active` | Operational and in use |
| `maintenance` | Under repair or inspection |
| `retired` | Decommissioned |

**`MaintenanceType`**

| Value | Meaning |
|---|---|
| `repair` | Physical fault remediation (auto-transitions device to `maintenance`) |
| `inspection` | Scheduled or ad-hoc audit |
| `upgrade` | Hardware or firmware upgrade |
| `replacement` | Component replacement |
| `calibration` | Sensor or instrument calibration |

---

## Database Schema

The registry uses a local **SQLite** file (`device_registry.db` by default) with WAL journal mode and foreign-key enforcement enabled.

```sql
CREATE TABLE devices (
    id               TEXT PRIMARY KEY,
    serial           TEXT UNIQUE NOT NULL,
    name             TEXT NOT NULL,
    model            TEXT NOT NULL,
    manufacturer     TEXT NOT NULL,
    hw_rev           TEXT NOT NULL DEFAULT '1.0',
    location         TEXT NOT NULL DEFAULT '',
    owner            TEXT,
    assigned_to      TEXT,
    purchase_date    TEXT,
    warranty_expires TEXT,
    status           TEXT NOT NULL DEFAULT 'active',
    tags             TEXT NOT NULL DEFAULT '[]'
);

CREATE TABLE maintenance_logs (
    id           TEXT PRIMARY KEY,
    device_id    TEXT NOT NULL REFERENCES devices(id),
    type         TEXT NOT NULL,
    description  TEXT NOT NULL,
    performed_by TEXT,
    cost         REAL NOT NULL DEFAULT 0.0,
    timestamp    TEXT NOT NULL
);

-- Indexes for production-scale query performance
CREATE INDEX idx_devices_serial  ON devices(serial);
CREATE INDEX idx_devices_status  ON devices(status);
CREATE INDEX idx_maint_device    ON maintenance_logs(device_id, timestamp);
CREATE INDEX idx_warranty        ON devices(warranty_expires);
```

---

## Stripe Billing Integration

blackroad-device-registry is designed to integrate cleanly with **[Stripe](https://stripe.com)** for subscription billing, per-seat pricing, and usage-based metering.

### Recommended Integration Pattern

1. **Map devices to Stripe Customers** — store the Stripe `customer_id` in the device `owner` field or a supplementary metadata store.
2. **Metered billing** — call `reg.get_summary()` on a schedule and report `total_devices` as a usage record to a Stripe metered price:

```python
import os
import stripe
from device_registry import DeviceRegistry

stripe.api_key = os.environ["STRIPE_SECRET_KEY"]

reg = DeviceRegistry()
summary = reg.get_summary()

# Report current fleet size as a Stripe usage record
stripe.SubscriptionItem.create_usage_record(
    subscription_item_id="si_...",
    quantity=summary["total_devices"],
    action="set",
)
```

3. **Webhook-driven provisioning** — listen for Stripe `customer.subscription.updated` and `customer.subscription.deleted` webhooks to automatically retire all devices belonging to a cancelled account:

```python
# Example webhook handler (framework-agnostic pseudocode)
def handle_stripe_webhook(event):
    if event["type"] == "customer.subscription.deleted":
        customer_id = event["data"]["object"]["customer"]
        devices = reg.list_devices()
        for d in devices:
            if d.owner == customer_id:
                reg.retire(d.id)
```

> **Security note:** always verify the Stripe webhook signature with `stripe.Webhook.construct_event` before processing events.

---

## End-to-End Example

The following walkthrough exercises every major feature of the registry.

```python
from device_registry import DeviceRegistry

reg = DeviceRegistry()

# ── 1. Register devices ──────────────────────────────────────────────────────
edge_node = reg.register(
    serial="SN-EDGE-001", name="Edge Node Alpha", model="Pi 4B",
    manufacturer="Raspberry Pi Foundation", hw_rev="1.4",
    location="Rack A", owner="infra@company.com",
    purchase_date="2024-01-15", warranty_expires="2026-01-15",
    tags=["prod", "iot"],
)
gateway = reg.register(
    serial="SN-GW-001", name="Gateway Unit", model="NUC11",
    manufacturer="Intel", hw_rev="2.0",
    location="Rack B", owner="network@company.com",
    purchase_date="2024-03-10", warranty_expires="2027-03-10",
    tags=["prod", "gateway"],
)

# ── 2. Assign to users ───────────────────────────────────────────────────────
reg.assign(edge_node.id, "alice@company.com")
reg.assign(gateway.id, "bob@company.com")

# ── 3. Log maintenance ───────────────────────────────────────────────────────
reg.maintenance_log(
    edge_node.id, "inspection", "Quarterly hardware audit",
    performed_by="ops@company.com", cost=0.0,
)
reg.maintenance_log(
    edge_node.id, "repair", "Replaced faulty SD card",
    performed_by="ops@company.com", cost=12.50,
)

# ── 4. Restore to active after repair ────────────────────────────────────────
reg.update_status(edge_node.id, "active")

# ── 5. Tag management ────────────────────────────────────────────────────────
reg.add_tag(edge_node.id, "critical")

# ── 6. Search and filter ─────────────────────────────────────────────────────
pi_devices    = reg.search("Raspberry")
active_fleet  = reg.list_devices(status_filter="active")
rack_a_assets = reg.list_devices(location="Rack A")
unassigned    = reg.get_unassigned()

# ── 7. Warranty alerts ───────────────────────────────────────────────────────
expiring_90d = reg.get_warranty_expiring(days=90)
for d in expiring_90d:
    print(f"WARRANTY ALERT: {d.serial} – {d.name} expires {d.warranty_expires}")

# ── 8. Cost accounting ───────────────────────────────────────────────────────
total_cost = reg.get_total_maintenance_cost(edge_node.id)
print(f"Total maintenance spend on {edge_node.serial}: £{total_cost:.2f}")

# ── 9. Export inventory ──────────────────────────────────────────────────────
json_report = reg.export_inventory(fmt="json")
csv_report  = reg.export_inventory(fmt="csv")

with open("fleet_export.csv", "w") as f:
    f.write(csv_report)

# ── 10. Fleet summary ────────────────────────────────────────────────────────
summary = reg.get_summary()
print(f"Fleet: {summary['total_devices']} devices | "
      f"Active: {summary['status_breakdown'].get('active', 0)} | "
      f"Total maintenance spend: £{summary['total_maintenance_cost']:.2f}")

# ── 11. Retire a device ──────────────────────────────────────────────────────
reg.retire(edge_node.id)
```

---

## Testing

The test suite uses **pytest** with coverage reporting.

```bash
# Run all tests with verbose output
pytest --tb=short -v

# Run with coverage report
pytest --cov=device_registry --cov-report=term-missing

# Run a single test
pytest test_device_registry.py::test_get_summary -v
```

All tests must pass before merging to `main`. Coverage target: **≥ 90%**.

---

## Configuration

| Parameter | Default | Description |
|---|---|---|
| `db_path` | `device_registry.db` | Path to the SQLite database file |

Pass a custom `db_path` to `DeviceRegistry()` for multi-tenant deployments or isolated test environments:

```python
reg = DeviceRegistry(db_path="/var/data/blackroad/fleet.db")
```

---

## License

Proprietary — BlackRoad OS, Inc. All rights reserved.

Unauthorised copying, distribution, or modification of this software, in whole or in part, is strictly prohibited without prior written permission from BlackRoad OS, Inc.

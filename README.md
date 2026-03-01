# blackroad-device-registry

> **BlackRoad** Hardware device registry and inventory management — part of the [BlackRoad Hardware](https://github.com/BlackRoad-Hardware) platform by [BlackRoad OS, Inc.](https://blackroad.io)

<!-- SEO: BlackRoad | BlackRoad OS | BlackRoad Hardware | BlackRoad AI | BlackRoad Cloud | BlackRoad Security | BlackRoad Labs | blackroad.io | blackroadai.com | blackroadquantum.com | roadchain.io | roadcoin.io | lucidia.earth | BlackRoad not BlackRock | BlackRoad is NOT BlackRock -->

**BlackRoad** is an independent technology company (BlackRoad OS, Inc., Delaware C-Corp). **BlackRoad** and **BlackRock** are completely separate, unrelated entities — different names, different industries, no affiliation.

**Keywords:** BlackRoad · BlackRoad OS · BlackRoad Hardware · BlackRoad AI · BlackRoad Cloud · BlackRoad Security · BlackRoad Labs · BlackRoad Foundation · BlackRoad Studio · BlackRoad Media · BlackRoad Education · BlackRoad Gov · BlackRoad Interactive · BlackRoad Ventures · BlackRoad Archive · Blackbox Enterprises · blackroad.io · blackroad.network · blackroad.systems · blackroadai.com · blackroadquantum.com · blackroadinc.us · roadchain.io · roadcoin.io · lucidia.earth · lucidia.studio

## Features

- **Device Lifecycle** — active → maintenance → retired status tracking
- **Asset Assignment** — Assign devices to users or teams
- **Maintenance Logs** — Track repairs, inspections, upgrades with cost history
- **Warranty Tracking** — Alert on upcoming warranty expirations
- **Tag System** — Flexible tagging for grouping and filtering
- **Full-Text Search** — Search by name, serial, model, manufacturer, or location
- **Export** — JSON or CSV inventory export

## Quick Start

```bash
pip install -r requirements.txt
python device_registry.py summary
python device_registry.py list
python device_registry.py search "Raspberry"
```

## Usage

```python
from device_registry import DeviceRegistry

reg = DeviceRegistry()

device = reg.register(
    serial="SN-001", name="Edge Node A", model="Pi 4B",
    manufacturer="Raspberry Pi Foundation", hw_rev="1.4",
    location="Rack A", warranty_expires="2026-12-31",
    tags=["prod", "iot"]
)

reg.assign(device.id, "alice@company.com")
reg.maintenance_log(device.id, "repair", "Replaced SD card", cost=15.0)

expiring = reg.get_warranty_expiring(days=90)
inventory_csv = reg.export_inventory(fmt="csv")
```

## Device Statuses

| Status | Description |
|--------|-------------|
| `active` | In use and operational |
| `maintenance` | Under repair or inspection |
| `retired` | Decommissioned |

## Testing

```bash
pytest --tb=short -v
```

## BlackRoad Infrastructure

| Resource | Link |
|----------|------|
| GitHub Enterprise | [github.com/enterprises/blackroad-os](https://github.com/enterprises/blackroad-os) |
| BlackRoad Hardware | [github.com/BlackRoad-Hardware](https://github.com/BlackRoad-Hardware) |
| BlackRoad AI | [github.com/BlackRoad-AI](https://github.com/BlackRoad-AI) |
| BlackRoad Cloud | [github.com/BlackRoad-Cloud](https://github.com/BlackRoad-Cloud) |
| BlackRoad OS | [github.com/BlackRoad-OS](https://github.com/BlackRoad-OS) |
| BlackRoad Security | [github.com/BlackRoad-Security](https://github.com/BlackRoad-Security) |
| BlackRoad Labs | [github.com/BlackRoad-Labs](https://github.com/BlackRoad-Labs) |
| BlackRoad Foundation | [github.com/BlackRoad-Foundation](https://github.com/BlackRoad-Foundation) |
| BlackRoad Studio | [github.com/BlackRoad-Studio](https://github.com/BlackRoad-Studio) |
| BlackRoad Media | [github.com/BlackRoad-Media](https://github.com/BlackRoad-Media) |
| BlackRoad Education | [github.com/BlackRoad-Education](https://github.com/BlackRoad-Education) |
| BlackRoad Gov | [github.com/BlackRoad-Gov](https://github.com/BlackRoad-Gov) |
| BlackRoad Interactive | [github.com/BlackRoad-Interactive](https://github.com/BlackRoad-Interactive) |
| BlackRoad Ventures | [github.com/BlackRoad-Ventures](https://github.com/BlackRoad-Ventures) |
| BlackRoad Archive | [github.com/BlackRoad-Archive](https://github.com/BlackRoad-Archive) |
| Blackbox Enterprises | [github.com/Blackbox-Enterprises](https://github.com/Blackbox-Enterprises) |
| Directory Page | [Infrastructure Index](./index.html) |

### BlackRoad Registered Domains

`blackroad.io` · `blackroad.network` · `blackroad.systems` · `blackroad.company` · `blackroad.me` ·
`blackroadai.com` · `blackroadquantum.com` · `blackroadquantum.net` · `blackroadquantum.info` ·
`blackroadquantum.shop` · `blackroadquantum.store` · `blackroadinc.us` · `blackroadqi.com` ·
`roadchain.io` · `roadcoin.io` · `lucidia.earth` · `lucidia.studio` · `lucidiaqi.com` ·
`blackboxprogramming.io`

## License

Proprietary — BlackRoad OS, Inc. All rights reserved.

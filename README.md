# Mensa API

> Inoffizielle REST API für den Speiseplan der Mensa Friedrichshafen (DHBW / HSKA).  
> Datenquelle: Max-Manager (Seezeit Bodensee)

![Status](https://img.shields.io/website?url=https%3A%2F%2Fmensa-api.timothyklimke.de&label=Mensa%20API&style=flat-square)

---

## Was ist das?

Die Mensa API wandelt die XML-Daten des Max-Managers in ein modernes JSON-Format um und stellt aktuelle sowie historische Speiseplandaten strukturiert bereit – inklusive Gerichten, Zusatzstoffen, Tags (vegan, vegetarisch, …) und Preisen für alle Nutzergruppen.

**Base URL:** `https://mensa-api.timothyklimke.de`

---

## Quick Start

```bash
curl -H "x-api-key: DEIN_KEY" \
  "https://mensa-api.timothyklimke.de/api/v1/dish-schedule?from=2026-04-21&to=2026-04-25"
```

```python
import requests

response = requests.get(
    "https://mensa-api.timothyklimke.de/api/v1/dish-schedule",
    params={"from": "2026-04-21", "to": "2026-04-25"},
    headers={"x-api-key": "DEIN_KEY"}
)

print(response.json())
```

---

## Endpunkte

| Methode | Endpunkt | Beschreibung |
| :--- | :--- | :--- |
| `GET` | `/api/v1/dish-schedule` | Speiseplan für einen Zeitraum (max. 15 Tage) |
| `GET` | `/api/v1/dishes` | Alle bekannten Gerichte |
| `GET` | `/api/v1/dishes/id` | Gerichtdetails per UUID |
| `GET` | `/api/v1/prices` | Aktuelle Preisstruktur |

→ Vollständige Dokumentation: [`DOCS.md`](./DOCS.md)

---

## Zugang

Die API ist **invite-only**.

---

## Deployment

Gehostet auf AWS Lightsail.

---

## Datenschutz

→ [`PRIVACY.md`](./PRIVACY.md)

# Mensa API

> Inoffizielle REST-API für den Speiseplan der DHBW-Mensa Friedrichshafen.
> Datenquelle: Max-Manager (Seezeit Bodensee)

---

## Überblick

Die Mensa API wandelt die XML-Daten des Max-Managers in ein modernes JSON-Format um
und stellt aktuelle wie historische Speiseplandaten strukturiert bereit – inklusive
Gerichten, Zusatzstoffen, Tags (vegan, vegetarisch, …) und Preisen für alle
Nutzergruppen.

**Base URL:** `https://api.mensa-fn.de`
**Authentifizierung:** API-Key im Header `x-api-key` (alle Endpunkte)

---

## Quick Start

Speiseplan für einen Zeitraum abrufen:

```bash
curl -H "x-api-key: API_KEY" \
  "https://api.mensa-fn.de/api/v1/dish-schedule?from=2026-06-08&to=2026-06-12"
```

```python
import requests

response = requests.get(
    "https://api.mensa-fn.de/api/v1/dish-schedule",
    params={"from": "2026-06-08", "to": "2026-06-12"},
    headers={"x-api-key": "DEIN_KEY"},
)

print(response.json())
```

Tipp: Mit `&include_all=true` liefert der Speiseplan Gericht **und** Preis bereits
eingebettet – dann genügt eine einzige Anfrage:

```bash
curl -H "x-api-key: API_KEY" \
  "https://api.mensa-fn.de/api/v1/dish-schedule?from=2026-06-08&to=2026-06-12&include_all=true"
```

---

## Endpunkte

| Methode | Endpunkt | Beschreibung |
| :--- | :--- | :--- |
| `GET` | `/api/v1/dish-schedule` | Speiseplan für einen Zeitraum (max. 16 Tage) |
| `GET` | `/api/v1/dishes` | Alle bekannten Gerichte |
| `GET` | `/api/v1/dishes/id` | Gerichtdetails per UUID |
| `GET` | `/api/v1/prices` | Aktuelle Preisstruktur |

→ Vollständige Dokumentation: [`DOCUMENTATION.md`](./DOCUMENTATION.md)

---

## Deployment

Gehostet auf AWS Lightsail.

---

## Datenschutz

→ [`PRIVACY.md`](./PRIVACY.md)

# Mensa API – Dokumentation

> **Inoffizielle API** | Betreiber: Timothy Klimke | Basis-URL: `https://mensa-api.timothyklimke.de`

Diese API wandelt die XML-Daten des Max-Managers (Seezeit) in ein modernes JSON-Format um und stellt aktuelle sowie historische Speiseplandaten strukturiert bereit.

---

## Inhaltsverzeichnis

1. [Authentifizierung](#1-authentifizierung)
2. [Endpunkte](#2-endpunkte)
   - [GET /api/v1/dish-schedule](#get-apiv1dish-schedule)
   - [GET /api/v1/dishes/id](#get-apiv1dishesid)
   - [GET /api/v1/dishes](#get-apiv1dishes)
   - [GET /api/v1/prices](#get-apiv1prices)
3. [Fehlerbehandlung](#3-fehlerbehandlung)
4. [Beispiel-Implementierung](#4-beispiel-implementierung)

---

## 1. Authentifizierung

Alle Anfragen müssen einen gültigen API-Key enthalten. Dieser wird als HTTP-Header übergeben.

| Header | Beschreibung |
| :--- | :--- |
| `x-api-key` | Dein persönlicher API-Key |
| `Content-Type` | `application/json` |

> **Hinweis:** Anfragen ohne gültigen `x-api-key`-Header werden mit `401 Unauthorized` abgelehnt.

---

## 2. Endpunkte

### GET /api/v1/dish-schedule

Gibt alle Speiseplaneinträge für einen definierten Zeitraum zurück.

**Parameter**

| Parameter | Typ | Pflicht | Beschreibung |
| :--- | :--- | :--- | :--- |
| `from` | `string` | Nein | Startdatum im Format `YYYY-MM-DD` |
| `to` | `string` | Nein | Enddatum im Format `YYYY-MM-DD` |

Werden keine Parameter angegeben, gibt die API den aktuellen Speiseplan zurück.

> **Hinweis:** Der maximal abfragbare Zeitraum beträgt **16 Tage**. Anfragen mit einem längeren Zeitraum werden mit `400 INVALID_DATE_RANGE_TOO_LONG` abgelehnt. Außerdem muss `from` ≤ `to` sein, andernfalls antwortet die API mit `400 INVALID_DATE_RANGE`.

**Beispiel-Request**

```
GET /api/v1/dish-schedule?from=2026-04-21&to=2026-04-30
```

**Antwortstruktur**

| Feld | Typ | Beschreibung |
| :--- | :--- | :--- |
| `plan_id` | `string (UUID)` | Eindeutige ID dieses Speiseplaneintrags |
| `date` | `string (ISO 8601)` | Datum, an dem das Gericht angeboten wird |
| `category` | `string` | Ausgabestation (z. B. `"Menü I"`, `"Menü II"`) |
| `dish_id` | `string (UUID)` | Referenz auf das Gericht → siehe [`/api/v1/dishes/id`](#get-apiv1dishesid) |
| `price_id` | `string (UUID)` | Referenz auf die Preisstruktur → siehe [`/api/v1/prices`](#get-apiv1prices) |
| `as_of` | `string (ISO 8601)` | Zeitstempel der letzten Aktualisierung durch den Crawler |

**Beispiel-Antwort**

```json
{
    "plan_id": "019db039-1ca1-7ee0-a5ac-bbbc2a408bd7",
    "date": "2026-04-30T00:00:00.000Z",
    "category": "Menü II",
    "dish_id": "019db039-1ca0-7605-b89b-a8f251ef1c01",
    "price_id": "019db039-1c4c-7776-9671-3d568f9ad2b5",
    "as_of": "2026-04-22T03:16:00.205Z"
}
```

---

### GET /api/v1/dishes/id

Gibt detaillierte Informationen zu einem oder mehreren Gerichten anhand ihrer UUIDs zurück.

**Parameter**

| Parameter | Typ | Pflicht | Beschreibung |
| :--- | :--- | :--- | :--- |
| `ids` | `string` | Ja | Kommagetrennte Liste von Gericht-UUIDs |

**Beispiel-Request**

```
GET /api/v1/dishes/id?ids=019db039-1c82-7f7f-a399-f40596ba35c2
```

Mehrere IDs:

```
GET /api/v1/dishes/id?ids=<uuid1>,<uuid2>,<uuid3>
```

---

### GET /api/v1/dishes

Gibt eine Liste aller in der Datenbank bekannten Gerichte zurück. Dieser Endpunkt eignet sich für Suchfunktionen, Statistiken oder das Aufbauen eines lokalen Caches.

**Keine Parameter erforderlich.**

**Antwortstruktur (pro Eintrag)**

| Feld | Typ | Beschreibung |
| :--- | :--- | :--- |
| `id` | `string (UUID)` | Eindeutige ID des Gerichts |
| `name` | `string` | Name des Gerichts (ohne Zusatzstoffnummern) |
| `name_additives` | `string` | Name des Gerichts inklusive Zusatzstoffnummern in Klammern |
| `additives` | `string[]` | Liste aller im Gericht enthaltenen Zusatzstoff-IDs |
| `tags` | `string[]` | Kennzeichnungen (z. B. `vegan`, `vegetarisch`, `schweinefleischfrei`) |
| `ingredients` | `object[]` | Array der einzelnen Komponenten (Hauptspeise, Beilage, Sauce etc.) |
| `last_served` | `string (ISO 8601)` | Datum, an dem das Gericht zuletzt auf dem Speiseplan stand |
| `as_of` | `string (ISO 8601)` | Zeitstempel der letzten Aktualisierung durch den Crawler |

**Struktur eines `ingredients`-Eintrags**

| Feld | Typ | Beschreibung |
| :--- | :--- | :--- |
| `name` | `string` | Name der Komponente |
| `additives` | `string[]` | Zusatzstoff-IDs dieser Komponente |
| `sub_ingredients` | `object[]` | Untergeordnete Zutaten (kann leer sein) |

**Beispiel-Antwort**

```json
{
    "id": "019db039-1c82-7f7f-a399-f40596ba35c2",
    "name": "drei Gemüsefrikadellen mit Tomatensauce dazu Kartoffel-Wirsing Püree und Blattsalat",
    "name_additives": "drei Gemüsefrikadellen (25a,28) mit Tomatensauce dazu Kartoffel-Wirsing Püree (3,31) und Blattsalat",
    "additives": ["25a", "3", "28", "31"],
    "tags": ["3", "31"],
    "ingredients": [
        {
            "name": "drei Gemüsefrikadellen",
            "additives": ["25a", "28"],
            "sub_ingredients": []
        },
        {
            "name": "Tomatensauce",
            "additives": [],
            "sub_ingredients": []
        },
        {
            "name": "Kartoffel-Wirsing Püree",
            "additives": ["3", "31"],
            "sub_ingredients": []
        },
        {
            "name": "Blattsalat",
            "additives": [],
            "sub_ingredients": []
        }
    ],
    "last_served": "2026-04-27T00:00:00.000Z",
    "as_of": "2026-04-22T03:16:00.205Z"
}
```

---

### GET /api/v1/prices

Gibt die aktuelle Preisstruktur für alle Nutzergruppen zurück.

**Keine Parameter erforderlich.**

**Antwortstruktur**

| Feld | Typ | Beschreibung |
| :--- | :--- | :--- |
| `id` | `string (UUID)` | Eindeutige ID dieser Preisstruktur |
| `price.internal_students` | `string` | Preis für Studierende der eigenen Hochschule (in EUR) |
| `price.external_students` | `string` | Preis für Studierende anderer Hochschulen (in EUR) |
| `price.employees` | `string` | Preis für Hochschulmitarbeiter (in EUR) |
| `price.guests` | `string` | Preis für Gäste und sonstige Externe (in EUR) |

**Beispiel-Antwort**

```json
{
    "id": "019db039-1c43-7a8f-ace9-0ebb67efa1b4",
    "price": {
        "internal_students": "4.70",
        "external_students": "8.70",
        "employees": "5.70",
        "guests": "6.00"
    }
}
```

---

## 3. Fehlerbehandlung

Die API gibt standardisierte HTTP-Statuscodes mit einem JSON-Body zurück. Jede Fehlerantwort enthält die Felder `error` (maschinenlesbarer Code) und `error_message` (Beschreibung).

**Antwortstruktur bei Fehlern**

```json
{
    "error": "ERROR_CODE",
    "error_message": "ERROR_CODE"
}
```

**Fehlercodes**

| Statuscode | `error` | Beschreibung |
| :--- | :--- | :--- |
| `200 OK` | – | Anfrage erfolgreich |
| `400 Bad Request` | `INVALID_DATE_RANGE` | `from` liegt nach `to` (Zeitraum ist umgekehrt) |
| `400 Bad Request` | `INVALID_DATE_RANGE_TOO_LONG` | Angefragter Zeitraum überschreitet das Maximum von 16 Tagen |
| `401 Unauthorized` | `INVALID_API_KEY` | Fehlender oder ungültiger API-Key |
| `404 Not Found` | – | Ressource nicht gefunden (z. B. unbekannte UUID) |
| `500 Internal Server Error` | – | Interner Serverfehler |

**Beispiele**

```
401 Unauthorized
```
```json
{"error": "INVALID_API_KEY", "error_message": "INVALID_API_KEY"}
```

```
400 Bad Request  →  from > to
```
```json
{"error": "INVALID_DATE_RANGE", "error_message": "INVALID_DATE_RANGE"}
```

```
400 Bad Request  →  Zeitraum > 16 Tage
```
```json
{"error": "INVALID_DATE_RANGE_TOO_LONG", "error_message": "INVALID_DATE_RANGE_TOO_LONG"}
```

---

## 4. Beispiel-Implementierung

Das folgende Python-Beispiel ruft den Speiseplan für einen definierten Zeitraum ab und gibt alle Gerichte mit Name und Datum aus.

```python
import requests

API_KEY = "DEIN_API_KEY"
BASE_URL = "https://mensa-api.timothyklimke.de/api/v1"

headers = {
    "x-api-key": API_KEY,
    "Content-Type": "application/json"
}

# Schritt 1: Speiseplan abrufen
schedule_response = requests.get(
    f"{BASE_URL}/dish-schedule",
    params={"from": "2026-04-21", "to": "2026-04-30"},
    headers=headers
)

if schedule_response.status_code != 200:
    print(f"Fehler beim Abrufen des Speiseplans: {schedule_response.status_code}")
    exit()

schedule = schedule_response.json()

# Schritt 2: Alle eindeutigen dish_ids sammeln
dish_ids = list({entry["dish_id"] for entry in schedule})

# Schritt 3: Gerichtdetails abrufen
dishes_response = requests.get(
    f"{BASE_URL}/dishes/id",
    params={"ids": ",".join(dish_ids)},
    headers=headers
)

if dishes_response.status_code != 200:
    print(f"Fehler beim Abrufen der Gerichte: {dishes_response.status_code}")
    exit()

dishes = {dish["id"]: dish for dish in dishes_response.json()}

# Schritt 4: Ausgabe
for entry in schedule:
    dish = dishes.get(entry["dish_id"])
    date = entry["date"][:10]
    name = dish["name"] if dish else "Unbekanntes Gericht"
    print(f"{date} | {entry['category']}: {name}")
```

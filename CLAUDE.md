# CLAUDE.md – Kontext & Wiederkehrende Aufgaben für Claude

Dieses File erklärt Claude, worum es bei diesem Repository geht und wie
wiederkehrende Aufgaben (insbesondere die Amazon-Synchronisierung) durchzuführen sind.

---

## Projekt-Überblick

**Website:** https://www.theafox-books.com/
**GitHub Repo:** https://github.com/theafox01/theafox-books.com-webseite
**Amazon Autorenprofil:** https://www.amazon.de/stores/Andrea-Kramer/author/B0F6LS5XR3/allbooks
**GitHub Account:** theafox01

Die Website präsentiert alle Bücher der Autorin **Andrea Kramer** (Pseudonym theafox).
Bücher werden auf **Amazon KDP** veröffentlicht (Kindle & Paperback).

---

## Repo-Struktur

```
theafox-books.com-webseite/
├── index.html              # Hauptseite (lädt Bücher aus JSON)
├── style.css               # Styling
├── theafox_books.json      # ← ZENTRALE DATENDATEI (alle Bücher)
├── covers/                 # Buchcover-Bilder (Dateiname = ASIN.jpg)
│   ├── B0F61WN4WW.jpg
│   ├── B0F62QWMZS.jpg
│   └── ...
├── legal-notice.html
├── privacy-policy.html
└── CLAUDE.md               # Diese Datei
```

---

## JSON-Datenstruktur (`theafox_books.json`)

Jedes Buch hat folgende Felder (Struktur NICHT verändern):

```json
{
  "title": "Buchtitel",
  "subtitle": "Untertitel oder leer",
  "img": "covers/ASIN.jpg",
  "href": "https://www.amazon.de/dp/ASIN",
  "category": "Word Search",
  "author": "Andrea Kramer",
  "featured": false,
  "platform": "Amazon"
}
```

**Kategorien** (bestehende verwenden):
- `"Word Search"` – Wortsuchrätsel (Großteil der Bücher)
- `"Puzzles"` – Sudoku, Nurikabe, Mazes
- `"Children's Books"` – Kinderbücher (Runi the Rootling, etc.)
- `"Notebooks"` – Notizbücher, Sketchbooks
- `"Mindfulness"` – Chakra, Spiritual, Color-Bücher
- `"Health"` – Gesundheitsbücher (Payhip-Bücher)

**Platforms:** `"Amazon"` oder `"Payhip"`

---

## Wiederkehrende Aufgabe: Amazon-Synchronisierung

### Wann nötig?
Wenn neue Bücher auf Amazon erschienen sind, die noch nicht in der JSON-Datei stehen.

### Vorgehensweise (Schritt für Schritt)

**Schritt 1 – Repo klonen**
```bash
gh repo clone theafox01/theafox-books.com-webseite /tmp/theafox-books
```

**Schritt 2 – Abhängigkeiten installieren**
```bash
pip3 install playwright requests beautifulsoup4
python3 -m playwright install chromium
```

**Schritt 3 – Amazon scrapen (Suche, nicht Autorenseite!)**

Die Autorenseite (`/stores/...`) lädt Bücher per JavaScript und ist schlecht scrapbar.
Stattdessen die **Amazon-Suche** verwenden (paginiert, zuverlässig):

```python
# Kindle-Bücher
https://www.amazon.de/s?k=Andrea+Kramer&i=digital-text&search-type=ss&field-author=Andrea+Kramer&s=date-desc-rank

# Physische Bücher
https://www.amazon.de/s?k=Andrea+Kramer&rh=p_27%3AAndrea+Kramer&search-type=ss&s=date-desc-rank
```

**Schritt 4 – Filter anwenden**

Nur **B0\* ASINs** behalten (= Amazon KDP Bücher der richtigen Andrea Kramer).
Numerische ISBNs (wie `3965433792`) sind andere Autorinnen mit gleichem Namen → ausschließen.

Bekannte falsche ASINs (andere Andrea Kramers) ausschließen:
- `B0F9K9Y94Y` – Belletristik, andere Autorin
- `B01HAJOXC4` – alte Kurzgeschichten, andere Autorin
- `B007R77DMM`, `B007I4AG1O` – amerikanische Autorin
- `B07583YCLJ`, `B0756MS7S6` – TV-Forschungsbücher, akademisch
- `B00XJ5ONHO` – akademisch
- `B0BQ8F88P1` – Mathematik-Journal

**Schritt 5 – Cover herunterladen**

Für jedes neue Buch: Amazon-Produktseite aufrufen, Bild-URL extrahieren, als `covers/ASIN.jpg` speichern.

Fallback-URL wenn kein Bild gefunden:
```
https://images-na.ssl-images-amazon.com/images/P/ASIN.01._SCLZZZZZZZ_.jpg
```

**Schritt 6 – JSON aktualisieren**

Neue Einträge am Ende der JSON-Liste anhängen. Bestehende Einträge NICHT verändern.

**Schritt 7 – Commit & Push**
```bash
git add theafox_books.json covers/*.jpg
git commit -m "Sync: X neue Bücher von Amazon hinzugefügt"
git push origin main
```

---

## Wichtige Hinweise

- **Keine Duplikate:** Vor dem Hinzufügen immer prüfen ob ASIN bereits in JSON vorhanden
- **Encoding:** JSON mit `ensure_ascii=False` speichern (deutsche Umlaute bleiben erhalten)
- **Cover-Dateinamen:** Immer `ASIN.jpg` (Großbuchstaben, kein Suffix außer .jpg)
- **Amazon blockiert** direkte HTTP-Requests → **Playwright** mit Chromium verwenden
- **Git-Identität** (falls nicht global gesetzt):
  ```bash
  git config user.name "theafox01"
  git config user.email "theafox01@users.noreply.github.com"
  ```

---

## Stand der letzten Synchronisierung

- **Datum:** 2026-04-05
- **JSON-Einträge vorher:** 151
- **JSON-Einträge nachher:** 214
- **Neu hinzugefügt:** 63 Bücher
- **Amazon zeigt insgesamt:** ~272 Bücher
- **Noch nicht erfasst (~58):** Vermutlich Kindle-Duplikate von Paperback-ASINs
  oder sehr neue Bücher die noch nicht in der Suche indexiert sind

---

## Sync-Skripte

Die fertigen Python-Skripte liegen im `claude-setup` Repository:
```
https://github.com/theafox01/claude-setup
```

Oder direkt neu schreiben – der Aufbau ist in diesem CLAUDE.md vollständig beschrieben.

# 🚐🧘‍♀️ YogaVan AI – KI-Routen- und Kapazitätsplaner für mobile Dienstleister

## 1. Ziel des Projekts
**YogaVan AI** ist ein automatisierter, KI-gestützter Planungsassistent für mobile Dienstleister (am Beispiel eines mobilen Yoga-Studios im Wohnwagen). 

Mobile Unternehmer stehen wöchentlich vor der komplexen Herausforderung, drei dynamische Faktoren unter einen Hut zu bringen:
1. **Wetterbedingungen** (Outdoor-Yoga ist stark wetterabhängig),
2. **Logistische Machbarkeit** (Stellplätze mit ausreichend Platz für Wohnwagen-Gespanne und Stromanschluss),
3. **Lokale Nachfrage** (touristische Auslastung und Zielgruppen-Potenzial vor Ort).

**Ziel dieses Projekts** ist es, den zeitintensiven, manuellen Planungsprozess durch einen automatisierten **n8n-Workflow** zu ersetzen. Das System liest potenzielle Tour-Standorte ein, reichert diese automatisch mit aktuellen 7-Tage-Wetterprognosen an und lässt **Google Gemini** eine fundierte, wirtschaftlich optimierte Routen- und Aufenthaltsempfehlung erstellen.

---

## 2. Beschreibung des Workflows

Der Workflow automatisiert die komplette Datenbeschaffung, Aufbereitung und Entscheidungsfindung in sechs Schritten:

1. **Trigger (`Planung starten`):** Der Workflow wird manuell (oder optional wöchentlich per Zeitplan) ausgelöst.
2. **Standortdaten laden (`Standorte CSV laden` & `CSV zu JSON parsen`):** Die Datei `standorte_beispieldaten.csv` mit möglichen Zielregionen (z. B. Bodensee, Allgäu, Chiemsee), Koordinaten, Stellplatzkosten, Gespann-Eignung und erwarteter Kundennachfrage wird eingelesen.
3. **Echtzeit-Wetterdaten abrufen (`Wetterprognose abrufen (Open-Meteo)`):** Für jeden Standort wird automatisiert die Open-Meteo-API angefragt, um Maximaltemperaturen, Niederschlagsmengen und Sonnenstunden der nächsten 7 Tage zu ermitteln.
4. **Daten-Aggregation (`Standorte & Wetter zusammenführen`):** Ein JavaScript-Code-Node verknüpft die CSV-Stammdaten mit den jeweiligen Wetterdaten, berechnet Durchschnittswerte und bündelt alle Standorte in einem strukturierten Prompt-Payload für die KI.
5. **KI-Routenoptimierung (`KI-Routenplaner (Google Gemini)`):** Über die Google Gemini API (`gemini-2.5-flash`) analysiert das Sprachmodell alle Standorte anhand klar definierter Business-Regeln (z. B. wenig Regen, hohe Nachfrage, strikte Gespann-Tauglichkeit, sinnvolle geografische Reihenfolge) und generiert einen konkreten 7-Tage-Tourenplan inkl. Begründung und wirtschaftlicher Einschätzung.
6. **Ausgabe & Export (`Tourenplan aufbereiten`, `In Markdown-Datei umwandeln` & `Tourenplan speichern`):** Das Ergebnis wird als übersichtlicher Markdown-Report (`tourenplan_ergebnis.md`) gespeichert und kann optional per E-Mail oder Telegram versendet werden.

### Visuelle Darstellung des Workflows

```mermaid
flowchart LR
    A([▶️ Trigger: Planung starten]) --> B[📂 CSV laden:<br/>standorte_beispieldaten.csv]
    B --> C[📄 CSV zu JSON parsen]
    C --> D[🌤️ HTTP Request:<br/>Open-Meteo Wetter-API]
    D --> E[⚙️ Code Node:<br/>Daten konsolidieren & Prompt bauen]
    E --> F[🤖 KI-Analyse:<br/>Google Gemini 2.5 Flash]
    F --> G[📝 Tourenplan formatieren]
    G --> H[(💾 Export:<br/>tourenplan_ergebnis.md)]
```

---

## 3. Verwendete Dienste & Technologien

| Dienst / Tool | Einsatzbereich im Projekt | Besonderheit |
| :--- | :--- | :--- |
| **n8n (Self-Hosted / Docker)** | Zentrale Automatisierungs- & Workflow-Engine | Visuelle Orchestrierung, volle Datenkontrolle |
| **Open-Meteo API** | Abruf von 7-Tage-Wetterprognosen (Temperatur, Regen, Sonne) | Kostenlos, DSGVO-freundlich, kein API-Key nötig |
| **Google Gemini API (`gemini-2.5-flash`)** | Bewertung der Standorte & Erstellung des Routenplans | Schnelles, kosteneffizientes LLM über Google AI Studio |
| **JavaScript (n8n Code Node)** | Datenbereinigung, Aggregation & JSON-Payload-Bau | Verhindert unnötige mehrfache LLM-Aufrufe |
| **CSV-Dateisystem** | Datenhaltung für potenzielle Tour-Standorte | Einfach pflegbar (z. B. über Excel oder Google Sheets) |

---

## 4. Zusätzliche Dateien im Repository

* `workflow.json` – Exportierter, direkt importierbarer n8n-Workflow (vorkonfiguriert für Google Gemini).
* `standorte_beispieldaten.csv` – Beispieldatensatz mit 6 Zielregionen in Süddeutschland inkl. Koordinaten, Stellplatzkosten, Gespann-Eignung und Nachfrage-Score.
* `tourenplan_ergebnis.md` *(wird bei Ausführung generiert)* – Fertiger KI-Tourenplan als Markdown-Dokument.

---

## 5. Screenshots

> **Hinweis für die Abgabe:** Lege in deinem Repository einen Ordner `screenshots/` an und füge dort die unten aufgeführten Bilder aus deiner laufenden n8n-Instanz ein.

### Gesamtansicht des n8n-Workflows
![Gesamtansicht des n8n-Workflows](screenshots/01_workflow_uebersicht.png)
*Abbildung 1: Vollständiger Workflow in n8n von der CSV-Einlesung bis zum KI-Export.*

### Wetterdaten-Abruf & Daten-Aggregation
![Open-Meteo API Response](screenshots/02_wetter_und_aggregation.png)
*Abbildung 2: Erfolgreicher Abruf der 7-Tage-Wetterdaten über die Open-Meteo API und Zusammenführung im Code-Node.*

### Generierter KI-Tourenplan (Output)
![Generierter Tourenplan](screenshots/03_ki_ergebnis_tourenplan.png)
*Abbildung 3: Beispiel-Output von Google Gemini mit empfohlener Route, Aufenthaltsdauer und Wetter-/Nachfrage-Begründung.*

---

## 6. Besondere Herausforderungen

1. **Verarbeitung mehrerer Items vor dem KI-Aufruf (Aggregation):**
   Standardmäßig führt n8n jeden nachfolgenden Node für *jedes einzelne Item* (also für jeden Standort einzeln) aus. Würde man Google Gemini direkt nach der Wetter-API anbinden, bekäme das LLM 6 getrennte Anfragen und könnte die Standorte nicht miteinander vergleichen. **Lösung:** Einsatz eines JavaScript-Code-Nodes (`$input.all()`), der alle Standorte inklusive ihrer Wetterdaten in ein einziges JSON-Payload zusammenführt.
2. **Zuverlässigkeit & Struktur der KI-Antworten (Prompt Engineering):**
   Ohne klare Vorgaben neigen LLMs dazu, zu allgemeine Reisetipps zu geben oder wichtige Restriktionen (wie die Gespann-Tauglichkeit des Wohnwagens) zu ignorieren. **Lösung:** Verwendung präziser Business-Regeln im Prompt mit klaren Ausschlusskriterien (harte Filter wie `gespann_geeignet = Nein`) und einer fest definierten Markdown-Ausgabestruktur.
3. **Sauberes JSON-Escaping für die Gemini REST-API:**
   Beim direkten Einfügen mehrzeiliger Texte in einen HTTP-Request-Body entstehen schnell ungültige JSON-Strings. **Lösung:** Das komplette `contents`-Objekt für die Gemini API wird bereits sauber im vorgeschalteten Code-Node als JavaScript-Objekt konstruiert und im HTTP-Node per `JSON.stringify()` übergeben.

---

## 7. Lernerfahrungen

* **Datenfluss-Verständnis in n8n:** Ich habe ein tiefes Verständnis dafür entwickelt, wie n8n Daten als Listen von Items verarbeitet und wann eine Aggregation (Many-to-One) notwendig ist, um komplexe Vergleiche durch eine KI durchführen zu lassen.
* **Kombination aus deterministischer Logik und KI:** Nicht alles sollte die KI erledigen. Das Berechnen von Durchschnittstemperaturen und Regenmengen im Code-Node spart Token-Kosten, reduziert Halluzinationen und macht den Workflow deutlich robuster.
* **Praxisnahe API-Integration:** Durch die Anbindung externer REST-APIs (Open-Meteo und Google Gemini) habe ich gelernt, dynamische Parameter aus vorherigen Nodes sauber zu übergeben und verschachtelte JSON-Antworten (`candidates[0].content.parts[0].text`) gezielt auszulesen.
* **Wirtschaftlicher Blickwinkel:** Bei der Entwicklung wurde klar, dass der größte Mehrwert nicht in der reinen Technik liegt, sondern in der Verknüpfung von Echtzeitdaten (Wetter) mit betriebswirtschaftlichen Kennzahlen (Stellplatzkosten vs. erwartete Kurseinnahmen).
# Projektabmachungen

Was hier steht, gilt für das ganze Repo und für alle, die daran arbeiten.
Änderungen an diesen Abmachungen werden gemeinsam beschlossen und hier nachgeführt.

## Aufbau

- Das Repo ist ein **Monorepo**.
- **Jeder Knoten hat einen eigenen Ordner**, z. B. ein Gateway pro
  Hardware-Variante. Ein Knotenordner ist in sich abgeschlossen:
  Konfiguration, Code und Doku liegen beisammen.

## Arbeitsbereiche

- **Ein Chat gehört zu genau einem Knoten** und arbeitet ausschliesslich in
  dessen Ordner.
- Alles ausserhalb der Knotenordner (`projekt.md`, `README.md`, `docs/`,
  `shared/`, `.gitignore`) gehört dem **Projekt-Chat**.
- **Neue Knoten legt der Projekt-Chat an**, aber nur als Grundgerüst:
  Pflichtdateien, Grundkonfiguration und Vorlage für die Zugangsdaten. Danach
  übernimmt ein eigener Chat den Knoten, und der Projekt-Chat arbeitet dort
  nicht mehr.
- In einem fremden Bereich darf nur gelesen werden. Ändern, Umbenennen und
  Löschen sind dort verboten, auch bei kleinen oder schon erledigten Dingen.

### Posteingang

Muss ein anderer Bereich angepasst werden, wird das nicht dort umgesetzt,
sondern **als Auftrag in dessen Posteingang geschrieben**. Wer den Bereich
bearbeitet, greift den Auftrag auf.

- Jeder Knoten hat seinen Posteingang in `<knoten>/POSTEINGANG.md`, der
  Projekt-Chat in `POSTEINGANG.md` im Hauptordner.
- Der Posteingang ist Teil des Repos.
- Aufträge werden nur **unten angefügt**. Bestehende Einträge bleiben
  unverändert. Fehlt die Datei, wird sie angelegt.
- Jeder Auftrag nennt Datum, Absender und genug Zusammenhang, dass er ohne die
  absendende Sitzung verständlich ist.
- Nur der Besitzer des Bereichs markiert einen Auftrag als erledigt oder
  räumt ihn weg.

Vorlage für einen Auftrag:

```markdown
## JJJJ-MM-TT – <kurzer Titel>

**Von:** <absendender Knoten oder Projekt-Chat>
**Status:** offen

<Was ist aufgefallen, was soll geschehen, warum>
```

## Pflichtdateien

Jeder Knotenordner enthält mindestens:

| Datei            | Inhalt                                                        |
|------------------|---------------------------------------------------------------|
| `README.md`      | Zweck, Hardware, Verdrahtung, Inbetriebnahme, aktuelle Version |
| `CHANGELOG.md`   | Alle Änderungen pro Version                                   |

Ein Knoten ohne diese beiden Dateien gilt als unvollständig. Der
`POSTEINGANG.md` entsteht mit dem ersten Auftrag.

## Versionierung

- Versioniert wird nach **[Semantic Versioning](https://semver.org/lang/de/)**:
  `MAJOR.MINOR.PATCH`.
  - **MAJOR**: inkompatible Änderung, z. B. an gemeinsamen Paketen, sodass
    alle Knoten mitgezogen werden müssen
  - **MINOR**: neue Funktion, rückwärtskompatibel
  - **PATCH**: Fehlerbehebung ohne neue Funktion
- **Jeder Knoten wird für sich versioniert**, MINOR und PATCH zählen pro Knoten.
- **Die MAJOR-Version ist bei allen Knoten gleich.** Sie steht für den Stand
  des Gesamtsystems. Eine inkompatible Änderung hebt deshalb MAJOR bei
  **allen** Knoten zugleich an. MINOR und PATCH beginnen dabei überall wieder
  bei `0`.
- Vor `1.0.0` gilt das System als Entwicklungsstand. Inkompatible Änderungen
  erhöhen dann MINOR beim betroffenen Knoten und stehen im CHANGELOG. Mit
  `1.0.0` starten alle Knoten gemeinsam.
- Eine neue Version wird an allen Stellen zugleich nachgeführt:
  1. Version im `README.md` des Knotens
  2. `version` unter `esphome: project:` in der Konfiguration
  3. neuer Abschnitt im `CHANGELOG.md` des Knotens
  4. Git-Tag in der Form `<knoten>/vMAJOR.MINOR.PATCH`, z. B. `gateway/v0.2.0`

## Changelog

Aufbau nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/):
neueste Version oben, Datum im Format `JJJJ-MM-TT`, Einträge gruppiert nach
*Hinzugefügt*, *Geändert*, *Behoben*, *Entfernt*. Laufende Änderungen sammeln
sich unter *Unveröffentlicht*, bis eine Version entsteht.

## Zugangsdaten

Echte Werte (WLAN, API-Schlüssel, OTA-Passwort) stehen nur in gitignorierten
Dateien (`secrets.yaml`, `secrets.h`). Ins Repo kommt nur eine Vorlage mit
Platzhaltern.

# Konvention für Knotenordner

Jeder Knoten hat einen eigenen Ordner unter `nodes/`. Der Ordnername ist
zugleich der Knotenname (Kleinbuchstaben, Bindestriche, z. B. `gateway-eth`).
Pflichtdateien und Versionierung: siehe [projekt.md](../projekt.md).

## ESPHome-Knoten

```
nodes/<knoten>/
  README.md              Zweck, Hardware, Inbetriebnahme, Version
  CHANGELOG.md
  POSTEINGANG.md         Aufträge von anderen Bereichen (entsteht mit dem ersten)
  <knoten>.yaml          ESPHome-Konfiguration
  secrets.yaml           gitignoriert
  secrets.yaml.example   Vorlage mit Platzhaltern
```

Gemeinsame Bausteine kommen aus `shared/esphome/` über `packages:`.

Gibt es von einem Knoten mehrere gleich gebaute Geräte (z. B. Gateways pro
Stockwerk), ist `<knoten>.yaml` der gemeinsame Baustein. Pro Gerät liegt dann
eine kurze Instanzdatei unter `instanzen/` mit Name, Schlüsseln und
gerätespezifischen Einstellungen, die den Baustein über `packages:` einbindet.

## Zugangsdaten

Echte Werte (WLAN, API-Schlüssel) stehen nur in `secrets.yaml`.
Der Name ist in `.gitignore` erfasst. Ins Repo geht nur eine Vorlage mit
Platzhaltern.

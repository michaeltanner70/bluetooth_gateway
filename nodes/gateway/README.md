# gateway

**Version:** 1.0.0 · Änderungen: [CHANGELOG.md](CHANGELOG.md)

ESPHome-Knoten. Minimalistischer Bluetooth-Gateway: empfängt BLE-Geräte in der
Umgebung und leitet sie als Bluetooth-Proxy an Home Assistant weiter.

## Stand

Läuft auf dem XIAO ESP32-C6 und ist in Home Assistant eingebunden. Ausgelegt
für mehrere Gateways gleicher Bauart, z. B. einer pro Stockwerk. Jeder Gateway
meldet sich in Home Assistant als eigener Bluetooth-Adapter. BLE-Geräte in
Reichweite erscheinen als neu entdeckte Integrationen, Home Assistant wählt pro
Gerät den Gateway mit dem besten Empfang. Dazu kommen die Diagnose-Entitäten
aus `common/diagnostics.yaml` und ein Fallback-Hotspot.

Kontrolle in Home Assistant: *Einstellungen → Geräte & Dienste → Bluetooth →
bt-gateway-<id> → Konfigurieren* zeigt die empfangenen BLE-Geräte
(Advertisement Monitor) und die belegten Verbindungsplätze.

## Gateways

| Gateway            | Antenne | Standort |
|--------------------|---------|----------|
| `bt-gateway-47b10` | extern  | offen    |
| `bt-gateway-a8fc4` | intern  | offen    |

MAC-Adressen stehen bewusst nicht im Repo. Die vollständige MAC eines Gateways
zeigt Home Assistant beim Gerät (Entität „3.1 Geräte MAC“).

## Aufbau der Konfiguration

| Datei im Repo                   | Im Add-on                  | Inhalt                                   |
|---------------------------------|----------------------------|------------------------------------------|
| `gateway.yaml`                  | `common/bt-gateway.yaml`   | gemeinsamer Baustein für alle Gateways   |
| `instanzen/bt-gateway-<id>.yaml`| `bt-gateway-<id>.yaml`     | pro Gateway: Name, Antenne, Schlüssel    |

- **Name:** `bt-gateway-` plus die letzten 5 Stellen der WLAN-MAC, in
  Kleinbuchstaben. Beispiel mit erfundener MAC: AA:BB:CC:12:34:56 ergibt
  `bt-gateway-23456`.
- **Antenne:** Substitution `antenne` in der Instanzdatei, `extern` (U.FL) oder
  `intern` (Keramik). Ein anderer Wert lässt die Prüfung mit einer Meldung
  abbrechen.
- **Reihenfolge der Packages:** zuerst `diagnostics`, dann `gateway`. Spätere
  Packages überschreiben frühere, so gelten die Heap-Grenzwerte des Bausteins.
- Eine Änderung an `gateway.yaml` gilt für alle Gateways. Danach jeden Gateway
  neu flashen.

## Hardware

| Teil           | Wahl                                              |
|----------------|---------------------------------------------------|
| Board          | Seeed Studio XIAO ESP32-C6 (`seeed_xiao_esp32c6`) |
| Speisung       | USB-C, z. B. ein beliebiges USB-Netzteil          |
| Antenne        | pro Gateway: extern (U.FL) oder eingebaut         |
| Weitere Teile  | keine, ausser allenfalls der externen Antenne     |

Es gibt keine Verdrahtung: Board ans USB-Netzteil, fertig.

Das Board hat eine eingebaute Keramikantenne und einen U.FL-Anschluss für eine
externe Antenne. Gewählt wird pro Gateway mit der Substitution `antenne` in der
Instanzdatei (Vorgabe: `extern`). Die Pins setzt `gateway.yaml` beim Start:

| Pin    | Pegel                          | Bedeutung                   |
|--------|--------------------------------|-----------------------------|
| GPIO3  | LOW                            | HF-Schalter eingeschaltet   |
| GPIO14 | HIGH bei `extern`, LOW bei `intern` | Antennenwahl           |

### Status-LED

Die gelbe Benutzer-LED des Boards (GPIO15, leuchtet bei LOW) zeigt den Zustand:

| Muster                         | Bedeutung                                              |
|--------------------------------|--------------------------------------------------------|
| 0,5 s an, 0,5 s aus            | sucht WLAN oder wartet auf Home Assistant (API)        |
| jede Sekunde ein Blitz (100 ms)| WLAN verbunden und Home Assistant verbunden: Betrieb   |
| aus oder dauernd an            | keine Speisung, Firmware startet oder hängt            |

Empfangene BLE-Pakete zeigt die LED bewusst nicht an. Der Gateway hört auch
fremde Geräte, die LED würde deshalb fast dauernd flackern.

Gemessen am selben Standort (WLAN-Pegel, gleicher Access Point):

| Einstellung                     | Pegel           |
|---------------------------------|-----------------|
| Pins nicht gesetzt (bis 0.2.0)  | -73 bis -79 dBm |
| eingebaute Antenne, fest        | -54 bis -56 dBm |
| externe Antenne, fest           | -52 bis -55 dBm |

## Neuen Gateway hinzufügen

Genau so wurde `bt-gateway-a8fc4` als zweiter Gateway angelegt. Die Anleitung
ist so geschrieben, dass sie ohne Vorwissen Schritt für Schritt nachgebaut
werden kann, auch von einer neuen Claude-Sitzung.

### Umgebung

| Was                       | Wo                                                         |
|---------------------------|------------------------------------------------------------|
| dieses Repo               | `C:\_Daten\Synology Drive\twm\_git\bluetooth_gateway`       |
| Add-on-Verzeichnis (live) | `\\172.16.100.228\config\esphome` (Samba-Freigabe von HA)  |
| Spiegel zum Prüfen        | `C:\_Daten\Synology Drive\twm\_git\home_assistant\ESPHome`  |
| Instanzdateien            | `nodes\gateway\instanzen\bt-gateway-<id>.yaml`             |
| Baustein                  | `nodes\gateway\gateway.yaml`                               |

- ESPHome lokal: **2026.10.1 oder neuer** (`esphome version`). Das ESPHome-Add-on
  in Home Assistant muss dieselbe Version haben. Aktualisieren lokal:
  `python -m pip install --no-cache-dir --upgrade esphome==<version>`
  (`--no-cache-dir`, weil der pip-Cache sonst mit „Permission denied“ abbricht).
- **Kompilieren und flashen nur in PowerShell**, nicht in Git Bash. In Git Bash
  (MSYS) bricht die Installation von ESP-IDF mit „MSys/Mingw is not supported“ ab.
- Die `secrets.yaml` im Add-on und im Spiegel sind **identisch** und müssen es
  bleiben. Jede Änderung in beiden Dateien gleich machen. Secret-Werte nie
  ausgeben, nie ins Repo committen.

### Schritt 1: Board anschliessen und MAC auslesen

1. Neuen XIAO ESP32-C6 per USB-C (Datenkabel!) an den PC stecken. Erkennt Windows
   ihn nicht: BOOT gedrückt halten, einstecken, BOOT loslassen.
2. Port finden. Espressif-Boards haben die USB-Kennung `303A:1001`, die
   USB-Seriennummer ist die MAC:

   ```powershell
   python -c "import serial.tools.list_ports as l; [print(p.device, p.hwid) for p in l.comports()]"
   ```
3. MAC am Chip bestätigen (Beispiel COM6):

   ```powershell
   python -m esptool --port COM6 read-mac
   ```

   Massgebend ist die Zeile `BASE MAC`. Das ist die WLAN-MAC.
4. Die **`<id>`** bilden: die letzten 5 Zeichen der MAC ohne Doppelpunkte, in
   Kleinbuchstaben. Beispiel mit erfundener MAC: `AA:BB:CC:12:34:56` →
   `AABBCC123456` → **`23456`**, Gerätename `bt-gateway-23456`.
5. **Die MAC nirgends ins Repo schreiben**, weder in Dateien noch in Kommentare
   oder Commit-Meldungen. Nur die `<id>` kommt ins Repo.

### Schritt 2: Instanzdatei anlegen

1. `nodes\gateway\instanzen\bt-gateway-47b10.yaml` kopieren nach
   `nodes\gateway\instanzen\bt-gateway-<id>.yaml`.
2. In der neuen Datei ersetzen:

   | Stelle                         | neuer Wert                                   |
   |--------------------------------|----------------------------------------------|
   | Kommentarzeile 1               | `# Bluetooth-Gateway <id>`                   |
   | Kommentar „Liegt im Add-on …“  | `bt-gateway-<id>.yaml`                       |
   | `name:`                        | `bt-gateway-<id>`                            |
   | `friendly_name:`               | `Bluetooth Gateway <id>`                     |
   | `antenne:`                     | `extern` (U.FL) oder `intern` (Keramik)      |
   | `key: !secret …`               | `bt_gateway_<id>_api_key`                    |
   | `ssid: !secret …`              | `bt_gateway_<id>_fallback_ap_ssid`           |
   | `password: !secret …`          | `bt_gateway_<id>_fallback_ap_password`       |

   Die Reihenfolge unter `packages:` nicht ändern (zuerst `diagnostics`, dann
   `gateway`).

### Schritt 3: Secrets ergänzen

In **beiden** `secrets.yaml` (Add-on und Spiegel) unten denselben Block
anfügen. Vorher prüfen, dass die beiden Dateien identisch sind und der Block
noch nicht existiert.

```yaml

# --- Geraet: bt-gateway-<id> (Repo bluetooth_gateway) -------------------------
# API-Schluessel, gilt auch fuer die OTA-Verschluesselung
bt_gateway_<id>_api_key: "<32 Zufallsbytes, Base64>"
bt_gateway_<id>_fallback_ap_ssid: "bt-gateway-<id>"
bt_gateway_<id>_fallback_ap_password: "<12 Zeichen, Buchstaben und Ziffern>"
```

Werte erzeugen, ohne sie anzuzeigen, z. B. in ein Skript schreiben, das direkt
anhängt:

```powershell
python -c "import os,base64;print(base64.b64encode(os.urandom(32)).decode())"
python -c "import secrets,string;a=string.ascii_letters+string.digits;print(''.join(secrets.choice(a) for _ in range(12)))"
```

Danach prüfen: beide Dateien identisch, und die drei neuen Schlüsselnamen sind da.

### Schritt 4: Prüfen im Spiegel

Der Spiegel hat `common/diagnostics.yaml` und die `secrets.yaml` am richtigen
Ort. Baustein und Instanzdatei nur für die Prüfung dorthin kopieren und danach
wieder entfernen:

```powershell
$N='C:\_Daten\Synology Drive\twm\_git\bluetooth_gateway\nodes\gateway'
$L='C:\_Daten\Synology Drive\twm\_git\home_assistant\ESPHome'
Copy-Item "$N\gateway.yaml" "$L\common\bt-gateway.yaml"
Copy-Item "$N\instanzen\bt-gateway-<id>.yaml" "$L\bt-gateway-<id>.yaml"
Set-Location $L
esphome config bt-gateway-<id>.yaml
```

Erwartet: `Configuration is valid!`. In der Ausgabe kontrollieren:
`diag_min_heap_ok: '80000'` und beim Schalter auf GPIO14 `restore_mode:
ALWAYS_ON` (extern) bzw. `ALWAYS_OFF` (intern). Steht bei `name` ein
`!secret …`, ist das nur eine Anzeige: ESPHome zeigt Werte so, die zufällig einem
Secret gleichen (hier die Hotspot-SSID).

### Schritt 5: Ins Add-on kopieren

```powershell
$H='\\172.16.100.228\config\esphome'
Copy-Item "$N\instanzen\bt-gateway-<id>.yaml" "$H\bt-gateway-<id>.yaml"
Copy-Item "$N\gateway.yaml" "$H\common\bt-gateway.yaml"
```

### Schritt 6: Erstes Flashen per USB

Aus dem Spiegel heraus (dort liegen die Kopien aus Schritt 4 noch), mit dem Port
aus Schritt 1:

```powershell
Set-Location $L
esphome run bt-gateway-<id>.yaml --device COM6 --no-logs
```

Erwartet: `Hash of data verified.` und `Successfully uploaded program.` Danach
die Prüfkopien wieder entfernen:

```powershell
Remove-Item "$L\bt-gateway-<id>.yaml", "$L\common\bt-gateway.yaml"
```

Ob der Gateway ins WLAN kommt, zeigt das serielle Log (Zeile `IP Address:`) oder
die Status-LED: sie wechselt vom 0,5-s-Blinken auf einen kurzen Blitz pro Sekunde.

Spätere Updates gehen per OTA über die IP-Adresse, mit `--device <IP>` statt
des COM-Ports, oder im ESPHome-Dashboard mit „Install“.

### Schritt 7: Home Assistant

1. Home Assistant bindet den Gateway **selbst** ein, weil das Add-on den
   Schlüssel kennt. Es entstehen zwei Einträge:
   - **ESPHome:** Gerät „Bluetooth Gateway <id>“ mit 15 Diagnose-Entitäten
     (`sensor.bluetooth_gateway_<id>_…`).
   - **Bluetooth:** ein Adapter-Gerät. Das hat grundsätzlich keine Entitäten,
     das ist richtig so.
   Erscheint der Gateway nur unter „Neu entdeckt“: dort auf *Hinzufügen*.
2. Adapter-Gerät umbenennen in „Bluetooth Gateway <id> (Adapter)“ und beiden
   Geräten den Bereich zuweisen.
3. Den Titel des Bluetooth-Integrationseintrags kann nur die Oberfläche ändern:
   *Einstellungen → Geräte & Dienste → Bluetooth*, beim Eintrag ⋮ →
   *Umbenennen* → `bt-gateway-<id> (<Bluetooth-MAC>)`.
4. Kontrolle: Diagnose-Entitäten zeigen Werte, „4.0 System Gesundheit“ ist
   🟢 Stabil, und unter Bluetooth → Adapter → *Konfigurieren* hört der neue
   Gateway BLE-Geräte.

### Schritt 8: Doku und Repo

1. Tabelle **Gateways** oben um die neue Zeile ergänzen (Name, WLAN-MAC,
   Antenne, Standort).
2. Im `CHANGELOG.md` unter *Unveröffentlicht* eintragen.
3. Committen und pushen. Ins Repo kommen nur die Instanzdatei und die Doku,
   keine Secrets.

### Stolpersteine

- **Gerät umbenennen:** Das ESPHome-Dashboard merkt sich in
  `config/esphome/.device-builder.json` die MAC pro YAML-Datei. Bleibt nach einer
  Umbenennung der alte Eintrag stehen, zeigt das Dashboard das Gerät beim alten,
  leeren Eintrag. Abhilfe: Add-on stoppen, den alten Schlüssel aus der Datei
  entfernen, Add-on starten.
- **Packages-Reihenfolge:** Steht `gateway` vor `diagnostics`, überschreibt das
  Diagnose-Package die Heap-Grenzwerte (Ampel „4.0“ zeigt dann falsch).
- **Ungültige Antenne:** Ein Tippfehler bei `antenne` lässt `esphome config`
  mit `antenne_muss_extern_oder_intern_sein` abbrechen.

## Betrieb im ESPHome-Add-on

Im Livesystem läuft die Konfiguration im ESPHome-Add-on von Home Assistant.
Massgebend sind die Dateien in diesem Repo, nach jeder Änderung werden sie ins
Add-on-Verzeichnis (`config/esphome/`) kopiert (Zuordnung siehe *Aufbau der
Konfiguration*).

- Schlüssel in der `secrets.yaml` des Add-ons, pro Gateway:
  `bt_gateway_<id>_api_key`, `bt_gateway_<id>_fallback_ap_ssid` und
  `bt_gateway_<id>_fallback_ap_password`. Dazu `wifi_ssid`, `wifi_password` und
  `mac_bssid_*`, die alle Geräte gemeinsam nutzen. Der API-Schlüssel sichert
  auch OTA, ein OTA-Passwort gibt es nicht.
- Fallback-Hotspot: Findet ein Gateway das WLAN nicht, öffnet er den Hotspot
  `bt-gateway-<id>`. Darüber lassen sich im Captive Portal neue WLAN-Daten
  eingeben.
- Diagnose: eingebunden wird `common/diagnostics.yaml` (Version 1.4.1, 5 Access
  Points) aus dem Add-on-Verzeichnis, wie bei allen anderen Geräten (Original
  im Repo `esphome_apps`). Die Datei liegt nicht in diesem Repo.
- Lokal prüfen im Spiegel `home_assistant/ESPHome`, wo `common/` und
  `secrets.yaml` nebeneinanderliegen: Baustein als `common/bt-gateway.yaml` und
  Instanzdatei dorthin kopieren, dann `esphome config bt-gateway-<id>.yaml`.

# wfx_cdr – Workflow Guide / Workflow-Anleitung

**English version below.** → [Deutsche Version weiter unten](#deutsch)

## English

`wfx_cdr` represents CD/DVD burn projects as a virtual file system inside
Total Commander. Each project is a virtual folder at the plugin root; files
inside it are only **references** to real files on disk (nothing gets
copied). Commands like *** Burn appear as virtual folders/files **inside**
the respective project.

Most steps below are **order-independent** — noted explicitly per section.
Only "create project" always has to come first, and "burn" last, for
obvious reasons.

### Prerequisites (one-time, not per project)

`*** Settings (global)` at the plugin root:

- **Device** — determine via the `Scan` button (finds SATA/AHCI burners
  directly through Windows; additionally scans for USB burners via
  `cdrecord -scanbus`, if present).
- **Tools** — folder containing `cdrecord.exe`/`mkisofs.exe`/`vcdimager.exe`
  etc. (default: next to the plugin DLL).
- **Administrator rights (UAC)** — needed on most systems for cdrecord to
  access the burner at all. With *"Keep elevated for the whole session"*,
  one UAC prompt per TC session is enough instead of one per burn step —
  **exception:** a Data project running with *On-the-fly* enabled still
  needs its own UAC prompt every single time (technical limitation, see
  below).
- Speed, FIFO, BurnFree, language, theme (light/dark) — optional, apply to
  all projects.

These settings also serve as the **default template** for newly created
projects (Volume ID and boot image excluded — those are always
project-specific).

### Creating a Data CD/DVD

#### 1. Create a project (always first)

Two equivalent ways:

- Enter **`*** New project`** at the root → type a name → choose a type.
- **F7** at the plugin root → the same name/type dialog.

#### 2. Choose the project type (set at creation, not changeable afterward)

| Type | When to use |
|---|---|
| **Data CD/DVD** | The normal case: any files/folders on a data CD/DVD. |
| **DVD-Video** | The folder structure already contains a finished `VIDEO_TS` folder. |
| **ISO image (burn later)** | An existing `.iso` file should be burned 1:1 (no mkisofs run). |
| **Data (linked folder)** | One or more real folders should be freshly (recursively) re-read on **every** burn — useful for folders that change between burn attempts. |

#### 3. Add content (order-independent with step 4)

- **Data CD/DVD, DVD-Video:** copy files/folders directly into the project
  folder, as often as you like, in any order, and add/remove more later.
- **ISO image:** copy an `.iso` file in (replaces any existing one).
  Alternatively, copy an `.iso` straight into the plugin **root** — this
  automatically creates a new ISO project with that name.
- **Linked folder:** copy an entire real folder in — it isn't actually
  copied, just remembered as a reference (shows up as one entry named after
  the folder). Add further folders the same way; individual files can
  additionally be placed directly at the project root. Removing a linked
  folder (<kbd>Del</kbd>) only removes the reference, never the real files.

#### 4. Settings (optional, order-independent with step 3)

`*** Settings` inside the project (not for ISO projects, which burn 1:1):

- **File system:** Joliet (+ long names), Rock Ridge (+ Rational Rock), UDF,
  ISO level.
- **Disc info:** Volume ID (auto-derived from the project name when
  created, ISO9660-compliant, editable), publisher, preparer.
- **Bootable CD (El Torito):** boot image (a file/folder inside the
  project), emulation mode.
- **Burn options:** Multisession (leave disc open), On-the-fly (image and
  burn in one pass with no intermediate file — needs its own UAC prompt
  every time when elevation is on, see above), *Image only* (no actual
  burn, just an `.iso` file), *Keep image*, *Verify after burn*, target
  medium (CD/DVD variants, drives the capacity check).

Most changes made here (file system, medium, burn options, …) are carried
over as the new default template for **future newly created** projects —
**except** Volume ID and boot image, which always stay purely
project-specific and never affect the global template.

#### 5. Burn

- **`*** Burn`** — supports multisession if enabled above (disc stays open).
- **`*** Burn & close`** — always closes the disc.

Sequence: pre-check for missing source files (Ignore / Update project /
Abort) → check for a writable medium in the drive → confirmation dialog
(shows device, speed, simulation/elevation note if applicable) → progress
window (with a taskbar entry, cancellable).

#### 6. Follow-up (optional, any time afterward)

- **`*** Tools` → `*** Finalize`** — closes a disc that was left open for
  multisession, without writing any new data.
- **`*** Delete project`** — removes only the virtual project folder (the
  referenced source files are left untouched).
- The project folder can also be **copied** normally via copy/paste (a real
  copy of the project file, not a reference) or **exported** out of the
  plugin into a real folder (only the project file itself ends up there,
  none of the referenced source files).

### Creating an Audio CD

#### 1.+2. Create a project, choose type **Audio CD**

Same as above (`*** New project` or F7).

#### 3. Add tracks (order-independent with steps 4 and 5)

- Copy MP3/Ogg/FLAC/WAV files directly into the project folder (flat list
  only, no subfolders).
- **Track order = alphabetical sort of the file names** — to reorder,
  rename the files inside the project (e.g. number prefixes `01_`, `02_`, …).
- Missing/deleted source files are only checked at burn time (see below),
  not immediately when adding.

#### 4. Edit CD-Text (optional)

**`*** Edit CD-Text`** — album title/performer plus per-track title/performer.
Only takes effect if *CD-Text* is enabled at burn time (see settings).

#### 5. Settings (optional, order-independent with steps 3/4)

`*** Settings` inside the project:

- Write mode DAO/TAO.
- Pre-emphasis, copy-protection flags (SCMS/copy permitted).
- Pause before each track (default 150 sectors = 2 s).
- ReplayGain (level adjustment before burning, source files stay
  unchanged — the adjustment runs on temporary copies).
- Target medium.

#### 6. Burn

Same as Data projects: `*** Burn` / `*** Burn & close`, same sequence
(missing files → medium check → confirmation → progress window). Non-WAV
sources are automatically decoded to WAV first (needs mpg123/oggdec/flac in
the tools folder).

#### 7. Follow-up

Same as Data projects (Finalize, delete project, copy/export).

### Shared tools (`*** Tools`, usable any time, independent of any project)

| Command | Effect |
|---|---|
| `*** Erase CD/DVD (CD-RW)` | `blank=fast` on the disc in the drive. |
| `*** Finalize` | Close the last open session without writing new data. |
| `*** Eject disc` | Open the tray. |
| `*** Load disc` | Close the tray. |
| `*** Disc info` | Show raw ATIP/TOC data for the inserted disc. |

All five are themselves virtual folders (so they always sort above real
entries in TC) and trigger their action immediately on entry.

### More notes

- **Simulated burn (dummy):** runs exactly like a real burn (including the
  SCSI handshake with the medium), just without the laser — so it still
  needs a writable medium in the drive, otherwise you now get a clear error
  message instead of a cryptic SCSI failure.
- **Auto-erase:** can be enabled in the global settings to automatically
  blank a non-empty CD-RW before every burn.
- **VideoCD/SVCD:** their own, separate project types (not part of the
  Data/Audio choice) — flat MPEG-1/MPEG-2 track list, no settings dialog of
  their own (always CD medium, single session). MP3/MP4 files aren't
  filtered by the plugin, but get rejected by `vcdimager` at burn time — the
  source files need to already be real MPEG-1/2.
- **Multilingual:** German/English/Russian/Ukrainian, switchable in the
  global settings.

---

## Deutsch

`wfx_cdr` bildet CD/DVD-Brennprojekte als virtuelles Dateisystem in Total
Commander ab. Jedes Projekt ist ein virtueller Ordner an der Plugin-Wurzel;
Dateien darin sind nur **Referenzen** auf echte Dateien auf der Platte (es
wird nichts kopiert). Befehle wie *** Brennen erscheinen als virtuelle
Ordner/Dateien **innerhalb** des jeweiligen Projekts.

Die meisten Schritte unten sind **reihenfolgeunabhängig** — das wird pro
Abschnitt explizit vermerkt. Nur "Projekt anlegen" muss immer zuerst
passieren, und "Brennen" logischerweise zuletzt.

### Voraussetzungen (einmalig, nicht pro Projekt)

`*** Einstellungen (global)` an der Plugin-Wurzel:

- **Gerät** — per `Scannen`-Knopf ermitteln (findet SATA/AHCI-Brenner direkt
  über Windows; zusätzlich USB-Brenner über `cdrecord -scanbus`, sofern
  vorhanden).
- **Tools** — Ordner mit `cdrecord.exe`/`mkisofs.exe`/`vcdimager.exe` usw.
  (Standard: neben der Plugin-DLL).
- **Adminrechte (UAC)** — auf den meisten Systemen nötig, damit cdrecord den
  Brenner überhaupt ansprechen darf. Mit *"Für die ganze Sitzung erhöht
  bleiben"* reicht ein UAC-Dialog pro TC-Sitzung statt einem pro
  Brennschritt — **Ausnahme:** läuft ein Daten-Projekt mit aktiviertem
  *On-the-fly*, braucht dieser eine Schritt trotzdem jedes Mal einen
  eigenen UAC-Dialog (technische Einschränkung, siehe unten).
- Speed, FIFO, BurnFree, Sprache, Design (hell/dunkel) — optional, betreffen
  alle Projekte gemeinsam.

Diese Einstellungen dienen zugleich als **Vorbelegung** für neu angelegte
Projekte (VolId und Boot-Image ausgenommen — die sind immer projektspezifisch).

### Daten-CD/DVD erstellen

#### 1. Projekt anlegen (immer zuerst)

Zwei gleichwertige Wege:

- **`*** Neues Projekt`** an der Wurzel betreten → Name eingeben → Typ wählen.
- **F7** in der Plugin-Wurzel → derselbe Name/Typ-Dialog.

#### 2. Projekttyp wählen (bei Anlage, danach nicht mehr änderbar)

| Typ | Wann verwenden |
|---|---|
| **Daten-CD/DVD** | Normalfall: beliebige Dateien/Ordner auf eine Daten-CD/DVD. |
| **DVD-Video** | Ordnerstruktur enthält bereits einen fertigen `VIDEO_TS`-Ordner. |
| **ISO-Image (später brennen)** | Eine fertige `.iso`-Datei soll 1:1 gebrannt werden (kein mkisofs-Lauf). |
| **Daten (verlinkter Ordner)** | Ein oder mehrere reale Ordner sollen bei **jedem** Brennvorgang frisch (rekursiv) neu eingelesen werden — praktisch für Ordner, die sich zwischen zwei Brennversuchen ändern. |

#### 3. Inhalte hinzufügen (reihenfolgeunabhängig zu Schritt 4)

- **Daten-CD/DVD, DVD-Video:** Dateien/Ordner direkt in den Projektordner
  kopieren, beliebig oft, beliebige Reihenfolge, auch nachträglich noch
  ergänzen/löschen.
- **ISO-Image:** eine `.iso`-Datei hineinkopieren (ersetzt eine evtl. schon
  vorhandene). Alternativ: eine `.iso` direkt in die Plugin-**Wurzel**
  kopieren — legt automatisch ein neues ISO-Projekt mit diesem Namen an.
- **Verlinkter Ordner:** einen kompletten realen Ordner hineinkopieren — er
  wird nicht wirklich kopiert, sondern nur als Verweis gemerkt (erscheint als
  ein Eintrag mit seinem Ordnernamen). Weitere Ordner genauso hinzufügen;
  einzelne Dateien lassen sich zusätzlich direkt im Projekt-Root ablegen.
  Einen verlinkten Ordner per <kbd>Entf</kbd> wieder entfernen löscht nur den
  Verweis, nie die echten Dateien.

#### 4. Einstellungen (optional, reihenfolgeunabhängig zu Schritt 3)

`*** Einstellungen` im Projekt (nicht bei ISO-Projekten, die brennen 1:1):

- **Dateisystem:** Joliet (+ lange Namen), Rock Ridge (+ Rational Rock), UDF,
  ISO-Level.
- **Datenträger-Info:** Volume-ID (wird beim Anlegen automatisch aus dem
  Projektnamen abgeleitet, ISO9660-konform, editierbar), Herausgeber, Ersteller.
- **Bootfähige CD (El Torito):** Boot-Image (Datei/Ordner im Projekt),
  Emulationsmodus.
- **Brennoptionen:** Multisession (Disc offen lassen), On-the-fly (Image und
  Brennen in einem Rutsch ohne Zwischendatei — braucht bei aktivierter
  Elevation jedes Mal einen eigenen UAC-Dialog, siehe oben), *Nur Image
  erzeugen* (kein Brennvorgang, nur eine `.iso`-Datei), *Image behalten*,
  *Nach Brennen prüfen* (Verify), Ziel-Medium (CD/DVD-Varianten, bestimmt die
  Kapazitätsprüfung).

Die meisten Änderungen hier (Dateisystem, Medium, Brennoptionen …) werden
als neue Vorbelegung für **künftig neu angelegte** Projekte übernommen —
**außer** Volume-ID und Boot-Image, die bleiben immer rein projektspezifisch
und beeinflussen nie die globale Vorbelegung.

#### 5. Brennen

- **`*** Brennen`** — Multisession-fähig, falls oben aktiviert (Disc bleibt offen).
- **`*** Brennen & abschließen`** — schließt die Disc in jedem Fall.

Ablauf: Vorab-Prüfung auf fehlende Quelldateien (Ignorieren / Projekt bereinigen /
Abbrechen) → Prüfung auf beschreibbares Medium im Laufwerk → Bestätigungsdialog
(zeigt Gerät, Speed, ggf. Simulation-/Elevation-Hinweis) → Fortschrittsfenster
(mit Taskleisten-Eintrag, abbrechbar).

#### 6. Nachbereitung (optional, jederzeit danach)

- **`*** Werkzeuge` → `*** Finalisieren`** — schließt eine offen gelassene
  Multisession-Disc nachträglich ab, ohne neue Daten zu schreiben.
- **`*** Projekt löschen`** — entfernt nur den virtuellen Projektordner
  (die referenzierten Quelldateien bleiben unangetastet).
- Projektordner lässt sich auch ganz normal per Copy/Paste **kopieren**
  (echte Kopie der Projektdatei, kein Verweis) oder aus dem Plugin heraus in
  einen echten Ordner **exportieren** (dabei landet nur die Projekt-Datei
  selbst dort, keine der referenzierten Quelldateien).

### Audio-CD erstellen

#### 1.+2. Projekt anlegen, Typ **Audio-CD** wählen

Wie oben (`*** Neues Projekt` oder F7).

#### 3. Tracks hinzufügen (reihenfolgeunabhängig zu Schritt 4 und 5)

- MP3/Ogg/FLAC/WAV-Dateien direkt in den Projektordner kopieren (nur flache
  Liste, keine Unterordner).
- **Track-Reihenfolge = alphabetische Sortierung der Dateinamen** — zum
  Umsortieren die Dateien im Projekt umbenennen (z. B. Zahlenpräfixe
  `01_`, `02_`, …).
- Fehlende/gelöschte Quelldateien werden erst beim Brennen geprüft (siehe
  unten), nicht sofort beim Hinzufügen.

#### 4. CD-Text bearbeiten (optional)

**`*** CD-Text bearbeiten`** — Albumtitel/-Interpret sowie Titel/Interpret
pro Track. Nur wirksam, wenn beim Brennen *CD-Text* aktiviert ist (siehe
Einstellungen).

#### 5. Einstellungen (optional, reihenfolgeunabhängig zu Schritt 3/4)

`*** Einstellungen` im Projekt:

- Schreibmodus DAO/TAO.
- Pre-Emphasis, Kopierschutz-Flags (SCMS/Kopieren erlaubt).
- Pause vor jedem Track (Standard 150 Sektoren = 2 s).
- ReplayGain (Pegelanpassung vor dem Brennen, Quelldateien bleiben
  unverändert — Anpassung läuft über temporäre Kopien).
- Ziel-Medium.

#### 6. Brennen

Wie bei Daten-Projekten: `*** Brennen` / `*** Brennen & abschließen`, gleicher
Ablauf (fehlende Dateien → Medium-Prüfung → Bestätigung → Fortschrittsfenster).
Nicht-WAV-Quellen werden vorher automatisch nach WAV dekodiert
(mpg123/oggdec/flac im Tools-Ordner nötig).

#### 7. Nachbereitung

Wie bei Daten-Projekten (Finalisieren, Projekt löschen, kopieren/exportieren).

### Gemeinsame Werkzeuge (`*** Werkzeuge`, jederzeit nutzbar, unabhängig von einem Projekt)

| Befehl | Wirkung |
|---|---|
| `*** CD/DVD löschen (CD-RW)` | `blank=fast` auf die eingelegte CD-RW/DVD-RW. |
| `*** Finalisieren` | Letzte offene Sitzung abschließen, ohne neue Daten zu schreiben. |
| `*** Disk auswerfen` | Schublade öffnen. |
| `*** Disk einziehen` | Schublade schließen. |
| `*** Disc-Info` | ATIP/TOC-Rohdaten der eingelegten Disc anzeigen. |

Alle fünf sind selbst virtuelle Ordner (sortieren deshalb in TC immer oberhalb
echter Einträge) und lösen ihre Aktion sofort beim Betreten aus.

### Weitere Hinweise

- **Simuliertes Brennen (Dummy):** läuft komplett wie ein echter Brennvorgang
  (inkl. SCSI-Handshake mit dem Medium), nur ohne Laser — braucht also
  trotzdem ein beschreibbares Medium im Laufwerk, sonst kommt eine klare
  Fehlermeldung statt eines kryptischen SCSI-Fehlers.
- **Auto-Erase:** kann in den globalen Einstellungen aktiviert werden, um vor
  jedem Brennvorgang automatisch eine nicht-leere CD-RW zu löschen.
- **VideoCD/SVCD:** eigene, separate Projekttypen (nicht Teil der
  Daten-/Audio-Auswahl) — flache MPEG-1/MPEG-2-Track-Liste, kein eigener
  Einstellungsdialog (immer CD-Medium, Single-Session). MP3/MP4-Dateien
  werden vom Plugin nicht gefiltert, aber von `vcdimager` beim Brennen
  abgelehnt — die Quelldateien müssen bereits echtes MPEG-1/2 sein.
- **Mehrsprachig:** Deutsch/Englisch/Russisch/Ukrainisch, umschaltbar in den
  globalen Einstellungen.

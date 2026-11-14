# CTA 2026 – Rückblick auf den CTA-Workshop

Vortrag über den 5. CERN Tape Archive (CTA) Workshop, 9.–10. November 2026 bei DESY in Hamburg.
Gebaut mit [Slidev](https://sli.dev).

## Inhalt

1. **CTA heute:** was das CERN Tape Archive ist und wo es nach Run-3 steht
2. **Workshop & Community:** Programm, Eindrücke und Site-Reports anderer Institute
3. **Neu in CTA v6:** gRPC-Frontend, PostgreSQL-Scheduler, neuer Tape Daemon, einheitliche Konfiguration
4. **Roadmap:** Data Challenge 2027, Long Shutdown 3 und der Weg zum HL-LHC
5. **IBM Diamondback:** die Rack-Library im Langzeittest am CERN
6. **Ausblick:** wie es bei uns weitergeht

Die Sprechernotizen zu jeder Folie (mit Quellenangaben) stehen in `Mein-Vortrag/slides.md`.

## Ordner

| Pfad | Inhalt |
|---|---|
| `Mein-Vortrag/slides.md` | Folien und Sprechernotizen |
| `Mein-Vortrag/components/` | eine Vue-Komponente pro Folie |
| `Mein-Vortrag/public/` | Bilder |
| `Mein-Vortrag/CTA-2026-Vortrag.pdf` | fertiger Export, eine Seite pro Klick |

Die Original-Workshop-Folien sind nicht im Repo. Sie liegen auf Indico: <https://indico.desy.de/event/52689>

## Auf dem eigenen Rechner ausführen

Voraussetzung: [Node.js](https://nodejs.org) 20 oder neuer (getestet mit Node 24).

```bash
git clone https://github.com/arabila/CERN-Tape-Vortrag.git
cd CERN-Tape-Vortrag/Mein-Vortrag
npm install
npm run dev
```

Dann <http://localhost:3030> im Browser öffnen.

- Pfeiltasten oder Leertaste: weiter / zurück
- `F`: Vollbild
- Präsentationsansicht mit Notizen: <http://localhost:3030/presenter>

## Als PDF exportieren

```bash
cd Mein-Vortrag
npx playwright install chromium   # nur beim ersten Mal
npx slidev export --with-clicks --output CTA-2026-Vortrag.pdf --wait 9000
```

`--with-clicks` macht eine Seite pro Klick. `--wait` gibt den Animationen Zeit, fertig zu laufen, bevor die Seite gespeichert wird.

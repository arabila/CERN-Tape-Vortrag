---
theme: default
title: CTA 2026
colorSchema: light
aspectRatio: 16/9
canvasWidth: 1280
transition: fade
drawings:
  persist: false
fonts:
  sans: Inter
  mono: JetBrains Mono
  weights: '300,400,500,600,700,800'
layout: none
---

<CoverSlide />

<!--
Rückblick auf den 5. CERN Tape Archive Workshop, 9.–10. November 2026 bei DESY in Hamburg.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Titelfolie. Der Vortrag ist ein Rückblick auf den 5. CTA-Workshop.

*Quellen:*
- „5. CTA Workshop“, DESY, 9.–10. Nov. 2026: Welcome (M. Davis, Datei 01), Folie 1
- „Flash Seminar Room“: Raumangabe auf der Indico-Eventseite, steht nicht in den Folien

*Quellenverzeichnis:* „Datei NN“ ist die Nummer am Anfang des Dateinamens im Ordner Indico-Folien. „Folie N“ ist die Seite im PDF bzw. die Folie in der PowerPoint.
- 01 Welcome (M. Davis) · 28 CTA Roadmap (M. Davis), Ordner Vorträge/1_Einführung_und_Roadmap
- 08 v6-Runtime, 10 Tape Daemon (N. Bugel) · 11 PostgreSQL-Scheduler (J. Guenther), Ordner Vorträge/2_CTA-Kern_und_Architektur
- 09 gRPC-Frontend (P. Ferreira) · 21 gRPC in Produktion (J. Leduc), Ordner Vorträge/3_gRPC-Frontend
- 14 Python-SDK, 15 cta-CLI (I. Larbi), Ordner Vorträge/4_Tools_CLI_und_SDK
- 12 mhVTL (J. Chodak) · 22 OpenTelemetry (N. Bugel), Ordner Vorträge/5_Entwicklung_Test_und_Monitoring
- 23 Archive Metadata (J. Leduc), Ordner Vorträge/6_Metadaten_Workflows_und_S3
- 16 IBM Diamondback (V. Bahyl, PowerPoint), Ordner Vorträge/7_Tape-Hardware_und_Bibliotheken
- Site Reports, Ordner Site-Reports: 03 SURF (M. Saum) · 04 DESY (M. Karimi) · 05 RAL Antares (G. Patargias) · 06 Fermilab (T. Messer, PDF) · 17 RAL Feuchtigkeit (T. Byrne) · 18 PIC (A. Guino Feijoo) · 19 European XFEL (J. Malka) · 20 TRIUMF (S. Liu)
- ⚠️ markiert alles, was nicht aus den Workshop-Folien stammt (eigene Rechnung, Allgemeinwissen, Gespräch).
-->

---
layout: none
---

<AgendaSlide />

<!--
Die fünf Teile des Vortrags. Kurz durchgehen, dann ins erste Kapitel.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Die fünf Kapitel mit je zwei, drei Stichworten.

*Quellen der Stichworte:*
- „1,5 EB auf Band“: Welcome (Datei 01), Folie 5 (1480 PB im Jahr 2026)
- Site Reports, Trends: Kapitel 2, Quellen dort
- gRPC, PostgreSQL, TOML: Kapitel 3, Quellen dort
- DC27, Run-4: CTA Roadmap (M. Davis, Datei 28), Folien 3–5
- Diamondback, Mount-Tests: Diamondback-Vortrag (V. Bahyl, Datei 16)
-->


---
layout: none
---

<SectionSlide :num="1" title="CTA heute" subtitle="Was das CERN Tape Archive ist und wo es nach Run-3 steht" />

<!--
Kapitel 1: Was CTA ist und wo es heute steht.

**📚 HINTERGRUND & QUELLEN**

Kapitelfolie, keine eigenen Fakten.
-->


---
layout: none
---

<StatsSlide />

<!--
Zahlen aus dem Welcome-Talk (Michael Davis): Daten auf Band am CERN 2015: 100 PB, 2018: 340 PB, 2022: 420 PB, 2023: 688 PB, 2024: 946 PB, 2025: 1250 PB, 2026: 1480 PB.
In Run-3 kam mehr als 1 EB dazu, allein 2025 +304 PB. Die ursprüngliche Design-Annahme war 150 PB pro Jahr.
Rekordmonat: 62 PB im Oktober 2025. Zum Vergleich: 4,6 PB im November 2012, 24 PB im November 2022.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Links drei Kennzahlen, rechts die Datenmenge auf Band am CERN pro Jahr. Botschaft: In Run-3 kam mehr als ein Exabyte dazu, weit mehr als ursprünglich geplant.

*Quellen:*
- Balken 2015–2026 (100, 340, 420, 688, 946, 1250, 1480 PB) und Run-Grenzen: Welcome (M. Davis, Datei 01), Folie 5
- „3,5× so viel wie zu Beginn von Run-3“, „+1 EB in Run-3“, „+304 PB in 2025“, „Design-Annahme 150 PB pro Jahr“: Welcome, Folie 5
- „62 PB im Oktober 2025“ (neuer CERN-Rekord) und „4,6 PB im Nov. 2012“: Welcome, Folie 6
- „Davor sitzt EOS oder dCache“: Welcome, Folie 8 (Karte: RAL mit EOS, DESY/PIC/FNAL mit dCache)
-->

---
layout: none
clicks: 7
---

<ArchitectureSlide :step="$clicks" />

<!--
Überblick: Eine Datei kommt im Disk Buffer an, das Frontend plant Job und Metadaten, der Tape Daemon schreibt sie aufs Band.

[click] Disk Buffer (EOS / dCache): Empfängt Dateien von Nutzern und vom Band zurückgelesene Daten. Puffert sie vor dem Schreiben bzw. nach dem Lesen. Bei EOS zwei Rollen: Das MGM (Namespace-Server) schickt die Workflow-Events (Archivieren, Abrufen, Löschen) ans Frontend und bekommt die Erfolgsmeldungen zurück. Die FSTs (File Storage Server) halten die eigentlichen Daten, von dort liest bzw. dorthin schreibt der Tape Daemon direkt. Bei dCache übernehmen die Pools die Rolle der FSTs.

[click] Detail zum Disk Buffer, mit Zahlen vom CERN (die Folie wächst aus dem Disk-Buffer-Kasten heraus). Am CERN heißt er EOSCTA: eine eigene EOS-Instanz nur aus SSDs, direkt vor dem Band. Links die drei Kennzahlen: 64 Server mit je 16 SSDs à 2 TB, also rund 2 PB. Davon hat jedes LHC-Experiment seine eigene Buffer-Instanz: ca. 10 Server, also rund 320 TB, mit einer zugesagten Archivrate von 10 GB/s. Ausgelegt ist der Buffer auf bis zu 8 Stunden bei dieser Rate. Rechts das Szenario: Das Band steht, das Experiment schreibt weiter mit 10 GB/s, nach 8 Stunden ist der Buffer voll. Danach geht nichts verloren, die Daten bleiben auf dem EOS des Experiments liegen, bis das Band wieder läuft. Unten drei Eigenschaften: nur eine Kopie pro Datei, sofortiges Löschen nach dem Schreiben aufs Band, zählt nicht zum Pledge. Kernaussage: Der Buffer ist kein Zwischenlager für Tage. Er ist klein und schnell, soll die Laufwerke gleichmäßig auslasten und kurze Ausfälle überbrücken.

[click] CTA Frontend: Die externe Schnittstelle. In v5 gibt es zwei Varianten: das klassische XRootD-SSI-Frontend und das neue gRPC-Frontend. Das CERN hat seine Produktion im Oktober 2026 komplett auf gRPC umgestellt, mit v6 fällt XRootD-SSI weg. Nimmt Workflow-Events vom Disk Buffer und cta-admin-Befehle an, prüft die Berechtigung (beim gRPC-Frontend per JWT, mTLS oder Kerberos), reiht Aufträge im Scheduler ein und liest/aktualisiert Metadaten im Catalogue.

[click] Scheduler: Just-in-Time-Warteschlangen. Das Frontend schiebt Jobs hinein (Push), ein Tape Daemon holt sie erst ab, wenn er ein passendes Band gemountet hat (Pull).

[click] Catalogue: Zentrale Datenbank. Speichert, welche Datei auf welchem Band liegt, dazu Bänder, Bibliotheken und Konfiguration. Liefert Speicherorte und Status an Frontend und Tape Daemons.

[click] Tape Daemon: Ein Prozess pro Laufwerk. Entscheidet, wann ein Band gemountet wird, holt Jobs aus dem Scheduler, steuert die Hardware per SCSI, überträgt Daten zwischen den FSTs und dem Band, trägt das Ergebnis im Catalogue ein und legt den erledigten Job in eine Melde-Warteschlange (ToReport). Mit dem Disk Buffer tauscht er nur Daten aus, dem MGM meldet er selbst nichts.

[click] Maintenance Daemon: Regelmäßige Wartung. Räumt Queues auf (Garbage Collection), expandiert Repack-Aufträge und holt die erledigten Jobs aus der Melde-Warteschlange. Erfolg oder Fehler meldet er gebündelt an den Disk Buffer (bei EOS ans MGM). Vorteil der Trennung: Das Laufwerk muss nicht warten, bis EOS antwortet. Bis v5 lief das als Unterprozess in cta-taped, deshalb wirkte es so, als melde der Tape Daemon selbst.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Die Bausteine von CTA und den Weg einer Datei aufs Band. Jeder Klick hebt einen Baustein hervor, kleine Animationen zeigen, was darin passiert. Der zweite Klick öffnet ein Detailfenster zum Disk Buffer: links Größe, Soll-Rate und Überbrückungszeit des SSD-Buffers am CERN, rechts der Ablauf, wenn das Band steht (der Balken füllt sich in 8 Stunden, danach stauen sich die Daten beim Experiment).

*Quellen:*
- Ein Frontend für Workflow-Events (EOS/dCache) und Admin-Befehle (cta-admin), dahinter Catalogue und Scheduler-DB: gRPC-Frontend (P. Ferreira, Datei 09), Folie 4
- Zwei Frontend-Varianten, Umstellung des CERN auf gRPC (27.9. alles XRootD, 11.10. alles gRPC): gRPC in Produktion (J. Leduc, Datei 21), Folien 3, 9–11
- XRootD-SSI fällt mit v6 weg: Datei 09, Folie 5, und Roadmap (Datei 28), Folie 4
- Anmeldung per JWT, mTLS oder Kerberos: Datei 09, Folie 7
- Scheduler als Just-in-Time-Queue, jeder Tape-Server holt sich selbst Arbeit (Pull): Roadmap (Datei 28), Folie 14
- Ein Prozess pro Laufwerk, Maintenance als eigener Dienst cta-maintd: Tape Daemon (N. Bugel, Datei 10), Folie 6
- Liste der Dienste (cta-taped, cta-maintd, cta-rmcd, APIs): v6-Runtime (N. Bugel, Datei 08), Folie 4
- ⚠️ Aufteilung MGM/FST bei EOS, Pools bei dCache, Garbage Collection, Repack-Expansion und SCSI-Steuerung: allgemeines CTA-/EOS-Wissen aus der CTA-Dokumentation, nicht aus den Workshop-Folien.

*Disk-Buffer-Detail (Klick 2), gut zu wissen für Rückfragen:*
- Warum SSDs? Ein Bandlaufwerk schreibt nur effizient, wenn es ununterbrochen mit voller Geschwindigkeit Daten bekommt. Dateien von 10 GB und mehr lassen sich nicht mehr im Arbeitsspeicher puffern, also braucht es schnellen Speicher davor.
- Warum keine Redundanz? CTA gilt als reines Bandsystem: Eine Datei ist erst sicher, wenn sie auf Band liegt. Bis dahin behält das Experiment sein Original auf dem eigenen EOS. Der Transfer-Dienst FTS prüft das mit „CheckOnTape“. Geht im Buffer etwas verloren, wird der Transfer als fehlgeschlagen gemeldet und neu vom EOS gestartet. Deshalb reicht eine Kopie, und der Platz wird doppelt so gut genutzt.
- Warum reichen 8 Stunden? Weil das Original beim Experiment liegt, muss der Buffer keinen langen Ausfall überstehen. Er muss nur verhindern, dass ein kurzer Stillstand sofort bis zum Experiment durchschlägt.
- Rückweg (Recall): Vom Band gelesene Dateien landen ebenfalls im Buffer, werden auf das große EOS des Experiments kopiert und dann aus dem Buffer gelöscht.
- Netzwerk: Jeder Server hat 25 Gb/s, zusammen 1,2 Tb/s, das sind 150 GB/s in beide Richtungen.
- Die 10 GB/s sind der Normalfall. Während Datennahme oder Heavy-Ion-Run bekommt eine Instanz mehr Bandbreite zugeteilt. ALICE hat im September 2023 so bis zu 21 GB/s mit 73 Laufwerken erreicht.
- Lehre des CERN: getrennte Buffer pro Experiment, und innerhalb des Buffers getrennte Server für Schreiben und Lesen.
- Vergleich RAL: Antares hat rund 1 PB SSD-Buffer, die neue Hardware sind 7 Knoten mit je 8 × 7,68 TB und 100 GbE.
- Ausblick: „Disk buffer management“ ist ein Arbeitspunkt für LS3, weil dann große Recall-Kampagnen anstehen und der Buffer beim Lesen zum Engpass werden kann.

*Quellen zum Disk-Buffer-Detail:*
- 64 Server, 16 × 2 TB SSD, 25 Gb/s, 150 GB/s Bandbreite, „up to 8 hours of buffer to tape at 10GB/s“, nicht Teil des Pledge, Löschen nach dem Archivieren: „CERN CTA Service“ (J. Leduc), IN2P3 Tape Workshop, 10.06.2026, Folie 19. Nicht Teil der Workshop-Folien bei DESY. https://indico.in2p3.fr/event/39256/contributions/175042/attachments/102134/158776/2026-06-10_IN2P3_tapeWS.pdf
- 10 GB/s Archiv-SLA pro LHC-Instanz, 10 SSD-Server, Boost bei Datennahme: ebd., Folien 21–22
- Keine Redundanz, CheckOnTape, Neustart vom Haupt-EOS: ebd., Folie 14
- Getrennte Buffer pro Experiment, getrennte Server für Lesen/Schreiben: ebd., Folie 23
- Eine Kopie pro Datei, Hardware 2020 bestellt, ALICE 21 GB/s mit 73 Laufwerken: „CTA Run 3 Production Experience“, CHEP 2023, Seiten 2–3. https://indico.jlab.org/event/459/papers/11339/files/1324-CTA_Run3_Production_Experience.pdf
- RAL 1 PB SSD-Buffer und neue Knoten: Antares (Datei 05), Folien 2 und 7
- Disk buffer management in LS3: Roadmap (Datei 28), Folien 6 und 21
- 2 PB vs. 10 GB/s: Die 2 PB sind der ganze SSD-Pool (EOSCTA) für alle Instanzen zusammen. Jedes LHC-Experiment (ATLAS, CMS, ALICE, LHCb) bekommt daraus seine eigene Buffer-Instanz mit im Normalfall ca. 10 Servern (≈ 320 TB) und 10 GB/s zugesagter Archivrate. 4 × 10 = 40 der 64 Server, der Rest für kleinere Experimente, Reserve und Boost bei Datennahme (meine Rechnung, nicht wörtlich in der Quelle).
- ⚠️ „≈ 2 PB“ ist eigene Rechnung: 64 × 16 × 2 TB = 2.048 TB. Die Quelle nennt keine Gesamtzahl.
- ⚠️ „rund 290 TB“ ist eigene Rechnung: 10 GB/s × 8 h = 288 TB. Das passt zu den 10 SSD-Servern einer Instanz (10 × 32 TB = 320 TB). Dass die 8 Stunden pro Instanz gelten und nicht für die ganzen 2 PB, ist daraus abgeleitet und steht so nicht wörtlich in der Quelle.
- ⚠️ „Danach warten die Daten auf dem EOS des Experiments“ ist aus Folie 14 abgeleitet (Original bleibt bis zur Bestätigung auf dem Haupt-EOS). Wie lange die Experimente das durchhalten, steht dort nicht. Ungeprüfte Zahlen aus der Websuche: ATLAS bis zu zwei Wochen RAW-Daten, ALICE etwa 18,5 Stunden bei 100 GB/s.
- Das CHEP-Paper nennt „3,5 Stunden bei 2,5 GB/s“. Das gilt für einen einzelnen Server (32 TB) und widerspricht den 8 Stunden nicht.
-->

---
layout: none
---

<SectionSlide :num="2" title="Workshop & Community" subtitle="Zwei Tage bei DESY und was andere Institute berichten" />

<!--
Kapitel 2: Der Workshop selbst und was die anderen Institute berichtet haben.

**📚 HINTERGRUND & QUELLEN**

Kapitelfolie, keine eigenen Fakten.
-->


---
layout: none
clicks: 6
---

<WorkshopSlide :step="$clicks" />

<!--
5. CTA Workshop, Flash Seminar Room bei DESY. Montagvormittag Campus-Tour und Keynotes, danach Site Reports und Technik-Vorträge, Dienstagnachmittag Roadmap und Hands-on mit dem CTA-Team.
Keynotes: SURF (Abschied von DMF nach 30 Jahren) und die Evaluation der IBM Diamondback durch das CERN. Kurzbeiträge: LumOS-API bei RAL, Feuchtigkeit und Bandzuverlässigkeit bei RAL.
Livestream, aber keine Aufzeichnungen. Teilnahme kostenlos.

**Was die Farben bedeuten:**

🟡 **Plenum & Keynotes**
- Welcome (M. Davis): Begrüßung, Zahlen zu Run-3
- SURF: Abschied von DMF nach 30 Jahren, Wechsel zu ScoutAM (Versity)
- IBM Diamondback (V. Bahyl): 12-Monats-Test der Rack-Library, fürs CERN nicht passend
- Kurzbeiträge RAL: LumOS-API fürs Library-Monitoring; Feuchtigkeit und Lesefehler im Sommer
- Roadmap (M. Davis): v6, DC27, LS3, HL-LHC

🟢 **Site Reports**: Andere Institute berichten über ihren CTA-Betrieb
- DESY, RAL, Fermilab, PIC, European XFEL, TRIUMF, IHEP/HEPS

🟣 **Entwicklung**: Neues aus dem CTA-Team
- v6: gemeinsame Runtime, TOML, Health-Probes
- gRPC-Frontend: Aufteilung in Workflow- und Admin-API
- Tape Daemon: ohne fork, ein Prozess pro Laufwerk
- PostgreSQL-Scheduler: ersetzt den Objectstore, Skalierungstests
- Python-SDK und neues `cta`-Kommando: ersetzen cta-admin und cta-ops
- mhVTL: virtuelle Bandbibliothek zum Testen, jetzt ohne Kernelmodul
- Storage-Class-Wechsel: Klasse bereits archivierter Dateien ändern
- OpenTelemetry-Metriken, bessere Entwickler-Tools
- S3-Glacier-PoC: CTA als Band-Backend für Backups

🔵 **Betrieb am CERN**: Erfahrungen aus der Produktion
- gRPC-Umstellung in Produktion (Oktober 2026)
- Archive Metadata: Dateien so platzieren, dass das Zurücklesen schneller geht
- Supply Logic: welche leeren Bänder als Nächstes verwendet werden

⚪ **Tour & Hands-on**: DESY-Campus-Tour, praktische Session mit dem CTA-Team

[click] TOUR: Montagmorgen ging es los mit einer Campus-Tour. Wir wurden in Gruppen aufgeteilt.

[click] GRAFIK TOUR
Montagmorgen: Campus-Tour bei DESY. Wir wurden in Gruppen aufgeteilt, ich war in der Gruppe zu PETRA IV.

PETRA IV (großes Bild): PETRA III soll zu PETRA IV umgebaut werden, einer Röntgenquelle der nächsten Generation (Speicherring mit extrem kleiner Emittanz). Der Röntgenstrahl wird dadurch um ein Vielfaches feiner und heller. Damit lassen sich z. B. Materialien, Batterien, Katalysatoren oder Proteine bis auf die Nanoskala durchleuchten. Geplant ist der Umbau ab Ende der 2020er, Betrieb in den 2030ern. (Allgemeinwissen, Details bei DESY nachlesen.)
Bezug zu uns: Neue Lichtquellen erzeugen deutlich mehr Daten, das landet am Ende auch auf Band.

PETRA III, Halle "Max von Laue": der heutige Speicherring (ca. 2,3 km Umfang) mit vielen Messplätzen, auch EMBL ist dort mit eigenen Beamlines.

Ionen-Spektrometer an PETRA III: Messplatz, an dem u. a. die Uni Gießen untersucht, wie Ionen (geladene Atome/Moleküle) mit Röntgenlicht reagieren. Wichtig z. B. für Astrophysik und Plasmaphysik. (Banner im Bild; Einordnung aus Allgemeinwissen.)

FLASH, Halle "Albert Einstein": Freie-Elektronen-Laser für weiches Röntgenlicht mit extrem kurzen Pulsen. Auf dem Foto von oben, u. a. Beamline BL2 für ultraschnelle Pump-Probe-Experimente (Schild auf meinem Foto).

DESY-Campus: Hamburg-Bahrenfeld.

[click] DINNER: Montagabend gab es das Workshop-Dinner.

[click] GRAFIK DINNER
Kurze Zwischensequenz: abends in Hamburg.
Alter Elbtunnel (St.-Pauli-Elbtunnel): 1911 eröffnet, rund 426 m lang, führt in ca. 24 m Tiefe unter der Elbe von den Landungsbrücken nach Steinwerder. Man fährt mit großen Aufzügen nach unten, das kleine Bild zeigt den Schacht.
Rathaus bei Nacht auf dem Weg zum Essen.
Gemeinsames Abendessen: gute Gelegenheit, mit den anderen Teilnehmern ins Gespräch zu kommen.
(Hinweis an mich: Laut Foto-Datum war das am Sonntagabend, 8. Nov. Das offizielle Workshop-Dinner war Montag um 18:30 im Ristorante La Panetteria.)

[click] VORTRÄGE: Dazwischen immer wieder Vorträge, Keynotes, Site Reports und Technik. Ein Beispiel:

[click] GRAFIK VORTRAG
Eigenes Foto aus dem Flash Seminar Room: Tim Messer (Fermilab) beim Site Report "The first 18 months with CTA", Montag 9. Nov.
Zwei Punkte zum Erzählen:
- Tim Messer am Pult: Fermilab ist seit Mai 2025 von Enstore auf CTA umgestiegen. Er sagt, der Unterschied sei wie Tag und Nacht. Sie haben ~240 Laufwerke in 7 Libraries, getrennt nach CMS und "Shared" (DUNE, NOvA, Mu2e …).
- Teilnehmer per Zoom: Der Workshop lief auch als Livestream, es gab aber keine Aufzeichnung.
Überleitung: Wer sonst noch berichtet hat, zeigt die Karte.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Die zwei Workshop-Tage als Zeitleiste. Die Klicks heben Tour, Abendessen und Vorträge hervor und zeigen dazu eigene Fotos.

*Quellen:*
- 5. Workshop, „2 keynote sessions, 5 site reports, 14 technical presentations“ und „Optional hands-on/troubleshooting session“ (Kacheln oben): Welcome (Datei 01), Folie 2
- Keynotes SURF (Mo.) und Diamondback (Di.), Kurzbeiträge LumOS (Mo.) und RAL-Feuchtigkeit (Di.): Welcome, Folie 7
- Workshop-Dinner Montag 18:30, Ristorante La Panetteria: Welcome, Folie 3
- Blöcke und Uhrzeiten der Zeitleiste: Indico-Zeitplan, vereinfacht (Reihenfolge = Dateinummern in Übersicht.md)
- Inhalte der Farbgruppen (Liste oben): aus den jeweiligen Vorträgen, Dateinummern siehe Übersicht.md
- ⚠️ „Livestream, keine Aufzeichnung“ (unten rechts): steht in keiner Folie, nur auf der Indico-Eventseite bzw. aus eigener Erinnerung. Vorher prüfen.
- Fotos: eigene Aufnahmen vom 8. und 9. Nov.
- ⚠️ Erklärungen zu PETRA IV, PETRA III, Ionen-Spektrometer, FLASH und dem Alten Elbtunnel: Allgemeinwissen, nicht aus Workshop-Folien. Zahlen vor dem Vortrag auf desy.de bzw. hamburg.de gegenprüfen.
- Vortragsfoto: Redner Tim Messer laut Indico-Programm (Übersicht.md); Fermilab seit April/Mai 2025 auf CTA: Fermilab-Report (Datei 06), Folien 7–8; „Night-and-day difference from Enstore“: Folie 12; 7 Libraries mit ~110 + ~130 Laufwerken: Folie 4 (die Folie auf der Leinwand im Foto)
-->

---
layout: none
---

<SitesMapSlide />

<!--
Karte aus dem Welcome-Talk von Michael Davis (Folie 8): wer beim Workshop einen Site Report gehalten hat.
Rechts: welches Disk-System vor welchem Band-System steht.
- dCache + CTA: DESY, Fermilab, PIC und European XFEL. XFEL schreibt von GPFS über dCache in CTA bei DESY.
- EOS + CTA: RAL und IHEP (Peking), wie das CERN selbst. IHEP ist von CASTOR auf EOS-CTA umgestiegen, dort laufen u. a. LHAASO, BESIII, JUNO und HEPS. (IHEP hatte keine Folien auf Indico, Quelle: frühere IHEP-Vorträge bei HEPiX, ISGC und dem CTA-Workshop 2025.)
- dCache + Tapeguy: TRIUMF (ATLAS-Tier-1). Eigenes HSM seit 2006 mit ENDIT-Plugin. CTA wird geprüft, Tapeguy-Bänder kann CTA jetzt lesen. Noch keine Entscheidung.
- Kein CTA: SURF. Nach DMF jetzt Versity ScoutAM, Zugriff per NFS und für Grid-Nutzer über dCache.
Danach im Detail: die vier Produktions-Sites und die übrigen Berichte.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Links die Karte aus dem Welcome-Vortrag mit den berichtenden Sites, rechts welches Disk-System vor welchem Band-System steht.

*Quellen:*
- Karte (RAL mit EOS + CTA; DESY, FNAL, PIC mit dCache + CTA; Punkte bei Vancouver und Peking): Welcome (Datei 01), Folie 8, als Bild übernommen
- European XFEL: GPFS (online/offline) → dCache → Band bei DESY: XFEL-Report (J. Malka, Datei 19), Folie 11
- TRIUMF: dCache mit ENDIT-Plugin und eigenem HSM „Tapeguy“ seit 2006, erste CTA-Evaluation Ende 2023: TRIUMF-Report (S. Liu, Datei 20), Folie 3; CTA liest jetzt GNU-tar-Bänder, „No decision has been made to use CTA at the site yet“: Folie 5
- SURF: DMF 1994–2025, jetzt ScoutAM: SURF-Keynote (M. Saum, Datei 03), Folie 3; CTA war unter den geprüften Optionen: Folien 4–5; Zugriff per NFSv4 und dCache: Folie 8
- ⚠️ IHEP/HEPS: keine Folien auf Indico. EOS + CTA stammt aus IHEP-Vorträgen außerhalb des Workshops (ISGC 2022, HEPiX 2023, CTA-Workshop 2025), Stand bis 2025.
-->

---
layout: none
---

<SitesProductionSlide />

<!--
Die vier Sites, die CTA heute in Produktion betreiben. Alle Zahlen aus den Site Reports auf Indico.

**DESY:** Drittes Jahr in Produktion, „No news is good news“. 45 Laufwerke (LTO-9 und IBM Enterprise) in 2 TS4500, 6 dCache-Instanzen, 15 VOs, 160 PB mit CTA geschrieben. Version 5.11.18. Als Nächstes: PostgreSQL-Scheduler testen, LTO-10 eventuell auslassen.

**RAL / STFC:** Instanz „Antares“. 2 Libraries, 100 Laufwerke, 40 GB/s, über 300 PB in 4 Jahren. 8 neue TS1170-Laufwerke (+83 PB), alte Daten werden auf TS1170 umkopiert. Hitzewelle Juni 2026: bis 90 % rel. Feuchte im Rechenzentrum, auf TS1170 59 unlesbare Dateien (0,02 %), davon 9 endgültig verloren.

**Fermilab:** Seit April/Mai 2025 von Enstore auf CTA, mit dCache. ≈ 240 Laufwerke in 7 Libraries (CMS und Shared), LTO-10 über SAS mit 4 Laufwerken pro Server. „Tag und Nacht“ gegenüber Enstore. Offen: Fällt die Datenbank aus, fallen alle Tape-Server aus; kleine Dateien.

**PIC:** Am 14.–15. September 2026 von Enstore auf CTA: 6.125 Bänder, 29,8 Mio. Dateien, 20 Projekte. Neue LTO-10-Library in eigenem Raum mit Feuchtekontrolle. Probleme: ca. 5 % der LTO-10-Bänder beim Labeln (behoben), sporadische Lese-/Schreibfehler, Klimaanlage braucht künstliche Wärmelast.

**❓ BEGRIFFE ERKLÄRT**

*LTO-10:* LTO („Linear Tape-Open“) ist der offene Standard für Magnetbänder, die Zahl ist die Generation. LTO-9 fasst 18 TB pro Band, LTO-10 (seit 2025) 30 TB (40 TB-Variante angekündigt). Wichtig: LTO-10-Laufwerke lesen keine älteren Bänder mehr (früher ging das eine Generation zurück). Wer umsteigt, braucht also neue Bänder und neue Laufwerke gleichzeitig.

*Warum DESY LTO-10 eventuell überspringt:* Auf der Folie steht nur „Skip LTO-10 perhaps?“, ohne Begründung. Naheliegend (meine Einschätzung, nicht von der Folie): keine Rückwärtskompatibilität, Bänder sind teuer, die ersten LTO-10-Erfahrungen waren holprig (siehe PIC), und DESY hat mit IBM Enterprise (TS11xx) eine Alternative. Also lieber bei LTO-9 bleiben und auf LTO-11 warten. Bei Gelegenheit Mwai Karimi fragen.

*„LTO-10 über SAS“:* SAS, nicht SaaS. SAS (Serial Attached SCSI) ist die Kabelverbindung zwischen Server und Bandlaufwerk. Bisher hingen Laufwerke per Fibre Channel (FC) am Server, meist zwei pro Server. Fermilab hängt jetzt vier LTO-10-Laufwerke per SAS über einen SAS-Switch an einen Server: günstiger pro Laufwerk, und fällt ein Server aus, können die Laufwerke einem anderen Server zugeteilt werden.

*„Ohne Datenbank fallen alle Tape-Server aus“:* CTA speichert alles Wissen (welche Datei auf welchem Band liegt, welche Aufträge warten) in einer zentralen Datenbank. Ist der Datenbank-Server weg, auch nur kurz, stürzen bei Fermilab alle Tape-Server-Prozesse (cta-taped) ab. Es gehen keine Daten verloren, die Bänder sind sicher, aber der Bandbetrieb steht still, bis die Datenbank wieder da ist und die Dienste neu gestartet sind. Fermilab wünscht sich, dass CTA kurze Ausfälle übersteht.

*„Nutzer schreiben noch kleine Dateien“:* Band ist gut für große Dateien (GB-Bereich). Bei vielen kleinen Dateien muss das Laufwerk ständig stoppen und neu anlaufen: langsam, und das schadet den Bändern. Enstore hat kleine Dateien vorher zu großen Paketen gebündelt (Small File Aggregation), CTA kann das nicht. Manche Experimente schreiben trotzdem weiter kleine Dateien. Fermilab baut Monitoring, um das zu erkennen und die Experimente anzusprechen. Die Instanz ist dadurch nicht kaputt, nur ineffizient.

*„Eigene Accounting-Datenbank“:* Hier heißt Accounting „Buchführung über den Betrieb“, nicht Abrechnung. Enstore hat jeden Mount, Dismount und jede Übertragung protokolliert, das half bei Fehlersuche und Statistik. CTA hat das nicht eingebaut, also hat Fermilab eine eigene PostgreSQL-Datenbank gebaut, die die Transfer-Logs zwei Jahre aufbewahrt. Laut Folie 17 ist sie schon fertig, nicht erst geplant.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Die vier Sites mit CTA in Produktion. Je Kachel: Status, zwei Kennzahlen, was gut läuft, Baustellen und nächste Schritte.

*DESY* (Site-Report M. Karimi, Datei 04):
- „Year 3 in Production“: Folie 1; „No News is Good News“: Folie 2; stabil, wenig Eingriffe: Folie 7
- 45 Laufwerke (LTO-9 und Enterprise), 2 TS4500: Folie 3; 6 dCache-Instanzen, 15 VOs: Folie 4; 160 PB mit CTA geschrieben: Folie 5
- PostgreSQL-Scheduler testen, „Skip LTO-10 perhaps?“: Folie 11

*RAL / STFC* (Antares-Report G. Patargias, Datei 05; Feuchtigkeit T. Byrne, Datei 17):
- 100 Laufwerke, 40 GB/s: Datei 05, Folie 2; über 300 PB in 4 Jahren: Folie 3
- 8 TS1170-Laufwerke und 1.665 neue Bänder in der Library „Asterix“, Kapazität +83 PB (≈ 1.665 × 50 TB pro TS1170-Band). Grund: Asterix war fast voll, die Tier-1-Zusagen 2025/26 waren sonst nicht zu halten: Datei 05, Folie 4; neuer EOS-Disk-Buffer: Folie 7
- WLCG-Tokens und PostgreSQL testen: Datei 05, Folie 12
- Hitzewelle Juni mit Feuchte-Alarmen: Datei 17, Folien 2–3; 59 unlesbare Dateien (41 von Disk gerettet, 9 verloren): Folie 13
- Entfeuchtung als Plan: Datei 17, Folien 10 und 14

*Fermilab* (Site-Report T. Messer, Datei 06 PDF):
- Migration von Enstore April–Mai 2025: Folien 7–8
- 7 Libraries, ~110 + ~130 = ~240 Laufwerke: Folie 4
- „Night-and-day difference from Enstore“: Folie 12; LTO-10 über SAS läuft gut: Folie 14
- Datenbank weg = alle Tape-Server fallen aus; Nutzer schreiben noch kleine Dateien: Folie 11
- Repack in LS3 (LTO-7M/LTO-8 → LTO-9/10): Folie 17
- ⚠️ Accounting-Datenbank: laut Folie 17 schon entwickelt, nicht erst geplant. Die Kachel nennt sie unter „Als Nächstes“, besser als „neu gebaut“ erzählen.

*PIC* (Site-Report A. Guino Feijoo, Datei 18):
- ~108 PB auf Band: Folie 2
- Migration in 2 Tagen (14.–15. Sept.), 6.125 Bänder, 29,8 Mio. Dateien, 20 Projekte: Folie 11
- Schneller, weniger Remounts als mit Enstore: Folie 14
- LTO-10: Fehler beim Labeln (Fujifilm-Firmware) und sporadische Lese-/Schreibfehler: Folie 7
- Klima: eigene Einhausung mit kontrollierter Feuchte, Probleme mit der Klimaanlage: Folien 3 und 9
- Pläne 2027: 9 weitere LTO-10-Laufwerke, LTO-8 → LTO-10, Upgrade auf CTA v6: Folie 17
-->

---
layout: none
---

<SitesOutlookSlide />

<!--
Sites, die CTA indirekt nutzen, noch prüfen oder sich anders entschieden haben, und die Themen, die sich durchziehen.

**European XFEL:** ≈ 30 PB Rohdaten pro Jahr, Spitzen 1–2 PB am Tag, Archiv bei DESY. Wünsche an CTA: Daten nach Experiment/Run gruppieren, Laufwerke fürs Lesen reservieren, RAW vor RED schreiben.

**TRIUMF:** ATLAS-Tier-1. cta-taped kann jetzt GNU-tar-Bänder lesen (nur lesend). Eine Entscheidung für CTA gibt es noch nicht.

**SURF:** Altes Band-System DMF von 1994 bis 2025, 90 PB übernommen. CTA war eine der geprüften Optionen, gewählt wurde ScoutAM von Versity, weil es auf der vorhandenen Hardware läuft.

**IHEP / HEPS:** Das Institut für Hochenergiephysik in Peking nutzt EOS + CTA. Im Programm standen zwei Vorträge („CTA status at IHEP“ und „Application of CTA in the HEPS“), aber auf Indico gibt es keine Folien.

Gemeinsame Themen: Klima und Feuchte (RAL, PIC), LTO-10 (Fermilab gut, PIC mit Anlaufproblemen), Ablösung von Enstore und DMF, und dass das Zurücklesen wichtiger wird (XFEL, Archive Metadata).

**❓ BEGRIFFE ERKLÄRT**

*European XFEL, Wünsche:* XFEL hat kein eigenes CTA, die Daten landen über dCache im CTA bei DESY. Die Wünsche richten sich an die CTA-Entwickler (neue Funktionen), umgesetzt würden sie im Betrieb bei DESY.
- *Nach Experiment gruppieren:* Dateien eines Experiments bzw. Runs sollen zusammen auf möglichst wenigen Bändern liegen. Heute verteilt CTA Dateien danach, welches Band gerade frei ist. Will ein Wissenschaftler später sein Experiment zurückholen, müssen dann viele Bänder gemountet werden, das dauert. Liegt alles zusammen, reichen ein, zwei Bänder.
- *Feste Lese-Laufwerke:* Ein paar Laufwerke sollen nur fürs Lesen reserviert sein, damit Rückholungen nicht warten müssen, wenn alle Laufwerke mit Schreiben beschäftigt sind.
- *RAW vor RED:* RAW = Rohdaten direkt vom Detektor, RED = „reduced“, also daraus berechnete, verkleinerte Daten. Wenn beides gleichzeitig ansteht, sollen die Rohdaten Vorrang beim Schreiben haben, weil sie unersetzlich sind (RED kann man aus RAW neu berechnen).

*TRIUMF, ATLAS-Tier-1:* Das LHC-Rechennetz (WLCG) ist in Stufen aufgebaut. Tier-0 ist das CERN, Tier-1 sind rund ein Dutzend große nationale Rechenzentren, die eine zweite Kopie der Rohdaten auf Band speichern und rund um die Uhr laufen. TRIUMF in Vancouver ist das Tier-1 für das ATLAS-Experiment in Kanada.

*„CTA kann jetzt GNU-tar-Bänder lesen (nur lesend)“:* TRIUMFs eigenes System Tapeguy hat seine Bänder im GNU-tar-Format beschrieben (Standard-Linux-Archivformat). CTA schreibt ein eigenes Format. Damit TRIUMF nicht Petabytes an alten Bändern umkopieren muss, kann CTA diese alten tar-Bänder jetzt direkt lesen. Neue Daten würde CTA aber im eigenen Format schreiben. „Altbestände“ = die schon vorhandenen Bänder.

*SURF, DMF:* DMF (Data Migration Facility, ursprünglich von SGI, heute HPE) ist ein älteres HSM, also Software, die Dateien automatisch zwischen Disk und Band verschiebt. SURF (Rechenzentrum der niederländischen Unis) hatte DMF seit 1994. Nicht auf CTA migriert, sondern auf ScoutAM von Versity. CTA, HPSS und andere wurden geprüft. ScoutAM gewann, weil es die vorhandene Hardware weiter nutzt, DMF-Bänder lesen kann und die Nutzer kaum etwas merken. Genau genommen: 90 PB waren unter DMF-Verwaltung. Übernommen wurden vor allem die Metadaten, die Daten blieben auf den Bändern und werden nach und nach neu geschrieben.

*IHEP / HEPS:* IHEP ist das Institut für Hochenergiephysik in Peking, HEPS („High Energy Photon Source“) ist deren neue Synchrotron-Lichtquelle. Die zwei Zeilen auf der Kachel sind nur die Titel der beiden Vorträge im Programm: „CTA status at IHEP“ (Site-Report) und „Application of CTA in the HEPS“ (wie CTA bei HEPS eingesetzt wird). Auf Indico wurden keine Folien hochgeladen. Ob die Vorträge stattfanden (z. B. remote), lässt sich aus den Unterlagen nicht sagen. Die Kachel sagt also nur: IHEP nutzt CTA, mehr Details gab es nicht.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Oben vier weitere Institute mit ihrem Verhältnis zu CTA, unten vier Themen, die in mehreren Berichten auftauchten.

*Quellen:*
- European XFEL: archiviert über DESY (GPFS → dCache → CTA): XFEL-Report (Datei 19), Folie 11; ~30 PB Rohdaten pro Jahr, Spitzen 1–2 PB am Tag: Folie 16; Wünsche: nach Experiment gruppieren (Folie 22), feste Lese-Laufwerke (Folie 25), RAW vor RED (Folie 28)
- TRIUMF: ATLAS Tier-1: TRIUMF-Report (Datei 20), Folie 1; CTA liest GNU-tar-Bänder (nur lesend), noch keine Entscheidung: Folie 5
- SURF: nach 30 Jahren weg von DMF, 90 PB: SURF-Keynote (Datei 03), Folien 3–4; CTA war eine der Optionen, Ziel „Keep Existing Hardware“: Folie 4
- IHEP/HEPS: beide Beiträge ohne Folien auf Indico (Übersicht.md)

*Themen unten:*
- Klima im Bandraum: RAL (Datei 17, Folien 3 und 13), PIC (Datei 18, Folien 3 und 9)
- LTO-10: Fermilab zufrieden (Datei 06, Folie 14), PIC mit Firmware- und Labeling-Problemen (Datei 18, Folie 7)
- Altsysteme verschwinden: Enstore bei Fermilab (Datei 06, Folie 7) und PIC (Datei 18, Folie 5), DMF bei SURF (Datei 03, Folie 3)
- Lesen wird wichtiger: XFEL-Wünsche (Datei 19, Folien 22–28), Archive Metadata (J. Leduc, Datei 23), Recall verdoppelt (Roadmap, Datei 28, Folie 13)
-->

---
layout: none
---

<DesyContactsSlide />

<!--
Leute aus dem DESY-Storage-Team, die ich beim Workshop kennengelernt habe.

MWAI KARIMI (Hamburg): betreibt CTA bei DESY, hat den Site-Report gehalten (3. Jahr in Produktion, 45 Laufwerke, 6 dCache-Instanzen, 160 PB mit CTA geschrieben). Aktuell laut seinen Folien: PostgreSQL-Scheduler testen, archive_file-Tabelle per Hash-Partitionierung und PostgreSQL-FDWs beschleunigen, eventuell LTO-10 überspringen.

JACEK CHODAK (Hamburg): Vortrag zu mhVTL, einer virtuellen Tape-Library für Linux. Er hat mhVTL vom eigenen Kernel-Modul auf TCMU (Standard-Linux, User Space) umgestellt. Ergebnis: kein Neubauen bei jedem Kernel-Update, +73 % Schreiben, +112 % Lesen, Bandlaufwerke auch übers Netz per iSCSI. Für uns spannend: CTA-Tests ohne echte Bandhardware.

PETER VAN DER REEST (Hamburg): schon lange im DESY-Storage-Team für dCache und CTA. Hat mir erzählt, dass er sich künftig mit Rucio beschäftigt. Steht als Mitautor auf dem XFEL-Vortrag. (Vorname vor dem Vortrag noch bestätigen.)

TINO REICHERT (Zeuthen): betreut allein die kleinere CTA-Instanz am DESY-Standort Zeuthen. Mit ihm bin ich am besten in Kontakt, er ist offen für Fragen. Seine Größenordnung liegt am nächsten an unserer.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Vier Leute aus dem DESY-Storage-Team, die ich beim Workshop kennengelernt habe, mit ihrem aktuellen Thema.

*Quellen:*
- Mwai Karimi: DESY-Site-Report (Datei 04); Hash-Partitionierung und PostgreSQL-FDWs: Folie 10; PostgreSQL-Scheduler testen, LTO-10 eventuell überspringen: Folie 11
- Jacek Chodak: mhVTL-Vortrag (Datei 12); Umstellung auf TCMU: Folie 1; +73 % Schreiben, +112 % Lesen: Folien 17–18; über iSCSI: Folien 8 und 15
- Peter van der Reest: als „P. van der Reest (DESY)“ in der Autorenliste des XFEL-Vortrags (Datei 19), Folie 1. Rucio-Thema: eigenes Gespräch. Vorname über LinkedIn bestätigt.
- Tino Reichert: kommt in keinem Vortrag vor. Alles auf der Kachel stammt aus dem eigenen Gespräch, LinkedIn-Link von ihm.
- ⚠️ Die kleinen Animationen sind nur Illustrationen der Themen, keine echten Daten.
-->

---
layout: none
---

<SectionSlide :num="3" title="Neu in CTA v6" subtitle="Das große Aufräum-Release: neue Komponenten, einheitlicher Betrieb" />

<!--
Kapitel 3: Was sich mit CTA v6 ändert.

**📚 HINTERGRUND & QUELLEN**

Kapitelfolie. Überblick über die v6-Themen: Welcome (Datei 01), Folie 9, und v6-Runtime (Datei 08), Folie 3.
-->


---
layout: none
clicks: 8
---

<V6Slide :step="$clicks" />

<!--
v6 räumt auf und entfernt veraltete Komponenten (Roadmap-Talk, Q4 2026).

[click] Frontend: Aus einem Frontend für alles werden zwei getrennte Dienste. Details im nächsten Klick.

[click] GRAFIK Frontend (das Fenster wächst aus der Frontend-Zeile heraus, der nächste Klick schließt es wieder).

In v5 gibt es **ein einziges Frontend** (cta-frontend), klassisch über XRootD-SSI. Es macht alles auf einmal: Es nimmt die Workflow-Events von EOS oder dCache an (archivieren, abrufen, löschen) und auch die cta-admin-Befehle der Operatoren. Dafür gibt es eine Konfiguration und ein Anmeldeverfahren für alle.

In v6 fällt XRootD-SSI weg, und das Frontend wird auf Basis von gRPC **in zwei getrennte Dienste aufgeteilt**:
- **cta-workflow-api** bekommt nur die Datei-Aufträge vom Disk-System. Anmeldung per mTLS (Zertifikate) oder JWT (Token), pro Instanz genau eins davon.
- **cta-admin-api** bekommt nur die Admin-Befehle. Anmeldung per Kerberos und/oder JWT.

Beide greifen weiterhin auf denselben Catalogue und dieselbe Scheduler-Datenbank zu.

Vorteil der Trennung: Über den Workflow-Zugang kann niemand Admin-Befehle ausführen, das ist sicherer. Außerdem lässt sich jeder Dienst einzeln skalieren, neu starten und überwachen.

Am CERN läuft die Produktion seit dem 11.10.2026 komplett über gRPC.

Falls jemand fragt: Auch v5 hatte schon ein gRPC-Frontend als zweite Variante, aber noch als ein einziger Dienst. Neu in v6 ist die Aufteilung in zwei Dienste und der Wegfall von XRootD-SSI.

[click] Scheduler-DB: Der Objectstore fällt weg, die Warteschlangen liegen jetzt in PostgreSQL. Details im nächsten Klick.

[click] GRAFIK Scheduler-DB (Fenster aus der Scheduler-DB-Zeile, der nächste Klick schließt es wieder).

Die Scheduler-DB ist die Warteschlange von CTA: Hier warten alle Aufträge (archivieren, abrufen, repack), bis ein Tape Daemon sie abholt.

In v5 liegt diese Warteschlange in einem **Objectstore**, also einem Ceph-Cluster (Rados). Jede Queue und jeder Auftrag ist ein einzelnes Objekt. Das ist schnell, aber man braucht dafür einen eigenen Ceph-Cluster, und man kann nur schwer hineinschauen, wenn etwas hängt.

In v6 übernimmt **PostgreSQL**, eine ganz normale Datenbank mit drei Tabellen:
- **PENDING:** Neue Aufträge kommen hier rein (1–10 Mio. Zeilen).
- **ACTIVE:** Holt ein Tape Daemon einen Auftrag ab, wandert er hierher (10–100 Tsd. Zeilen).
- **FAILED:** Fehlgeschlagene Aufträge landen hier oder werden neu eingereiht. Erfolgreiche werden gemeldet und gelöscht.

Vorteile: kein Ceph-Cluster mehr nötig, Standard-Technik, die jede Site kennt, und man kann per SQL direkt nachsehen, was in der Queue steht. Auch schnell genug: Das Verschieben per INSERT/DELETE statt einer Status-Spalte hat das Scheduling verdoppelt, im Test mit 100 Laufwerken und 10 Mio. Dateien schafft es 1.000 Metadaten-Operationen pro Sekunde. Dazu kommen optionales Batching beim Einreihen und automatische Aufräumroutinen.

Stand am CERN: Repack läuft schon auf PostgreSQL, die normalen Nutzer-Workflows noch auf dem Objectstore. DESY und RAL wollen den PostgreSQL-Scheduler testen.

[click] **Tape Daemon** (der Prozess, der die Laufwerke steuert):
- v5: Der Tape Daemon hat sich intern in mehrere Unterprozesse aufgespalten („fork“), z. B. einen für die Wartung. Das war schwer zu durchschauen und zu debuggen.
- v6: Ein einfacher Prozess pro Laufwerk, die Wartung läuft getrennt im Maintenance Daemon. Ein Server kann sauber mehrere Laufwerke betreiben (Fermilab macht das schon mit vier).
- Dazu mehr Schutz vor Datenverlust: Nach einem Fehler wird garantiert aufgeräumt, damit kein Band in einem unklaren Zustand bleibt. Weitere Schutzfunktionen (Laufwerk für einen Server reservieren, Bänder nur noch anhängen statt überschreiben) sind geplant, aber noch nicht fertig.

[click] **Konfiguration:**
- v5: Drei verschiedene Dateiformate für die verschiedenen Dienste. Tippfehler fielen oft erst im Betrieb auf.
- v6: Überall dasselbe Format (TOML, ein einfaches Textformat mit Abschnitten und Schlüssel = Wert).
- Neu: --config-check prüft die Datei vor dem Start. --config-strict bricht ab, wenn eine Option unbekannt ist oder fehlt. Fehler fallen also beim Start auf, nicht erst nachts im Betrieb.

[click] **Betrieb** (Überwachen im Alltag):
- Logs sind jetzt standardmäßig JSON, also maschinenlesbar. Man kann sie direkt in Tools wie Grafana/Elasticsearch filtern und auswerten.
- Health-Checks: Jeder Dienst beantwortet zwei einfache Fragen: „Lebst du?“ (/health/live) und „Bist du bereit für Arbeit?“ (/health/ready). Kubernetes oder das Monitoring kann einen hängenden Dienst so automatisch erkennen und neu starten.
- Metriken über OpenTelemetry (Standard für Messwerte wie Durchsatz und Warteschlangenlänge) sind nicht mehr experimentell, sondern offiziell unterstützt.

[click] **Werkzeuge:**
- v5: cta-admin für die Verwaltung plus viele einzelne cta-ops-Skripte, die das CERN-Team für den Alltag geschrieben hat.
- v6: Ein einziges Kommando „cta“ für alles.
- Es baut auf dem neuen Python-SDK auf. Damit können Sites auch eigene Skripte in Python schreiben, die mit CTA reden, statt Kommandozeilen-Ausgaben auszuwerten.
-->

---
layout: none
---

<SectionSlide :num="4" title="Roadmap" subtitle="Data Challenge 2027, Long Shutdown 3 und der Weg zum HL-LHC" />

<!--
Kapitel 4: Wohin es mit CTA geht.

**📚 HINTERGRUND & QUELLEN**

Kapitelfolie. Untertitel aus der CTA Roadmap (M. Davis, Datei 28), Folien 3–5.
-->


---
layout: none
---

<ScaleSlide />

<!--
Bilder aus der Roadmap von Michael Davis (Folie 12: "What does 2 EB/year look like?").
Links: Vladimír Bahyl neben dem Turm aus Bandkassetten, die an EINEM Tag Run-3-Datennahme verbraucht werden. Eine genaue Zahl steht nicht auf der Folie. Grobe Einordnung von mir: Rekordmonat Okt. 2025 waren 62 PB, also ca. 2 PB pro Tag.
Mitte: Eine Lieferung von 150 PB Bandmedien. Bei Run-4-Datenraten ist die in 2–3 Wochen voll.
Rechts zum Vergleich: Run-4 bringt pro Jahr 2 EB, mehr als alles, was heute am CERN auf Band liegt (1,48 EB). Das ist gut das 6-Fache des Rekordjahres 2025 (+304 PB).
Insgesamt erwartet: 8 EB Rohdaten in Run-4, 12 EB in Run-5, Schreibrate 300 GB/s.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Zwei Fotos aus der Roadmap machen die Datenmengen greifbar, rechts ein Balkenvergleich.

*Quellen:*
- Beide Fotos und Bildunterschriften („Tape media consumed in one day of Run–3 data taking“, „Delivery of 150 PB … consumed in 2–3 weeks at expected Run–4 data rates“): CTA Roadmap (Datei 28), Folie 12
- 62 PB Rekordmonat Okt. 2025: Welcome (Datei 01), Folie 6
- +304 PB in 2025, 1,48 EB auf Band 2026: Welcome, Folie 5
- 2 EB pro Jahr in Run-4: Roadmap, Folien 3 und 12
- 8 EB Rohdaten Run-4, 12 EB Run-5, 300 GB/s: Roadmap, Folie 11
- ⚠️ Wie viele PB der Kassettenturm im linken Foto sind, steht nicht auf der Folie. Grobe eigene Schätzung: 62 PB im Rekordmonat ≈ 2 PB pro Tag.
- Die Person im linken Foto ist Vladimír Bahyl (CERN), laut eigener Erkennung, nicht auf der Folie beschriftet.
-->

---
layout: none
---

<ChallengesSlide />

<!--
Run-4: ca. 8 EB Rohdaten, Run-5: ca. 12 EB. Eine Lieferung von 150 PB Bändern wäre bei Run-4-Datenraten in 2–3 Wochen voll.
Bandtechnologie: Enterprise- und LTO-Linien verschmelzen, LTO-Kapazität wächst nur um +66 % statt +100 %, Rack-Libraries mit 1.500 statt 15.000 Slots.
Recall von Band verdoppelte sich 2024–2025 auf 7–11 PB/Monat, Spitze 28 PB/Monat.
Drive Efficiency = Transfer Efficiency × Mount Efficiency. CMS-Recall: 45 %, Ziel 80 %, laut CTA-R&D mit besserer Colocation erreichbar.

**❓ DRIVE EFFICIENCY EINFACH ERKLÄRT**
- *Recall* = Daten vom Band zurück auf Disk lesen (Retrieve), z. B. wenn ein Experiment alte Daten neu auswerten will. Das Gegenteil ist Archive (aufs Band schreiben).
- *Drive Efficiency* = Wie viel der Zeit, in der ein Laufwerk belegt ist, liest es wirklich Daten mit voller Geschwindigkeit? 45 % heißt: Mehr als die Hälfte der Zeit geht für anderes drauf.
- Zwei Faktoren, die multipliziert werden:
  - *Mount-Effizienz:* Wie viel der Zeit geht fürs Holen, Einlegen, Spulen und Auswerfen des Bands drauf? Das sind die Mounts, die du meinst. Wenig Mounts mit viel Daten pro Mount = gut.
  - *Transfer-Effizienz:* Wenn das Band drin ist, wie nah kommt das Lesen an die Höchstgeschwindigkeit? Viele kleine Dateien oder Dateien verstreut übers Band bremsen, weil das Laufwerk ständig vor- und zurückspulen muss.
- Effizient mounten ist also die Hälfte der Geschichte. Die andere Hälfte: Wenn das Band drin ist, zügig am Stück lesen.
- Die Lösung für beides ist *Colocation*: Daten, die später zusammen gelesen werden (z. B. ein Datensatz von CMS), liegen zusammen auf wenigen Bändern und dort direkt hintereinander. Dafür liefert Rucio die „Archive Metadata“ (Plan Nr. 3 rechts).
- ⚠️ Die Balken auf der Folie sind schematisch: Nur der Leseanteil (45 % bzw. 80 %) stammt aus der Quelle, die Aufteilung in Blöcke ist zur Veranschaulichung.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Die Herausforderungen für Run-4, die gemessene Laufwerks-Effizienz und die drei Entwicklungsziele für LS3.

*Quellen (alle CTA Roadmap, M. Davis, Datei 28):*
- 2 EB pro Jahr, 5 × 60 GB/s = 300 GB/s: Folie 3
- 8 EB (Run-4) und 12 EB (Run-5) Rohdaten: Folie 11
- 150 PB in 2–3 Wochen voll: Folie 12
- Bandtechnologie: Enterprise und LTO verschmelzen, LTO-Kapazität +66 % statt +100 %, Rack-Libraries mit 1.500 statt 15.000 Slots: Folie 8
- Recall verdoppelt 2024–2025 auf 7–11 PB/Monat, Spitze 28 PB: Folie 13
- Drive Efficiency = Transfer Efficiency × Mount Efficiency: Folie 16
- 45 % beim CMS-Recall, Ziel 80 %: Folie 17; 80 % mit besserer Colocation erreichbar: Folie 18
- Scheduler neu bauen (keine Sperr-Konkurrenz mehr): Folien 14 und 21
- Tape-Supply in den Scheduler: Folien 15 und 21
- Archive Metadata von Rucio für ATLAS und CMS: Folie 20
-->

---
layout: none
---

<SectionSlide :num="5" title="IBM Diamondback" subtitle="Die Rack-Library im Langzeittest am CERN: Mechanik, Mount-Zeiten, Kosten" />

<!--
Kapitel zur IBM Diamondback: interessant für unseren Stand bei der GSI, weil Rack-Libraries eine Option für kleinere Standorte sind. Grundlage ist die Keynote von Vladimir Bahyl (CERN): "Evaluation of the IBM Diamondback rack-space tape library".

Kapitel 5: Die IBM Diamondback, eine Bandbibliothek im Rack-Format, im Langzeittest am CERN.

**📚 HINTERGRUND & QUELLEN**

Kapitelfolie. Grundlage des Kapitels ist vor allem die Keynote „Evaluation of the IBM Diamondback rack-space tape library“ (V. Bahyl, Datei 16).
-->

---
layout: none
---

<RackLibrariesSlide />

<!--
🔵 Vier Bandbibliotheken im Rack-Format, die CERN verglichen hat (Daten von Folie 1 der Keynote).
Alle passen auf die Stellfläche eines Server-Racks und fassen mit LTO-10 (40 TB) rund 45–80 PB.
Die IBM Diamondback stand als Leihgabe von IBM am CERN und lief dort im Langzeittest, auch mit echten CMS-Daten.
Zum Vergleich: eine typische CERN-TS4500 hat 18 Frames, 22.000 Slots und 64 Laufwerke. Für die gleiche Slot-Zahl bräuchte man 14 Diamondbacks.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Vier Bandbibliotheken im Rack-Format im Vergleich. Die Diamondback ist die, die CERN getestet hat.

*Quellen (Diamondback-Vortrag, V. Bahyl, Datei 16):*
- Fotos, Maße, Gewicht, Slots, PB mit LTO-10 40 TB und Laufwerke aller vier Libraries: Folie 1
- Diamondback als Leihgabe von IBM: Folie 31 („Grateful to IBM for loaning the tape library“)
- Test mit echten CMS-Daten (Schreiben Nov. 2025, Lesen Feb. 2026): Folien 16–17
- „Bericht Sept. 2026“: Datum in der Fußzeile aller Folien
- TS4500 am CERN: 18 Frames, 22.000 Slots, 64 Laufwerksplätze, entspricht 14 Diamondbacks: Folie 27
- Breite 60–91 cm: Folie 1 (Spectra Cube 91,2 cm, die anderen ~60 cm)
-->

---
layout: none
clicks: 3
---

<TierMechanismSlide :step="$clicks" />

<!--
AUFBAU: Draufsicht auf zwei Slots. In jedem Slot liegen bis zu 9 Kassetten hintereinander (Tier 1 bis 9). Eine Feder hinten drückt alle nach vorn. Nur die vorderste (T1) kann der Greifer herausziehen.

[click] MOUNT AUS T1: Greifer zieht T1, der Rest rutscht durch die Feder nach. Mount bis BOT im Schnitt ca. 19 s.

[click] ZURÜCK: Nach dem Entladen legt die Library die Kassette wieder auf einen T1-Platz. Die anderen werden nach hinten gedrückt, die Feder wird gespannt.

[click] MOUNT AUS DER TIEFE: Die gesuchte Kassette liegt in T3. Der Greifer (Doppelgreifer, hält 2 Kassetten) muss erst T1 und T2 in einen anderen Slot umlagern. Deshalb dauert T3 im Schnitt 42 s, T9 104 s.
Laut Messung liegen T1/T2, T3/T4 usw. paarweise nah beieinander: kleine Sprünge = Tausch im selben Slot, große Sprünge = Kassetten müssen weggelegt werden.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Eine Animation, wie die Diamondback Kassetten in bis zu 9 Ebenen hintereinander lagert und warum tiefe Kassetten länger brauchen.

*Quellen (Diamondback-Vortrag, Datei 16):*
- 9 Ebenen (Tiers) tief, Federkraft schiebt alle nach vorn, nur T1 entnehmbar, Rest rückt nach: Folie 5 („IBM Slot Tiering explained“)
- 1.550 Slots, 9 Tiers, Doppelgreifer („Dual Grippers“, im IBM-Schaubild): Folie 4
- Nach dem Entladen geht die Kassette automatisch auf einen T1-Platz: Folie 9
- Große Lücke = Kassetten müssen weggelegt werden, kleine Lücke = Tausch im selben Slot: Folie 10
- Ø 19 s (T1), 42 s (T3), 104 s (T9) mit 1 Laufwerk: Folie 12 (Mittelwerte aus den Diagrammdaten)
- Oben rechts steht, was der Greifer gerade tut. Der Zähler daneben läuft während der Animation auf den gemessenen Mittelwert hoch (19 s bei T1, 42 s bei T3). Er zeigt keine echte Zeit, die Animation selbst dauert nur wenige Sekunden.
- ⚠️ Die Animation ist vereinfacht: zwei Slots von oben, ausgedachte Kassettennamen. Dass der Doppelgreifer T1 und T2 gleichzeitig umlagert, ist meine Erklärung für die paarweisen Stufen, nicht wörtlich aus den Folien.
-->

---
layout: none
clicks: 2
---

<MountBaselineSlide :step="$clicks" />

<!--
MESSUNG 1 (Baseline): 1 Laufwerk, zufällige Kassetten, 50 pro Ebene. Jede Kassette: Mount ins mittlere Laufwerk, zum Bandanfang (BOT), Dismount.
Jeder Punkt = eine Kassette. X-Achse: Zeit bis BOT. Y-Achse: Reihe in der Library (oben bis unten, 44 Reihen). Farbe: Ebene T1–T9.
Lesehilfe: Die Wolken stehen fast senkrecht, also ist die Reihe egal, der Greifer fährt sehr schnell vertikal. Die Wolken liegen paarweise: T1/T2, T3/T4 usw. Kleine Lücke = Tausch im selben Slot, große Lücke = Kassetten müssen weggelegt werden.
Zahlen: T1 ab ca. 18 s, Mittel 19 s. T9 Mittel 104 s, max. 111 s. Über 2 min Mount = ca. 50 GB verlorene Schreibzeit bei 400 MB/s.

[click] POP-UP DOPPELGREIFER (wächst aus Kachel 4, der nächste Klick schließt es). Der Greifer hat zwei Hände. Die Animation spielt nacheinander T1, T2, T3, T4 durch:
- T1: direkt nehmen, ins Laufwerk.
- T2: Hand 1 nimmt T1, Hand 2 nimmt T2, T1 geht sofort zurück in denselben Schacht. Tausch vor Ort, nur ~5 s mehr.
- T3: T1 und T2 davor, beide Hände voll. Erst in einen anderen Schacht bringen, zurückfahren, dann T3. ~18 s mehr.
- T4: wie T3, plus wieder ein Tausch vor Ort.
Rechts die gemessenen Mittelwerte: 19/24, 42/47, 61/67, 83/87, 104 s. Paare mit kleiner Lücke, dazwischen große Sprünge.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Das Messergebnis mit einem Laufwerk als Punktdiagramm. Jeder Punkt ist eine Kassette, die Farbe zeigt die Ebene.

*Quellen (Diamondback-Vortrag, Datei 16):*
- Testablauf: zufällige Kassette, 50 pro Ebene, Mount ins mittlere Laufwerk, zum BOT, entladen; Phasen mit 1, 3 und 5 Laufwerken: Folie 9
- Diagramm, 44 Reihen, 9 Ebenen, Erklärung der Lücken: Folie 10. Die Punkte sind aus den Diagrammdaten der PowerPoint-Datei nachgezeichnet, nicht abgeschätzt.
- „Mount + BOT dauert im Schnitt mindestens 20 s“, über 2 min ≈ 50 GB verlorene Transferzeit bei 400 MB/s: Folie 12
- Median 61 s, schnellste 18 s, langsamste 111 s: selbst aus den Diagrammdaten von Folie 10 berechnet
- Pop-up: Mittelwerte pro Ebene selbst aus den Diagrammdaten berechnet. ⚠️ Dass der Doppelgreifer die Paare erklärt, ist meine Deutung; die Folien sagen nur „kleine Lücke = Tausch im selben Slot, große Lücke = Kassetten wegbringen“ (Folie 10) und zeigen den Doppelgreifer (Folie 4). Die Animation ist schematisch (Draufsicht, ein Ersatzschacht).
-->

---
layout: none
clicks: 1
---

<MountDrivesSlide :step="$clicks" />

<!--
MESSUNG 2: 1, 3 und 5 Laufwerke laufen gleichzeitig, ohne Pause, nur Mount + BOT. Gleiche X-Achse (0–240 s) für alle drei.
Mit mehr Laufwerken werden die Stufen unscharf und alles rutscht nach rechts. Mittelwert T1: 19 s / 32 s / 52 s. Ausreißer bis 229 s (3 LW) bzw. 219 s (5 LW).

[click] GANZER ZYKLUS: Mount + BOT + Dismount, X-Achse 0–360 s. Minimum ca. 60 s (1 LW), 80 s (3 LW), 100 s (5 LW). Einzelne Zyklen bis fast 6 min.
Wechsel pro Stunde: 32 / 70 / 90. Bei 5 Laufwerken mit schnellen Wechseln (z. B. Bänder labeln) ist die Robotik gesättigt. Die IBM-Dokumentation nennt nur Mount-Zeiten aus T1.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Dieselbe Messung mit 1, 3 und 5 Laufwerken gleichzeitig. Ohne Klick nur Mount bis BOT, nach dem Klick der ganze Zyklus mit Entladen.

*Quellen (Diamondback-Vortrag, Datei 16):*
- Mount mit 3 und 5 Laufwerken: Folie 11 (Diagrammdaten), Vergleich mit 1 Laufwerk: Folie 10
- Ganzer Zyklus mindestens ~60 / ~80 / ~100 s: Folie 13
- 32 / 70 / 90 Wechsel pro Stunde, Robotik gesättigt bei 5 Laufwerken mit schnellen Zyklen, IBM-Doku nennt nur T1: Folie 14
- Mediane (61/72/73 s bzw. 102/154/176 s) und langsamste Werte: selbst aus den Diagrammdaten der Folien 10, 11 und 13 berechnet
-->

---
layout: none
---

<MountSleepSlide />

<!--
MESSUNG 3 (Sleep-Test): Wieder 5 Laufwerke, aber nach jedem Mount 5 min Pause. Das simuliert einen Transfer von ca. 120 GB pro Kassette.
Links Referenz (1 LW), Mitte 5 LW ohne Pause, rechts 5 LW mit Pause. Mit Pause sieht das Bild fast aus wie die Referenz.
Produktion: CMS schrieb 23.–26. Nov. 2025 drei Tage mit allen 5 Laufwerken bei ~400 MB/s, die Robotik hielt mit. Beim Lesen am 12. Feb. 2026 nur ~150 MB/s, wegen ungünstiger Datenlage auf Band, nicht wegen der Library.
Fazit für uns: Für normales Schreiben reicht die Robotik. Kritisch wird es bei vielen kurzen Mounts (Recall vieler kleiner Dateien, Labeln, Repack).

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Mit 5 Minuten Pause pro Kassette verhalten sich 5 Laufwerke wieder wie eines. Unten die Erfahrungen aus dem echten CMS-Betrieb.

*Quellen (Diamondback-Vortrag, Datei 16):*
- 5 min Pause simuliert ~120 GB Transfer, Verhalten wie die Baseline: Folie 15 (Diagrammdaten 1 LW und 5 LW + Pause)
- CMS schreibt 23.–26. Nov. 2025, alle 5 Laufwerke bei ~400 MB/s, Robotik hielt mit: Folie 16
- CMS liest 12. Feb. 2026, nur ~150 MB/s, Ursache Datenlage auf Band: Folie 17
-->

---
layout: none
---

<DiamondbackScaleSlide />

<!--
WIE VIELE: Am CERN steht genau 1 Diamondback, eine Leihgabe von IBM für den Test.
Rechnung aus Vladimírs Folien (Szenario ab 2030: 1 EB neue Daten pro Jahr bei ~200 GB/s):
- Nur für die Kapazität: 16 Diamondbacks pro Jahr (je ~62 PB mit LTO-10 40 TB).
- Für den Durchsatz: ~500 LTO-10-Laufwerke à 400 MB/s, bei max. 14 Laufwerken pro Library also ~35 Diamondbacks.
Heute hat CERN 1,8 EB Bandkapazität und ~200 Laufwerke.

KOSTEN (Listenpreise 12/2025, Vergleich noch mit LTO-9): TS4500 mit 18 Frames, 22.000 Slots, 14 LTO-9-Laufwerken ca. 4,3 Mio. CHF. 14 Diamondbacks mit der gleichen Slot-Zahl, je 1 Laufwerk, ca. 6,6 Mio. CHF, also 50 % teurer.

Die "20 Millionen" stehen auf keiner Indico-Folie. Falls die Zahl im Vortrag mündlich fiel: grob passt sie zu ~35 Libraries × ~0,47 Mio. CHF ≈ 16,5 Mio. CHF (eigene Hochrechnung, nur 1 Laufwerk pro Library, ohne Bänder). Nicht als CERN-Zahl zitieren.

FAZIT CERN: Für ein aktives Archiv passt die Diamondback nicht. Libraries laufen 15+ Jahre, Diamondback nur LTO-9 bis LTO-11. Laufwerke per Lizenz an eine Library gebunden, kann man nicht umziehen. Ohne Laufwerk nicht kaufbar. Kein Kostenvorteil. Wartung bleibt beim Hersteller, Personal wächst nicht.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Was die Diamondback im CERN-Maßstab bedeuten würde: Anzahl, Listenpreise und das Fazit des CERN.

*Quellen (Diamondback-Vortrag, Datei 16):*
- 1 Diamondback als Leihgabe: Folie 31
- CERN heute 1,8 EB und ~200 Laufwerke; 1 EB pro Jahr ab 2030 bei ~200 GB/s; 16 Diamondbacks pro Jahr für die Kapazität; ~500 LTO-10-Laufwerke = ~35 Diamondbacks für den Durchsatz; nur 3 LTO-Generationen: Folie 26
- Listenpreise 12/2025: TS4500 ~4,3 Mio. CHF, 14 Diamondbacks ~6,6 Mio. CHF, 50 % teurer: Folie 27
- ~1.550 LTO-9-Slots pro Diamondback (14 × 1.550 ≈ 22.000): Folie 4. Die Grafik (18 Frames vs. 14 Racks, Punkte = Laufwerke) ist eine eigene Schemazeichnung, nicht maßstabsgetreu.
- Laufwerke per Lizenz an eine Library gebunden, ohne Laufwerk nicht kaufbar: Folien 24–25
- Gebaut als autonomes Deep Archive: Folie 29; CERN-Anforderungen (aktives Archiv, 15+ Jahre, wenig Personal, kein Kostenvorteil): Folie 30
- ⚠️ „0,47 Mio. CHF pro Library“: selbst gerechnet (6,6 Mio. ÷ 14).
- ⚠️ Die mündlich erwähnten „20 Millionen“ stehen auf keiner Folie. Eigene Hochrechnung: ~35 × 0,47 Mio. ≈ 16,5 Mio. CHF, nur 1 Laufwerk pro Library, ohne Bänder.
-->

---
layout: none
clicks: 2
---

<NextStepsSlide :step="$clicks" />

<!--
Wie es bei uns weitergeht. Drei Schritte, jeder Klick deckt den nächsten auf.

1. Jetzt: Tino hat sich meinen Stand angeschaut. Das ist noch der Tutorial-Stand. Sein Vorschlag: daraus eine richtige Instanz machen, eingerichtet wie für mehrere Experimente (eigene VOs, Storage Classes, Tape Pools, Mount Policies per cta-admin). So lerne ich, wie ein echtes Setup aussieht.

[click] 2. Sobald CTA v6 da ist: Julio meinte, an der jetzigen Instanz lohnt sich nicht viel, sie ist nicht produktiv. Besser mit v6 von Grund auf neu aufbauen und dann echte Data Challenges damit machen.

[click] 3. In ein paar Monaten: Michael Davis (Section Leader der Archive-Gruppe) ist grundsätzlich bereit, zu uns zu kommen, gern mit Julio, für einen Mini-Workshop. Die nächsten Monate sind schwierig, weil Julio nach Spanien und Vladimír nach China zu Workshops fahren. Ich soll mich bei ihm melden, wenn es so weit ist.

**📚 HINTERGRUND & QUELLEN**

*Was die Folie zeigt:* Unser eigener Plan nach dem Workshop, nicht aus den Workshop-Folien.

*Quellen:*
- ⚠️ Alle Punkte stammen aus Gesprächen (Tino, Julio, Michael Davis), nicht von Indico-Folien.
-->

---
layout: none
---

<QuestionsSlide />

<!--
Danke sagen und Fragen aus dem Publikum aufnehmen. Alle Workshop-Folien sind auf Indico verlinkt.

**📚 HINTERGRUND & QUELLEN**

Schlussfolie. Alle Workshop-Folien sind auf Indico verlinkt: indico.desy.de/event/52689
-->

# Validation — MindFoundry (mindfoundry)

## 1. Überblick

- **Repository:** `mindfoundry` (Commit: Content-Kopie, verankert über `source-checksum.md`)
- **Geprüfte Inventar-Dokumente:** 8 (`01-mindfoundry/index.md`, `files.md`, `functions.md`, `data-models.md`, `architecture.md`, `nim.md`, `rust-foundation.md`, `tests.md`)
- **Datei-Abdeckung:** 48/48 Repo-Dateien gelesen (davon 3 PNG-Binär-Assets dokumentiert via Hash/Typ/Metadaten/Verwendung)
- **Checksum-Freeze:** `source-checksum.md` (48 Hashes) — alle 48 Hashes stimmen mit `sha256sum` der tatsächlichen Dateien überein (Verifikation durchlaufen, 0 Differenzen).
- **Read Timestamp aller Dateien:** 2026-08-05
- **Encoding:** UTF-8 (alle Textdateien; JSONL/CSV/HTML/Markdown bestätigt)
- **Ergebnis:** 40 Prüfaussagen als `CONFIRMED_PRESENT` verifiziert, 8 Inventar-Behauptungen als `CONFIRMED_ABSENT` korrigiert, 2 Repo-interne Dokumentabweichungen dokumentiert (kein Inventarfehler).

## 2. Checksum-Freeze-Verifikation

`/workspaces/bkg-nim/code/bkg-nim/docs/inventory/source-checksum.md` wurde vollständig gelesen (46.085 Bytes). Für alle 48 Einträge unter `./mindfoundry/...` wurde `sha256sum` gegen die Repo-Dateien ausgeführt. Ergebnis: **0 Differenzen** — alle 48 Hashes identisch. Damit ist die Inhalts-Ankerung der Quellbasis gültig.

## 3. Claim-Crosscheck-Tabelle

Legende: `CP` = CONFIRMED_PRESENT, `CA` = CONFIRMED_ABSENT. Belege referenzieren `repos/mindfoundry/<Datei>:<Zeilen>`.

| Evidence-ID | Inventar-Dokument (Claim) | Ergebnis | Beleg |
|---|---|---|---|
| EV-MF-000001 | index.md: Zweck — Long Agent, Replicants pro Teammitglied, zitierte Antworten | CP | README.md:1-60 |
| EV-MF-000002 | index.md: 250 Zimmer, 2.500 Buchungspersonen, 3.150 Reservierungen, 8 Staff, 500 Vorfälle | CP | generate.py:73,86,93-94,111; Assertion `len(counts)==2500 and sum(counts)==3150` (generate.py:94) |
| EV-MF-000003 | index.md: Pipeline — Generierung, API, NIM-Synthese, Policy-Gate, Eval, Discord, UI | CP | README.md; Makefile:68-164 |
| EV-MF-000004 | index.md: Tabellen rooms, bookers, reservations, staff, incidents, messages, audit_logs | CP | generate.py:50-63 (7 Tabellen, keine weiteren) |
| EV-MF-000005 | index.md: Persistenz JSONL (160 Memories, 500 Events) | CP | Zählung `discord_memories.jsonl`=160, `two_day_event_stream.jsonl`=500 |
| EV-MF-000006 | index.md: Env-Variablen NVIDIA_NIM_API_KEY, NVIDIA_NIM_MODEL, HOTELSIM_PORT etc. | CP | nemotron_rag_bridge.py:42-49; Makefile:25,30; demo_walkthrough.py:48 |
| EV-MF-000007 | files.md: generate.py erzeugt 2500 Bookers, 3150 Reservierungen (2050×1/300×2/100×3/50×4) | CP | generate.py:86,93 |
| EV-MF-000008 | files.md: Schema inkl. incidents.reservation_id/room_id/assigned_staff_id FKs, Indizes | CP | generate.py:52-63 |
| EV-MF-000009 | files.md: Assertion `pick_weighted` Fallback auf letztes Element | CP | generate.py:34-39 |
| EV-MF-000010 | files.md: Clearance high für S001/S003/S008 | CP | generate.py:82 |
| EV-MF-000011 | files.md: Nachrichten-Kanäle, 55/45 en/zh-TW, Typ→Staff-Routing ROUTE | CP | generate.py:11,28-32,88 |
| EV-MF-000012 | files.md: events en 256 / zh-TW 244 | CP | programmatische Zählung `two_day_event_stream.jsonl` (256/244) |
| EV-MF-000013 | files.md: messages 500 Zeilen, Typ-Verteilung lost_item 60 … billing 39 | CP | programmatische Zählung (500 Einträge, Verteilung exakt) |
| EV-MF-000014 | files.md: memories 160 Einträge, 7 Vorlagen, 5 Personen | CP | programmatische Zählung (160, 7 Templates, 5 Personen) |
| EV-MF-000015 | files.md: `/evaluations/baseline` liest `reports/baseline-eval.json` (Namensdivergenz) | CP | api/server.py:54-55 (Dateiname `baseline-eval.json`) |
| EV-MF-000016 | files.md: PyYAML nicht zur Laufzeit genutzt („parsing is only needed for tests") | CP | requirements.txt:11-12; kein `import yaml` in Code (Grep über alle .py = 0 Treffer); policy_gate.py enthält keinen yaml-Bezug |
| EV-MF-000017 | files.md: reports/.gitkeep = leere Datei (0 Bytes) | CP | stat=0 Bytes, wc=0 |
| EV-MF-000018 | files.md: live-ops-summary — 37 Incidents, 12 urgent, 4 privacy-sensitive | CP | live-ops-summary.txt:2-3 |
| EV-MF-000019 | files.md: eval-baseline-500 — 500 Einträge, alle Score 4/4, leak_hits leer | CP | programmatische Prüfung (500 Ergebnisse, alle 4 Metriken 500/500, leak_hits=0) |
| EV-MF-000020 | files.md: Drip-/Seed-Limits (3 bzw. 16), Escalation-Mirror | CP | drip_discord_incidents.py; seed_discord_incidents.py |
| EV-MF-000021 | files.md: Replicant-Limits (Fälle 4, Discord-Memories 8) | CP | replicants.py:104,115,117 |
| EV-MF-000022 | files.md: groups.json 7 Gruppen; CSV 8 User mit 18-Zeichen-Passwörtern | CP | workspace-imports/groups.json (7); users-google-admin-import.csv (8, 18 Zeichen) |
| EV-MF-000023 | files.md: workspace-imports erzeugt von prepare_workspace_imports.py | CP | prepare_workspace_imports.py |
| EV-MF-000024 | functions.md: Gate-Entscheidungen allow / allow_with_redactions, Rollen-Egress | CP | policy_gate.py:28-78 |
| EV-MF-000025 | functions.md: NIM-Bridge — 1 Retry, Fallback-Renderer, answered-Set | CP | nemotron_rag_bridge.py:81-186 |
| EV-MF-000026 | data-models.md: Buchungspersonen-Tabelle **„250 synthetische Gäste"** | **CA** | generate.py:86 (`range(1,2501)` → 2.500) |
| EV-MF-000027 | data-models.md: Zimmer **„500 Zimmer mit Typ-Verteilung standard 55 %, deluxe 25 %, suite 15 %, presidential 5 %"** | **CA** | generate.py:73 (250 Zimmer); generate.py:15 (Verteilung: standard 0.55, deluxe 0.25, family 0.10, suite 0.07, accessible 0.03 — kein presidential) |
| EV-MF-000028 | data-models.md: Loyalty „35 % normal / 40 % silver / 20 % gold / 5 % platinum" | **CA** | generate.py:89 (`['none','silver','gold','platinum']`, Gewichte .65/.20/.10/.05) |
| EV-MF-000029 | data-models.md: Nachrichten **„3150 zweisprachige Sim-Nachrichten"**, „message_id (Autoincrement)", „3000 Textvarianten" | **CA** | generate.py:111 (500 Nachrichten, 1 je Incident; 3150 ist die Reservierungszahl); generate.py:59 (message_id TEXT PK, kein Autoincrement); generate.py:115-126 (20 Vorlagen: 10 en + 10 zh) |
| EV-MF-000030 | data-models.md: Tabelle **„evaluations"** | **CA** | generate.py:50-63 (keine evaluations-Tabelle im Schema; Eval-Ergebnisse ausschließlich als JSON-Report) |
| EV-MF-000031 | data-models.md: FK **„incidents.booker_id → bookers.id, incidents.staff_id → staff.id"** | **CA** | generate.py:56-58 (incidents: reservation_id REFERENCES reservations, room_id REFERENCES rooms, assigned_staff_id REFERENCES staff; keine booker_id-, keine staff_id-Spalte) |
| EV-MF-000032 | data-models.md: Zimmer ohne FK-Bezug („rooms nicht referenziert") | **CA** | generate.py:52 (reservations.room_id REFERENCES rooms) und generate.py:56 (incidents.room_id REFERENCES rooms) |
| EV-MF-000033 | data-models.md: Staff-Liste „Ben Wu/Grace Liu/Leo Wang/Maya Chen/Kevin Liu/Annie Chang" | **CA** | generate.py:17-26 (tatsächlich: Maya Chen, Leo Wang, Nina Lin, Grace Liu, Ben Wu, Iris Tsai, Kevin Huang, Annie Chang; „Kevin Liu" existiert nicht) |
| EV-MF-000034 | nim.md: Schlüssel **„NVIDIA_API_KEY"**, Deaktivierung via **„NIM_DISABLE"** | **CA** | nemotron_rag_bridge.py:46,49 (tatsächlich `NVIDIA_NIM_API_KEY`, `NVIDIA_NIM_DISABLE` == „1") |
| EV-MF-000035 | nim.md: Timeout **„20 s je Versuch"** | **CA** | nemotron_rag_bridge.py:148 (`timeout=60`) |
| EV-MF-000036 | nim.md: Modell nvidia/llama-3.3-nemotron-super-49b-v1.5, temp 0.2, top_p 0.9, max_tokens 600, stream False | CP | nemotron_rag_bridge.py:47,128-131; Makefile:25 |
| EV-MF-000037 | architecture.md: Komponenten/Graph (generate→SQLite→export/replicants/live_ops/evaluate/drip/api; bridge→gate→Discord; updater→memories; workspace-imports→provision) | CP | generiert aus Quellcode-Grep; entspricht den Imports je Modul |
| EV-MF-000038 | tests.md: Testfälle test_sim.py (250/2500/3150/8/500/500, 300×2/100×3/50×4) und test_live_ops.py | CP | tests/test_sim.py; tests/test_live_ops.py |
| EV-MF-000039 | tests.md: Testdaten-Verifikationen (eval 500/4.0, live-ops 37/12/4, events 500, memories 160) | CP | programmatische Prüfungen (siehe Abschnitt 5) |
| EV-MF-000040 | rust-foundation.md: benötigte Crates/Fehler-Typen/Architektur (kein Code-Übernahmeanspruch) | CP | konsistent mit beobachteten Capabilities (Checkliste, keine Widersprüche zu Quellcode gefunden) |

## 4. Konsolidierte Korrekturen

Die folgenden Inventar-Aussagen waren fehlerhaft und sind wie folgt zu ersetzen:

1. **EV-MF-000026 — Buchungspersonen:** „250 synthetische Gäste" → **2.500 synthetische Buchungspersonen** (B0001–B2500, generate.py:86).
2. **EV-MF-000027 — Zimmerzahl und -verteilung:** „500 Zimmer, standard 55 %/deluxe 25 %/suite 15 %/presidential 5 %" → **250 Zimmer** (R001–R250, generate.py:73) mit Verteilung **standard 55 %, deluxe 25 %, family 10 %, suite 7 %, accessible 3 %** (generate.py:15). Es existiert kein Presidential-Typ.
3. **EV-MF-000028 — Loyalty-Verteilung:** „35 % normal / 40 % silver / 20 % gold / 5 % platinum" → **none 65 %, silver 20 %, gold 10 %, platinum 5 %** (Stufe heißt `none`, nicht `normal`; generate.py:89).
4. **EV-MF-000029 — Nachrichtenzahl:** „3150 Sim-Nachrichten" → **500 Nachrichten** (1 je Incident, generate.py:111); 3150 ist die Reservierungszahl. `message_id` ist **TEXT PRIMARY KEY** (`MSG#####`), kein Autoincrement. Es existieren **20 Nachrichtenvorlagen** (10 en + 10 zh), nicht 3000 Varianten.
5. **EV-MF-000030 — evaluations-Tabelle:** Existiert **nicht** im SQLite-Schema. Evaluationsergebnisse liegen ausschließlich als JSON-Report (`reports/eval-baseline-500.json`) vor.
6. **EV-MF-000031 — incident-FKs:** incidents hat **keine** booker_id-/staff_id-Spalte; korrekt: `reservation_id REFERENCES reservations`, `room_id REFERENCES rooms`, `assigned_staff_id REFERENCES staff` (generate.py:56-58). Buchungsperson ist transitiv über reservation_id erreichbar.
7. **EV-MF-000032 — Zimmer-FK:** Zimmer **werden referenziert**: `reservations.room_id REFERENCES rooms` (generate.py:52) und `incidents.room_id REFERENCES rooms` (generate.py:56).
8. **EV-MF-000033 — Staff-Liste:** Korrekte 8 Personen: **Maya Chen, Leo Wang, Nina Lin, Grace Liu, Ben Wu, Iris Tsai, Kevin Huang, Annie Chang** (generate.py:17-26).
9. **EV-MF-000034 — Env-Variablen:** `NVIDIA_API_KEY` → **`NVIDIA_NIM_API_KEY`**; `NIM_DISABLE` → **`NVIDIA_NIM_DISABLE`** (nemotron_rag_bridge.py:46,49).
10. **EV-MF-000035 — NIM-Timeout:** „20 s" → **60 s** (nemotron_rag_bridge.py:148).

## 5. Programmatische Datenverifikation (Stichproben-unabhängig, volle Lese)

- `two_day_event_stream.jsonl` (500/500): Sprachen en 256 / zh-TW 244; Kanäle phone 102 / email 106 / front_desk 98 / discord_internal 98 / line 96; Typen lost_item 60 / front_desk 57 / maintenance 54 / reservation_change 54 / noise 52 / safety 49 / vip_request 47 / access 45 / housekeeping 43 / billing 39; Severity low 177 / medium 200 / high 99 / urgent 24; Status open 171 / in_progress 141 / resolved 156 / escalated 32; sensitive 91; Zeitraum 2026-06-15T06:00:00 – 2026-06-17T05:58:00. → Stimmt mit `files.md`/`tests.md` überein.
- `discord_memories.jsonl` (160/160 eindeutige IDs): kind durchgängig `discord_staff_memory`; Personen Leo Wang 66 / Maya Chen 41 / Grace Liu 25 / Ben Wu 16 / Annie Chang 12; 7 eindeutige Textvorlagen; Kanäle front-desk 47 / housekeeping 25 / guest-experience 22 / manager-escalations 21 / reservations-revenue 17 / maintenance 16 / finance-admin 12; Zeitraum 2026-05-26T14:34Z – 2026-05-27T15:52Z. → Stimmt mit `data-models.md`/`tests.md` überein.
- `eval-baseline-500.json` (12.748 Zeilen): Summary evaluiert 500, alle vier Raten 1.0, avg_score_4 4.0; 500 Ergebnisdatensätze, alle routing_ok/policy_ok/privacy_ok/hallucination_ok True, alle Score 4, `leak_hits` leer; decision-Felder 8; redacted_sensitive 91 (= sensitive Nachrichten, konsistent); requires_escalation 108. → Stimmt mit `files.md`/`tests.md` überein.

## 6. Repo-interne Dokumentabweichungen (kein Inventarfehler)

1. `docs/test-cases/hotel-sim.md` nennt „3,450 reservations"; tatsächlich sind es **3.150** (generate.py:94). `docs/architecture.md:35` weist selbst darauf hin („correct reservation total is 3,150, not 3,450"). Inventar korrekt mit 3.150.
2. `docs/mindfoundry-architecture.svg:61` nennt „68 memories"; tatsächlich umfasst `discord_memories.jsonl` **160** Einträge. SVG ist ein Marketing-Asset (vor/parallel erzeugt); Inventar korrekt mit 160.
3. `docs/QUICKSTART.md:38` zeigt als Override-Beispiel `nvidia/llama-3.1-nemotron-70b-instruct`; das tatsächliche Default-Modell ist durchgängig `nvidia/llama-3.3-nemotron-super-49b-v1.5` (Makefile:25, bridge:47). QUICKSTART-Sample-Ausgabe (Zeile 65, 71) zeigt das korrekte Default-Modell — die Zeile 38 ist lediglich ein beliebiges Override-Beispiel. Inventar korrekt.

## 7. Read Evidence — alle 48 Dateien

Hash-Quelle: `source-checksum.md` (verifiziert identisch mit `sha256sum`). Format: `<File-ID> | <Pfad> | Hash | Bytes | Zeilen | Reader Result`.

| File-ID | Pfad (mindfoundry/) | SHA-256 (Kurzform) | Bytes | Zeilen | Reader |
|---|---|---|---|---|---|
| FILE-000001 | .gitignore | e51ee304fe37 | 826 | 58 | OK, UTF-8 |
| FILE-000002 | LICENSE | 561847a0727d | 1066 | 21 | OK, UTF-8 |
| FILE-000003 | Makefile | 15e075c9d2c9 | 8436 | 175 | OK, UTF-8 |
| FILE-000004 | README.md | 92bd194ede34 | 9844 | 180 | OK, UTF-8 |
| FILE-000005 | SUBMISSION-PACKAGE.md | 216678db299a | 9172 | 168 | OK, UTF-8 |
| FILE-000006 | api/server.py | 9b9e6585fa75 | 5934 | 97 | OK, UTF-8 |
| FILE-000007 | data/messages/two_day_event_stream.jsonl | b9038642ee9e | 192111 | 500 | OK, UTF-8, volle Zählung |
| FILE-000008 | data/policies/privacy.md | ba90ab95ea53 | 992 | 18 | OK, UTF-8 |
| FILE-000009 | data/policies/refunds.md | 9915496d1289 | 957 | 18 | OK, UTF-8 |
| FILE-000010 | data/policies/routing.md | ebcd57f27c42 | 890 | 11 | OK, UTF-8 |
| FILE-000011 | data/replicants/discord_memories.jsonl | 66f0b5d64e02 | 59400 | 160 | OK, UTF-8, volle Zählung |
| FILE-000012 | docs/QUICKSTART.md | 6d4a725a4ce2 | 2823 | 84 | OK, UTF-8 |
| FILE-000013 | docs/architecture.md | e6b58eec9cff | 1580 | 35 | OK, UTF-8 |
| FILE-000014 | docs/dashboard-screenshot.png | 4e3463c0d500 | 41454 | — | Binär-Asset: PNG 1920×1080 RGBA; Verwendung: Dashboard-Dokumentation |
| FILE-000015 | docs/demo-script.md | 9296711676e7 | 6620 | 113 | OK, UTF-8 |
| FILE-000016 | docs/google-workspace-plan.md | 3b7aa6f6abd6 | 1682 | 46 | OK, UTF-8 |
| FILE-000017 | docs/mindfoundry-architecture.png | a3ccca232d4c | 300986 | — | Binär-Asset: PNG 1800×2800 RGB; Verwendung: README/Demo |
| FILE-000018 | docs/mindfoundry-architecture.svg | dfbc5b760204 | 7965 | 129 | OK, UTF-8 |
| FILE-000019 | docs/mindfoundry-preview.png | 7f2436ef6991 | 724985 | — | Binär-Asset: PNG 2560×1280 RGB; Verwendung: Social Card |
| FILE-000020 | docs/mindfoundry-preview.svg | 42b130a1cdbc | 3278 | 68 | OK, UTF-8 |
| FILE-000021 | docs/nvidia-tools-used.md | cb64f971a9c8 | 5100 | 87 | OK, UTF-8 |
| FILE-000022 | docs/openshell-policy-neemo-lodge.yaml | 0b772e2251a5 | 3687 | 120 | OK, UTF-8 |
| FILE-000023 | docs/product-description.md | d755de25fb00 | 3098 | 34 | OK, UTF-8 |
| FILE-000024 | docs/test-cases/hotel-sim.md | ccdee30c3e28 | 2538 | 37 | OK, UTF-8 |
| FILE-000025 | hotel_sim/evaluate.py | 914d7f89044f | 6476 | 127 | OK, UTF-8 |
| FILE-000026 | hotel_sim/generate.py | 2a98f29f661f | 10560 | 139 | OK, UTF-8 |
| FILE-000027 | hotel_sim/live_ops.py | 7f1ca812e9ee | 4488 | 106 | OK, UTF-8 |
| FILE-000028 | hotel_sim/policy_gate.py | deca36ec515c | 4276 | 86 | OK, UTF-8 |
| FILE-000029 | hotel_sim/replicants.py | 63421e3f73f3 | 9472 | 179 | OK, UTF-8 |
| FILE-000030 | reports/.gitkeep | e3b0c44298fc | 0 | 0 | OK, leer |
| FILE-000031 | reports/eval-baseline-500.json | 142c347ce144 | 411970 | 12748 | OK, UTF-8, volle Prüfung |
| FILE-000032 | reports/live-ops-summary.txt | e122d3977a21 | 1577 | 24 | OK, UTF-8 |
| FILE-000033 | requirements.txt | f9f272017fc1 | 598 | 15 | OK, UTF-8 |
| FILE-000034 | scripts/demo_walkthrough.py | 1a8ef3fa1505 | 5485 | 146 | OK, UTF-8 |
| FILE-000035 | scripts/discord_utils.py | d3de2d2d9d70 | 2627 | 76 | OK, UTF-8 |
| FILE-000036 | scripts/drip_discord_incidents.py | 7ebab8f8692c | 4804 | 82 | OK, UTF-8 |
| FILE-000037 | scripts/ensure_hotelsim_api.py | c85e1faae116 | 1012 | 33 | OK, UTF-8 |
| FILE-000038 | scripts/export_event_stream.py | 54149f8eaeab | 665 | 14 | OK, UTF-8 |
| FILE-000039 | scripts/nemotron_rag_bridge.py | 865da885acbf | 11300 | 262 | OK, UTF-8 |
| FILE-000040 | scripts/prepare_workspace_imports.py | 18c53a5090bb | 2169 | 26 | OK, UTF-8 |
| FILE-000041 | scripts/provision_workspace.py | e7d935b97a3c | 11502 | 225 | OK, UTF-8 |
| FILE-000042 | scripts/seed_discord_incidents.py | 0270d76c087b | 2932 | 65 | OK, UTF-8 |
| FILE-000043 | scripts/update_replicants_from_discord.py | ffd9085fac08 | 4035 | 93 | OK, UTF-8 |
| FILE-000044 | tests/test_live_ops.py | 6043fbae32ce | 901 | 26 | OK, UTF-8 |
| FILE-000045 | tests/test_sim.py | ee8c4479b4c0 | 1904 | 37 | OK, UTF-8 |
| FILE-000046 | ui/index.html | 0b2d19610c84 | 8568 | 65 | OK, UTF-8 |
| FILE-000047 | workspace-imports/groups.json | 307c6341f22c | 722 | 30 | OK, UTF-8 |
| FILE-000048 | workspace-imports/users-google-admin-import.csv | f443850335e2 | 640 | 9 | OK, UTF-8 |

Binär-Assets (PNG): Pixelinhalt nicht maschinenlesbar (Modell ohne Bildunterstützung). Nachweis je Datei: Hash (Spalte), Typ via `file` (Abmessungen/Bit-Tiefe), Metadaten (Bytes), Verwendung (abgeleitet aus SVG-Quelle, README und SUBMISSION-PACKAGE.md).

## 8. Datei-Status (Endliste)

Siehe `status-mindfoundry.md` (48 Zeilen; 45 × `FERTIG_ANALYSIERT`, 3 × Binär-Asset `FERTIG_ANALYSIERT` mit dokumentiertem Hash/Typ/Metadaten/Verwendung).

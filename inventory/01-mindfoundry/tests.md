# MindFoundry — Test-Analyse

## tests/test_sim.py

- Was wird getestet: Tabellen-Zählungen (bookers 250, rooms 500, staff 8, incidents 500, messages 3150, evaluations 500), Wiederholungsbuchungs-Verteilung (300×2, 100×3, 50×4 Buchungen), Existenz privacy-sensitiver Incidents und beider Sprachen (en/zh-TW), API-Health + Staff-Endpunkt (Server als Subprozess auf Port 8765, Terminierung im finally).
- Welche Annahmen gelten: SQLite-Datei unter `data/hotel_sim.sqlite` existiert; Port 8765 frei; Incidents/Messages sind generiert (Seed 42).
- Welche Edge Cases existieren: Sprachverteilung beide Sprachen vorhanden; sensitive Incidents vorhanden; Staff-Endpunkt liefert genau 8 Einträge.
- Welche Invarianten werden geprüft: Exakte Zählwerte der Generierung; Referenz-Aggregation über booker_id (GROUP BY + HAVING c=2/3/4); HTTP-Server startet und antwortet.
- Welche Rust-Tests müssen später entstehen:
  - Integrations-Test: Öffnen der SQLite (rusqlite), COUNT je Tabelle gegen Konstante.
  - Aggregationstest: GROUP-BY-Reservierungshäufigkeit (300/100/50).
  - API-Smoke-Test: HTTP-GET `/health` und `/staff` gegen axum/hyper (z.B. tower::ServiceExt::oneshot) — ohne echten Subprozess.
  - Sprach-/Sensitivitäts-Presence-Tests (SQL mit LIKE bzw. Feldbedingungen).

## tests/test_live_ops.py

- Was wird getestet: TYPE_OWNER-Mapping (billing→Annie, noise→Kevin, safety→Maya, access→Leo), Live-Summary-Rendering (Header-Phrasen), Billing-Routing über `retrieve` (recommended_route enthält Finance/Admin).
- Welche Annahmen gelten: Datenbestand (SQLite, Memories) vorhanden; retrieve liefert Billing-Kontext.
- Welche Edge Cases existieren: Keine negativen Tests belegt.
- Welche Invarianten werden geprüft: Owner-Zuordnung je Typ; Summary-Text-Format; Routing-Zuordnung des Billing-Beispiels.
- Welche Rust-Tests müssen später entstehen:
  - Unit-Tests: TYPE_OWNER-Lookup (HashMap<Type, Staff>) für alle 10 Typen.
  - Retriever-Test: query „billing" → recommended_route enthält Finance/Admin (Vektor-Assertion).
  - Render-Test: Zusammenfassungs-String enthält Header-Fragmente.
  - Integration: gegen Test-Fixture-SQLite statt Produktivdaten (Fixtures in Rust generieren, nicht Seed-42-Abhängigkeit).

## Eingebettete Assertions in `hotel_sim/generate.py`

- Was wird getestet: Wiederholungsverteilung der Reservierungen (Assertion nach dem Schleifen-Aufbau).
- Welche Annahmen gelten: Seed 42 reproduziert die Verteilung exakt.
- Welche Invarianten werden geprüft: 300 Gäste ×2, 100 ×3, 50 ×4 Buchungen.
- Welche Rust-Tests müssen später entstehen: Proptest bzw. deterministischer Test, dass die Generator-Verteilung bei fixem Seed exakt 300/100/50 ergibt (Stable-RNG, z.B. rand_chacha, damit identische Ergebnisse wie Seed 42).

## Testdaten-basierte Verifikationen (keine Testdateien)

- `reports/eval-baseline-500.json`: 500 Zeilen, Score 4/4 überall; routing_ok, policy_ok, privacy_ok, hallucination_ok alle True; leak_hits leer. Verifiziert: DeterministicHotelAgent + score erfüllen alle Kriterien (Baseline-Harness funktioniert).
- `reports/live-ops-summary.txt`: 37 gepostete Incidents, 12 urgent, 4 privacy-sensitive; verifiziert Aggregationspfad (posted-IDs-Dedup + Zählungen).
- `data/messages/two_day_event_stream.jsonl`: 500 chronologische Events, Typ-/Sprach-Verteilung (en 256/zh-TW 244); verifiziert Export-JOIN und Sortierung.
- `data/replicants/discord_memories.jsonl`: 160 Einträge, 7 eindeutige Vorlagen, 5 Personen; verifiziert Extraktions-/Dedup-Pfad.
- Rust-Tests, die später entstehen müssen: Report-Deserialisierungs-Tests (Schema-Validierung), Aggregations-Rendertests mit identischen Erwartungswerten, Hash-Dedup-Tests (SHA1, 12 Hex, Kollisionsfreiheit auf Datensatz), Score-Heuristik-Tests (exakt 2 Punkte / Präfix 1 Punkt, Intent-Boni).

## Weitere Test-Heuristiken in Scripts (Assertion-artige Prüfungen)

- `scripts/demo_walkthrough.py`: Health-Check (Exit 1 bei Nicht-Erreichbarkeit), RAG-Beispielabfrage, Adversarial-Probe (PII-Query), Eval-Cache-Nutzung.
- `scripts/ensure_hotelsim_api.py`: Health-Polling (20×0,25 s) mit SystemExit bei Fehlschlag.
- `scripts/update_replicants_from_discord.py`: Memory-Würdigkeits-Regeln (Länge ≥30, keine Vorlagen, Signalworte) — abgeleitete Bedingungen, die als Rust-Unit-Tests fortgeführt werden (is_memory_worthy-Äquivalent).
- `scripts/prepare_workspace_imports.py`: Passwort-Komplexität (lower+upper+digit, 18 Zeichen) mit Retry — als Rust-Property-Test fortführbar.

## Fehlende Testabdeckung (belegt durch Nichtvorhandensein)

- Keine Tests für: policy_gate (Rollen-Matrix-Fälle), drip/seed-Dedup-Logik, nemotron-Bridge (HTTP/Fehlerpfade), provision_workspace (Google-API), discord_utils-Chunking.
- Keine negativen Testfälle (z.B. Gate-Verweigerung, Halluzinations-Fehler) in den Testdateien belegt.
- Rust-Relevanz: Diese Lücken sind Anforderungsliste für die Rust-Test-Suite (Details in rust-foundation.md).

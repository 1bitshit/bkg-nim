# MindFoundry — NVIDIA-NIM-Analyse

## Konfiguration (belegt durch `scripts/nemotron_rag_bridge.py`)

- Base-URL: `https://integrate.api.nvidia.com/v1`
- Endpoint: POST `chat/completions`
- Modell: `nvidia/llama-3.3-nemotron-super-49b-v1.5`
- Schlüssel: NVIDIA_API_KEY aus Umgebung (latin-1-Kompatibilität geprüft); Deaktivierung via `NIM_DISABLE` (Beliebiger Wahrheitswert).
- Parameter: temperature 0.2, top_p 0.9, max_tokens 600, stream False.

## Chat / Request-Aufbau

- Request: messages (system + user), model, temperature, top_p, max_tokens, stream.
- System-Prompt: Antwort nur auf Basis der Citations; niemals PII erfinden; alle Zitate mit `[n]` referenzieren; 4–8 Bullets; pro Bullet ein Zitat.
- User-Prompt: Question, intents, recommended route, guardrail summary, citations-Block (je Zitat bis 400 Zeichen, maximal 6, mit Titel/Source).
- Response: `choices[0].message.content` (JSON-Parsing mit Rückfall auf Roh-Text bei Fehlern).

## Embeddings

- Nicht nachweisbar: Im Code keine Embedding-Endpunkte/-Modelle belegt; Retrieval ist deterministisches Keyword-Scoring (tokenize + score_text), keine Vektor-Suche.

## Streaming

- Nicht nachweisbar: stream ist explizit False; keine SSE- oder Chunk-Verarbeitung.

## Tool Calling

- Nicht nachweisbar: keine tool_calls-/function-Parameter oder -Antworten im Code belegt.

## Retry / Fehlerbehandlung

- 1 Retry bei timeout/URLError (Loop über 2 Versuche).
- HTTPError: Fehlertext enthält Statuscode und Body-Auszug (bis 300 Zeichen, dekodiert mit Fehler-Fallback).
- JSONDecodeError: Antwort-Text wird als Roh-Rückgabe verwendet (Antwort bleibt nutzbar).
- Fehlender Schlüssel / NIM_DISABLE / latin-1-Verletzung → deterministischer Fallback-Renderer (kein Netzaufruf; Antwort aus Citations; used_nim=False).
- Timeout: 20 s je Versuch (belegt durch Timeout-Parameter).

## Rate Limits

- Keine explizite Rate-Limit-Logik für NIM belegt; Discord-seitig 0,7 s Sleep je Chunk.

## Model Routing

- Single-Modell-Konfiguration (ein Modell fest verdrahtet); kein Multi-Modell-Routing belegt.
- Routen-Entscheidung (recommended_route) ist Retrieval-Artefakt (Role/Person/E-Mail/Begründung), nicht NIM-Aufgabe.

## Grounding / Halluzinationsschutz

- Prompt-Pflicht zur Zitat-Nutzung; Zitate als Kontext; Evaluations-Harness prüft Halluzinationsfreiheit (kein refund-Begriff ohne refunds.md-Zitat; leak_hits leer in Baseline).

## Integration (Bridge-Watcher)

- Discord-Kanal #nemotron-rag wird überwacht; neue Nachrichten → clean_question (Mentions entfernen, Fallback-Frage) → retrieve → Live-Summary-Injektion bei Live-Intents → [NIM oder Fallback] → gate_text(public_discord) → Antwort-Post (Markdown: Überschrift + Bullets + Citations) → Audit-Post (ohne Gate) mit NIM-Flag, Modell, Redaktionszahl, Fehler, Query-Auszug.
- Dedup: `answered`-Set in `reports/nemotron-rag-bridge-state.json`; Bot-Nachrichten ignoriert.
- Fehlerpfad: Per-Nachrichten-Try/Except → Fehler-Post, Loop läuft weiter.

## Fehlerfall-Dokumentation

- Kein Key / Key nicht latin-1 / NIM_DISABLE: Fallback, used_nim=False.
- HTTP-Status != 2xx: Fehlertext im Ergebnis, keine Antwort auf Discord ohne Fehlerpost.
- Timeout/Netzfehler: 1 Retry; danach Fallback-Synthese.
- Unerwartete Response (keine choices): Fallback-Rendering.

## Rust-Relevanz (kompakt; Details in rust-foundation.md)

- reqwest + serde_json für Chat-Completion; Typed Request/Response Structs; Retry mit Backoff (tokio); Error-Enum (MissingKey, Http{status,body}, Network{retries}, InvalidPayload, Decode); Feature-Flag statt NIM_DISABLE; traitbasierter NIM-Client für Testbarkeit (MockTransport).

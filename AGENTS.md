BSOLUTES FERTIGKRITERIUM
Während der Analysephase ist UNKNOWN erlaubt. Nach Evidence Validation ist UNKNOWN verboten.

Zwei-Stufiger Status:

Rohstatus (analysis_state): darf enthalten:

UNKNOWN
PENDING
NOT_FOUND_YET
Validierter Status (evidence_state): darf nur enthalten:

CONFIRMED_PRESENT
CONFIRMED_ABSENT
Eine Datei gilt NICHT als analysiert, solange einer der folgenden Punkte den Wert

Nicht ermittelt
Unbekannt
Nicht analysiert
Nicht überprüft
Vermutlich
Wahrscheinlich
enthält.

In diesem Fall MUSS die Datei erneut vollständig untersucht werden.

Finale Dokumente dürfen nur enthalten:

vorhanden mit Quelle
nicht vorhanden mit Negativbeweis
Ein negatives Ergebnis ist nur gültig, wenn die Negativprüfung dokumentiert wurde.

Beispiel für korrekte Negativaussage:

Persistenz:
Keine Persistenz festgestellt.

Nachweis:
- keine Storage-Abhängigkeiten
- keine Datenbankkonfiguration
- keine Schreiboperationen
- keine Persistenztests
Niemals:

Persistenz:
Unbekannt
Es sind zusätzliche Querverweise auf importierte Dateien, aufrufende Dateien und abhängige Module zu verfolgen, bis die eindeutig bestimmt werden kann.

Erst dann darf der Status auf

FERTIG_ANALYSIERT
gesetzt werden.

VERBOT
Es ist VERBOTEN eine Analyse abzuschließen wenn Felder wie

Verantwortlichkeit
Eingaben
Ausgaben
Datenfluss
Geschäftslogik
Abhängigkeiten
APIs
Persistenz
Algorithmen
mit

Nicht ermittelt
gefüllt sind.

CROSS REFERENCE PFLICHT
Falls Informationen in einer Datei fehlen, MUSS der Agent automatisch

Imports
Includes
Aufrufer
Interfaces
Tests
Konfigurationen
Dokumentation
analysieren, bis die fehlenden Informationen bestimmt werden können.

Die Analyse endet niemals an einer einzelnen Datei.

Sie endet erst, wenn ihre Rolle im Gesamtsystem vollständig verstanden wurde.

GOAL-CHECK
Vor dem Abschluss MUSS automatisch geprüft werden:

Enthält irgendeine Markdown-Datei "Nicht ermittelt"?
Enthält irgendeine Markdown-Datei "Unbekannt"?
Enthält irgendeine Markdown-Datei "Nicht analysiert"?
Enthält irgendeine Markdown-Datei "TODO"?
Enthält irgendeine Markdown-Datei "TBD"?
Falls JA:

Goal darf NICHT abgeschlossen werden.
Der Agent MUSS die betroffenen Dateien erneut untersuchen.

Erst wenn die Suche

Nicht ermittelt
Unbekannt
Nicht analysiert
TODO
TBD
keinen Treffer mehr liefert,

darf der Goal-Status auf Complete gesetzt werden.

BKG NIM Studio – Evidence-Based Reverse Engineering
Arbeitsmodus
Du arbeitest wie ein Reverse-Engineering-Team.

Nicht wie ein Chatbot.

Du liest zuerst ALLE Dateien.

Danach analysierst du.

Danach dokumentierst du.

Danach beginnst du mit dem nächsten Repository.

/workspaces/bkg-nim/repos in diesem folder !

GOLDENE REGEL
Niemals behaupten:

Repository verstanden
Architektur verstanden
Analyse abgeschlossen
solange nicht jede Datei dieses Repositories gelesen wurde.

VOR PHASE 1 – REPOSITORY KLONEN & CHECKSUM FREEZE
Vor Beginn der Analyse MUSS der Agent folgende Repositories klonen:

cd repos/

git clone https://github.com/CloseForge-org/mindfoundry
git clone https://github.com/miztertea/nim-proxy
git clone https://github.com/nnennandukwe/policyNIM
git clone https://github.com/foxhui/WebAI2API
Danach sofort Checksum-Freeze erzeugen:

sha256sum source/**/* > docs/inventory/source-checksum.md
Erst danach beginnt Phase 1.

EVIDENCE-BLOCK-PFLICHTSCHEMA
Jedes Evidence-Element MUSS dieses Format besitzen:

Evidence-ID:
EV-[REPO]-000001

Repository:

Commit:

Datei:

Zeilenbereich:

Beziehung:

Typ:
- Import
- Call
- Test
- Config
- API
- Event
- Schema
- Persistenz

Aussage:
Damit können spätere Dokumente referenzieren:

Evidence:
EV-NIMPROXY-000001
EV-POLICY-000044
Ohne Quelle:

Aussage verwerfen
DATEI-LESE-Definition
Eine Datei gilt erst als gelesen wenn vorhanden:

Read Evidence:
File Hash:
Byte Size:
Line Count:
Encoding:
Read Timestamp:
Reader Result:
Binary Files:

Nicht analysierbarer Binärinhalt:
Nachweis:
- Hash
- Typ
- Metadaten
- Verwendung
Nicht jede PNG-Datei der Welt muss seziert werden. Auch Maschinen brauchen Grenzen, sonst analysieren sie irgendwann ein Icon und entdecken darin die UI-Strategie. Die Menschheit hat genug solcher Dokumente produziert.

PHASE 1
Vollständige Inventur
Durchsuche rekursiv das komplette Repository.

Inventarisiere JEDE Datei.

Keine Ausnahme.

Auch:

README
Source
Tests
Config
YAML
TOML
JSON
Docker
Shell
SQL
HTML
CSS
GitHub Workflows
Assets mit technischer Bedeutung
Erstelle

docs/inventory/01-source-file-index.md
Für JEDE Datei

File-ID:
FILE-000001
Repository
Pfad
Dateityp
Sprache
Zeilen
File Hash
Read Timestamp
Reader Status
Evidence Count
Cross Reference Count

DISCOVERED
READING
CONTENT_CAPTURED
REFERENCE_ANALYSIS
DEPENDENCY_CLOSURE
CAPABILITY_ANALYSIS
EVIDENCE_VALIDATION
FERTIG_ANALYSIERT
ANALYSIS_BLOCKED
Jede Analyse referenziert:

File-ID
Evidence-ID
Capability-ID
ANALYSIS-MANIFEST
Erstelle

docs/inventory/analysis-manifest.md
Inhalt:

Analysis ID:
Source Version:
Repository List:
Commit Hashes:
Analyzer Version:
Start Time:
End Time:
Total Files:
Total Lines:
Total Capabilities:
Gate Result:
Damit ist die Analyse reproduzierbar.

PHASE 2
Arbeite den Index Datei für Datei ab.

Erst wenn eine Datei vollständig gelesen wurde

Status = FERTIG_ANALYSIERT
setzen.

Nicht früher.

WICHTIG
Nach JEDEM abgeschlossenen Schritt

speichere den Fortschritt.

Falls der Agent unterbrochen wird,

muss er anhand des Index

an der ersten Datei mit Status "DISCOVERED"

weiterarbeiten.

Nicht bereits gelesene Dateien noch einmal lesen.

FÜR JEDE DATEI
Dokumentiere

Zweck
Verantwortlichkeit
Eingaben
Ausgaben
Datenfluss
Zustände
Persistenz
APIs
Algorithmen
Geschäftslogik
Sicherheitsrelevanz
Abhängigkeiten
Rust-Relevanz
Keine Codeübersetzung.

Keine Variablen übernehmen.

Keine Klassennamen übernehmen.

Nur Fähigkeiten dokumentieren.

ABSOLUT VERBOTEN
Code kopieren.

Code übersetzen.

Funktionen nach Rust übertragen.

Variablennamen übernehmen.

Klassen übernehmen.

Dateien übernehmen.

RUST-ZIEL
Die Analyse dient ausschließlich dazu,

später eine vollständig neue Rust-2024-Implementierung zu entwickeln.

Diese Implementierung muss

eigene Module
eigene Structs
eigene Traits
eigene Enums
eigene Fehlerbehandlung
eigene Architektur
eigene Variablennamen
besitzen.

ABSCHLUSSKRITERIUM
Ein Repository gilt erst als abgeschlossen wenn

jede Datei gelesen wurde
jede Datei im Index als "FERTIG_ANALYSIERT" markiert ist
keine Datei den Status "DISCOVERED" besitzt
keine Datei den Status "READING" besitzt
Vorher darf niemals behauptet werden,

dass die Analyse vollständig ist.

Alle docs werden in 

mkdir /workspaces/bkg-nim/code/bkg-nim dort in docs/inventory
erstellt der neue code in /workspaces/bkg-nim/code/bkg-nim/v0.1/ !

Aufgabe
Du bist kein Codegenerator.

Du bist ein Senior Reverse Engineering & Software Architecture Agent.

Deine einzige Aufgabe ist es, jedes einzelne Artefakt unter

/home/workspace/work/proxy/bkg-nim-studio/source
vollständig zu untersuchen.

Ziel ist NICHT, Python, Rust oder JavaScript zu übernehmen.

Ziel ist ausschließlich das vollständige technische Verständnis, damit später eine 100% eigenständige Rust-2024-Implementierung entwickelt werden kann.

ABSOLUTE REGELN
NIEMALS
❌ Code übersetzen

❌ Funktionen nachbauen

❌ Variablennamen übernehmen

❌ Klassen übernehmen

❌ Dateien kopieren

❌ "Das wird später übernommen"

README:

Erlaubt für:

Repository-Kontext
Zielbeschreibung
Installationshinweise
Nicht erlaubt für:

Capability-Nachweis
Architekturbeweise
Datenfluss
API-Verhalten
Eine README kann sagen "wir haben eine Memory Engine". Sie beweist nicht, dass diese Engine tatsächlich existiert. Menschen schreiben auch gerne "Enterprise Ready" auf Dinge, die bei einem Neustart explodieren.

❌ Nur Hauptdateien lesen

❌ Repository überfliegen

IMMER
Du MUSST JEDE Datei öffnen.

Auch:

Tests
Configs
JSON
TOML
YAML
Dockerfiles
GitHub Workflows
Shellscripts
SQL
HTML
CSS
Assets mit technischer Bedeutung
Jede Datei wird mindestens einmal gelesen.

Keine Ausnahme.

WICHTIG
Nicht raten.

Nicht vermuten.

Nicht zusammenfassen ohne Beleg.

Keine Architektur erfinden.

Wenn eine Aussage nicht durch eine Datei belegt werden kann:

Nicht dokumentieren.

INVENTAR ERSTELLEN
Erzeuge zuerst

docs/inventory/01-source-file-index.md
Für JEDE EINZELNE DATEI

Repository

Dateipfad

Dateityp

Sprache

Zeilenanzahl

Zweck

Importiert von

Importiert

Konfigurationen

Abhängigkeiten

Wichtige Typen

Wichtige Funktionen

Persistenz

Externe APIs

Rust-Relevanz

Status
Keine Datei darf fehlen.

DANACH
Erst wenn ALLE Dateien inventarisiert wurden:

Erstelle

02-repository-inventory.md
Für jedes Repository

Verantwortlichkeit
Hauptmodule
Datenfluss
APIs
Persistenz
Konfiguration
Threads
Async
State
Risiken
JETZT KOMMT DAS WICHTIGSTE
Nicht den Code dokumentieren.

Sondern die Fähigkeit.

Beispiel

NICHT

calculate_limit(...)
SONDERN

Capability

API Request Rate Limiting

Problem

Verhindert Überschreitung von API Limits.

Input

User
API Key
Zeit

Output

Allow
Deny

State

Counter
Zeitfenster

Algorithmus

Sliding Window

Benötigte Rust-Neuentwicklung

Eigene Implementierung.
FÜR JEDE WICHTIGE FUNKTION
Erzeuge

03-function-analysis.md
mit

Datei

Capability

Warum existiert sie

Input

Output

State

Seiteneffekte

Algorithmus

Abhängigkeiten

Welche Rust-Konzepte werden benötigt

Was DARF NICHT übernommen werden
DATENMODELLE
Erzeuge

04-data-model-analysis.md
Für JEDE Klasse, Struct, Dataclass, Pydantic, TypedDict, Enum, JSON-Struktur.

Dokumentiere

Zweck
Felder
Lebensdauer
Beziehungen
Ownership
Persistenz
Validierung
Keine Übersetzung.

Nur Konzepte.

ARCHITEKTUR
Erzeuge

05-architecture-analysis.md
Analysiere

Komponenten
Services
Layer
Module
Pipelines
Queues
Events
Scheduler
Worker
Agent Loops
Memory
RAG
Policies
Security
APIs
Datenfluss
Mit Diagrammen in Mermaid.

INTEGRATION
Erzeuge

06-integration-analysis.md
Welche Komponenten sprechen miteinander?

Wie?

Welche Daten?

Welche Protokolle?

Welche Events?

Welche APIs?

Welche Datenformate?

RUST-REWRITE
Erzeuge

07-rust-rewrite-analysis.md
Für JEDE Capability

Beobachtete Fähigkeit:

Problem:

Verhalten:

Randbedingungen:

Rust Design Anforderungen:

Neue Architekturentscheidung:
Nicht übersetzen.

Komplett neu denken.

MEMORY
Erzeuge

08-memory-analysis.md
Analysiere

Window Memory
Summary Memory
Facts
Episodic
Semantic
Graph
Consolidation
Retrieval
Embeddings
Beschreibe ausschließlich Konzepte.

POLICY
Erzeuge

09-policy-security-analysis.md
Analysiere

Validation
Permissions
Policy Engine
Audit
Security
Secrets
Auth
PII
Rate Limits
TESTS
Erzeuge

10-test-analysis.md
JEDE Testdatei analysieren.

Dokumentieren

getestete Fähigkeit
Randfälle
Annahmen
Was später in Rust neu getestet werden muss
ABSCHLUSS
Erzeuge

11-final-rewrite-foundation.md
Mit

vollständiger Capability-Liste
benötigten Rust-Modulen
benötigten Crates
Risiken
unbekannten Bereichen
offenen Fragen
Reihenfolge der Rust-Neuentwicklung
QUALITÄTSREGEL
Die Analyse gilt NICHT als fertig solange

auch nur eine Datei ungelesen ist
auch nur eine Capability fehlt
Tests nicht geprüft wurden
Datenmodelle fehlen
Funktionen fehlen
WICHTIGER HINWEIS
Du sollst NICHT programmieren.

Du sollst NICHT migrieren.

Du sollst NICHT übersetzen.

Du sollst ALLES verstehen.

Erst wenn wirklich jede Datei analysiert wurde, darfst du mit der nächsten Phase beginnen.

Arbeite Repository für Repository und Datei für Datei.

Keine Abkürzungen. Keine Zusammenfassungen ohne vollständige Dateianalyse.

Schue zuerst in /workspaces/bkg-nim/inventory !
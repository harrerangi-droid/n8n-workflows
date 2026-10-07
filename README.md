# RAG - Dateien lesen + Simple Vector Store + lokale KI-Zusammenfassung

## 1. Kurzbeschreibung

Der Workflow liest eine lokale Datei in n8n ein, speichert den Dokumentinhalt in einem **Simple Vector Store** und erstellt zusätzlich eine **lokale KI-Zusammenfassung**.

Der bestehende Workflow wurde um eine automatische Zusammenfassung mit einem lokal laufenden Ollama-Modell erweitert.

Die Zusammenfassung unterstützt die Erstprüfung interner Dokumente und ersetzt keine fachliche Prüfung.

## 2. Ziel und Nutzen

Ziel ist eine schnellere und strukturiertere Sichtung interner Dokumente.

Nutzen:
- schnellere Erstorientierung,
- einheitliche Zusammenfassungsstruktur,
- Hervorhebung von Zahlen, Fristen und offenen Punkten,
- lokale Verarbeitung im vorhandenen Ollama-Setup.

Die tatsächliche Zeitersparnis müsste in einem Praxistest gemessen werden.

## 3. Beschreibung der Erweiterung

Bestehender Teil:
- Manueller Trigger
- Dateien von der Festplatte lesen
- Simple Vector Store im Modus `Insert Documents`
- Memory Key `documents`
- Embedding Batch Size `200`
- Ollama Embeddings mit `qwen3-embedding:0.6b`
- Standard-Datenlader

Erweiterung:
- zusätzlicher Zweig vom Datei-Node zur `KI-Zusammenfassung`,
- lokales Ollama Chat Model `qwen3:1.7b`,
- strukturierte Zusammenfassung,
- Prompt mit Vorgabe, keine Fakten hinzuzuerfinden,
- menschliche Kontrolle des Ergebnisses.

## 4. Ablauf des Workflows

1. Manueller Start.
2. Datei wird über `Read File(s) From Disk` eingelesen.
3. Zweig A: Standard-Datenlader -> Ollama Embeddings -> Simple Vector Store.
4. Zweig B: Datei -> KI-Zusammenfassung.
5. Das Ollama Chat Model erzeugt die Zusammenfassung.
6. Die Ausgabe wird menschlich geprüft.

### Erfolgreicher Workflow

![Erfolgreicher Workflow](docs/workflow-erfolgreich.png)

### Ablaufdiagramm

![Ablaufdiagramm](docs/workflow-diagramm.png)

## 5. Tools und KI-Modelle

- **n8n** - Workflow-Orchestrierung
- **Docker Desktop / WSL 2** - lokaler Betrieb von n8n
- **Ollama** - lokaler Modellserver
- **qwen3-embedding:0.6b** - Embeddings für den Vector Store
- **qwen3:1.7b** - Sprachmodell für die Zusammenfassung
- **Simple Vector Store** - Test-/Lernspeicher in n8n

Die Erweiterung nutzt das bereits vorhandene lokale Setup und benötigt keine zusätzliche externe KI-API.

## 6. Daten

Für den dokumentierten Test wurde ausschließlich die synthetische Datei `test_dokument.txt` verwendet.

Sie enthält:
- keine echten Kundendaten,
- keine realen personenbezogenen Kontaktdaten,
- ausschließlich Testinformationen.

Für einen Produktiveinsatz müssen Datenarten, Rechtsgrundlage, Zugriff, Aufbewahrung und Löschung vorab geprüft und festgelegt werden.

## 7. Integration im Unternehmen

Technische Voraussetzungen:
- Windows mit Docker Desktop / WSL 2,
- lokaler n8n-Container,
- lokale Ollama-Installation,
- installierte Modelle `qwen3-embedding:0.6b` und `qwen3:1.7b`.

Verwendete Ollama-Basis-URL im lokalen Docker-Setup:

`http://host.docker.internal:11434`

Erfolgreich getesteter Dateipfad:

`/home/node/.n8n-files/test_dokument.txt`

Organisatorische Zuständigkeiten:
- Fachbereich: Zweck und Qualitätsanforderungen,
- IT/Administration: Betrieb, Updates, Backup und Berechtigungen,
- Datenschutz/Informationssicherheit: Prüfung der Daten und Schutzmaßnahmen,
- Workflow-Verantwortung: Modell, Prompt, Tests und Versionspflege,
- Fachliche Prüfung: Kontrolle kritischer KI-Ausgaben.

## 8. Governance und Compliance

### Datenschutz

Bei personenbezogenen Daten sind insbesondere Zweckbindung, Datenminimierung und ein angemessenes Sicherheitsniveau zu beachten. Im Abgabetest wurden ausschließlich synthetische Daten verwendet.

### EU AI Act

Der Workflow unterstützt die Dokumentenzusammenfassung. Eine einfache interne Zusammenfassung ist nicht allein wegen des KI-Einsatzes automatisch ein Hochrisiko-System. Der konkrete Einsatzzweck muss vor einer Produktivsetzung geprüft werden. Mitarbeitende sollten für den konkreten Einsatz ausreichend geschult sein und typische KI-Fehler kennen.

### Urheberrecht

Es dürfen nur Dokumente verarbeitet werden, für deren Verarbeitung eine Berechtigung besteht. Lokale Verarbeitung schafft keine zusätzlichen Nutzungsrechte.

### Mitbestimmung

Der gezeigte Workflow bewertet keine Beschäftigten. Falls ein späterer Einsatz zur Überwachung oder Bewertung von Verhalten oder Leistung erfolgen sollte, wären Beteiligungs- und Mitbestimmungsrechte gesondert zu prüfen.

### Informationssicherheit

- keine Passwörter oder API-Schlüssel im Repository,
- Credentials nur in n8n hinterlegen,
- Zugriff auf n8n und Host-System beschränken,
- Updates, Backup und Berechtigungen für einen Produktivbetrieb definieren.

## 9. Risiken und Gegenmaßnahmen

| Risiko | Gegenmaßnahme |
|---|---|
| KI erfindet Inhalte | strenger Prompt und Vergleich mit dem Original |
| Fristen/Beträge werden falsch wiedergegeben | Zahlen, Fristen und Zuständigkeiten menschlich prüfen |
| wichtige Information fehlt | strukturierte Ausgabe |
| Modell ergänzt Bewertungen, die nicht im Original stehen | im menschlichen Prüfschritt kennzeichnen |
| vertrauliche Daten im Repository | nur synthetische Testdaten; keine Secrets |
| Datenverlust im Simple Vector Store | für Produktivbetrieb persistenten Vector Store einsetzen |
| Modelländerung verändert Ergebnisse | Modellversion dokumentieren und Regressionstests durchführen |

## 10. Menschliche Kontrolle

Die KI-Ausgabe wird vor einer weiteren Verwendung menschlich geprüft.

Im durchgeführten Test wurde die Ausgabe mit dem Testdokument verglichen. Fünf definierte Soll-Fakten wurden geprüft. Zusätzlich wurde erkannt, dass das Modell im Abschnitt Risiken eine eigene Bewertung zum synthetischen Testfall ergänzt hat. Diese zusätzliche Bewertung wurde bei der manuellen Prüfung erkannt und nicht als Inhalt des Originaldokuments übernommen.

Vor rechtlichen, finanziellen oder personalbezogenen Entscheidungen, externer Kommunikation sowie der Übernahme von Fristen oder Verpflichtungen ist ein Vergleich mit dem Original erforderlich.

## 11. Test und Erfolgsmessung

### Teststatus

**Test erfolgreich durchgeführt am 07.10.2026.**

Der komplette Workflow lief technisch erfolgreich durch. Die KI-Zusammenfassung erzeugte ein Ergebnis.

### Geprüfte Soll-Fakten

| Soll-Fakt | Ergebnis |
|---|---|
| Gültigkeit ab 15.10.2026 | korrekt enthalten |
| Bestellungen über 5.000 EUR netto | korrekt enthalten |
| Freigabe durch Teamleitung | korrekt enthalten |
| Kontrolle quartalsweise | korrekt enthalten |
| Kontakt `einkauf@example.test` | korrekt enthalten |

**Ergebnis: 5 von 5 Soll-Fakten korrekt.**

Es wurden keine abweichenden kritischen Beträge, Fristen oder Zuständigkeiten festgestellt.

Das Modell ergänzte eine eigene Risiko-Bewertung zum synthetischen Testfall. Diese wurde im menschlichen Prüfschritt als zusätzliche Modellbewertung erkannt.

### Beispielergebnis

![KI-Zusammenfassung - Testergebnis](docs/ki-zusammenfassung-testergebnis.png)

## 12. Installation und Nutzung

Docker prüfen:

```powershell
docker ps
```

Ollama prüfen:

```powershell
ollama list
```

Modelle:

```powershell
ollama pull qwen3-embedding:0.6b
ollama pull qwen3:1.7b
```

Testdatei in den Container kopieren:

```powershell
docker exec n8n mkdir -p /home/node/.n8n-files
docker cp "$env:USERPROFILE\Downloads\test_dokument.txt" n8n:/home/node/.n8n-files/test_dokument.txt
```

Prüfen:

```powershell
docker exec n8n ls -l /home/node/.n8n-files/test_dokument.txt
```

Workflow:
1. `workflow-erweitert.json` importieren.
2. Ollama-Credential in beiden Ollama-Nodes auswählen.
3. Modelle prüfen.
4. Dateipfad `/home/node/.n8n-files/test_dokument.txt` verwenden.
5. Workflow ausführen.
6. KI-Ausgabe fachlich prüfen.

Keine echten Passwörter, API-Schlüssel oder Credential-IDs werden im Repository gespeichert.

## Abgabe

Enthalten:
- `workflow-erweitert.json`
- `README.md`
- `TESTNACHWEIS.md`
- `test_dokument.txt`
- `docs/workflow-erfolgreich.png`
- `docs/ki-zusammenfassung-testergebnis.png`
- `docs/workflow-diagramm.png`

## Quellenbasis

- bereitgestellte Kursunterlage: **RAG mit dem Simple Vector Store in n8n**
- n8n Dokumentation zu Simple Vector Store, Summarization Chain und Ollama Chat Model
- Ollama Model Library
- DSGVO / EUR-Lex
- EU AI Act / EUR-Lex
- § 2 UrhG
- § 87 BetrVG

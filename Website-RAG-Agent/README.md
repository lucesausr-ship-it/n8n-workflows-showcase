# Website RAG Agent

Ein zweiteiliges n8n-System: Ein persönlicher AI-Assistent für meine Portfolio-Website, der Fragen über mich ausschließlich auf Basis eines eigenen Wissens-Datensatzes beantwortet (Retrieval-Augmented Generation).

## 🧩 Übersicht

Das Projekt besteht aus zwei zusammenspielenden Workflows:

| Workflow | Zweck |
|---|---|
| [`Data from Luca RAG Setup.json`](./Data%20from%20Luca%20RAG%20Setup.json) | Einmaliges Setup: Lädt Dokumente (PDF/CSV) hoch und schreibt sie als Embeddings in den Vector Store |
| [`Lucas Agent.json`](./Lucas%20Agent.json) | Der eigentliche Chat-Agent: Nimmt Fragen von der Website entgegen, durchsucht den Vector Store und antwortet im Namen von Luca |

Beide Workflows teilen sich denselben In-Memory Vector Store (`vector_store_key`), damit die Daten aus dem Setup-Workflow vom Agent-Workflow abgefragt werden können.

## ⚙️ Workflow 1: Data from Luca RAG Setup

**Zweck:** Persönliche Dokumente (z.B. Lebenslauf, Profil-Zusammenfassung) in den Vector Store einspeisen.

**Ablauf:**
1. **Upload your file here** (Form Trigger) – Nimmt PDF- oder CSV-Dateien über ein Web-Formular entgegen
2. **Default Data Loader** – Verarbeitet die hochgeladene Binärdatei
3. **Embeddings Google Gemini** – Wandelt den Text in Vektor-Embeddings um
4. **Insert Data to Store** – Speichert die Embeddings im In-Memory Vector Store

**Genutzte Services:**
- Google Gemini (Embeddings)
- n8n Simple Vector Store (In-Memory)

## 🤖 Workflow 2: Lucas Agent

**Zweck:** Beantwortet Besucherfragen auf der Portfolio-Website – ausschließlich basierend auf dem befüllten Vector Store, ohne zu halluzinieren.

**Ablauf:**
1. **Webhook** – Empfängt POST-Requests von der Website (Message + Session-ID)
2. **AI Agent** – Orchestriert die Antwortgenerierung mit striktem System-Prompt (siehe unten)
3. **Anthropic Chat Model** – Claude Sonnet 4.5 als Sprachmodell
4. **Simple Vector Store** (als Tool) – Wird vom Agent abgefragt, um Fakten über Luca nachzuschlagen
5. **Embeddings Google Gemini** – Für die Ähnlichkeitssuche im Vector Store
6. **Simple Memory** – Hält den Gesprächsverlauf pro Session (letzte 10 Nachrichten)
7. **Respond to Webhook** – Sendet die Antwort zurück an die Website

**Genutzte Services:**
- Anthropic Claude (Sonnet 4.5)
- Google Gemini (Embeddings)
- n8n Simple Vector Store + Memory Buffer

### System-Prompt-Prinzipien
Der Agent ist bewusst restriktiv konfiguriert:
- Beantwortet **ausschließlich** Fragen über Luca – lehnt Off-Topic-Anfragen freundlich ab
- **Keine Halluzinationen**: Antwortet nur mit Infos aus dem Vector Store, erfindet nichts
- Antwortet immer in der Sprache der Anfrage
- Kurze, lockere Antworten (2-4 Sätze), kein Markdown, gerne mit Emojis

## 🚀 Setup / Verwendung

1. Beide JSONs in n8n importieren (`Import from File`)
2. Credentials einrichten:
   - **Google Gemini (PaLM) API** – für beide Workflows
   - **Anthropic API** – nur für den Agent-Workflow
3. Zuerst **Data from Luca RAG Setup** ausführen und eigene Dokumente hochladen, um den Vector Store zu befüllen
4. Danach **Lucas Agent** aktivieren – der Webhook ist dann bereit, Anfragen von der Website entgegenzunehmen
5. Webhook-URL in das eigene Frontend einbinden (POST-Request mit `message` und `sessionId` im Body)

## ⚠️ Hinweis

- Der In-Memory Vector Store hält die Daten nur, solange die n8n-Instanz läuft – bei Neustart muss der Setup-Workflow erneut ausgeführt werden. Für produktiven Dauerbetrieb empfiehlt sich ein persistenter Vector Store (z.B. Pinecone, Qdrant, Supabase).
- Der System-Prompt ist personalisiert auf "Luca" – bei Eigennutzung entsprechend anpassen.
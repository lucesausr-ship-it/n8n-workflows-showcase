# --- Secrets & Credentials ---
# Falls du mal testweise Zugangsdaten lokal speicherst
.env
.env.local
*.env
credentials.json
secrets.json
*_credentials.json
*.key
*.pem

# --- n8n-spezifisch ---
# Falls du n8n lokal mit SQLite-DB im Repo-Ordner laufen lässt (Config-Reste)
.n8n/
n8n-data/

# --- Windows-spezifischer Müll ---
Thumbs.db
desktop.ini
$RECYCLE.BIN/

# --- VS Code ---
.vscode/
*.code-workspace

# --- Backup-/Temp-Dateien ---
*.tmp
*.bak
*.log
~$*
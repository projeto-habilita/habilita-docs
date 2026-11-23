# Ambiente local

```bash
docker compose up          # API (FastAPI), frontend, emuladores Auth/Firestore e Supabase Storage local (ADR 0010)
pip install -r requirements-docs.txt
mkdocs serve               # docs em http://localhost:8000
```

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: comandos e emuladores de Auth e Firestore. Do Tech Lead: emulador de Storage, exigido para o Habilita (C13). |
| 1.1 | 03/11/2026 | @osamuelleite | Plano Custo Zero (ADR 0010): o emulador de Storage do Firebase dá lugar ao Supabase Storage local no Compose. |

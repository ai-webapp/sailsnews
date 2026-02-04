# Fix workflow n8n – un solo articolo invece di tutti

## Problema
Il nodo **Code in JavaScript** riceve **un solo item** (l’output di Add Metadata: `{ status, timestamp, count, news: [...] }`).  
Con `$input.all()` e `items.map(item => item.json)` costruisci `news: [ quell'oggetto ]`, cioè un array con **un solo elemento** (l’intero payload). SailNews quindi vede un solo “articolo”.

## Soluzione
L’output di **Add Metadata** è già nel formato giusto: `{ status, timestamp, count, news: [ articolo1, articolo2, ... ] }`. Non serve il Code.

### Cosa fare in n8n
1. **Elimina** il nodo **"Code in JavaScript"**.
2. **Collega** l’uscita di **"Add Metadata"** direttamente al nodo **"Respond to Webhook"** (al posto del Code).
3. Nel **Respond to Webhook** lascia:
   - **Respond With:** JSON (o Text se usi `JSON.stringify($json)`).
   - **Response Body:** `{{ JSON.stringify($json) }}` oppure `{{ $json }}` se il nodo accetta JSON diretto.
4. Salva e attiva il workflow.

### Flusso corretto
```
Webhook → RSS (x3) → Format (x3) → Aggregate All → Add Metadata → Respond to Webhook
```

Così la risposta del webhook sarà:
```json
{
  "status": "success",
  "timestamp": "2026-02-...",
  "count": N,
  "news": [
    { "title", "url", "created", "description", "source", "imageUrl" },
    ...
  ]
}
```
e SailNews mostrerà tutte le card.

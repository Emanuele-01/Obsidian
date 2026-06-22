# Approfondimento Ingegneristico: Chat Templates e Mascheramento del Gradiente (Loss Masking)

Nell'addestramento dei modelli di linguaggio (LLM) tramite **SFT (Supervised Fine-Tuning)**, il formato dei dati gioca un ruolo cruciale. Questo documento approfondisce come i dialoghi vengano convertiti in stringhe di testo attraverso i **Chat Templates** (usando Jinja2) e come, a livello di codice PyTorch, si eviti che il modello impari a prevedere le domande dell'utente tramite il **Loss Masking** (mascheramento del gradiente).

---

## 1. Chat Templates: Standardizzare i Dialoghi

Un dataset di dialogo viene salvato in formati strutturati (come JSON o JSONL) composti da liste di messaggi:

```json
[
  {"role": "system", "content": "Rispondi in formato JSON."},
  {"role": "user", "content": "Qual è la capitale della Francia?"},
  {"role": "assistant", "content": "{\"capitale\": \"Parigi\"}"}
]
```

Poiché i Transformer elaborano solo sequenze lineari di testo, questa struttura deve essere linearizzata inserendo dei delimitatori speciali (token speciali) che segnalino al modello l'inizio e la fine dei turni di parola e il ruolo di chi sta parlando.

### I Formati Standard
1. **ChatML (utilizzato da Qwen e molti modelli open-source):**
   ```text
   <|im_start|>system
   Rispondi in formato JSON.<|im_end|>
   <|im_start|>user
   Qual è la capitale della Francia?<|im_end|>
   <|im_start|>assistant
   {"capitale": "Parigi"}<|im_end|>
   ```

2. **Llama 3 / 3.1 / 3.2 Instruct Format:**
   ```text
   <|begin_of_text|><|start_header_id|>system<|end_header_id|>

   Rispondi in formato JSON.<|eot_id|><|start_header_id|>user<|end_header_id|>

   Qual è la capitale della Francia?<|eot_id|><|start_header_id|>assistant<|end_header_id|>

   {"capitale": "Parigi"}<|eot_id|>
   ```

### Jinja2 nei Tokenizzatori Hugging Face
Per evitare che l'utente debba formattare manualmente le stringhe per ogni modello, Hugging Face integra all'interno della configurazione del tokenizzatore un template **Jinja2** dinamico (`tokenizer.chat_template`). Lo script Python esegue il rendering in questo modo:

```python
# Rendering automatico con il template nativo del modello
formatted_text = tokenizer.apply_chat_template(messages, tokenize=False)
```

---

## 2. Mascheramento della Loss (Target-only Loss Masking)

Quando un modello impara a completare il testo, la funzione di perdita standard (**Cross-Entropy Loss**) viene calcolata prevedendo ogni singolo token successivo nella sequenza, da sinistra a destra.

Nel fine-tuning basato su istruzioni, tuttavia, c'è un problema fondamentale: **non vogliamo che il modello impari a prevedere il prompt dell'utente**. Vogliamo che impari a prevedere esclusivamente la risposta dell'assistente.

### Le conseguenze di una Loss non mascherata:
Se calcoliamo la perdita sull'intera sequenza (domanda + risposta):
1. Il modello spreca capacità computazionale per memorizzare i pattern e lo stile con cui gli utenti scrivono i prompt.
2. Si riscontra un degrado della qualità delle risposte (*style copycatting* o ripetizione del prompt di input).
3. L'apprendimento delle regole sintattiche specifiche (es. formattazione Obsidian o codice Go) risulta più debole.

### La Soluzione: Il valore magico `-100`
In PyTorch, la classe `torch.nn.CrossEntropyLoss` ha un parametro predefinito denominato `ignore_index` impostato a `-100`. Qualsiasi etichetta (target) contrassegnata con il valore `-100` viene **ignorata durante il calcolo del gradiente** e della perdita.

Durante la fase di tokenizzazione e preparazione del dataset, si creano due vettori della stessa lunghezza:
* **`input_ids`:** Contiene i token reali dell'intera conversazione (domanda + risposta) inviati al modello.
* **`labels`:** Contiene la copia esatta di `input_ids`, ma con una variazione fondamentale: tutti i token appartenenti al sistema e all'utente (compresi i delimitatori iniziali) vengono sovrascritti con `-100`. Solo i token appartenenti alla risposta dell'assistente (compreso il token speciale di fine sequenza `<|im_end|>` o `<|eot_id|>`) mantengono il loro ID originale.

```
Visualizzazione del Mascheramento dei Token:

Input IDs: [ <|im_start|>, user, \n, C, a, o, <|im_end|>, <|im_start|>, assistant, \n, H, e, l, l, o, <|im_end|> ]
Labels:    [    -100,    -100, -100, -100,-100,-100,  -100,      -100,      -100, -100, H, e, l, l, o, <|im_end|> ]
           \<-------------- MASKED (Loss = 0) ------------->/\<--------------- TRAINED (Loss Calcolata) ------->/
```

Grazie a questo mascheramento, il modello aggiorna i suoi pesi solo ed esclusivamente sulla precisione con cui risponde alla domanda, massimizzando l'efficacia del training e la fedeltà al formato.

---

## Collegamenti Consigliati
* Per comprendere come la loss calcolata sui token attivi aggiorni i pesi del modello nel ciclo di addestramento, vedi [[I 4 Step Fondamentali di Training#2. Loss Calculation]].
* Per vedere l'integrazione di questi formati con Obsidian o codice specialistico, leggi [[Guida Completa al Fine-Tuning#2. Il Fine-Tuning di un Modello Open-Source]].
* Per capire come ORPO integri nativamente le risposte positive e negative nel calcolo della loss, vedi [[Approfondimento ORPO]].

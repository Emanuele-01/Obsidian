# Glossario e Concetti Chiave del Fine-Tuning (Edizione 2026)

Questo documento funge da guida complementare per comprendere a fondo le dinamiche del **Fine-Tuning** e la terminologia tecnica avanzata utilizzata nelle moderne pipeline di addestramento dei modelli di linguaggio (LLM).

---

## Cos'è il Fine-Tuning? (Spiegazione Semplificata)

Immagina un **Modello Base** (Foundation Model) come un neolaureato brillantissimo che ha letto l'intera libreria mondiale: possiede un'eccellente cultura generale, conosce la grammatica di tutte le lingue e sa scrivere in modo fluido, ma non ha mai lavorato in un settore specifico.

Il **Fine-Tuning** è l'equivalente di un *Master Specialistico* o di un periodo di affiancamento aziendale mirato. Durante questo processo, il modello non impara una nuova lingua da zero, ma viene addestrato su un set di dati selezionato per:
1. **Specializzarsi in un dominio:** Comprendere gerghi tecnici complessi (es. log di infrastrutture IT, sintassi di librerie private).
2. **Modificare il comportamento:** Rispondere rispettando regole rigide (es. produrre esclusivamente file in formato Markdown strutturato, blocchi di codice puliti o risposte JSON valide per API).

---

## Dizionario delle Parole Cardine

Di seguito vengono analizzati e spiegati i termini tecnici fondamentali utilizzati per descrivere le metodologie più efficienti del 2026.

### 1. Pre-training (Pre-addestramento)
È la prima fase di vita di un LLM. Il modello analizza quantità enormi di dati non strutturati (miliardi di pagine web, libri, repository di codice) per imparare a prevedere la parola successiva in una frase. Questa fase richiede supercomputer e mesi di calcolo, generando la struttura base dell'IA.

### 2. VRAM (Video RAM)
La memoria ad accesso casuale situata sulla scheda video (GPU). Nel fine-tuning, la VRAM è la risorsa più critica in assoluto: contiene i pesi del modello, i gradienti e gli stati dell'ottimizzatore. Se la VRAM è insufficiente, il processo fallisce con un errore di *Out of Memory (OOM)*.

### 3. Pesi (Weights / Parametri)
I "pesi" sono i coefficienti matematici (i valori numerici nelle matrici del modello) che determinano quanta importanza dare a una parola rispetto a un'altra all'interno del contesto. Fare fine-tuning significa aggiornare questi numeri per correggere l'output del modello.

### 4. Quantizzazione (Quantization)
Una tecnica di compressione che riduce la precisione numerica dei pesi del modello (ad esempio passando da numeri a 16-bit a numeri a 4-bit). Questo permette di ridurre drasticamente lo spazio occupato in VRAM, consentendo a modelli massicci di essere eseguiti o addestrati su hardware consumer.

### 5. PEFT (Parameter-Efficient Fine-Tuning)
Invece di aggiornare tutti i miliardi di parametri di un modello (operazione pesantissima), le metodologie PEFT mantengono congelati i pesi originali del modello e addestrano solo un piccolo gruppo di parametri aggiuntivi inseriti ad hoc.

### 6. LoRA (Low-Rank Adaptation)
La tecnica PEFT più famosa. Invece di modificare le enormi matrici di peso originali del modello, LoRA inserisce delle matrici parallele molto più piccole e "strette" (di rango inferiore, o *Low-Rank*) all'interno degli strati neurali. Solo queste matrici minori vengono addestrate, riducendo i tempi e l'uso di memoria.

### 7. QLoRA (Quantized LoRA)
Un'evoluzione che combina la **Quantizzazione** a 4-bit del modello base con l'aggiunta dei moduli **LoRA** a 16-bit. È la metodologia regina per chi fa auto-ospitalità (*self-hosting*), poiché permette di fare fine-tuning di modelli avanzati su una singola scheda video da 24GB di VRAM.

### 8. DoRA (Weight-Decomposed Low-Rank Adaptation)
Un'ottimizzazione di LoRA che separa matematicamente la magnitudo (l'intensità) e la direzione dei pesi durante l'addestramento. Questo permette alle matrici LoRA di aggiornarsi con una precisione quasi identica a un addestramento completo (Full Fine-Tuning), pur mantenendo l'efficienza millimetrica di LoRA.

### 9. SFT (Supervised Fine-Tuning / Instruction Tuning)
Il processo di addestramento basato su esempi espliciti composti da una domanda (o istruzione) e dalla relativa risposta ideale fornita da un supervisore. Insegna al modello come comportarsi da vero e proprio assistente.

### 10. DPO e ORPO (Allineamento delle Preferenze)
* **DPO (Direct Preference Optimization):** Una tecnica che insegna al modello cosa scegliere e cosa evitare mostrandogli coppie di risposte (una considerata "buona" e una "mancata/allucinata"). Semplifica i vecchi processi eliminando la necessità di modelli di ricompensa complessi.
* **ORPO (Odds Ratio Preference Optimization):** Una metodologia avanzata che unisce la fase di SFT e quella di DPO in un unico step logico, ottimizzando i tempi di calcolo e impedendo al modello di dimenticare le sue capacità generali durante la specializzazione.

### 11. FlashAttention
Un algoritmo che ottimizza il modo in che la GPU calcola il meccanismo di attenzione (il modo in cui il modello collega le parole all'interno di un lungo testo). Evita colli di bottiglia nella memoria della GPU, velocizzando i calcoli fino a 2-4 volte.

### 12. GGUF / AWQ
Formati di esportazione e compressione dei modelli finiti:
* **GGUF:** Ottimizzato per l'inferenza anche su CPU o sistemi con poca GPU (molto usato con Ollama).
* **AWQ:** Ottimizzato per un'inferenza ultra-veloce basata puramente su GPU in ambienti di produzione (usato con vLLM).
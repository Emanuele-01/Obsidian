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
Una tecnica di compressione che riduce la precisione numerica dei pesi del modello. Maggiori dettagli sulla quantizzazione NF4 e sulla doppia quantizzazione si trovano in [[Approfondimento NF4 e Quantizzazione]].

### 5. PEFT (Parameter-Efficient Fine-Tuning)
Invece di aggiornare tutti i parametri di un modello, le metodologie PEFT mantengono congelati i pesi originali e addestrano solo un piccolo gruppo aggiuntivo (vedi [[I 5 Iperparametri di Configurazione QLoRA|Iperparametri di Configurazione LoRA]]).

### 6. LoRA (Low-Rank Adaptation)
La tecnica PEFT più famosa. Inserisce delle matrici parallele a basso rango all'interno degli strati neurali, come descritto in [[Glossario e Concetti su QLoRA#6. Matrici di Rango Inferiore (Low-Rank Matrices / Adapters)|Matrici di Rango Inferiore]].

### 7. QLoRA (Quantized LoRA)
Un'evoluzione che combina la quantizzazione a 4-bit del modello base con l'aggiunta di moduli LoRA a 16-bit. Per i dettagli vedi [[Approfondimento Tecnico QLoRA]].

### 8. DoRA (Weight-Decomposed Low-Rank Adaptation)
Un'ottimizzazione di LoRA che scompone matematicamente i pesi in magnitudo e direzione. Per un'analisi delle equazioni, vedi [[Approfondimento DoRA]].

### 9. SFT (Supervised Fine-Tuning / Instruction Tuning)
Il processo di addestramento basato su esempi espliciti composti da una domanda (o istruzione) e dalla relativa risposta ideale (Ground Truth). Per vedere dove si inserisce nel flusso di training, vedi [[I 4 Step Fondamentali di Training#2. Loss Calculation]].

### 10. DPO e ORPO (Allineamento delle Preferenze)
* **DPO (Direct Preference Optimization):** Insegna al modello le preferenze umane tramite coppie di risposte (scelte/rifiutate) evitando l'addestramento di un modello di ricompensa. Vedi [[Approfondimento DPO]].
* **ORPO (Odds Ratio Preference Optimization):** Una metodologia avanzata che unisce SFT e allineamento delle preferenze in un unico step logico. Vedi [[Approfondimento ORPO]].

### 11. FlashAttention
Un algoritmo che ottimizza il calcolo dell'attenzione riducendo i colli di bottiglia di I/O della GPU. Per saperne di più sulle versioni 1, 2 e 3, consulta [[Approfondimento FlashAttention]].

### 12. GGUF / AWQ
Formati di esportazione e compressione dei modelli finiti dopo la quantizzazione post-training:
* **GGUF:** Ottimizzato per l'inferenza anche su CPU o sistemi con poca GPU.
* **AWQ:** Ottimizzato per un'inferenza ultra-veloce basata puramente su GPU in ambienti di produzione.
* **EXL2:** Formato a bitrate variabile altamente ottimizzato per GPU desktop e schede consumer.
* Per un confronto dettagliato tra questi formati, consulta [[Approfondimento Formati Inferenza GGUF AWQ EXL2]].
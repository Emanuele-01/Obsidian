# Guida Completa al Fine-Tuning dei Modelli di Linguaggio (Aggiornata al 2026)

Il **fine-tuning** (o affinamento) è il processo mediante il quale un modello di Intelligenza Artificiale pre-addestrato (noto come *Foundation Model* o modello base) viene ulteriormente addestrato su un set di dati specifico e più circoscritto. Questo processo consente di adattare le capacità generali del modello a un dominio verticale, a un compito specialistico o a un determinato tono di voce.

Nel contesto dello sviluppo software moderno e dei sistemi auto-ospitati (*self-hosted*), il fine-tuning rappresenta il ponte tra un'intelligenza artificiale generica e un assistente aziendale o tecnico specializzato, capace di rispettare vincoli di formattazione (es. output strutturati in JSON, costrutti specifici di linguaggi come Go o Flutter) e logiche di business proprietarie.

---

## 1. Cosa significa il Fine-Tuning di un Modello

Un modello di linguaggio viene inizialmente sottoposto a un **pre-addestramento (Pre-training)** su una quantità massiva di dati testuali non strutturati (web scraping, libri, codice sorgente). In questa fase, il modello apprende la grammatica, le relazioni statistiche tra le parole e una vasta conoscenza enciclopedica del mondo. Tuttavia, non sa ancora come rispondere a un'istruzione in modo preciso o come comportarsi in un contesto specialistico.

Il **Fine-Tuning** interviene in una fase successiva. Invece di addestrare un modello da zero (operazione dal costo di milioni di dollari), si modificano i pesi del modello esistente utilizzando un dataset mirato (composto da migliaia o decine di migliaia di esempi accurati).

### Architettura Concettuale: Pre-training vs Fine-tuning

```
[ Dataset Massivo e Generico ] ---> PRE-TRAINING ---> [ Modello Base / Foundation Model ]
                                                                 |
                                                                 v
[ Dataset Specifico / Verticale ] -> FINE-TUNING  ---> [ Modello Specializzato (Fine-Tuned) ]
```

### Principali tipologie di Fine-Tuning:
1. **Instruction Fine-Tuning (SFT - Supervised Fine-Tuning):** Il modello viene addestrato su coppie di `[Istruzione, Risposta Corretta]`. È la tecnica fondamentale per trasformare un modello predittivo di testo in un assistente conversazionale (es. allineare il modello a rispondere in formato Markdown per Obsidian o a generare snippet di codice puliti).
2. **Domain Adaptation:** Il modello viene esposto a una grande quantità di testi di un settore specifico (es. documentazione interna di rete, registri di log di Kubernetes, codice sorgente di librerie private) per apprenderne il gergo e la sintassi prima o durante l'allineamento.

---

## 2. Il Fine-Tuning di un Modello Open-Source

Il fine-tuning di modelli open-source (come la famiglia **Llama 3/3.1/3.2**, **Mistral/Mixtral**, o **Qwen**) differisce profondamente dall'utilizzo di modelli closed-source (come quelli di OpenAI o Anthropic) accessibili solo via API.

Quando si effettua il fine-tuning di un modello open-source, si ha il controllo totale sulla pipeline di addestramento:

* **Accesso completo ai pesi (Weights):** È possibile modificare direttamente i parametri interni del modello, consentendo ottimizzazioni matematiche avanzate non applicabili tramite API esterne.
* **Privacy e Sovranità dei Dati:** I dati di addestramento (che spesso includono segreti aziendali, stringhe di configurazione, codice proprietario o dati sensibili) non lasciano mai l'infrastruttura locale o il VPS dedicato.
* **Personalizzazione del Formato di Output:** Si può addestrare il modello a rispondere nativamente rispettando token di sistema personalizzati, formati speciali o strutture dati complesse. Questo avviene applicando appositi [[Approfondimento Chat Templates e Tokenizzazione#1. Chat Templates: Standardizzare i Dialoghi|Chat Templates]] in Jinja2 e mascherando la loss sui prompt dell'utente tramite il [[Approfondimento Chat Templates e Tokenizzazione#2. Mascheramento della Loss (Target-only Loss Masking)|Loss Masking]].
* **Ottimizzazione dei Costi di Inferenza:** Un modello open-source da 8B o 14B parametri, accuratamente rifinito su un compito specifico, può superare in accuratezza modelli commerciali molto più grandi, riducendo drasticamente i costi computazionali di inferenza se eseguito su hardware dedicato o cluster auto-ospitati (es. tramite Docker ed Ollama/vLLM).

---

## 3. Le Migliori Metodologie per il 2026

Nel 2026, il panorama del fine-tuning si è consolidato attorno all'efficienza energetica, alla riduzione del consumo di VRAM e all'allineamento basato sulle preferenze senza la necessità di complessi sistemi di ricompensa esterni. Le metodologie dominanti e più efficienti includono:

### A. PEFT (Parameter-Efficient Fine-Tuning) e varianti avanzate
L'addestramento completo (*Full Fine-Tuning*) di tutti i parametri richiede un'enorme quantità di memoria GPU. Le tecniche PEFT congelano la maggior parte del modello base e addestrano solo un numero ridotto di parametri aggiuntivi.

* **LoRA (Low-Rank Adaptation):** Inietta matrici di decomposizione di rango inferiore negli strati di attenzione del modello. È governata da parametri descritti in [[I 5 Iperparametri di Configurazione QLoRA|Iperparametri di Configurazione]].
* **QLoRA (Quantized LoRA):** Rappresenta lo standard di riferimento per il self-hosting. Il modello base viene caricato in una quantizzazione a 4-bit ad alta precisione (NF4), mentre i moduli LoRA rimangono a 16-bit. Questo permette di effettuare il fine-tuning di un modello da 8B o 14B parametri su una singola GPU consumer (vedi [[Approfondimento Tecnico QLoRA]] e l'analisi su [[Approfondimento NF4 e Quantizzazione]]).
* **DoRA (Weight-Decomposed Low-Rank Adaptation):** Evoluzione di LoRA introdotta recentemente, che scompone i pesi del modello in componenti di magnitudo e direzione. DoRA ottimizza entrambe in modo indipendente, permettendo a LoRA di approssimare le performance del Full Fine-Tuning con la stessa efficienza computazionale (vedi [[Approfondimento DoRA]]).

### B. Allineamento Avanzato e Post-Training (Sostituti di RLHF)
L'apprendimento per rinforzo basato sui feedback umani (RLHF) del passato era complesso e instabile. Nel 2026 si utilizzano tecniche dirette:

* **DPO (Direct Preference Optimization):** Elimina la necessità di addestrare un modello di ricompensa separato. DPO ottimizza il modello direttamente sui dati di preferenza utilizzando una funzione di perdita binaria semplice ed elegante (vedi [[Approfondimento DPO]]).
* **ORPO (Odds Ratio Preference Optimization):** Integra la fase di Supervised Fine-Tuning (SFT) e l'allineamento delle preferenze in un unico step logico. Riduce drasticamente i tempi di calcolo e previene la degradazione delle performance del modello su compiti generici durante l'allineamento specialistico (vedi [[Approfondimento ORPO]]).

### C. Pipeline di Generazione Sintetica del Dataset
La qualità del fine-tuning dipende interamente dalla qualità dei dati. Nel 2026, la metodologia standard prevede l'uso di modelli "Frontier" (es. tramite architetture di tipo *Judge* o modelli commerciali superiori) per generare, filtrare e validare sinteticamente migliaia di esempi di addestramento partendo da documentazione grezza o file di codice, garantendo una pulizia del dataset senza precedenti prima dell'addestramento locale (descritta nel capitolo sull'SFT di [[I 4 Step Fondamentali di Training#2. Loss Calculation|Loss Calculation]]).

---

## Sintesi Operativa per l'Infrastruttura

Per implementare queste metodologie nel 2026, lo stack tecnologico raccomandato prevede l'utilizzo di librerie open-source consolidate gestite tramite container o ambienti di sviluppo isolati:

1. **Dataset Curation:** `Hugging Face Datasets` e script di parsing custom (con standardizzazione tramite [[Approfondimento Chat Templates e Tokenizzazione|Chat Templates]]).
2. **Training Engine:** `Axolotl` o `TRL (Transformer Reinforcement Learning)` integrati con `DeepSpeed` per il partizionamento dei dati in memoria (ZeRO Stage 1/2/3, vedi [[Approfondimento DeepSpeed e ZeRO]]).
3. **Hardware Efficiency:** Abilitazione nativa di `FlashAttention-2` o `FlashAttention-3` per ottimizzare l'uso dei Tensor Core delle GPU e accelerare il calcolo dei meccanismi di attenzione (vedi [[Approfondimento FlashAttention]]).
4. **Deployment:** Esportazione dei pesi finali (fusi con il modello base) quantizzati nei formati ottimizzati per l'inferenza target: `GGUF` per CPU/GPU ibrida, o `AWQ`/`EXL2` per GPU (vedi [[Approfondimento Formati Inferenza GGUF AWQ EXL2]]).
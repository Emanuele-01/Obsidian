# Approfondimento Tecnico: Formati di Inferenza e Quantizzazione (GGUF, AWQ, EXL2)

Una volta completato il fine-tuning di un modello di linguaggio, i suoi pesi vengono salvati in precisione nativa a 16-bit ($FP16$ o $BF16$). Caricare ed eseguire questi modelli in produzione richiede una quantità di risorse hardware significativa. Per ottimizzare l'inferenza, si ricorre alla **Quantizzazione Post-Training (PTQ - Post-Training Quantization)**, che comprime i pesi del modello finito.

Nel 2026, l'ecosistema dell'inferenza locale e self-hosted si divide principalmente su tre formati ottimizzati per diversi tipi di hardware: **GGUF**, **AWQ** e **EXL2**.

---

## 1. GGUF (GPT-Generated Unified Format)

Sviluppato dal team di **llama.cpp**, il formato **GGUF** è il successore del vecchio GGML. È diventato lo standard di fatto per l'inferenza su CPU e per sistemi ibridi CPU+GPU (come i chip Apple Silicon M-series o schede video consumer abbinate alla RAM del computer).

### Caratteristiche Principali:
* **Single-File Distribution:** GGUF memorizza l'intero modello, i pesi quantizzati, i token speciali e tutti i metadati strutturali (iperparametri, architettura, configurazione del tokenizzatore) in un singolo file binario. Questo previene problemi di compatibilità tra i file dei pesi e i tokenizzatori.
* **Hybrid CPU/GPU Offloading:** Permette di dividere matematicamente i layer del modello: ad esempio, su una GPU con soli 8GB di VRAM, è possibile caricare i primi 20 layer sulla scheda video e delegare i restanti 12 layer alla CPU (RAM di sistema), evitando crash per *Out of Memory*.
* **Quantizzazioni a Blocchi (K-Quantization):** GGUF utilizza una quantizzazione lineare basata su blocchi (es. `Q4_K_M`, `Q5_K_S`). Nella quantizzazione `Q4_K_M`, ad esempio:
  * I pesi sono suddivisi in super-blocchi di 256 parametri, a loro volta divisi in sotto-blocchi di 32 parametri.
  * Le costanti di scala (*scale factors*) e i minimi vengono quantizzati con precisioni diverse (es. a 6-bit) per ridurre drasticamente la perdita di precisione (perplexity) mantenendo il peso medio a circa 4.5 bit per parametro (bpw).

---

## 2. AWQ (Activation-aware Weight Quantization)

Il formato **AWQ** è progettato specificamente per massimizzare la velocità di inferenza e il parallelismo su GPU in ambienti di produzione ad alto traffico (come i server che utilizzano il motore **vLLM** o **TGI**).

### La Teoria di Base:
AWQ si basa sulla scoperta empirica che **non tutti i pesi di un LLM hanno la stessa importanza**. Solo una piccolissima percentuale (circa l'1%) delle connessioni neurali controlla l'accuratezza logica del modello. Questi pesi "salienti" corrispondono ai canali che registrano i picchi di attivazione più elevati durante il passaggio dei dati.

* **Protezione dei Pesi Chiave:** Invece di quantizzare uniformemente tutti i parametri (il che danneggerebbe l'accuratezza), AWQ identifica questi pesi importanti analizzando i profili di attivazione del modello su un dataset di calibrazione.
* **Scaling e Quantizzazione:** AWQ applica una scala di moltiplicazione per proteggere i pesi salienti, comprimendo poi gli altri pesi a 4-bit integer ($INT4$). Questo azzera quasi del tutto la perdita di accuratezza logica rispetto a modelli non compressi a 16-bit.
* **Prestazioni GPU:** Sfrutta direttamente i Tensor Core delle GPU NVIDIA ottimizzati per calcoli a bassa precisione in virgola mobile e intera, garantendo velocità di generazione dei token (*throughput*) eccezionali.

---

## 3. EXL2 (ExLlamaV2 Format)

Il formato **EXL2** è un formato proprietario della libreria **ExLlamaV2**, ottimizzato in modo estremo per l'esecuzione di LLM su schede video consumer (GPU singole o multiple collegate via PCIe) in ambienti desktop o home server.

### Caratteristiche Principali:
* **Bitrate Variabile Granulare:** A differenza di GGUF o AWQ che applicano lo stesso livello di quantizzazione a tutto il modello (es. fisso a 4-bit), EXL2 supporta quantization grid flessibili (da 2.0 a 8.0 bit per parametro, con precisioni decimali come `4.25 bpw` o `5.0 bpw`).
* **Misurazione dell'Errore per Layer:** Durante la compilazione del modello EXL2, uno script analizza l'errore di quantizzazione layer per layer su un dataset di calibrazione. Il compilatore assegna automaticamente un bitrate più alto (es. 5-bit o 6-bit) ai layer di attenzione critici o a quelli che registrano perdite di accuratezza elevate, compensando con un bitrate molto basso (es. 2-bit o 3-bit) sui layer del Feed-Forward Network meno influenti.
* **Kernel CUDA personalizzati:** I kernel di calcolo sono scritti interamente in C++/CUDA per minimizzare la latenza di avvio del token (*Time to First Token - TTFT*), offrendo velocità di generazione notevolmente superiori rispetto a GGUF a parità di hardware GPU.

---

## Collegamenti Consigliati
* Per comprendere come convertire ed esportare i pesi fusi a fine training in questi formati, vedi [[Guida Completa al Fine-Tuning#Sintesi Operativa per l'Infrastruttura]].
* Per la terminologia correlata, consulta [[Glossario e Concetti Chiave del Fine-Tuning#12. GGUF / AWQ]].
* Per esplorare come la quantizzazione avvenga invece durante la fase di training (NF4), leggi [[Approfondimento NF4 e Quantizzazione]].

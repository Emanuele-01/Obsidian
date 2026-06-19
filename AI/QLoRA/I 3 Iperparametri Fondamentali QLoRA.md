# I 3 Iperparametri Fondamentali per il Training in QLoRA

Mentre gli iperparametri strutturali (come *Rank*, *Alpha* e *Quantization*) definiscono la forma geometrica delle matrici di adattamento, gli **iperparametri di training** determinano la dinamica algoritmica con cui il modello apprende dai dati nel tempo. 

Basandosi sull'architettura di configurazione del ciclo di ottimizzazione, ecco l'analisi dettagliata dei 3 punti cardine del processo di addestramento:

---

## 1. Epochs (Epoche)

Un'**Epoca** rappresenta un passaggio completo (in avanti e all'indietro, ovvero *forward* e *backward pass*) dell'intero dataset di addestramento attraverso la rete neurale.

* **Cosa significa:** Indica quante volte il modello "vede" l'intero set di dati a disposizione. Se si imposta un valore pari a `3`, significa che la pipeline esporrà la sequenza di addestramento per tre cicli completi.
* **Best Practice nel 2026:** Nel fine-tuning degli LLM con QLoRA, il numero di epoche è generalmente molto basso rispetto al deep learning tradizionale (spesso compreso **tra 1 e 3 epoche**). Poiché il modello base possiede già una forte comprensione strutturale del linguaggio, esporlo troppe volte agli stessi identici dati specifici rischia di distruggere le sue capacità di generalizzazione (*overfitting*), portando il modello a ripetere a memoria i testi del dataset anziché comprenderne le logiche sottostanti.

---

## 2. Batch Size (Dimensione del Batch)

Il **Batch Size** definisce il numero di esempi di addestramento (campioni di testo o sotto-sequenze) elaborati contemporaneamente dalla GPU prima di calcolare i gradienti e aggiornare i pesi adattatori.

* **L'equilibrio computazionale:** * Un Batch Size **più grande** rende il calcolo dei gradienti più stabile e sfrutta al massimo il parallelismo dei Tensor Core della GPU, velocizzando l'addestramento. Tuttavia, richiede una quantità enorme di VRAM.
  * Un Batch Size **più piccolo** riduce il consumo di memoria video, ma rende l'ottimizzazione più instabile (*rumorosa*) e frammentata.
* **Strategia di Micro-Batch e Gradient Accumulation:** Nelle moderne configurazioni su GPU singole o server locali, per superare i limiti fisici della VRAM si scompone il parametro in due parti:
  1. **Per-Device Train Batch Size (Micro-Batch):** Il numero di campioni caricati fisicamente in memoria nello stesso istante (es. `2` o `4`).
  2. **Gradient Accumulation Steps:** Il numero di iterazioni consecutive in cui i gradienti vengono calcolati e accumulati in memoria *prima* di effettuare l'aggiornamento effettivo dei pesi (es. `8` o `16`). 
  
  *La moltiplicazione tra questi due fattori determina il **Batch Size Effettivo** (es. $2 \times 16 = 32$), consentendo stabilità matematica senza saturare la VRAM.*

---

## 3. Learning Rate (Tasso di Apprendimento)

Il **Learning Rate (LR)** è l'iperparametro più critico in assoluto: determina la dimensione del "passo" che l'ottimizzatore (es. AdamW) compie quando aggiorna i parametri numerici (i pesi) del modello per ridurre l'errore.

* **La dinamica del passo:**
  * Se il Learning Rate è **troppo alto**, l'ottimizzatore compie passi troppo lunghi, rischiando di scavalcare la soluzione ideale o di destabilizzare completamente le matrici adattatrici (divergenza del gradiente).
  * Se il Learning Rate è **troppo basso**, i passi saranno microscopici, allungando a dismisura i tempi di addestramento e rischiando di far bloccare il modello in punti di ottimizzazione scadenti (*local minima*).
* **Configurazione tipica per QLoRA:** Poiché le matrici LoRA partono da una configurazione pre-ottimizzata (dove una matrice è inizializzata a zero e l'altra con valori casuali), si utilizzano valori molto piccoli, tipicamente compresi tra **$1 	imes 10^{-4}$ ($0.0001$) e $2 	imes 10^{-5}$ ($0.00002$)**. 
* **Uso dei Warmup Steps e dei Decay Scheduler:** Nel 2026 è prassi standard abbinare il Learning Rate a un *Cosine Scheduler*. Durante il primo 3-5% dell'addestramento (*Warmup*), il Learning Rate sale gradualmente da zero al suo valore massimo per stabilizzare i pesi; successivamente, decresce seguendo una curva cosidettata a coseno fino a raggiungere un valore prossimo allo zero alla fine dell'ultima epoca, garantendo una convergenza pulita e precisa.
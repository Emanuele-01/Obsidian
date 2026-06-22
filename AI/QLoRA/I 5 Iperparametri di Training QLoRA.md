# I 5 Iperparametri di Training in QLoRA

Mentre gli iperparametri strutturali (come *Rank*, *Alpha* e *Quantization* descritti in [[I 5 Iperparametri di Configurazione QLoRA|Iperparametri di Configurazione]]) definiscono la forma geometrica delle matrici di adattamento, gli **iperparametri di training** determinano la dinamica algoritmica, l'ottimizzazione e la gestione hardware con cui il modello apprende dai dati nel tempo.

Basandosi sull'architettura avanzata del ciclo di ottimizzazione (vedi [[I 4 Step Fondamentali di Training|4 Step Fondamentali di Training]]), ecco l'analisi dettagliata dei 5 punti cardine del processo di addestramento:

---

## 1. Epochs (Epoche)

Un'**Epoca** rappresenta un passaggio completo (in avanti e all'indietro, ovvero *forward* e *backward pass*) dell'intero dataset di addestramento attraverso la rete neurale.

* **Cosa significa:** Indica quante volte il modello "vede" l'intero set di dati a disposizione. Se si imposta un valore pari a `3`, significa che la pipeline esporrà la sequenza di addestramento per tre cicli completi.
* **Best Practice nel 2026:** Nel fine-tuning degli LLM con QLoRA, il numero di epoche è generalmente molto basso rispetto al deep learning tradizionale (spesso compreso **tra 1 e 3 epoche**). Poiché il modello base possiede già una forte comprensione strutturale del linguaggio, esporlo troppe volte agli stessi identici dati specifici rischia di distruggere le sue capacità di generalizzazione (*overfitting*), portando il modello a ripetere a memoria i testi del dataset anziché comprenderne le logiche sottostanti. Per mitigare l'overfitting si consiglia di regolare anche il [[I 5 Iperparametri di Configurazione QLoRA#5. Dropout (LoRA Dropout)|Dropout]].

---

## 2. Batch Size (Dimensione del Batch)

Il **Batch Size** definisce il numero di esempi di addestramento (campioni di testo o sotto-sequenze) elaborati contemporaneamente dalla GPU prima di calcolare i gradienti e aggiornare i pesi adattatori.

* **L'equilibrio computazionale:**
  * Un Batch Size **più grande** rende il calcolo dei gradienti più stabile e sfrutta al massimo il parallelismo dei Tensor Core della GPU (soprattutto se abbinato ad algoritmi come [[Approfondimento FlashAttention|FlashAttention]]), velocizzando l'addestramento. Tuttavia, richiede una quantità enorme di VRAM.
  * Un Batch Size **più piccolo** riduce il consumo di memoria video, ma rende l'ottimizzazione più instabile (*rumorosa*) e frammentata.
* **Configurazione tipica:** Nei sistemi locali o su singola GPU, si tende a tenere basso il *Per-Device Train Batch Size* (chiamato anche **Micro-Batch Size**, ad esempio impostato a `2` o `4`) per evitare errori di memoria, compensando la granularità del calcolo con il punto successivo.

---

## 3. Gradient Accumulation (Accumulo del Gradiente)

Il **Gradient Accumulation** è la tecnica ingegneristica fondamentale che permette di simulare Batch Size virtuali di grandi dimensioni quando l'hardware locale ha vincoli rigidi di memoria (VRAM).

* **Come funziona:** Invece di aggiornare i pesi del modello dopo ogni singolo micro-batch, il sistema esegue diversi passaggi in avanti e all'indietro (descritti nei [[I 4 Step Fondamentali di Training|Step di Training]]) accumulando i gradienti calcolati in una memoria temporanea. L'aggiornamento effettivo dei pesi tramite l'ottimizzatore avviene solo dopo che è stato raggiunto il numero di passaggi impostato (es. `8` o `16` step).
* **Il vantaggio matematico:** La moltiplicazione tra il Micro-Batch Size e i Gradient Accumulation Steps definisce il **Batch Size Effettivo (Globale)**. 
  $$\text{Global Batch Size} = \text{Micro-Batch Size} \times \text{Gradient Accumulation Steps}$$
  Ad esempio, configurando un micro-batch di `2` e un'accumulazione di `16`, si ottiene un batch globale di `32`. Questo consente di avere la stessa stabilità matematica e convergenza di un server industriale con cluster di GPU, pur addestrando su una singola scheda video desktop.

---

## 4. Learning Rate (Tasso di Apprendimento)

Il **Learning Rate (LR)** è l'iperparametro più critico in assoluto: determina la dimensione del "passo" che l'ottimizzatore compie quando aggiorna i parametri numerici (i pesi) del modello per ridurre l'errore.

* **La dinamica del passo:**
  * Se il Learning Rate è **troppo alto**, l'ottimizzatore compie passi troppo lunghi, rischiando di scavalcare la soluzione ideale o di destabilizzare completamente le matrici adattatrici (divergenza del gradiente).
  * Se il Learning Rate è **troppo basso**, i passi saranno microscopici, allungando a dismisura i tempi di addestramento e rischiando di far bloccare il modello in punti di ottimizzazione scadenti (*local minima*).
* **Configurazione tipica per QLoRA:** Si utilizzano valori molto piccoli, tipicamente compresi tra **$1 \times 10^{-4}$ ($0.0001$) e $2 \times 10^{-5}$ ($0.00002$)**, abbinati a un *Cosine Scheduler* e a passaggi di *Warmup* per stabilizzare l'addestramento nelle prime fasi. È importante calibrare questo parametro in base al fattore di scala [[I 5 Iperparametri di Configurazione QLoRA#3. Alpha (LoRA Alpha)|Alpha ($\alpha$)]].

---

## 5. Optimizer (Ottimizzatore)

L'**Ottimizzatore** è l'algoritmo matematico incaricato di calcolare l'esatta traiettoria di aggiornamento dei pesi basandosi sui gradienti accumulati e sul learning rate.

* **Lo Standard nel 2026:** Il re incontrastato del fine-tuning è **AdamW** (Adam con *Weight Decay* integrato). AdamW traccia in modo indipendente la media mobile del gradiente (momento) e il suo quadrato per ogni singolo parametro, adattando il passo in modo dinamico e applicando una penalizzazione (regolarizzazione) per evitare che i pesi crescano a dismisura.
* **Varianti Ottimizzate per il Self-Hosting:** Poiché AdamW richiede di mantenere in VRAM due stati aggiuntivi per ogni parametro addestrabile, nelle pipeline QLoRA si utilizzano varianti ottimizzate come:
  * **paged_adamw_8bit / bnb_8bit_adamw:** Versioni quantizzate a 8-bit fornite dalla libreria `bitsandbytes`. Riducono lo spazio occupato dagli stati dell'ottimizzatore del 75%. Inoltre, la funzionalità *Paged* permette di effettuare il paging sulla RAM di sistema (CPU) in caso di saturazione improvvisa della VRAM, fungendo da rete di sicurezza contro i crash. Per una spiegazione del funzionamento della paginazione vedi [[Glossario e Concetti su QLoRA#4. Ottimizzatori Paginati (Paged Optimizers)|Ottimizzatori Paginati]].
curezza contro i crash.
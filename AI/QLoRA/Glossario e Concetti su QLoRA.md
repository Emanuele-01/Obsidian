# Glossario e Concetti Chiave su QLoRA (Edizione 2026)

Questo documento funge da guida complementare per comprendere a fondo i meccanismi interni di **QLoRA (Quantized Low-Rank Adaptation)** e la terminologia tecnica avanzata utilizzata nell'ambito del fine-tuning efficiente e dell'ottimizzazione degli LLM open-source.

---

## Cos'è il QLoRA? (Spiegazione Semplificata)

Immagina di dover ristrutturare un intero grattacielo (il **Modello Base**). La ristrutturazione classica (*Full Fine-Tuning*) richiederebbe di svuotare l'intero edificio, ridipingere ogni singola stanza e modificare ogni muro, un'operazione che richiede risorse immense e macchinari costosissimi.

Con la tecnica **LoRA**, decidi di non toccare la struttura portante dell'edificio, ma di costruire una piccola dépendance efficiente o una stanza prefabbricata sul tetto per gestire le nuove attività. Allenerai solo quella.

**QLoRA** fa un passo fondamentale in più per risparmiare spazio e costi: prende il grattacielo originale e ne comprime la rappresentazione (tramite la **Quantizzazione** a 4-bit) trasformandolo, metaforicamente, in una struttura prefabbricata ultra-compatta ma perfettamente solida. Durante il lavoro, i muratori guardano la struttura compressa solo quando serve, ma fanno le modifiche effettive e ad alta precisione unicamente sulla dépendance (le matrici adattatrici a 16-bit). 

In questo modo, un processo che prima richiedeva server industriali in cloud ora può essere completato interamente su una singola scheda video commerciale.

---

## Dizionario delle Parole Cardine di QLoRA

Di seguito vengono analizzati e spiegati i termini ingegneristici e matematici utilizzati per descrivere il funzionamento di QLoRA.

### 1. Quantizzazione a 4-bit (4-bit Quantization)
È il processo di riduzione della precisione numerica dei parametri del modello. Invece di usare 16 bit per rappresentare ogni singolo numero (peso), se ne usano solo 4. Questo riduce la dimensione del modello sul disco e in memoria di ben 4 volte, consentendo a modelli giganti di entrare in GPU molto più piccole.

### 2. NF4 (NormalFloat 4)
È un tipo di dato a 4-bit innovativo, costruito appositamente per i modelli di linguaggio. Poiché i pesi dei neuroni tendono a distribuirsi naturalmente secondo una curva gaussiana (a campana), l'NF4 assegna i valori quantizzati in modo non lineare, concentrando la precisione dove si trovano la maggior parte dei numeri. Questo azzera quasi completamente la perdita di informazioni tipica delle vecchie quantizzazioni lineari (INT4).

### 3. Doppia Quantizzazione (Double Quantization)
Nelle quantizzazioni standard, vengono salvate delle costanti di calcolo (scaling factors) per poter ricostruire i numeri originali. Queste costanti occupano memoria. QLoRA applica una seconda quantizzazione (a 8-bit) direttamente su queste costanti, risparmiando ulteriore spazio prezioso in VRAM (circa 32 bit per blocco).

### 4. Ottimizzatori Paginati (Paged Optimizers)
Una tecnologia ispirata alla memoria virtuale dei sistemi operativi. Quando la memoria video (VRAM) della scheda grafica si satura a causa di una sequenza di testo troppo lunga, l'ottimizzatore sposta temporaneamente i dati in eccesso sulla RAM del computer (CPU) attraverso il canale PCIe, evitando il crash del sistema per *Out of Memory*.

### 5. De-quantizzazione al volo (De-quantization on-the-fly)
Il modello base memorizzato in NF4 a 4-bit non può fare calcoli matematici diretti in quella forma. Durante l'addestramento, quando un blocco di dati deve passare attraverso un determinato strato neurale, i pesi a 4-bit vengono temporaneamente convertiti (de-quantizzati) in formato a 16-bit (BF16), viene eseguito il calcolo e poi vengono immediatamente cancellati dalla memoria cache, mantenendo l'occupazione della VRAM al minimo.

### 6. Matrici di Rango Inferiore (Low-Rank Matrices / Adapters)
Le componenti aggiuntive introdotte da LoRA/QLoRA. Invece di una matrice quadrata enorme (es. $4096 \times 4096$), LoRA usa due matrici rettangolari molto più piccole e sottili posizionate in sequenza (es. $4096 \times 8$ e $8 \times 4096$). Il numero fisso (in questo esempio, `8`) rappresenta il **Rango ($r$)**. Meno parametri significano meno memoria e calcoli fulminei.

### 7. DoRA (Weight-Decomposed Low-Rank Adaptation)
Una variante avanzata introdotta per colmare il piccolissimo gap di accuratezza rimasto tra QLoRA e l'addestramento completo. DoRA scompone la matrice dei pesi in due vettori separati: uno gestisce la magnitudo (quanto è forte il segnale neurale) e uno la direzione. Modificando questi due elementi in modo indipendente, DoRA apprende con l'accuratezza di un addestramento completo pur consumando la stessa memoria di QLoRA.

### 8. rsLoRA (Rank-Stabilized LoRA)
Nelle implementazioni LoRA standard, aumentare il rango (es. impostarlo a 64 o 128 per compiti difficili) può rendere l'addestramento instabile. rsLoRA corregge la formula matematica di scaling del gradiente, moltiplicandolo per l'inverso della radice quadrata del rango ($1/\sqrt{r}$). Questo permette di scalare il rango in totale sicurezza senza rischiare che il modello impazzisca (esplosione del gradiente).

### 9. ORPO (Odds Ratio Preference Optimization)
Un metodo di allineamento che unisce la comprensione del testo e le preferenze umane. Tradizionalmente, dovevi prima insegnare al modello a parlare bene (SFT) e poi, in una fase separata, insegnargli cosa preferire (DPO). ORPO unisce tutto in un unico step applicando una penalità matematica immediata se il modello devia verso la risposta "sbagliata" o allucinata mentre sta ancora imparando la struttura del testo.

### 10. FlashAttention-3
La terza generazione dell'algoritmo di calcolo dell'attenzione neurale. Ottimizza in modo microscopico l'uso della memoria SRAM interna della GPU eliminando la necessità di leggere e scrivere continuamente sulla memoria video principale (HBM/VRAM). Sulle GPU moderne, accelera l'addestramento riducendo drasticamente i tempi morti computazionali.
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
È il processo di riduzione della precisione numerica dei pesi del modello. Invece di usare 16 bit per rappresentare ogni singolo numero (peso), se ne usano solo 4. È configurata tramite il parametro di [[I 5 Iperparametri di Configurazione QLoRA#4. Quantization (Quantizzazione del Modello Base)|Quantizzazione]].

### 2. NF4 (NormalFloat 4)
È un tipo di dato a 4-bit innovativo, costruito appositamente per i modelli di linguaggio. Poiché i pesi dei neuroni tendono a distribuirsi naturalmente secondo una curva gaussiana (a campana), l'NF4 assegna i valori quantizzati in modo non lineare. Per i dettagli matematici, vedi [[Approfondimento NF4 e Quantizzazione#1. Il tipo di dato NF4 (NormalFloat 4)|l'approfondimento su NF4]].

### 3. Doppia Quantizzazione (Double Quantization)
Nelle quantizzazioni standard, vengono salvate delle costanti di calcolo (scaling factors) per poter ricostruire i numeri originali. QLoRA applica una seconda quantizzazione (a 8-bit) direttamente su queste costanti, come spiegato in [[Approfondimento NF4 e Quantizzazione#2. Doppia Quantizzazione (Double Quantization)|Doppia Quantizzazione]].

### 4. Ottimizzatori Paginati (Paged Optimizers)
Una tecnologia ispirata alla memoria virtuale dei sistemi operativi. Se la VRAM si satura, l'ottimizzatore sposta temporaneamente i dati in eccesso sulla RAM del computer (CPU). Questa funzionalità è supportata da ottimizzatori avanzati descritti in [[I 5 Iperparametri di Training QLoRA#5. Optimizer (Ottimizzatore)|Ottimizzatori di Training]].

### 5. De-quantizzazione al volo (De-quantization on-the-fly)
Il modello base memorizzato in NF4 a 4-bit non può fare calcoli matematici diretti in quella forma. Durante l'addestramento, i pesi a 4-bit vengono temporaneamente convertiti in formato a 16-bit ($BF16$) per eseguire il calcolo dello strato neurale e poi vengono immediatamente cancellati dalla cache. Vedi il [[Approfondimento NF4 e Quantizzazione#3. Flusso dei Calcoli e De-quantizzazione al Volo (On-the-Fly)|flusso dettagliato dei calcoli]].

### 6. Matrici di Rango Inferiore (Low-Rank Matrices / Adapters)
Le componenti aggiuntive introdotte da LoRA/QLoRA. Invece di una matrice quadrata enorme, LoRA usa due matrici rettangolari molto più piccole posizionate in sequenza. La dimensione interna delle matrici è definita dal [[I 5 Iperparametri di Configurazione QLoRA#2. r (Rank / Rango)|Rango ($r$)]] ed è scalata dal parametro [[I 5 Iperparametri di Configurazione QLoRA#3. Alpha (LoRA Alpha)|Alpha ($\alpha$)]].

### 7. DoRA (Weight-Decomposed Low-Rank Adaptation)
Una variante avanzata introdotta per colmare il divario di accuratezza rimasto tra QLoRA e l'addestramento completo. DoRA scompone la matrice dei pesi in magnitudo e direzione. Per un'analisi approfondita delle sue equazioni, consulta [[Approfondimento DoRA]].

### 8. rsLoRA (Rank-Stabilized LoRA)
Nelle implementazioni LoRA standard, aumentare il rango può rendere l'addestramento instabile. rsLoRA corregge la formula matematica di scaling del gradiente, moltiplicandolo per l'inverso della radice quadrata del rango ($1/\sqrt{r}$). Maggiori dettagli su [[Approfondimento DoRA#1. Il Limite di LoRA|rsLoRA]].

### 9. ORPO (Odds Ratio Preference Optimization)
Un metodo di allineamento che unisce la comprensione del testo (SFT) e le preferenze umane (DPO) in un unico step logico, ottimizzando i tempi di calcolo e impedendo la degradazione del modello. Vedi [[Approfondimento ORPO]].

### 10. FlashAttention-3
La terza generazione dell'algoritmo di calcolo dell'attenzione neurale. Ottimizza in modo microscopico l'uso della memoria SRAM interna della GPU ed elimina i colli di bottiglia di I/O. Vedi [[Approfondimento FlashAttention]].
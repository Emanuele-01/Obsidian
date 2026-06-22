# Approfondimento Ingegneristico: DeepSpeed e la Tecnologia ZeRO (Zero Redundancy Optimizer)

Quando si effettua il fine-tuning di modelli di linguaggio di grandi dimensioni (LLM), la limitazione principale non è la capacità di calcolo della GPU, ma la quantità di memoria video (**VRAM**) disponibile. Durante l'addestramento con ottimizzatori classici come AdamW, i requisiti di memoria scalano rapidamente oltre la capacità fisica di una singola scheda video.

**DeepSpeed** è una libreria di ottimizzazione sviluppata da Microsoft che risolve questo problema. Il suo componente fondamentale è **ZeRO (Zero Redundancy Optimizer)**, una tecnologia di parallelizzazione della memoria che consente di addestrare modelli con miliardi di parametri riducendo al minimo le ridondanze.

---

## 1. Anatomia del Consumo di VRAM durante il Training

Per capire l'importanza di ZeRO, analizziamo dove viene consumata la memoria durante l'addestramento in precisione mista a 16-bit (FP16/BF16) con l'ottimizzatore **Adam**:

1. **Parametri del Modello (FP16/BF16):** Richiedono $2 \text{ byte}$ per parametro.
2. **Gradienti (FP16/BF16):** Richiedono $2 \text{ byte}$ per parametro.
3. **Stati dell'Ottimizzatore (Adam in FP32):**
   * Pesi Master in FP32 (per stabilità numerica): $4 \text{ byte}$.
   * Momento (Momentum) in FP32: $4 \text{ byte}$.
   * Varianza (Variance) in FP32: $4 \text{ byte}$.
   * **Totale Stati Ottimizzatore:** $12 \text{ byte}$ per parametro.

### Il Calcolo dei Pesi statici (OGP Memory):
Per ogni singolo parametro del modello, sono necessari:
$$\text{Memoria Statica} = 2 \text{ (pesi)} + 2 \text{ (gradienti)} + 12 \text{ (ottimizzatore)} = 16 \text{ byte per parametro}$$

Ad esempio, un modello da **7 miliardi di parametri (7B)** richiede:
$$7 \times 10^9 \times 16 \text{ byte} = 112 \text{ GB di VRAM}$$

Questo calcolo esclude le **attivazioni** (i tensori intermedi salvati durante il forward pass per calcolare i gradienti), che dipendono dalla lunghezza del contesto e dal batch size. Diventa evidente che un addestramento completo (*Full Fine-Tuning*) di un modello da 7B è impossibile su singole GPU consumer da 24GB (RTX 3090/4090) o enterprise da 80GB (A100/H100) senza tecniche di partizionamento.

---

## 2. I Tre Livelli di ZeRO (Zero Redundancy Optimizer)

Nelle tecniche tradizionali di *Data Parallelism (DP)*, ogni GPU mantiene una copia completa dei parametri, dei gradienti e degli stati dell'ottimizzatore. ZeRO elimina queste ridondanze dividendo i dati tra le GPU coinvolte nell'addestramento.

```
Visualizzazione del Partizionamento dei Dati (es. su 4 GPU):

DP Classico (Ridondanza Massima):
GPU 0: [ Parametri (100%) ] [ Gradienti (100%) ] [ Stati Ottimizzatore (100%) ]
GPU 1: [ Parametri (100%) ] [ Gradienti (100%) ] [ Stati Ottimizzatore (100%) ]

ZeRO-1 (Partizionamento Stati Ottimizzatore):
GPU 0: [ Parametri (100%) ] [ Gradienti (100%) ] [ Stati Ottimizzatore (25%)  ]
GPU 1: [ Parametri (100%) ] [ Gradienti (100%) ] [ Stati Ottimizzatore (25%)  ]

ZeRO-2 (Partizionamento Stati + Gradienti):
GPU 0: [ Parametri (100%) ] [ Gradienti (25%)  ] [ Stati Ottimizzatore (25%)  ]
GPU 1: [ Parametri (100%) ] [ Gradienti (25%)  ] [ Stati Ottimizzatore (25%)  ]

ZeRO-3 (Partizionamento Stati + Gradienti + Parametri):
GPU 0: [ Parametri (25%)  ] [ Gradienti (25%)  ] [ Stati Ottimizzatore (25%)  ]
GPU 1: [ Parametri (25%)  ] [ Gradienti (25%)  ] [ Stati Ottimizzatore (25%)  ]
```

### ZeRO-Stage 1: Partizionamento degli Stati dell'Ottimizzatore ($P_{os}$)
Gli stati dell'ottimizzatore Adam (12 byte per parametro) vengono divisi equamente tra le $N$ GPU. Ogni GPU aggiorna solo $1/N$ dei parametri totali. 
* **Risparmio:** Riduce la memoria degli stati dell'ottimizzatore di $N$ volte, abbattendo la memoria statica totale di circa il 4x per grossi cluster.

### ZeRO-Stage 2: Partizionamento dei Gradienti ($P_{g}$)
Oltre agli stati dell'ottimizzatore, anche i gradienti vengono partizionati. Ogni GPU mantiene in memoria e riduce solo i gradienti corrispondenti alla sua porzione di parametri da aggiornare.
* **Risparmio:** La memoria per gradienti e ottimizzatore si riduce drasticamente, lasciando praticamente solo i parametri del modello base come fattore statico.

### ZeRO-Stage 3: Partizionamento dei Parametri del Modello ($P_{p}$)
È il livello di partizionamento definitivo. Anche i parametri del modello vengono divisi tra le GPU. 
* **Funzionamento:** Durante il *Forward Pass*, quando uno strato specifico deve effettuare il calcolo, la GPU richiede i parametri mancanti dalle altre GPU tramite una comunicazione collettiva ultra-veloce (*All-Gather*). Eseguito il calcolo, i parametri esterni vengono immediatamente eliminati dalla memoria locale. Lo stesso avviene a ritroso nel *Backward Pass*.
* **Risparmio:** La memoria statica scala in modo lineare inversamente proporzionale al numero di GPU. Con un cluster sufficientemente grande, è possibile addestrare modelli da centinaia di miliardi di parametri.

---

## 3. ZeRO-Offload: Sfruttare la RAM di Sistema (CPU)

Nelle architetture hardware locali o self-hosted con una sola GPU o un numero ridotto di schede, **ZeRO-Offload** estende il concetto di ZeRO spostando i dati sulla memoria RAM di sistema (CPU) o su memorie SSD NVMe tramite il bus PCIe.

* **Offload degli Stati dell'Ottimizzatore (ZeRO-2 Offload):** Gli stati di AdamW FP32 (che occupano il 75% della memoria statica) vengono allocati interamente sulla RAM del computer. La CPU esegue i calcoli di aggiornamento dei pesi, mentre la GPU si concentra solo sul forward e backward pass.
* **Offload dei Parametri (ZeRO-3 Offload):** Anche le parti congelate o temporaneamente inattive del modello vengono allocate sulla RAM di sistema e trasferite sulla VRAM della GPU solo per il calcolo del layer attivo.

Sebbene l'offload introduca una latenza dovuta al trasferimento dati su PCIe, librerie moderne come DeepSpeed ottimizzano la pipeline sovrapponendo i calcoli della GPU con il trasferimento dei dati del blocco successivo (*Compute/Communication Overlap*), rendendo fattibile il fine-tuning di modelli da 13B o 34B su singole macchine da lavoro.

---

## Collegamenti Consigliati
* Per comprendere come configurare l'infrastruttura di training con queste tecniche, vedi [[Guida Completa al Fine-Tuning#Sintesi Operativa per l'Infrastruttura]].
* Per esplorare come la quantizzazione riduca i pesi all'origine riducendo il bisogno di ZeRO estremo, leggi [[Approfondimento NF4 e Quantizzazione]].
* Per consultare i termini hardware correlati, vedi [[Glossario e Concetti Chiave del Fine-Tuning#2. VRAM (Video RAM)]].

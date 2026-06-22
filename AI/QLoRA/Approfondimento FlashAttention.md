# Approfondimento Ingegneristico: FlashAttention (1, 2 e 3)

Il calcolo del meccanismo di auto-attenzione (*Self-Attention*) è l'operazione più onerosa all'interno delle architetture Transformer. Con l'aumentare della lunghezza del contesto ($N$), il costo computazionale e l'uso di memoria scalano in modo quadratico ($O(N^2)$).

**FlashAttention** è un algoritmo rivoluzionario progettato per velocizzare l'attenzione e ridurre drasticamente l'impronta di memoria della GPU, non riducendo il numero di operazioni matematiche (FLOP), ma ottimizzando l'uso della **gerarchia di memoria della GPU (I/O Bottleneck)**.

---

## 1. Il Collo di Bottiglia dell'Attenzione Classica (Memory-Bound)

Nelle GPU moderne, il collo di bottiglia della computazione spesso non è la velocità di calcolo dei chip (FLOPS dei Tensor Core), ma la velocità con cui i dati vengono trasferiti tra:
* **HBM (High Bandwidth Memory / VRAM):** La memoria video principale della GPU, capiente ma relativamente lenta da leggere/scrivere.
* **SRAM (Static RAM):** La memoria cache integrata direttamente sul chip GPU, velocissima ma di dimensioni molto ridotte (nell'ordine dei megabyte).

### L'Attenzione Classica (Standard Attention):
1. Carica le matrici di Query $Q$ e Key $K$ dalla HBM alla SRAM.
2. Calcola la matrice delle similarità $S = Q K^T$ ($N \times N$) e la **scrive nella HBM**.
3. Rilegge la matrice $S$ dalla HBM alla SRAM, calcola il Softmax $P = \text{softmax}(S)$ e **scrive $P$ ($N \times N$) nella HBM**.
4. Rilegge $P$ e Value $V$ dalla HBM alla SRAM, calcola l'output $O = P V$ e scrive $O$ nella HBM.

Questo andirivieni di matrici di dimensione quadratica $N \times N$ satura la larghezza di banda della VRAM, lasciando i core di calcolo della GPU inattivi (collo di bottiglia *Memory-Bound*).

---

## 2. FlashAttention-1: Tiling e Softmax Incrementale

**FlashAttention-1** elimina la necessità di scrivere e leggere le matrici $N \times N$ di dimensioni gigantesche dalla HBM, eseguendo l'intero calcolo in un unico passaggio all'interno della cache veloce SRAM.

```
Attenzione Classica (HBM-heavy):
[ Q, K ] in HBM -> Leggi in SRAM -> Calcola QK^T -> Scrivi S (N x N) in HBM -> Leggi S in SRAM -> Softmax -> Scrivi P (N x N) in HBM ...

FlashAttention (Tiling in SRAM):
[ Q, K, V ] suddivisi in Blocchi -> Caricati in SRAM -> Calcolo progressivo del Softmax (Online) -> Scrivi solo Output O in HBM
```

### Le due tecniche chiave:
1. **Tiling (Suddivisione in Blocchi):** Le matrici $Q, K, V$ vengono suddivise in blocchi che entrano interamente nella SRAM della GPU. L'attenzione viene calcolata blocco per blocco, accumulando i risultati intermedi.
2. **Softmax Online (Incrementale):** Poiché il Softmax classico richiede la somma di tutti gli elementi dell'intera riga per calcolare il denominatore:
   $$\text{softmax}(x)_i = \frac{e^{x_i - m}}{\sum e^{x_j - m}} \quad \text{dove } m = \max(x)$$
   Non sarebbe possibile calcolarlo a blocchi isolati. FlashAttention implementa una variante matematica del softmax che traccia e aggiorna dinamicamente due costanti per ogni riga durante la scansione dei blocchi: il massimo corrente $m^{(i)}$ e la somma parziale degli esponenziali $d^{(i)}$. Quando viene elaborato un nuovo blocco, i vecchi risultati intermedi vengono riscalati matematicamente al volo per riflettere i nuovi massimi e somme globali.

Durante il **Backward Pass**, FlashAttention non legge la matrice di attenzione dalla VRAM (poiché non è mai stata salvata). L'algoritmo **ricalcola al volo** la matrice di attenzione blocco per blocco in SRAM partendo da $O$ e dalle costanti del softmax salvate, un'operazione molto più rapida rispetto alla lettura di dati massivi da HBM.

---

## 3. FlashAttention-2: Ottimizzazione del Parallelismo e dei Warp

Rilasciata per ottimizzare ulteriormente i calcoli, **FlashAttention-2** introduce miglioramenti algoritmici mirati ad aumentare lo sfruttamento dei Tensor Core della GPU (fino a raggiungere il 70-75% del tetto massimo teorico delle GPU):

1. **Riduzione delle Operazioni Non-Matrice:** Modifica la sequenza matematica del Softmax online per ridurre il numero di moltiplicazioni ed esponentiazioni singole (non ottimizzate dai Tensor Core) a favore di moltiplicazioni di matrici pure (GEMM).
2. **Parallelizzazione sui Warp:** In una GPU, i thread lavorano in gruppi chiamati *warp*. FlashAttention-2 ottimizza la distribuzione del carico di lavoro dividendo il calcolo del blocco di attenzione tra i warp in modo da ridurre drasticamente la necessità di sincronizzazione e scrittura nella memoria condivisa tra i thread.
3. **Supporto Nativo per Sequenze Lunghe:** Permette di parallelizzare il calcolo non solo sulla dimensione del batch e delle testa di attenzione (*query heads*), ma anche sulla lunghezza stessa della sequenza, offrendo performance eccellenti per contesti da 32k a 128k token.

---

## 4. FlashAttention-3: Asincronicità e FP8 (Era Hopper e Blackwell)

Sulle GPU di architettura NVIDIA Hopper (es. H100) e Blackwell, **FlashAttention-3** introduce ottimizzazioni hardware-native estreme:

1. **Asincronicità dell'I/O (Overlapping):** Sfrutta il motore hardware **TMA (Tensor Memory Accelerator)** di Hopper. La GPU trasferisce i blocchi successivi di dati da HBM a SRAM in background, in modo asincrono, mentre i Tensor Core calcolano l'attenzione sul blocco corrente. Questo elimina completamente i tempi morti in cui la GPU attende che la memoria sia caricata.
2. **Supporto Nativo FP8:** Integra la computazione a precisione ridotta a 8-bit (FP8) mantenendo intatta la stabilità matematica grazie a tecniche di riscalamento dinamico dei quantili all'interno del Softmax online.
3. **Warp-Specialization:** Divide i thread GPU in due categorie specializzate: thread dedicati esclusivamente alla ricezione e al posizionamento dei dati, e thread dedicati solo alla computazione matematica dei Tensor Core, sincronizzati a livello hardware.

Questo permette a FlashAttention-3 di raggiungere velocità fino a **3 volte superiori** rispetto a FlashAttention-2 sulle schede H100/H200, riducendo i tempi di fine-tuning di ore su dataset massivi.

---

## Collegamenti Consigliati
* Per comprendere come FlashAttention si colloca nelle pipeline del 2026, consulta [[Approfondimento Tecnico QLoRA#C. Ingegneria del Dataset Sintetico e Validazione Automatizzata]].
* Per vedere dove FlashAttention interviene durante l'addestramento, leggi [[I 4 Step Fondamentali di Training#1. Forward Pass]].
* Per la terminologia legata all'attenzione, vedi [[Glossario e Concetti Chiave del Fine-Tuning#11. FlashAttention]].

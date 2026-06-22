# Approfondimento Matematico: DoRA (Weight-Decomposed Low-Rank Adaptation)

La tecnica **DoRA (Weight-Decomposed Low-Rank Adaptation)**, introdotta come evoluzione di LoRA, punta a colmare il divario prestazionale tra i metodi PEFT (Parameter-Efficient Fine-Tuning) e il *Full Fine-Tuning* tradizionale, mantenendo invariata l'efficienza computazionale all'inferenza.

Questo documento analizza la scomposizione matematica di DoRA, le differenze con LoRA e i motivi della sua superiore capacità di astrazione.

---

## 1. Il Limite di LoRA

In LoRA standard, la matrice dei pesi aggiornata $W \in \mathbb{R}^{d \times k}$ è definita come:

$$W = W_0 + \Delta W = W_0 + \frac{\alpha}{r} (B \cdot A)$$

Dove $W_0$ è la matrice pre-addestrata (congelata) e $B \cdot A$ è la decomposizione a basso rango dell'aggiornamento. 

Ricerche empiriche hanno dimostrato che il *Full Fine-Tuning* mostra una spiccata capacità di modificare in modo indipendente la **magnitudo (intensità)** e la **direzione** dei vettori dei pesi. LoRA invece accoppia intrinsecamente questi due aspetti: qualsiasi aggiornamento $\Delta W$ altera contemporaneamente e in modo rigidamente proporzionale sia la forza che la direzione del neurone. Questo limite strutturale rallenta o impedisce l'apprendimento ottimale in compiti complessi o molto distanti dal dominio di pre-training.

---

## 2. La Formulazione Matematica di DoRA

DoRA risolve questo vincolo introducendo una scomposizione del peso ispirata alla *Weight Normalization*. Qualsiasi matrice di pesi $W$ viene scomposta in:
1. Un vettore di magnitudo $m \in \mathbb{R}^{1 \times k}$ che scala la forza di ogni colonna (neurone).
2. Una matrice di direzione $V \in \mathbb{R}^{d \times k}$ che definisce l'orientamento spaziale dei pesi.

La formula fondamentale di DoRA definisce il peso finale come:

$$W = m \frac{V}{\|V\|_c}$$

Dove $\|\cdot\|_c$ rappresenta la **norma Euclidea ($L_2$) calcolata colonna per colonna**:

$$\|V\|_c = \left[ \|v_1\|_2, \|v_2\|_2, \dots, \|v_k\|_2 \right]$$

### Iniezione di LoRA nella Direzione
Per ottimizzare i parametri in modo efficiente, DoRA congela la direzione iniziale impostando $V = W_0$ e applica l'adattamento LoRA esclusivamente alla matrice di direzione $V$. La formula diventa quindi:

$$W = m \frac{W_0 + \Delta W}{\|W_0 + \Delta W\|_c} = m \frac{W_0 + B \cdot A}{\|W_0 + B \cdot A\|_c}$$

* **Parametri Addestrabili:** In DoRA, gli unici parametri addestrati sono il vettore di magnitudo $m$ (molto piccolo, pari a $k$ parametri) e le matrici LoRA $A$ e $B$.
* **Inizializzazione:** Per garantire che il modello parta esattamente dallo stato pre-addestrato, $m$ viene inizializzato come la norma delle colonne di $W_0$:
  $$m_{init} = \|W_0\|_c$$
  Mentre la matrice LoRA $A$ viene inizializzata con distribuzione gaussiana e $B$ a zero, rendendo inizialmente $\Delta W = 0$.

---

## 3. Perché DoRA è Superiore a LoRA

La scomposizione di DoRA introduce una dinamica di apprendimento che imita fedelmente il Full Fine-Tuning.

```
Visualizzazione concettuale degli aggiornamenti dei pesi:

   Full Fine-Tuning                 LoRA Standard                    DoRA (Weight-Decomposed)
      (Flessibile)               (Accoppiamento Rigido)                 (Indipendente)
         ^                               ^                                     ^
         |  / Direzione                  |  /                                  |  / Direzione
Magnitudo| /                  Magnitudo  | /  (Direzione e                  Magnitudo| /____
         |/____                          |/   Magnitudo cambiano               |/    
         +------>                        +------> assieme)                     +------>
```

1. **Decoupling (Disaccoppiamento):** DoRA consente al modello di apportare modifiche minime alla direzione e modifiche massicce alla magnitudo (o viceversa). Questa flessibilità permette di apprendere concetti verticali difficili senza distorcere la rappresentazione spaziale dei pesi pre-esistenti.
2. **Stabilità con Modelli Quantizzati (QDoRA):** DoRA può essere combinato con la quantizzazione NF4 (diventando QDoRA). In questo scenario, $W_0$ è congelato in NF4 a 4-bit, mentre la scomposizione in magnitudo e direzione a 16-bit permette di recuperare quasi il 100% dell'accuratezza persa a causa della quantizzazione.
3. **Nessun Overhead all'Inferenza (Zero Inference Latency):** Una volta completato il training, le matrici addestrate $m$, $A$ e $B$ possono essere fuse matematicamente (*folded*) nella matrice originaria $W_0$ calcolando la formula di scomposizione. Il modello risultante ha la stessa identica architettura e velocità del modello originale base, senza richiedere computazioni aggiuntive durante l'uso.

---

## Collegamenti Consigliati
* Per comprendere come implementare DoRA all'interno di un framework avanzato, vedi [[Approfondimento Tecnico QLoRA#3. Le Migliori Metodologie di Fine-Tuning per il 2026]].
* Per esplorare i parametri correlati come *Rank* ($r$) e *Alpha* ($\alpha$), consulta [[I 5 Iperparametri di Configurazione QLoRA]].
* Per confrontare DoRA con altri metodi di allineamento, leggi [[Approfondimento ORPO]].

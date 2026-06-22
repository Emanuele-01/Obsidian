# I 5 Iperparametri di Configurazione (Strutturali) in QLoRA

Quando si effettua il fine-tuning di un modello di linguaggio utilizzando **[[Glossario e Concetti Chiave del Fine-Tuning#7. QLoRA (Quantized LoRA)|QLoRA (Quantized Low-Rank Adaptation)]]**, la corretta configurazione degli iperparametri determina non solo l'efficienza nell'uso della VRAM, ma anche la capacità del modello di apprendere compiti complessi senza perdere la sua conoscenza di base (*overfitting* o *catastrophic forgetting*).

Di seguito vengono analizzati nel dettaglio i 5 iperparametri strutturali chiave di configurazione:

---

## 1. Target Modules (Moduli Target)

I **Target Modules** definiscono quali strati lineari (*linear layers*) all'interno dell'architettura del modello base riceveranno le matrici adattatrici LoRA. 

* **Cosa sono:** In un modello basato su Transformer (come Llama o Mistral), l'attenzione è divisa in diversi moduli di proiezione: `q_proj` (query), `k_proj` (key), `v_proj` (value) e `o_proj` (output), oltre ai moduli del Feed-Forward Network (`gate_proj`, `up_proj`, `down_proj`).
* **Best Practice:** Nei primi anni di LoRA, si applicavano gli adattatori solo a `q_proj` e `v_proj` per risparmiare memoria. Oggi, lo standard industriale prevede di **selezionare tutti i moduli lineari** (spesso indicati nei file di configurazione come `target_modules: all-linear` o elencandoli singolarmente). Questo approccio offre una capacità di apprendimento nettamente superiore e colma quasi del tutto il divario prestazionale rispetto a un *Full Fine-Tuning*, aumentando l'uso della VRAM in modo trascurabile.

---

## 2. r (Rank / Rango)

Il **[[Glossario e Concetti su QLoRA#6. Matrici di Rango Inferiore (Low-Rank Matrices / Adapters)|Rango ($r$)]]** determina la dimensione interna (la "larghezza" o il collo di bottiglia) delle matrici di decomposizione introdotte da LoRA.

* **Cosa fa:** Definisce il numero di parametri addestrabili negli adattatori. Più il rango è alto, più la matrice intermedia è grande, consentendo al modello di memorizzare pattern e informazioni più complessi. Di contro, un rango più elevato aumenta il consumo di VRAM e il rischio di overfitting.
* **Valori tipici e linee guida:**
  * **$r = 8$ o $16$:** Ideale per compiti di allineamento stilistico, cambio di tono di voce o compiti di classificazione semplici.
  * **$r = 32$ o $64$:** Lo standard raccomandato per scenari verticali (es. insegnare al modello a scrivere codice in un linguaggio specifico, rispondere rispettando schemi JSON complessi o apprendere una documentazione tecnica privata).
  * **$r = 128$ o superiore:** Utilizzato raramente, solo per domini radicalmente diversi dal dataset di pre-training (es. linguaggi di programmazione esotici o formule matematiche avanzate), spesso abbinato a varianti stabili come [[Glossario e Concetti su QLoRA#8. rsLoRA (Rank-Stabilized LoRA)|rsLoRA]].

---

## 3. Alpha (LoRA Alpha)

**[[Glossario e Concetti su QLoRA#6. Matrici di Rango Inferiore (Low-Rank Matrices / Adapters)|Alpha ($\alpha$)]]** è il fattore di scala costante applicato ai pesi delle matrici LoRA durante il calcolo dei gradienti. Funziona come un moltiplicatore che determina l'impatto degli adattatori rispetto alla conoscenza del modello base congelato.

* **La formula di scaling:** Il contributo della matrice LoRA viene pesato sul modello base secondo il rapporto: $\text{Scaling} = \frac{\alpha}{r}$.
* **Come configurarlo:** La regola empirica consolidata prevede di impostare **$\alpha$ pari al doppio del valore del Rango ($\alpha = 2r$)** (ad esempio, se $r = 32$, si imposta $\alpha = 64$; se $r = 64$, si imposta $\alpha = 128$). Mantenere questo rapporto fisso stabilizza la grandezza dei gradienti durante l'addestramento e permette di variare il rango senza dover ricalibrare drasticamente il tasso di apprendimento (*Learning Rate*).

---

## 4. Quantization (Quantizzazione del Modello Base)

La **Quantizzazione** definisce il formato di compressione con cui il modello base pre-addestrato viene caricato in memoria prima di agganciare le matrici LoRA. È l'elemento che differenzia QLoRA dal LoRA classico.

* **Configurazione Standard:** In QLoRA si utilizza nativamente la quantizzazione a **[[Approfondimento NF4 e Quantizzazione#1. Il tipo di dato NF4 (NormalFloat 4)|4-bit NF4 (NormalFloat4)]]**. Questo formato comprime i pesi a 16-bit originali in una rappresentazione a 4-bit ottimizzata per la distribuzione statistica dei parametri dei Transformer.
* **Doppia Quantizzazione:** Si raccomanda di abilitare sempre la [[Approfondimento NF4 e Quantizzazione#2. Doppia Quantizzazione (Double Quantization)|Doppia Quantizzazione]] per risparmiare circa $0.37$ bpw, riducendo l'uso complessivo di VRAM.
* **Impatto:** Consente di abbattere l'occupazione della VRAM del modello base del ~75%. Ad esempio, un modello da 8 miliardi di parametri (8B), che normalmente richiederebbe circa 16 GB di VRAM solo per essere caricato in precisione nativa (BF16), occupa solo circa 5.5 GB in NF4 a 4-bit, lasciando il resto della memoria della GPU a disposizione del contesto e dei calcoli di addestramento.

---

## 5. Dropout (LoRA Dropout)

Il **Dropout** è una tecnica di regolarizzazione fondamentale per prevenire l'overfitting, in particolare quando il dataset di fine-tuning è di dimensioni ridotte.

* **Cosa fa:** Durante ogni iterazione dell'addestramento (*step*), il Dropout disattiva casualmente una determinata percentuale di nodi (valori) all'interno delle matrici LoRA, impostandoli a zero. Questo costringe la rete neurale a non fare affidamento su specifici percorsi o parametri di memoria, spingendola a sviluppare una rappresentazione più robusta e generalizzabile del compito.
* **Valori tipici:**
  * **$0.0$ (Disattivato):** Utilizzato solo se si dispone di dataset sintetici massivi e perfettamente bilanciati, o quando si punta alle massime performance di pura memorizzazione.
  * **$0.05$ o $0.1$ (Raccomandato):** Lo standard operativo. Escludere il 5% o il 10% dei parametri a ogni ciclo protegge il modello dal rischio di replicare a memoria le risposte del dataset, garantendo una migliore flessibilità di inferenza su input inediti.
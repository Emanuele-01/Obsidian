# Approfondimento Matematico: ORPO (Odds Ratio Preference Optimization)

Nel panorama del post-training e dell'allineamento dei modelli di linguaggio (LLM), l'introduzione di **ORPO (Odds Ratio Preference Optimization)** ha rivoluzionato il modo in che i modelli vengono addestrati a seguire le preferenze umane, eliminando la necessità di pipeline multi-step complesse e costose in termini di risorse hardware.

Questo documento descrive la teoria, la formulazione matematica e i vantaggi computazionali di ORPO rispetto ai metodi precedenti (SFT, RLHF e DPO).

---

## 1. Il Problema dell'Allineamento Tradizionale (SFT + DPO/RLHF)

Fino a poco tempo fa, l'allineamento di un LLM richiedeva una pipeline sequenziale a più fasi:
1. **Supervised Fine-Tuning (SFT):** Il modello impara la struttura e lo stile dei dialoghi tramite esempi positivi. Il limite è che durante l'SFT aumenta anche la probabilità logaritmica (*log-probability*) di parole o stili indesiderati presenti implicitamente nel dataset.
2. **Direct Preference Optimization (DPO) o RLHF:** In una fase separata, si insegna al modello a scegliere la risposta preferita ($y_w$, *winning*) rispetto a quella rifiutata ($y_l$, *losing*).

### Il limite di DPO:
DPO richiede due modelli attivi in memoria contemporaneamente:
* Il modello in fase di addestramento (policy attiva $\pi_\theta$).
* Un modello di riferimento congelato (policy di riferimento $\pi_{ref}$, tipicamente il modello derivato dall'SFT).

Questo raddoppia i requisiti di memoria GPU (VRAM) e richiede una complessa calibrazione preliminare (SFT) che rende la pipeline instabile se il dataset SFT e quello delle preferenze non sono perfettamente allineati.

---

## 2. La Formulazione Matematica di ORPO

**ORPO** elimina interamente la necessità di un modello di riferimento e fonde l'SFT e l'allineamento delle preferenze in un **unico step di addestramento**.

La funzione di perdita (*Loss Function*) di ORPO è definita come:

$$\mathcal{L}_{ORPO} = \mathbb{E}_{(x, y_w, y_l)} \left[ \mathcal{L}_{SFT} + \lambda \mathcal{L}_{OR} \right]$$

Dove:
* $x$ è il prompt di input.
* $y_w$ è la risposta preferita (generata correttamente).
* $y_l$ è la risposta rifiutata (allucinata, errata o stilisticamente scadente).
* $\lambda$ è un iperparametro di scaling (tipicamente impostato tra $0.1$ e $0.5$).

### A. La componente SFT ($\mathcal{L}_{SFT}$)
È la classica entropia incrociata (*cross-entropy loss*) calcolata esclusivamente sulla risposta preferita $y_w$:

$$\mathcal{L}_{SFT} = -\frac{1}{|y_w|} \log P_\theta(y_w | x)$$

Questa componente spinge il modello ad apprendere la struttura grammaticale e la formattazione corretta della risposta target.

### B. La componente Odds Ratio ($\mathcal{L}_{OR}$)
La vera innovazione di ORPO risiede nel calcolo dell'**Odds Ratio**. L'Odds (probabilità a favore) di una sequenza $y$ rispetto a un input $x$ sotto i parametri del modello $\theta$ è definita come:

$$\text{Odds}_\theta(y | x) = \frac{P_\theta(y | x)}{1 - P_\theta(y | x)}$$

L'Odds indica quanto è più probabile che il modello generi la sequenza $y$ rispetto a non generarla. La perdita di Odds Ratio penalizza il modello se la probabilità della risposta rifiutata $y_l$ si avvicina a quella della risposta preferita $y_w$:

$$\mathcal{L}_{OR} = - \log \sigma \left( \log \frac{\text{Odds}_\theta(y_w | x)}{\text{Odds}_\theta(y_l | x)} \right)$$

Dove $\sigma$ è la funzione sigmoide. Se il modello inizia a preferire $y_w$ rispetto a $y_l$, il rapporto degli Odds cresce, spingendo la sigmoide verso $1$ e riducendo a zero il valore della perdita $\mathcal{L}_{OR}$.

---

## 3. Vantaggi Ingegneristici e Pratici di ORPO

```
Confronto delle Pipeline di Allineamento:

Metodo Classico (SFT + DPO):
[ Dataset SFT ] -> Training SFT -> [ Modello SFT ] 
                                           |
                                           v
[ Dataset Preferenze ] ------------------> Training DPO (Richiede Modello SFT + Modello DPO in VRAM) -> [ Modello Finito ]

Metodo ORPO:
[ Dataset Preferenze (x, yw, yl) ] -> Training ORPO (Singolo step, 1 solo modello in VRAM) -> [ Modello Finito ]
```

1. **Efficienza Hardware (VRAM):** Poiché non è necessario caricare un modello di riferimento ($\pi_{ref}$), l'addestramento consuma circa la metà della VRAM rispetto a DPO. Questo rende l'allineamento delle preferenze applicabile su singole GPU commerciali tramite QLoRA.
2. **Velocità di Convergenza:** ORPO riduce il tempo di addestramento complessivo del **50-60%**, dimezzando i cicli di calcolo rispetto alla pipeline sequenziale SFT + DPO.
3. **Prevenzione del "Catastrophic Forgetting":** Nelle pipeline tradizionali, durante la fase DPO il modello rischia di "dimenticare" come strutturare frasi complesse o formattare codice per focalizzarsi solo sulla preferenza. Poiché ORPO mantiene la componente $\mathcal{L}_{SFT}$ attiva per tutta la durata del processo, il modello preserva intatta la sua ricchezza espressiva e la fedeltà al formato.

---

## Collegamenti Consigliati
* Per vedere come ORPO si inserisce nel panorama del fine-tuning del 2026, leggi [[Approfondimento Tecnico QLoRA#B. Allineamento Singolo Step (Post-SFT)]].
* Per comprendere come il calcolo della loss impatta i pesi adattatori, vedi [[I 4 Step Fondamentali di Training#2. Loss Calculation]].
* Per la terminologia legata all'allineamento, vedi [[Glossario e Concetti Chiave del Fine-Tuning#10. DPO e ORPO (Allineamento delle Preferenze)]].

# Approfondimento Matematico: DPO (Direct Preference Optimization)

L'allineamento dei modelli di linguaggio con le preferenze umane è stato storicamente dominato da **RLHF (Reinforcement Learning from Human Feedback)**. Tuttavia, la complessità dell'addestramento basato su algoritmi di policy gradient (come PPO) ha spinto la ricerca verso soluzioni più stabili. **DPO (Direct Preference Optimization)** rappresenta una pietra miliare in questo campo, in quanto dimostra come ottimizzare le preferenze direttamente senza addestrare un modello di ricompensa separato.

Questo documento analizza la derivazione matematica di DPO e il suo flusso operativo.

---

## 1. La Matematica Dietro DPO: Dalla Ricompensa alla Loss Diretta

Nel RLHF tradizionale, l'obiettivo è massimizzare la ricompensa attesa $r(x, y)$ penalizzando la divergenza dal modello di partenza (policy di riferimento $\pi_{ref}$) per evitare che il modello "impazzisca" (*reward hacking*):

$$\max_{\pi} \mathbb{E}_{x \sim \mathcal{D}, y \sim \pi(y|x)} [r(x, y)] - \beta \mathbb{D}_{KL}(\pi(y|x) \,\|\, \pi_{ref}(y|x))$$

Dove $\mathbb{D}_{KL}$ è la divergenza di Kullback-Leibler e $\beta$ controlla l'intensità del vincolo.

### La Reparametrizzazione Chiave
La teoria dell'ottimizzazione vincolata dimostra che la soluzione ottimale $\pi^*$ per questo problema ha una forma chiusa che dipende direttamente dalla ricompensa $r(x, y)$:

$$\pi^*(y|x) = \frac{\pi_{ref}(y|x) \exp\left(\frac{1}{\beta} r(x, y)\right)}{Z(x)}$$

Dove $Z(x) = \sum_y \pi_{ref}(y|x) \exp\left(\frac{1}{\beta} r(x, y)\right)$ è la funzione di partizione (costante di normalizzazione). 

Risolvendo questa equazione per $r(x, y)$, ricaviamo la funzione di ricompensa espressa unicamente in termini di log-probabilità della policy ottimale e della policy di riferimento:

$$r(x, y) = \beta \log \frac{\pi^*(y|x)}{\pi_{ref}(y|x)} + \beta \log Z(x)$$

### Il Modello di Preferenza Bradley-Terry
Nelle coppie di preferenza, la probabilità che un essere umano preferisca la risposta $y_w$ (winning) alla risposta $y_l$ (losing) dato il prompt $x$ viene modellata tramite la formula di **Bradley-Terry**:

$$P(y_w \succ y_l \mid x) = \sigma\big(r(x, y_w) - r(x, y_l)\big)$$

Sostituendo l'espressione di $r(x, y)$ ricavata in precedenza all'interno del modello di Bradley-Terry, la funzione di normalizzazione $Z(x)$ si cancella matematicamente:

$$P(y_w \succ y_l \mid x) = \sigma\left( \beta \log \frac{\pi^*(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi^*(y_l|x)}{\pi_{ref}(y_l|x)} \right)$$

### La Loss Function di DPO
Sfruttando questa identità, possiamo definire la perdita dell'ottimizzazione tramite la classica *binary cross-entropy loss* sul dataset di preferenze $\mathcal{D}$:

$$\mathcal{L}_{DPO}(\theta; \pi_{ref}) = - \mathbb{E}_{(x, y_w, y_l) \sim \mathcal{D}} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(y_w \mid x)}{\pi_{ref}(y_w \mid x)} - \beta \log \frac{\pi_\theta(y_l \mid x)}{\pi_{ref}(y_l \mid x)} \right) \right]$$

In questo modo, la ricompensa implicita viene appresa direttamente aggiornando i pesi $\theta$ della policy $\pi_\theta$.

---

## 2. Il Flusso Operativo di DPO

Durante l'addestramento, il calcolo della loss richiede di passare lo stesso input $x$ e le risposte $y_w$ e $y_l$ attraverso due modelli distinti:

```
                  +-----------------------------------+
                  |         Prompt di Input (x)       |
                  +-------------------+---------------+
                                      |
                     +----------------+----------------+
                     |                                 |
                     v                                 v
        +------------+------------+       +------------+------------+
        | Policy Attiva \pi_\theta |       | Ref Policy \pi_{ref}   |
        |      (Addestrabile)     |       |      (Congelata)        |
        +------------+------------+       +------------+------------+
                     |                                 |
           Calcola log-prob per              Calcola log-prob per
             y_w e y_l                        y_w e y_l
                     |                                 |
                     +----------------+----------------+
                                      |
                                      v
                        +-------------+-------------+
                        |  Calcolo della Loss DPO   |
                        |     (Binary Cross-Ent)    |
                        +---------------------------+
```

1. **Forward Pass sul Modello Attivo:** Si calcolano $\log \pi_\theta(y_w|x)$ e $\log \pi_\theta(y_l|x)$.
2. **Forward Pass sul Modello di Riferimento:** Si calcolano $\log \pi_{ref}(y_w|x)$ e $\log \pi_{ref}(y_l|x)$.
3. **Calcolo della Loss:** Si applica la formula di DPO e si esegue il backward pass per aggiornare i pesi $\theta$.

---

## 3. Limiti di DPO e Evoluzione in ORPO

Sebbene DPO eviti l'instabilità di PPO, presenta due svantaggi significativi:
* **Impronta di Memoria (VRAM):** Mantenere sia $\pi_\theta$ che $\pi_{ref}$ in memoria raddoppia i requisiti hardware per i pesi base del modello.
* **Separazione delle Fasi:** DPO assume che la policy sia già stata parzialmente allineata con l'SFT. Se si esegue DPO direttamente su un modello base non ottimizzato, l'addestramento fallisce rapidamente.

Per superare queste limitazioni, nel 2026 lo standard industriale si è spostato verso **ORPO**, che elimina del tutto $\pi_{ref}$ e unifica SFT e preferenze in un unico step logico.

---

## Collegamenti Consigliati
* Per vedere l'evoluzione di DPO ad allineamenti a singolo step, leggi [[Approfondimento ORPO]].
* Per comprendere come DPO si colloca nel flusso completo di post-addestramento, consulta [[Guida Completa al Fine-Tuning#B. Allineamento Avanzato e Post-Training (Sostituti di RLHF)]].
* Per i concetti terminologici di base, vedi [[Glossario e Concetti Chiave del Fine-Tuning#10. DPO e ORPO (Allineamento delle Preferenze)]].

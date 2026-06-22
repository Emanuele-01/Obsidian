# Approfondimento Matematico: NF4 e Doppia Quantizzazione in QLoRA

La quantizzazione è il cuore pulsante di **QLoRA (Quantized Low-Rank Adaptation)**. Questo documento descrive in dettaglio la matematica e l'ingegneria dietro il tipo di dato **NF4 (NormalFloat 4)** e la **Doppia Quantizzazione (Double Quantization)**, spiegando come permettano di abbattere l'uso di memoria grafica senza compromettere l'accuratezza del modello.

---

## 1. Il tipo di dato NF4 (NormalFloat 4)

La maggior parte dei metodi di quantizzazione tradizionali (come INT4 o FP4) suddivide l'intervallo numerico in intervalli lineari o logaritmici uniformi. Questo approccio è sub-ottimale per i pesi delle reti neurali profonde (LLM), che per natura seguono una **distribuzione statistica gaussiana (normale)** con media zero e varianza finita:

$$W \sim \mathcal{N}(0, \sigma^2)$$

Il formato **NF4** risolve questo problema applicando la **Quantizzazione per Quantili (Information-Theoretic Quantile Quantization)**. L'obiettivo è fare in modo che ciascuno dei 16 valori discreti rappresentabili con 4 bit abbia la stessa probabilità teorica di essere assegnato. In questo modo, l'informazione trasportata da ciascun livello quantizzato viene massimizzata.

### Calcolo dei Quantili di NF4
I 16 punti di quantizzazione ($q_i$) vengono ricavati dividendo l'area della funzione di distribuzione cumulativa (CDF) di una gaussiana standardizzata $\mathcal{N}(0, 1)$ in intervalli di uguale probabilità:

1. Si determinano i quantili di una gaussiana per ricavare un tipo di dato simmetrico a 4 bit.
2. Poiché il valore esatto di `0` è fondamentale per rappresentare i pesi nulli (e i padding), lo schema viene modificato per garantire uno zero esatto (`0.0`).
3. L'intervallo risultante viene normalizzato nell'intervallo $[-1, 1]$.

I 16 valori esatti della mappa di quantizzazione NF4 standard (utilizzati nelle librerie come `bitsandbytes`) sono:

```
q = [-1.0, -0.6961917, -0.5250797, -0.3949175, 
     -0.2844414, -0.1847734, -0.0910502,  0.0, 
      0.0795803,  0.1609302,  0.2461159,  0.3379152, 
      0.4431098,  0.5715682,  0.7250228,  1.0]
```

### Formula di De-quantizzazione
Durante il calcolo dello strato lineare (moltiplicazione tra matrici), il peso quantizzato a 4 bit $W^{NF4}$ viene convertito al volo nel formato di attivazione a 16 bit (es. $BF16$) tramite il fattore di scala (costante di quantizzazione) $c$:

$$\tilde{W}^{BF16} = c \cdot q(W^{NF4})$$

---

## 2. Doppia Quantizzazione (Double Quantization)

Per quantizzare una matrice di pesi senza perdere precisione, i parametri vengono divisi in blocchi indipendenti (es. blocchi di dimensione $B = 64$). Per ogni blocco viene calcolato un fattore di scala $c$ (un numero FP32).

Sebbene questo approccio limiti l'errore di quantizzazione, introduce un sovraccarico di memoria:
* Con un blocco di dimensione 64, ogni parametro riceve un contributo di memoria dal fattore di scala pari a:
  $$\frac{32 \text{ bit (FP32)}}{64 \text{ pesi}} = 0.5 \text{ bit per parametro (bpw)}$$

La **Doppia Quantizzazione (DQ)** riduce questo overhead quantizzando a sua volta i fattori di scala $c$:

```
[ Pesi Base (16-bit) ] --(Quantizzazione 1: NF4, Blocco 64)--> [ Pesi Quantizzati (4-bit) ] + [ Costanti c_1 (FP32) ]
                                                                                                    |
                                                                                    (Quantizzazione 2: FP8, Blocco 256)
                                                                                                    v
                                                                                   [ Costanti c_1 (FP8) ] + [ Costanti c_2 (FP32) ]
```

### Il Processo Matematico di DQ:
1. I fattori di scala $c_1^{FP32}$ della prima quantizzazione (blocco $B_1 = 64$) vengono raggruppati in blocchi più grandi di dimensione $B_2 = 256$.
2. Questi fattori di scala vengono quantizzati in formato **FP8 a 8 bit** (o integer a 8 bit), generando la costante compressa $c_1^{FP8}$ e un secondo set di fattori di scala di secondo livello $c_2^{FP32}$.
3. Il risparmio di memoria risultante è significativo:
   $$\text{Memory Overhead (senza DQ)} = \frac{32}{64} = 0.500 \text{ bpw}$$
   $$\text{Memory Overhead (con DQ)} = \frac{8}{64} + \frac{32}{64 \times 256} = 0.125 + 0.002 = 0.127 \text{ bpw}$$
4. **Risparmio effettivo:** circa $0.373$ bit per parametro. Su un modello da 65 miliardi di parametri (65B), questo si traduce in circa **3 GB di VRAM risparmiati**, permettendo di allocare contesti più lunghi o batch size maggiori.

---

## 3. Flusso dei Calcoli e De-quantizzazione al Volo (On-the-Fly)

Una delle idee chiave di QLoRA è che il modello base a 4-bit non viene mai addestrato; i calcoli dei gradienti avvengono solo sulle matrici LoRA (adattatori) a 16-bit.

Durante il **Forward Pass** e il **Backward Pass**, il calcolo per uno strato lineare con input $X$ e pesi $W$ avviene seguendo questi step:

1. **Lettura:** I pesi $W^{NF4}$ (4-bit) e le costanti doppie $c_1^{FP8}, c_2^{FP32}$ vengono letti dalla VRAM.
2. **De-quantizzazione:** I pesi della matrice base vengono ricostruiti in memoria temporanea (cache della GPU):
   $$W^{BF16} = \text{dequant}(c_1^{FP8}, c_2^{FP32}) \cdot q(W^{NF4})$$
3. **Calcolo della Proiezione:** Viene calcolato il prodotto con l'input $X$:
   $$Y = X \cdot W^{BF16} + X \cdot A \cdot B$$
   Dove $A \in \mathbb{R}^{d \times r}$ e $B \in \mathbb{R}^{r \times k}$ sono le due matrici adattatrici LoRA a bassa dimensione (rango $r$) tenute costantemente a 16-bit.
4. **Liberazione Memoria:** Non appena il calcolo di $Y$ è completato per lo strato corrente, la matrice de-quantizzata $W^{BF16}$ viene immediatamente rimossa dalla memoria cache, mantenendo in VRAM solo la rappresentazione a 4-bit.

---

## Collegamenti Consigliati
* Per comprendere come questo flusso si inserisce nell'addestramento complessivo, vedi [[I 4 Step Fondamentali di Training]].
* Per impostare i parametri corretti relativi alla quantizzazione, consulta [[I 5 Iperparametri di Configurazione QLoRA]].
* Per la terminologia correlata, vedi [[Glossario e Concetti su QLoRA]] e [[Glossario e Concetti Chiave del Fine-Tuning]].

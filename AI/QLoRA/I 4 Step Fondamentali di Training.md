# I 4 Step Fondamentali del Ciclo di Training in QLoRA

Nell'addestramento dei modelli di linguaggio con **QLoRA (Quantized Low-Rank Adaptation)**, il processo di apprendimento si sviluppa attraverso un ciclo iterativo continuo composto da 4 step logici e matematici precisi. L'obiettivo fondamentale di questa pipeline è modificare i parametri addestrabili (gli adattatori LoRA) basandosi sui dati di addestramento, garantendo al contempo che il modello finale sia in grado di **generalizzare** su dati inediti e mai visti in precedenza.

Di seguito viene analizzato nel dettaglio ogni singolo step, evidenziando le peculiarità computazionali introdotte dalla quantizzazione di QLoRA:

---

## 1. Forward Pass (Passaggio in Avanti)

Il **Forward Pass** è la fase iniziale in cui il modello riceve i dati di input (i token del testo) e calcola una previsione probabilistica del risultato (*output prediction*).

* **Cosa succede matematicamente:** I dati fluiscono attraverso i vari strati della rete neurale. Ogni strato esegue moltiplicazioni tra matrici sfruttando i pesi interni del modello per mappare le relazioni tra le parole.
* **La particolarità in QLoRA:** Poiché il modello base è congelato e memorizzato nel formato compresso a **4-bit NF4**, la GPU non può eseguire direttamente i calcoli matematici in questa forma. Durante il forward pass, i pesi a 4-bit vengono **de-quantizzati al volo** in formato a 16-bit (*Bfloat16* o *Float16*), combinati temporaneamente con i pesi degli adattatori LoRA a 16-bit per calcolare l'output, e poi immediatamente rimossi dalla cache.

---

## 2. Loss Calculation (Calcolo della Perdita)

Una volta ottenuta la previsione dal forward pass, entra in gioco la **Loss Calculation**, ovvero la misurazione quantitativa dell'errore commesso dal modello.

* **Cosa succede matematicamente:** Il sistema confronta la previsione generata dal modello (*predict*) con la risposta ideale o reale fornita all'interno del dataset di addestramento, nota in gergo come **Ground Truth** (la verità sul campo).
* **Significato logico:** La funzione di perdita (es. *Cross-Entropy Loss*) genera un singolo valore numerico (la *Loss*). Più questo valore è vicino allo zero, più la risposta del modello è identica a quella desiderata. Questo valore funge da bussola per l'intero sistema: definisce l'esatta entità del divario che separa lo stato attuale del modello dalla precisione assoluta.

---

## 3. Backward Pass (Passaggio all'Indietro)

Il **Backward Pass** (o retropropagazione dell'errore) è la fase in cui il valore della *Loss* viene analizzato a ritroso attraverso l'architettura della rete per comprendere come correggere i parametri.

* **Cosa succede matematicamente:** Sfruttando il calcolo delle derivate parziali e la regola della catena (*chain rule*), l'algoritmo calcola i **gradienti**. Un gradiente è un vettore che indica la direzione e l'intensità di come ogni singolo parametro dovrebbe essere modificato (aumentato o diminuito) per ridurre il valore della perdita nel ciclo successivo.
* **La particolarità in QLoRA:** Questo è il momento in cui QLoRA dimostra la sua massima efficienza energetica e hardware. I gradienti **non vengono calcolati per i miliardi di parametri del modello base** (che rimangono congelati a 4-bit senza subire variazioni), ma vengono generati e tracciati in memoria **esclusivamente per i piccoli moduli adattatori LoRA** a 16-bit posizionati nei target modules. Ciò consente un risparmio mastodontico di memoria video durante la fase più critica del training.

---

## 4. Optimization (Ottimizzazione)

L'ultimo step del ciclo è l'**Optimization**, ovvero l'applicazione pratica delle correzioni calcolate nella fase precedente per aggiornare i pesi effettivi del modello.

* **Cosa succede matematicamente:** L'ottimizzatore (es. *AdamW a 8-bit*) prende i gradienti accumulati durante il backward pass e, pesandoli in base al valore del *Learning Rate* impostato, aggiorna i coefficienti numerici delle matrici adattatrici compiendo un piccolo passo (*a tiny step*) nella direzione ottimale.
* **Il risultato finale:** Terminato questo quarto step, il ciclo ricomincia da capo con il batch di dati successivo. Iterazione dopo iterazione, il valore della *Loss* si riduce progressivamente, e il modello si adatta stabilmente a rispondere rispettando le istruzioni, la sintassi e la formattazione richieste dal dataset.
Il Capitolo 3 delle fonti approfondisce lo studio delle **funzioni reali di una variabile reale** ($f: X \subseteq \mathbb{R} \to Y \subseteq \mathbb{R}$), esplorandone le operazioni, le proprietà, le tipologie elementari e fornendo un rigoroso inquadramento del concetto di limite [1].

Ecco i concetti principali spiegati nel dettaglio:

**1. Operazioni e Composizione**
Oltre a estendere le operazioni algebriche (somma, differenza, prodotto, frazione) dai numeri reali alle funzioni, il capitolo si sofferma sulla **composizione di funzioni** ($g \circ f$). Spiega che questa operazione definisce una nuova funzione valutando $g$ sui valori restituiti da $f$, avvisando che in generale la composizione non è commutativa, ovvero $f \circ g \neq g \circ f$ [2, 3].
[[Composizione ed Operazioni di una funzione|Approfondimento]].

**2. Proprietà delle Funzioni**
Le funzioni vengono classificate in base a diverse caratteristiche geometriche e analitiche:
*   **Invertibilità:** Vengono definite le funzioni iniettive (a ogni elemento del codominio corrisponde al più un elemento del dominio) e suriettive (il codominio è interamente "coperto") [4]. Una funzione è invertibile unicamente se è **biettiva** (sia iniettiva che suriettiva). Dal punto di vista grafico, l'iniettività significa che una qualsiasi retta orizzontale interseca il grafico al massimo una volta [5]. Il grafico della funzione inversa $f^{-1}$ è simmetrico rispetto alla bisettrice $y=x$ rispetto al grafico di $f$ [6]. 
## Funzione iniettiva 
 ![[Pasted image 20260526085222.png]]
*   **Limitatezza:** Le funzioni possono essere limitate superiormente (il grafico non supera un certo tetto $M$), inferiormente, o globalmente [7].
*   **Simmetria:** Si introducono le funzioni **pari** ($f(-x)=f(x)$), che presentano un grafico simmetrico rispetto all'asse delle ordinate, e **dispari** ($f(-x)=-f(x)$), il cui grafico è simmetrico rispetto all'origine del piano cartesiano [8].
*   **Monotonia e Periodicità:** Vengono classificate le funzioni crescenti e decrescenti e si introduce il concetto di funzioni periodiche (come il seno e il coseno), le quali ripetono i propri valori a intervalli regolari detti "periodi" $T$ [9].

**3. Funzioni Elementari**
Le fonti definiscono analiticamente le famiglie di funzioni di base necessarie per il corso:
*   **Polinomi e funzioni razionali:** I polinomi sono combinazioni lineari di monomi di vario grado, mentre le funzioni razionali derivano dal rapporto di due polinomi [10, 11].
*   **Potenze ed Esponenziali:** Viene affrontato il problema di dare senso all'elevamento a potenza con esponente irrazionale (come $a^\pi$ o $2^\pi$). Si risolve estendendo il concetto dagli esponenti razionali (noti tramite le radici) ai numeri reali tramite il concetto di limite di successioni crescenti e limitate [12, 13]. Ciò permette di formalizzare le funzioni potenza ($x^r$) ed esponenziali ($a^x$, prestando particolare attenzione alla base del numero di Nepero $e^x$) [14, 15].
*   **Funzioni Iperboliche:** Definite proprio tramite l'esponenziale: coseno iperbolico ($\cosh$, la cui forma grafica è detta "catenaria"), seno iperbolico ($\sinh$) e tangente iperbolica ($\tanh$) [16, 17].
*   **Funzioni Circolari (Trigonometriche):** Dopo aver introdotto la misurazione degli angoli in radianti, il capitolo illustra seno, coseno e tangente con le loro proprietà di parità e periodicità [17-19].

**4. Limiti delle Funzioni Reali**
Questa sezione è dedicata all'estensione del concetto di limite studiato sulle successioni verso le funzioni continue.
*   **Definizione tramite successioni:** Calcolare il limite per $x$ che tende a $c$ (dove $c$ è un "punto di accumulazione") significa verificare che per *qualsiasi* successione di punti che si avvicina a $c$, il corrispondente valore della funzione si avvicina sempre a un certo valore $l$. Durante l'avvicinamento, il punto $c$ esatto viene escluso per capire l'andamento del grafico "vicino" a esso [20-23].
*   **Limiti Destri e Sinistri:** Viene mostrato come la funzione si comporti avvicinandosi a un punto unicamente con valori più grandi (limite destro, $x \to c^+$) o più piccoli (limite sinistro, $x \to c^-$) [24, 25].
*   **Asintoti:** Se il limite per un punto finito diverge a $\pm\infty$, la funzione possiede un **asintoto verticale**. Se invece il limite all'infinito converge a un valore finito $l$, possiede un **asintoto orizzontale** [26].
*   **Regole di calcolo e Limiti Notevoli:** Le regole algebriche del limite e teoremi come quello dei **Carabinieri** valgono anche per le funzioni [27, 28]. Sfruttando proprio il Teorema dei Carabinieri, le fonti dimostrano tre importantissimi limiti notevoli che torneranno molto utili [29]:
    1.  $\lim_{x \to 0} \frac{\sin(x)}{x} = 1$ [29, 30]
    2.  $\lim_{x \to 0} \frac{1-\cos(x)}{x^2} = \frac{1}{2}$ [30, 31]
    3.  $\lim_{x \to 0} \frac{e^x - 1}{x} = 1$ [31, 32]
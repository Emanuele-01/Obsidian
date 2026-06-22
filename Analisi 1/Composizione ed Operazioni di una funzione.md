Le operazioni e la composizione sono i modi fondamentali con cui possiamo "combinare" due o più funzioni per crearne di nuove.
Vediamo nel dettaglio sia le operazioni algebriche sia la composizione, con definizioni pratiche ed esempi numerici.
## 1. Operazioni Algebriche tra Funzioni
Se abbiamo due funzioni, f(x) e g(x), possiamo sommarle, sottrarle, moltiplicarle o dividerle proprio come faresti con i normali numeri.
L'unica regola fondamentale da ricordare riguarda il **dominio**: la nuova funzione esisterà **solo dove esistono contemporaneamente sia f che g** (cioè nell'intersezione dei loro domini). Per la divisione, dovremo anche escludere i punti in cui la funzione al denominatore si annulla.
Ecco le definizioni formali e un esempio pratico per ciascuna, usando queste due funzioni di partenza:
* **$f(x) = x^2$** (Dominio: tutto $\mathbb{R}$)
* **$g(x) = x + 3$** (Dominio: tutto $\mathbb{R}$)
### Somma e Differenza
Si sommano o sottraggono semplicemente le due espressioni.
 * **Formula:** $$(f \pm g)(x) = f(x) \pm g(x)$$
 * **Esempio (Somma):** $$(f + g)(x) = x^2 + x + 3$$
* **Se calcoliamo il valore in:** $$x = 2: (f+g)(2) = 2^2 + 2 + 3 = 9$$**Che equivale a fare**: $$f(2) + g(2) = 4 + 5 = 9$$
### Prodotto
Si moltiplicano le due espressioni tra loro.
 * **Formula:** $$(f \cdot g)(x) = f(x) \cdot g(x)$$
 * **Esempio:** $$(f \cdot g)(x) = x^2 \cdot (x + 3) = x^3 + 3x^2$$
 * **Se calcoliamo il valore in:** $$x = 2:* (f \cdot g)(2) = 2^3 + 3(2^2) = 8 + 12 = 20$$
### Quoziente (Rapporto)
Si divide la prima funzione per la seconda, aggiungendo la condizione che il denominatore non sia zero.
 * **Formula:** $$\left(\frac{f}{g}\right)(x) = \frac{f(x)}{g(x)} con g(x) \neq 0$$
 * **Esempio:** $$\left(\frac{f}{g}\right)(x) = \frac{x^2}{x + 3}$$
 * *Vincolo sul dominio:* Poiché il denominatore non può essere zero, dobbiamo escludere **x = -3**. Il dominio di questa nuova funzione sarà tutto $\mathbb{R}$ tranne -3.
## 2. La Composizione di Funzioni ($g \circ f$)
La composizione non è un'operazione algebrica, ma una **applicazione in cascata**. Immagina le funzioni come macchinari industriali in una catena di montaggio:
 1. Prendi un numero x e lo inserisci nel primo macchinario f.
 2. Il macchinario f sforna un risultato, che chiamiamo f(x).
 3. Prendi questo risultato f(x) e lo usi direttamente come "materia prima" (input) dentro il secondo macchinario g.
 4. Il risultato finale sarà g(f(x)).
La scrittura (g $\circ$ f)(x) si legge **"g composto f di x"** oppure **"g di f di x"**. Nota che la funzione scritta a destra (f) è quella che agisce per prima.
### Il vincolo del Dominio nella Composizione
Affinché la composizione sia possibile, **il risultato di f(x) deve poter entrare legalmente in g**. Cioè, l'insieme delle immagini (i valori di uscita) di f deve trovarsi all'interno del dominio (i valori accettabili in ingresso) di g.
## Esempio Pratico: Perché l'ordine conta? ($f \circ g \neq g \circ f$)
La composizione **non è commutativa**. Cambiare l'ordine delle funzioni cambia completamente il risultato. Vediamolo con i numeri e con le formule usando:
 * f(x) = x^2 (la macchina che "eleva al quadrato")
 * g(x) = x + 3 (la macchina che "aggiunge 3")
### Caso A: Calcoliamo $(g \circ f)(x)$ ovvero $g(f(x))$
Qui f lavora per prima, e il suo risultato entra in g.
 * **Con un numero (x = 2):**
   1. Prima applichiamo f: $f(2) = 2^2 = 4$
   2. Ora passiamo 4 dentro g: $g(4) = 4 + 3 = \mathbf{7}$
 * **In formula:** Sostituiamo l'intera espressione di $f(x)$ dentro la x di $g(x)$:
   
### Caso B: Calcoliamo $(f \circ g)(x)$ ovvero $f(g(x))$
Qui g lavora per prima, e il suo risultato entra in f.
 * **Con lo stesso numero (x = 2):**
   1. Prima applichiamo g: $g(2) = 2 + 3 = 5$
   2. Ora passiamo 5 dentro f: $f(5) = 5^2 = \mathbf{25}$
 * **In formula:** Sostituiamo l'intera espressione di g(x) dentro la x di f(x):
   
> **Risultato:** Come vedi, $x^2 + 3$ è completamente diverso da $x^2 + 6x + 9$. Questo dimostra visivamente perché l'ordine nella composizione è cruciale.
> 
Ti è più chiaro adesso il meccanismo "in cascata" della composizione, o vorresti vedere un esempio con funzioni diverse, magari stringenti sul dominio (come radici o logaritmi)?

# 🧱 Widget Principali in Flutter

[← Fondamenti di Dart](./dart_fondamenti.md) | [Torna all'Hub](./index.md) | [Continua con le Tecniche di Sviluppo →](./sviluppo_ios_android.md)

---

In Flutter, **tutto è un Widget**. I widget sono i mattoni fondamentali utilizzati per costruire l'interfaccia utente (UI) di un'applicazione. Essi descrivono l'aspetto della UI in base alla loro configurazione corrente e al loro stato.

---

## 1. StatelessWidget vs StatefulWidget

La prima distinzione fondamentale in Flutter è tra widget senza stato e widget con stato. Entrambi estendono le classi base di Dart (per capire l'ereditarietà vedi la sezione [OOP in Dart](./dart_fondamenti.md#4-programmazione-orientata-agli-oggetti-oop)).

### StatelessWidget
Rappresenta un widget immutabile. Le sue proprietà non cambiano nel tempo una volta costruito.
*   **Caso d'uso:** Icone, etichette di testo statiche, pulsanti semplici.
*   **Metodo chiave:** `build(BuildContext context)`.

### StatefulWidget
Rappresenta un widget che può cambiare il proprio stato interno durante il ciclo di vita dell'applicazione. Quando lo stato cambia, viene chiamato `setState()` per notificare al framework di ridisegnare il widget.
*   **Caso d'uso:** Form compilabili, contatori, elementi interattivi.
*   **Struttura:** Si divide in due classi: la classe che estende `StatefulWidget` e la classe che estende `State<MyWidget>`.
*   **Ciclo di Vita Principale:**
    1.  `createState()`: Crea l'oggetto State.
    2.  `initState()`: Chiamato una sola volta all'avvio (ottimo per inizializzazioni).
    3.  `didChangeDependencies()`: Chiamato dopo `initState` o al cambio dei dati dipendenti (es. InheritedWidget).
    4.  `build()`: Costruisce la UI.
    5.  `setState()`: Richiede la ricostruzione del widget.
    6.  `dispose()`: Chiamato quando il widget viene rimosso definitivamente (usato per liberare risorse).

> [!WARNING]
> Un uso eccessivo di `setState` su widget di grandi dimensioni può causare problemi di performance. Per ovviare a questo, si usano pattern di architettura esterni come descritto in [State Management](./sviluppo_ios_android.md#1-architettura-e-gestione-dello-stato-state-management).

---

## 2. Widget Strutturali e di Layout Principali

I widget in Flutter sfruttano ampiamente i **parametri nominati** di Dart per configurare le loro proprietà. Maggiori dettagli sulla sintassi in [Parametri Nominati](./dart_fondamenti.md#2-parametri-nominati-named-parameters).

### Scaffold
Fornisce la struttura di layout visivo di base per le pagine secondo le linee guida del Material Design. Gestisce le aree standard di una schermata (barra superiore, corpo, menu laterale, ecc.).
*   **Parametri Principali:**
    *   `appBar`: (`PreferredSizeWidget`) La barra superiore della schermata (solitamente un widget `AppBar`).
    *   `body`: (`Widget`) L'area principale della schermata.
    *   `floatingActionButton`: (`Widget`) Un pulsante circolare che fluttua sopra il body.
    *   `drawer`: (`Widget`) Un pannello che scorre lateralmente (menu di navigazione).
    *   `backgroundColor`: (`Color`) Il colore di sfondo del body.

```dart
Scaffold(
  appBar: AppBar(title: Text('Home Page')),
  body: Center(child: Text('Contenuto Principale')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.add),
  ),
)
```

### AppBar
La barra delle applicazioni situata nella parte superiore dello `Scaffold`.
*   **Parametri Principali:**
    *   `title`: (`Widget`) Il titolo principale (solitamente un widget `Text`).
    *   `leading`: (`Widget`) Un widget prima del titolo (es. icona menu o tasto indietro).
    *   `actions`: (`List<Widget>`) Pulsanti o azioni allineati a destra.
    *   `backgroundColor`: (`Color`) Colore di sfondo della barra.

---

## 3. Widget di Spaziamento e Allineamento

### Container
Un widget versatile che combina pittura comune, posizionamento e dimensionamento. Permette di aggiungere margini, padding, bordi, colori di sfondo e forme.
*   **Parametri Principali:**
    *   `width` / `height`: (`double`) Dimensioni del container.
    *   `padding`: (`EdgeInsetsGeometry`) Spazio interno tra il container e il suo figlio.
    *   `margin`: (`EdgeInsetsGeometry`) Spazio esterno attorno al container.
    *   `color`: (`Color`) Colore di sfondo (da non usare insieme a `decoration`).
    *   `decoration`: (`Decoration`) Permette di definire sfondi complessi (gradienti, bordi arrotondati tramite `BoxDecoration`).
    *   `child`: (`Widget`) Il widget figlio all'interno.

```dart
Container(
  width: 200,
  height: 100,
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
  ),
  child: Text('Testo in Box', style: TextStyle(color: Colors.white)),
)
```

### Padding
Aggiunge spazio vuoto attorno al suo widget figlio. È una scelta più leggera rispetto a `Container` se si deve solo aggiungere spaziatura.
*   **Parametri Principali:**
    *   `padding`: (`EdgeInsetsGeometry`) Definisce la quantità di spazio (es. `EdgeInsets.all(8.0)` o `EdgeInsets.symmetric(horizontal: 16, vertical: 8)`).
    *   `child`: (`Widget`) Il widget da distanziare.

### SizedBox
Un box con dimensioni fisse. È ampiamente utilizzato per creare spazi vuoti tra widget (in `Row` o `Column`) o per forzare dimensioni specifiche su un widget figlio.
*   **Parametri Principali:**
    *   `width`: (`double`) Spaziatura orizzontale o larghezza fissa.
    *   `height`: (`double`) Spaziatura verticale o altezza fissa.
    *   `child`: (`Widget`) Opzionale. Il widget a cui imporre le dimensioni.

---

## 4. Widget Multi-Figlio (Layout di Flusso)

### Column & Row
Dispongono i loro figli rispettivamente in direzione verticale (`Column`) ed orizzontale (`Row`).
*   **Parametri Principali:**
    *   `children`: (`List<Widget>`) L'elenco dei widget da disporre.
    *   `mainAxisAlignment`: (`MainAxisAlignment`) Allineamento lungo l'asse principale (verticale per `Column`, orizzontale per `Row`).
    *   `crossAxisAlignment`: (`CrossAxisAlignment`) Allineamento lungo l'asse trasversale.

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('Primo Elemento'),
    SizedBox(height: 10),
    Text('Secondo Elemento'),
  ],
)
```

### Stack
Sovrappone i suoi figli uno sopra l'altro. Utile per sovrapporre elementi (es. badge sopra icone o testo su sfondi sfumati).
*   **Parametri Principali:**
    *   `children`: (`List<Widget>`) I widget sovrappostisi. L'ultimo elemento dell'elenco viene disegnato sopra gli altri.
    *   `alignment`: (`AlignmentGeometry`) Come allineare i widget non posizionati nello stack.
*   **Nota:** Spesso usato in combinazione con il widget **`Positioned`** per posizionare con precisione assoluta un figlio all'interno dello Stack (`top`, `bottom`, `left`, `right`).

---

## 5. Widget per Liste e Scorrimento

### ListView
Il widget di scorrimento più comune. Dispone i suoi figli uno dopo l'altro nella direzione di scorrimento.
*   **Parametri Principali:**
    *   `children`: (`List<Widget>`) Lista di elementi (per liste corte).
    *   `scrollDirection`: (`Axis`) Direzione dello scorrimento (`Axis.vertical` o `Axis.horizontal`).
*   **`ListView.builder`**: Variante ottimizzata per liste lunghe o infinite. Carica in memoria solo gli elementi attualmente visibili a schermo.
    > [!TIP]
    > L'uso di `ListView.builder` fa parte delle best-practice di ottimizzazione. Leggi di più su [Ottimizzazione delle Prestazioni](./sviluppo_ios_android.md#5-ottimizzazione-delle-prestazioni).
    *   `itemCount`: (`int`) Numero totale di elementi.
    *   `itemBuilder`: (`IndexedWidgetBuilder`) Funzione che ritorna il widget per un dato indice.

```dart
ListView.builder(
  itemCount: 100,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text('Elemento numero $index'),
    );
  },
)
```

---

## 6. Interazione e Input

### GestureDetector
Rileva le interazioni fisiche dell'utente sullo schermo. Non ha una resa visiva propria, ma avvolge altri widget per renderli interattivi.
*   **Parametri Principali:**
    *   `onTap`: Chiamato in seguito a un tocco singolo.
    *   `onDoubleTap`: Chiamato su un doppio tocco.
    *   `onLongPress`: Chiamato su una pressione prolungata.
    *   `child`: Il widget che deve reagire al tocco.

### ElevatedButton / TextButton / IconButton
Pulsanti preconfigurati conformi al Material Design.
*   `ElevatedButton`: Pulsante in rilievo con ombreggiatura.
*   `TextButton`: Pulsante piatto senza bordi definiti, ideale per azioni secondarie.
*   `IconButton`: Un'icona cliccabile.
*   **Parametri Principali:**
    *   `onPressed`: (`VoidCallback`) Funzione eseguita al click. Se impostato a `null`, il pulsante sarà disabilitato.
    *   `child`: (o `icon` per `IconButton`) Il widget interno del pulsante.

---

[← Fondamenti di Dart](./dart_fondamenti.md) | [Torna all'Hub](./index.md) | [Continua con le Tecniche di Sviluppo →](./sviluppo_ios_android.md)

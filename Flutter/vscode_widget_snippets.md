# 📝 Scorciatoie Flutter Widget Snippets (VS Code)

[← Strumenti VS Code](./vscode_flutter_tools.md) | [Torna all'Hub](./index.md)

---

L'estensione **Flutter Widget Snippets** (di *Nash*) è uno dei pacchetti di produttività più scaricati. Permette di generare interi blocchi di codice boilerplate per Widget, metodi del ciclo di vita o layout complessi digitando una semplice abbreviazione e premendo `Tab` o `Invio`.

Di seguito trovi la tabella di riferimento rapido di tutte le scorciatoie disponibili.

---

## 1. Cheat Sheet delle Scorciatoie

| Scorciatoia | Codice Generato | Descrizione |
| :--- | :--- | :--- |
| **`stless`** | `StatelessWidget` | Genera la classe base per un widget senza stato. |
| **`stful`** | `StatefulWidget` | Genera la classe principale + la classe State per un widget con stato. |
| **`stfulw`** | `StatefulWidget` + build | Simile a `stful`, ma posiziona immediatamente il cursore dentro il metodo `build`. |
| **`initS`** | `void initState() { ... }` | Override del metodo di inizializzazione dello stato (dentro `State`). |
| **`disp`** | `void dispose() { ... }` | Override del metodo di distruzione (chiusura controller, stream, ecc.). |
| **`setState`** | `setState(() { ... });` | Ridisegna il widget con stato aggiornato. |
| **`inheritedW`** | `InheritedWidget` | Crea un widget per la condivisione dello stato lungo l'albero dei widget. |
| **`customPaint`** | `CustomPainter` | Genera una classe per disegnare forme grafiche personalizzate a basso livello. |
| **`customScrollView`** | `CustomScrollView` | Struttura di scorrimento basata su Slivers. |
| **`listViewBuilder`** | `ListView.builder` | Struttura di lista performante per elenchi dinamici. |
| **`gridViewBuilder`** | `GridView.builder` | Griglia a scorrimento ottimizzata. |
| **`singleChildScrollView`** | `SingleChildScrollView` | Box singolo a scorrimento (usato per i form). |
| **`tweenAnimation`** | `TweenAnimationBuilder` | Genera il boilerplate per animazioni implicite basate su intervalli. |

---

## 2. Come Utilizzare gli Snippet con i Segnaposto (Placeholders)

Quando utilizzi uno snippet (ad esempio digitando `stless` e premendo `Invio`), l'estensione genera il codice e attiva i **segnaposto interattivi** (evidenziati in verde o blu nell'editor):

1.  **Digitazione iniziale**: Scrivi il nome del tuo Widget. L'editor cambierà automaticamente il nome della classe sia nella dichiarazione che nel costruttore in tempo reale.
2.  **Spostamento rapido**: Invece di usare il mouse, premi il tasto **`Tab`** sulla tastiera. Il cursore salterà automaticamente al punto successivo che richiede compilazione (es. le proprietà del widget o l'interno del metodo `build`).
3.  **Uscita**: Una volta completati i segnaposto, premi `Tab` un'ultima volta per tornare alla normale digitazione.

---

## 3. Scorciatoie per Metodi del Ciclo di Vita (dentro `State`)

Se ti trovi all'interno della classe `_MyWidgetState`, puoi usare queste scorciatoie veloci:

### `initS`
Genera istantaneamente il metodo per inizializzare variabili o caricare dati (vedi l'uso in [Schermata Profilo](./schermata_profile.md#1-codice-completo-commentato)):
```dart
@override
void initState() {
  super.initState();
  // Il tuo cursore si posizionerà automaticamente qui
}
```

### `disp`
Genera il metodo per pulire la memoria prima che il widget sia distrutto:
```dart
@override
void dispose() {
  // Rilascia controller o disdici stream qui
  super.dispose();
}
```

---

[← Strumenti VS Code](./vscode_flutter_tools.md) | [Torna all'Hub](./index.md)

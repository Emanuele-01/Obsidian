# 📝 Scorciatoie Flutter Widget Snippets (VS Code)

[← Strumenti VS Code](./vscode_flutter_tools.md) | [Torna all'Hub](../index.md)

---

L'estensione **Flutter Widget Snippets** (inclusi gli snippet Dart nativi e i pattern correlati) permette di generare velocemente blocchi di codice boilerplate. Digitando l'abbreviazione (prefisso) e premendo `Tab` o `Invio`, l'editor espanderà il codice pronto per la compilazione.

Di seguito trovi la raccolta completa delle scorciatoie catalogate per tipologia.

---

## 1. Scorciatoie per Flutter (Widget & UI)

Queste scorciatoie sono focalizzate sulla creazione rapida di componenti grafici ed elementi di layout di Flutter.

| Prefisso | Descrizione | Output Generato |
| :--- | :--- | :--- |
| **`fstless`** | StatelessWidget | Crea la classe per un widget senza stato (alternativo a `stless`). |
| **`fstful`** | StatefulWidget | Crea la classe principale + State per un widget con stato (alternativo a `stful`). |
| **`fscaff`** | Scaffold widget | Inserisce il widget `Scaffold` con i parametri strutturali di base. |
| **`ftxt`** | Text widget | Genera il widget `Text` pronto con la stringa di testo. |
| **`fcont`** | Container widget | Inserisce un widget `Container` vuoto. |
| **`fcent`** | Center widget | Inserisce un widget `Center` con la proprietà `child`. |
| **`frow`** | Row widget | Genera una `Row` con l'array `children`. |
| **`fcol`** | Column widget | Genera una `Column` con l'array `children`. |
| **`fex`** | Expanded widget | Avvolge un figlio all'interno di un widget `Expanded`. |
| **`fic`** | Icon widget | Inserisce un widget `Icon` richiedendo l'istanza `Icons`. |
| **`fszb`** | SizedBox (H & W) | Genera un `SizedBox` con argomenti sia di larghezza (`width`) sia di altezza (`height`). |
| **`fszbw`** | SizedBox (Solo W) | Inserisce un `SizedBox` con il solo argomento `width` (spaziatura orizzontale). |
| **`fszbh`** | SizedBox (Solo H) | Inserisce un `SizedBox` con il solo argomento `height` (spaziatura verticale). |
| **`felbtn`** | ElevatedButton | Genera un pulsante Material `ElevatedButton` con `onPressed` e `child`. |
| **`fstream`** | StreamBuilder | Inserisce la struttura di un `StreamBuilder` per gestire flussi asincroni. |
| **`finitlf`** | initState Method | Inserisce il metodo `initState()` all'interno dello stato del widget. |
| **`fimpmat`** | Import Material | Aggiunge in cima al file l'import: `import 'package:flutter/material.dart';`. |
| **`fedgall`** | EdgeInsets.all() | Scorciatoia per `EdgeInsets.all()` (padding uniforme su tutti i lati). |
| **`fedgonly`** | EdgeInsets.only() | Scorciatoia per `EdgeInsets.only()` (padding su lati specifici). |
| **`fedgsym`** | EdgeInsets.symmetric() | Genera `EdgeInsets.symmetric()` (padding orizzontale e verticale). |
| **`fedgsymv`** | EdgeInsets (Vertical) | Genera `EdgeInsets.symmetric(vertical: ...)` per spaziatura verticale. |
| **`fedgsymh`** | EdgeInsets (Horizontal)| Genera `EdgeInsets.symmetric(horizontal: ...)` per spaziatura orizzontale. |

---

## 2. Scorciatoie per Dart (Variabili, Funzioni & OOP)

Snippet progettati per accelerare la scrittura della logica in puro stile Dart.

| Prefisso | Descrizione | Output Generato |
| :--- | :--- | :--- |
| **`dvar`** | Variabile `var` | Dichiarazione di una variabile generica con inferenza. |
| **`dfinal`** | Variabile `final` | Dichiarazione di una variabile costante a runtime. |
| **`dconst`** | Variabile `const` | Dichiarazione di una costante nota a tempo di compilazione. |
| **`dinvar`** | Variabile d'istanza Pubblica | Aggiunge un attributo pubblico all'interno di una classe. |
| **`dprinvar`** | Variabile d'istanza Privata | Genera una variabile privata (con prefisso `_`). |
| **`dmt`** | Metodo Pubblico | Crea la firma di una funzione pubblica in una classe. |
| **`dprmt`** | Metodo Privato | Crea un metodo privato (con prefisso `_`). |
| **`darr`** | Arrow Function Pubblica | Funzione a riga singola che ritorna un valore (`=>`). |
| **`dprarr`** | Arrow Function Privata | Funzione a riga singola privata (`_funzione() => ...`). |
| **`dgetarr`** | Getter a freccia | Genera un metodo `get` sintetico ad una riga (`get variabile => _variabile;`). |
| **`dopnctor`** | Costruttore con opzioni | Costruttore di classe che usa i parametri nominati opzionali. |
| **`dan`** | Funzione Anonima | Genera una lambda function `() { ... }`. |
| **`dcla`** | Classe Standard | Crea una nuova classe Dart vuota con il costruttore di base. |
| **`dclae`** | Classe con Ereditarietà | Crea una classe che estende un'altra (`class A extends B { ... }`). |
| **`dlist`** | Collezione `List` | Dichiarazione di una lista tipizzata `List<T>`. |
| **`dmap`** | Collezione `Map` | Struttura per mappe chiave-valore `Map<K, V>`. |
| **`dset`** | Collezione `Set` | Raccolta ordinata di elementi univoci `Set<T>`. |
| **`dimpmeta`** | Import Meta | Aggiunge la libreria per l'annotazione `@required` o simili. |
| **`dconvert`** | Import Convert | Aggiunge l'import `dart:convert` (usato per JSON encoding/decoding). |
| **`dimpas`** | Import con alias | Inserisce la sintassi `import '...' as ...;`. |
| **`dimpshow`** | Import parziale (`show`) | Importa solo classi specifiche (`import '...' show ...;`). |
| **`dimphide`** | Import parziale (`hide`) | Importa nascondendo classi specifiche (`import '...' hide ...;`). |
| **`dimplazy`** | Import lazy (`deferred`) | Importa in modalità caricamento pigro (`import '...' deferred as ...;`). |
| **`dexshow`** | Export parziale (`show`) | Esporta solo determinati membri del file. |
| **`dexhide`** | Export parziale (`hide`) | Esporta nascondendo determinati membri del file. |

---

## 3. Scorciatoie per State Management (BLoC Pattern)

Per chi utilizza la gestione dello stato basata su Business Logic Components.

| Prefisso | Descrizione | Output Generato |
| :--- | :--- | :--- |
| **`fblocprov`** | BlocProvider | Crea il widget wrapper per registrare ed iniettare un BLoC nell'albero. |

---

## 4. Come Usare i Segnaposto (Placeholders)

Quando espandi uno snippet premendo `Invio` o `Tab` sull'abbreviazione:
1.  **Segnaposto Corrente**: L'editor seleziona automaticamente il primo parametro modificabile (es. il nome della classe). Scrivilo direttamente.
2.  **Navigazione (`Tab`)**: Premi il tasto **`Tab`** sulla tastiera per saltare istantaneamente al parametro successivo (es. le proprietà del widget o il corpo del costruttore).
3.  **Ritorno alla tastiera**: Al termine dei segnaposto definiti nello snippet, premendo `Tab` il cursore si riposizionerà alla fine del blocco di codice per consentire la normale digitazione.

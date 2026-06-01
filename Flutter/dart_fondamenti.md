# 🎯 Fondamenti di Dart

[← Torna all'Hub](./index.md) | [Continua con la Guida ai Widget →](./widget_principali.md)

---

Dart è il linguaggio di programmazione open-source sviluppato da Google ed è alla base del framework Flutter. È un linguaggio orientato agli oggetti, fortemente tipizzato, che supporta sia la compilazione JIT (Just-In-Time) per uno sviluppo rapido (Hot Reload) sia AOT (Ahead-Of-Time) per prestazioni native in produzione.

---

## 1. Variabili e Tipi di Dato

In Dart, tutto ciò che può essere inserito in una variabile è un *oggetto*. Di seguito i tipi principali e la gestione delle variabili.

### Dichiarazione delle Variabili
*   **Dinamica (con inferenza di tipo):** Usando la parola chiave `var`. Il tipo viene determinato al momento dell'assegnazione.
    ```dart
    var nome = 'Mario'; // Dart capisce che è una String
    ```
*   **Esplicita:** Definendo direttamente il tipo.
    ```dart
    String cognome = 'Rossi';
    int eta = 30;
    ```

### Costanti: `final` e `const`
*   `final`: La variabile può essere impostata una sola volta. Il valore può essere calcolato a runtime.
    ```dart
    final dataCorrente = DateTime.now(); // Calcolato a runtime
    ```
*   `const`: Definisce una costante a tempo di compilazione (compile-time constant).
    > [!TIP]
    > L'uso costante di `const` è fondamentale anche in Flutter per ottimizzare la UI. Leggi come influisce sulle performance nella sezione [Ottimizzazione delle Prestazioni](./sviluppo_ios_android.md#5-ottimizzazione-delle-prestazioni).
    ```dart
    const pi = 3.14159; // Valore noto a priori
    ```

### Tipi di Dato Principali
*   `Numbers`: `int` (interi) e `double` (numeri con virgola).
*   `Strings`: `String` (racchiuse tra singoli o doppi apici). Supportano l'interpolazione con `${variabile}`.
*   `Booleans`: `bool` (valori `true` o `false`).
*   `Lists`: Array ordinati di elementi (`List<T>`).
    ```dart
    List<String> frutti = ['Mela', 'Banana', 'Arancia'];
    ```
*   `Sets`: Raccolta ordinata di elementi unici.
    ```dart
    Set<int> numeriUnici = {1, 2, 3, 1}; // Il secondo 1 viene ignorato
    ```
*   `Maps`: Strutture chiave-valore (`Map<K, V>`).
    ```dart
    Map<String, dynamic> utente = {
      'nome': 'Luca',
      'eta': 25,
      'isDeveloper': true
    };
    ```

---

## 2. Null Safety

Dart adotta una gestione rigorosa della nullabilità (Sound Null Safety). Di default, le variabili non possono essere `null`.

*   **Tipi Nullabili:** Si usa l'operatore `?` dopo il tipo per indicare che può essere nullo.
    ```dart
    String? nomeNullabile; // Può essere String o null
    ```
*   **Operatore di asserzione non-nullo (`!`):** Forza Dart a considerare la variabile come non-nulla (da usare con cautela).
    ```dart
    String nome = nomeNullabile!; // Errore a runtime se nomeNullabile è null
    ```
*   **Operatore Null-Aware (`??`):** Fornisce un valore di fallback se la variabile è nulla.
    ```dart
    String nomeVisualizzato = nomeNullabile ?? 'Ospite';
    ```
*   **Accesso condizionale (`?.`):** Esegue il metodo o la proprietà solo se l'oggetto non è nullo.
    ```dart
    int? lunghezza = nomeNullabile?.length;
    ```
*   **La parola chiave `late`:** Utilizzata per dichiarare variabili non-nullabili che verranno inizializzate in un secondo momento (ma prima del loro utilizzo).
    ```dart
    late String descrizione;
    void init() {
      descrizione = 'Inizializzata';
    }
    ```

---

## 3. Funzioni

Le funzioni in Dart sono oggetti di prima classe, il che significa che possono essere passate come argomenti ad altre funzioni o assegnate a variabili.

### Sintassi Base
```dart
int somma(int a, int b) {
  return a + b;
}
```

### Funzioni Freccia (Arrow Functions)
Per funzioni contenenti una sola espressione.
```dart
int moltiplica(int a, int b) => a * b;
```

### Parametri delle Funzioni

#### 1. Parametri Posizionali Opzionali
Racchiusi tra parentesi quadre `[]`.
```dart
String saluta(String nome, [String? titolo]) {
  if (titolo != null) {
    return 'Ciao $titolo $nome';
  }
  return 'Ciao $nome';
}
```

#### 2. Parametri Nominati (Named Parameters)
Racchiusi tra parentesi graffe `{}`. Aumentano notevolmente la leggibilità e sono ampiamente utilizzati per definire le proprietà dei widget Flutter (vedi [Guida ai Widget](./widget_principali.md)).
```dart
void creaUtente({required String nome, int eta = 18}) {
  print('Utente: $nome, Età: $eta');
}

// Chiamata:
creaUtente(nome: 'Alice', eta: 30);
```

---

## 4. Programmazione Orientata agli Oggetti (OOP)

Dart è un linguaggio basato su classi e mixin. Ogni widget di Flutter è in realtà una classe Dart (leggi di più su [Stateless vs Stateful](./widget_principali.md#1-statelesswidget-vs-statefulwidget)).

> [!TIP]
> Questa sezione tratta i concetti base della programmazione ad oggetti. Per un'analisi approfondita sui membri privati, visibilità, getter/setter e costruttori factory, consulta la guida completa: **[OOP in Dart](./dart_oop.md)**.


### Definizione di una Classe e Costruttori
```dart
class Persona {
  String nome;
  int eta;

  // Costruttore abbreviato (Syntactic Sugar)
  Persona(this.nome, this.eta);

  // Costruttore nominato
  Persona.giovane(this.nome) : eta = 18;

  void saluta() {
    print('Ciao, sono $nome e ho $eta anni.');
  }
}
```

### Ereditarietà
Dart supporta l'ereditarietà singola tramite `extends`.
```dart
class Studente extends Persona {
  String corso;

  // Si usa super per passare i parametri al costruttore della classe padre
  Studente(super.nome, super.eta, this.corso);

  @override
  void saluta() {
    print('Ciao, sono lo studente $nome del corso di $corso.');
  }
}
```

### Mixins
I mixin consentono di riutilizzare il codice di una classe in più gerarchie di classi senza usare l'ereditarietà diretta. Si definiscono con `mixin` e si applicano con `with`.
```dart
mixin Volante {
  void vola() => print('Sto volando!');
}

class Uccello with Volante {}
```

---

## 5. Programmazione Asincrona

La programmazione asincrona in Dart si basa su due concetti chiave: `Future` (operazioni singole) e `Stream` (flussi di dati nel tempo). Questa logica è fondamentale per caricare dati da API o per implementare la [Gestione dello Stato](./sviluppo_ios_android.md#1-architettura-e-gestione-dello-stato-state-management).

### Future e Async/Await
Un `Future` rappresenta un valore che sarà disponibile in futuro (es. chiamata API).

```dart
Future<String> recuperaDatiDalServer() async {
  // Simula un ritardo di rete di 2 secondi
  await Future.delayed(Duration(seconds: 2));
  return 'Dati caricati con successo';
}

void main() async {
  print('Inizio caricamento...');
  try {
    String dati = await recuperaDatiDalServer();
    print(dati);
  } catch (e) {
    print('Errore: $e');
  }
  print('Fine operazione');
}
```

### Streams
Uno `Stream` è una sequenza di eventi asincroni. Viene gestito con `listen` o con un ciclo `await for`. È la colonna portante di architetture come BLoC (Business Logic Component).
```dart
Stream<int> conteggioAsincrono(int max) async* {
  for (int i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emette il valore corrente nello stream
  }
}
```

---

[← Torna all'Hub](./index.md) | [Continua con la Guida ai Widget →](./widget_principali.md)

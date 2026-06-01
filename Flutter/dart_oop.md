# 🏛️ Programmazione Orientata agli Oggetti (OOP) in Dart

[← Torna all'Hub](./index.md) | [← Fondamenti di Dart](./dart_fondamenti.md)

---

La Programmazione Orientata agli Oggetti (OOP) è il paradigma software su cui si fondano sia Dart sia Flutter. In Flutter, ogni schermata, bottone o tema è rappresentato da un oggetto derivato da una classe. Comprendere l'OOP in Dart è quindi indispensabile per scrivere codice pulito, robusto e scalabile.

---

## 1. Classi, Attributi e Metodi

Una **classe** è un modello (blueprint) per creare **oggetti**. Un oggetto è un'istanza concreta di una classe.

### Dichiarazione di una Classe Base
In Dart, una classe contiene campi (variabili) e metodi (funzioni).
```dart
class Auto {
  // Attributi (Stato dell'oggetto)
  String marca;
  String modello;
  int anno;

  // Costruttore standard
  Auto(this.marca, this.modello, this.anno);

  // Metodo (Comportamento dell'oggetto)
  void mostraDettagli() {
    print('Auto: $marca $modello ($anno)');
  }
}

void main() {
  // Istanziazione di un oggetto
  var miaAuto = Auto('Fiat', '500', 2022);
  miaAuto.mostraDettagli(); // Output: Auto: Fiat 500 (2022)
}
```

---

## 2. Visibilità: Membri Pubblici e Privati (`_`)

A differenza di linguaggi come Java o C++, Dart non ha parole chiave come `public`, `private` o `protected`. 

> [!IMPORTANT]
> In Dart, la privacy è definita a **livello di libreria** (cioè per singolo file `.dart`), non a livello di classe. 
> Qualsiasi identificatore (classe, variabile, metodo) che inizia con un **trattino basso (`_`)** è **privato** per il file in cui si trova.

### Esempio di Incapsulamento (file: `conto_bancario.dart`)
```dart
class ContoBancario {
  final String titolare;
  
  // Attributo privato: non accessibile al di fuori di questo file
  double _saldo;

  ContoBancario(this.titolare, this._saldo);

  // Metodo pubblico
  void deposita(double importo) {
    if (importo > 0) {
      _saldo += importo;
      _registraOperazione('Deposito di $importo');
    }
  }

  // Metodo privato: utilizzabile solo all'interno di questo file
  void _registraOperazione(String messaggio) {
    print('[LOG]: $messaggio. Nuovo saldo: $_saldo');
  }
}
```

Se importi `conto_bancario.dart` in un altro file, non potrai fare `conto._saldo` o `conto._registraOperazione()`. Otterrai un errore di compilazione.

---

## 3. Getter e Setter

I getter e i setter permettono di accedere e modificare gli attributi privati, inserendo al contempo logiche di controllo o formattazione.

```dart
class Termometro {
  double _celsius;

  Termometro(this._celsius);

  // GETTER: Calcola il valore in Fahrenheit al volo
  double get fahrenheit => (_celsius * 9 / 5) + 32;

  // SETTER: Modifica la temperatura in Celsius applicando un controllo di sicurezza
  set celsius(double valore) {
    if (valore < -273.15) {
      throw ArgumentError('Temperatura inferiore allo zero assoluto!');
    }
    _celsius = valore;
  }
}

void main() {
  var t = Termometro(25.0);
  print(t.fahrenheit); // Legge il getter: 77.0
  t.celsius = 30.0;     // Modifica tramite setter
}
```

---

## 4. Ereditarietà e Polimorfismo

*   **Ereditarietà (`extends`)**: Consente a una sottoclasse di ereditare proprietà e metodi da una superclasse.
*   **Polimorfismo (`@override`)**: Consente a una sottoclasse di riscrivere un metodo della classe padre per adattarlo al proprio comportamento.

```dart
abstract class Animale {
  void emettiVerso(); // Metodo astratto (senza corpo)
}

class Cane extends Animale {
  @override
  void emettiVerso() {
    print('Bau bau!');
  }
}

class Gatto extends Animale {
  @override
  void emettiVerso() {
    print('Miao!');
  }
}
```

---

## 5. Costruttori Avanzati

Dart offre costruttori unici pensati per ottimizzare lo sviluppo dei Widget in Flutter.

### 1. Costruttore Costante (`const`)
Se una classe produce solo dati immutabili (campi `final`), puoi definire un costruttore `const`. Consente a Flutter di riutilizzare la stessa istanza in memoria (leggi l'impatto in [Ottimizzazione UI](./sviluppo_ios_android.md#5-ottimizzazione-delle-prestazioni)).
```dart
class Punto {
  final double x;
  final double y;
  const Punto(this.x, this.y);
}
```

### 2. Costruttore Factory (`factory`)
Si usa quando non si vuole creare sempre una nuova istanza della classe, ad esempio per restituire un oggetto da una cache o mappare dati JSON.
```dart
class Configurazione {
  final String url;
  static Configurazione? _istanza;

  // Il costruttore factory può ritornare un'istanza esistente
  factory Configurazione.singoletto(String url) {
    _istanza ??= Configurazione._interna(url);
    return _istanza!;
  }

  // Costruttore privato nominato per uso interno
  Configurazione._interna(this.url);
}
```

---

## 6. Best Practices per lo Sviluppo in OOP (Dart/Flutter)

### 🎯 1. Favorisci l'Immutabilità
Utilizza variabili `final` per gli attributi delle tue classi. Rende il codice prevedibile ed evita bug derivanti da modifiche accidentali dello stato a runtime.
*   *In Flutter*: I widget ereditano da `StatelessWidget` o `StatefulWidget` e devono avere solo campi `final` (eccetto lo State dei widget con stato).

### 🎯 2. Single Responsibility Principle (SRP)
Una classe deve fare una sola cosa ed averne la piena responsabilità.
*   *Esempio*: Non mettere la logica di chiamata API all'interno della classe del Widget UI. Sposta la logica in una classe Repository separata.

### 🎯 3. Evita il "Deep Inheritance" (Ereditarietà Profonda)
L'ereditarietà profonda crea dipendenze strette e codice rigido. Preferisci la **composizione** (combinare oggetti più piccoli) o l'uso dei **mixins** per condividere comportamenti trasversali (vedi [Mixins in Dart](./dart_fondamenti.md#mixins)).

### 🎯 4. Utilizza i Costruttori Abbreviati
Usa l'inizializzazione formale direttamente nei costruttori per risparmiare righe di codice inutile:
```dart
// CORRETTO:
User(this.id, this.username);

// DA EVITARE (Sintassi verbosa stile Java):
User(String id, String username) {
  this.id = id;
  this.username = username;
}
```

---

[← Torna all'Hub](./index.md) | [← Fondamenti di Dart](./dart_fondamenti.md)

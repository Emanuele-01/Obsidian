# 🎯 Fondamenti di Rust

[← Torna all'Hub](./index.md) | [Continua con Ownership & Sicurezza →](./rust_ownership_safety.md)

---

Rust è un linguaggio di programmazione di sistema focalizzato su tre obiettivi primari: **sicurezza della memoria**, **velocità** e **concorrenza**. Questo documento introduce la sintassi di base e le caratteristiche principali per iniziare a scrivere codice Rust.

---

## 1. Variabili, Mutabilità e Costanti

In Rust, tutte le variabili sono **immutabili di default**. Questo previene modifiche accidentali dello stato e rende il codice concorrente sicuro a tempo di compilazione.

### Variabili Immutabili e Mutabili
Si usa `let` per dichiarare una variabile. Se si desidera renderla modificabile, occorre aggiungere la parola chiave `mut`.
```rust
fn main() {
    let x = 5; // Immutabile
    // x = 6;  // ERRORE: Non è possibile riassegnare una variabile immutabile

    let mut y = 10; // Mutabile
    println!("y iniziale: {y}");
    y = 15; // OK
    println!("y modificata: {y}");
}
```

### Costanti
Le costanti vengono dichiarate con `const` (non `let`). Devono essere annotate esplicitamente con il tipo di dato e il loro valore deve essere calcolabile a tempo di compilazione.
```rust
const SECONDI_IN_UN_GIORNO: u32 = 86_400;
```

### Shadowing
È possibile dichiarare una nuova variabile con lo stesso nome di una precedente. Questa tecnica (chiamata *shadowing*) consente anche di cambiare il tipo di dato mantenendo lo stesso nome.
```rust
let spazi = "   "; // Tipo: &str
let spazi = spazi.len(); // Tipo: usize (funziona!)
```

---

## 2. Tipi di Dato Principali

Rust è un linguaggio a tipizzazione statica, ma il compilatore è in grado di inferire quasi sempre il tipo.

*   **Interi**: `i8`, `i16`, `i32` (default), `i64`, `i128` (con segno) e `u8`, `u16`, `u32`, `u64`, `u128` (senza segno). `isize`/`usize` dipendono dall'architettura del sistema (32 o 64 bit).
*   **Decimali**: `f32` e `f64` (default).
*   **Booleani**: `bool` (`true` o `false`).
*   **Caratteri**: `char` (rappresenta un valore Unicode a 4 byte, es. `'a'` o `'🦁'`).
*   **Tuple**: Gruppo di valori di tipi diversi con lunghezza fissa.
    ```rust
    let tupla: (i32, f64, u8) = (500, 6.4, 1);
    let (x, y, z) = tupla; // Destrutturazione
    let primo = tupla.0;  // Accesso tramite indice
    ```
*   **Array**: Gruppo di elementi dello stesso tipo con lunghezza fissa memorizzato nello stack.
    ```rust
    let array: [i32; 3] = [1, 2, 3];
    ```

---

## 3. Funzioni ed Espressioni

Le funzioni vengono dichiarate con la parola chiave `fn`. La firma delle funzioni richiede sempre di specificare il tipo dei parametri e il tipo di ritorno (indicato con `->`).

### Istruzioni (Statements) vs Espressioni (Expressions)
*   Le **istruzioni** compiono azioni ma non restituiscono valori (terminano con `;`).
*   Le **espressioni** valutano un valore e **non** terminano con il punto e virgola.

```rust
fn somma(a: i32, b: i32) -> i32 {
    a + b // Questa è un'espressione. Ritorna implicitamente il valore della somma!
}
```

---

## 4. Controllo del Flusso

### Costrutto `if` come espressione
Dato che `if` in Rust è un'espressione, può essere utilizzato sul lato destro di un'assegnazione:
```rust
let condizione = true;
let numero = if condizione { 5 } else { 6 }; // I tipi dei due rami devono essere identici
```

### Cicli
Rust ha tre costrutti nativi per i cicli:
1.  **`loop`**: Ciclo infinito che si interrompe solo con `break`. Può anche ritornare un valore.
    ```rust
    let mut contatore = 0;
    let risultato = loop {
        contatore += 1;
        if contatore == 10 {
            break contatore * 2; // Ritorna 20
        }
    };
    ```
2.  **`while`**: Cicla finché la condizione è vera.
3.  **`for`**: Utilizzato per scorrere collezioni (es. array o intervalli).
    ```rust
    for numero in (1..4).rev() { // Converte l'intervallo 1, 2, 3 al contrario
        println!("{numero}!");
    }
    ```

---

## 5. Struct ed Enum

### Struct
Le strutture consentono di raggruppare valori correlati.
```rust
struct Utente {
    username: String,
    email: String,
    attivo: bool,
}

fn main() {
    let utente1 = Utente {
        username: String::from("mario88"),
        email: String::from("mario@example.com"),
        attivo: true,
    };
}
```

### Enum
Le enumerazioni in Rust sono molto potenti perché possono contenere dati associati all'interno delle loro varianti.
```rust
enum Messaggio {
    Svuota,                  // Nessun dato associato
    Sposta { x: i32, y: i32 }, // Contiene una struct anonima
    Scrivi(String),          // Contiene una String
}
```

---

## 6. Pattern Matching

Il costrutto principale per controllare il flusso in base ai valori degli enum è il `match`. Esso richiede un controllo **esaustivo** (tutti i casi possibili devono essere gestiti).

```rust
fn elabora_messaggio(msg: Messaggio) {
    match msg {
        Messaggio::Svuota => println!("Svuotato!"),
        Messaggio::Sposta { x, y } => println!("Sposta in x: {x}, y: {y}"),
        Messaggio::Scrivi(testo) => println!("Messaggio scritto: {testo}"),
    }
}
```

### Costrutto `if let`
Utilizzato per gestire un solo caso specifico ignorando tutti gli altri, riducendo la verbosità del codice rispetto a `match`.
```rust
let configurazione = Some(3);
if let Some(valore) = configurazione {
    println!("Il valore impostato è: {valore}");
}
```

---

## 7. Gestione dei Valori Nulli e degli Errori

Rust non ha un valore `null`. Al suo posto, utilizza la libreria standard ed in particolare due enum generici per gestire l'assenza di dati e gli errori.

### `Option<T>`
Rappresenta un valore che potrebbe essere presente (`Some(T)`) o assente (`None`).
```rust
let nome: Option<String> = Some(String::from("Alice"));
let eta: Option<i32> = None;
```

### `Result<T, E>`
Usato per operazioni che possono fallire. Rappresenta il successo (`Ok(T)`) o l'errore (`Err(E)`).
> [!IMPORTANT]
> A differenza di altri linguaggi, Rust non usa le eccezioni (`try/catch`). Gli errori sono semplici valori di tipo `Result` che devono essere propagati o gestiti esplicitamente. Leggi di più su [Best Practices di Gestione Errori](./rust_best_practices.md#2-gestione-degli-errori-result-e-option).

```rust
fn dividi(dividendo: f64, divisore: f64) -> Result<f64, String> {
    if divisore == 0.0 {
        Err(String::from("Divisione per zero non consentita!"))
    } else {
        Ok(dividendo / divisore)
    }
}
```

### L'operatore di propagazione `?`
Si usa per restituire immediatamente l'errore al chiamante se l'operazione fallisce, senza scrivere lunghi blocchi `match`.
```rust
fn leggi_e_somma() -> Result<i32, std::io::Error> {
    // Se una delle chiamate fallisce, ritorna l'errore immediatamente
    let num1 = leggi_da_file("file1.txt")?.parse::<i32>().unwrap();
    let num2 = leggi_da_file("file2.txt")?.parse::<i32>().unwrap();
    Ok(num1 + num2)
}
```

---

[← Torna all'Hub](./index.md) | [Continua con Ownership & Sicurezza →](./rust_ownership_safety.md)

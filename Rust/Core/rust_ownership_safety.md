# 🛡️ Ownership, References & Sicurezza della Memoria

[← Fondamenti di Rust](./rust_fondamenti.md) | [Torna all'Hub](../index.md) | [Continua con le Best Practices →](./rust_best_practices.md)

---

Il sistema di **Ownership** (Proprietà) è la caratteristica più distintiva di Rust. Consente al linguaggio di garantire la sicurezza della memoria senza l'utilizzo di un Garbage Collector (come Java, C# o Go) e senza che il programmatore debba allocare e deallocare manualmente la memoria (come C o C++).

---

## 1. Le Tre Regole dell'Ownership

Il compilatore di Rust controlla il rispetto delle seguenti regole a tempo di compilazione (durante la fase chiamata *borrow checking*):

1.  **Ogni valore in Rust ha un proprietario (una variabile).**
2.  **Può esserci solo un proprietario alla volta.**
3.  **Quando il proprietario esce dallo scope (l'area di validità racchiusa tra graffe `{}`), il valore viene rimosso dalla memoria (funzione `drop`).**

```rust
{
    let s = String::from("ciao"); // s è proprietario della stringa in memoria heap
    // qui puoi usare s
} // s esce dallo scope. Rust chiama drop() e libera la memoria!
```

---

## 2. Spostamento (Move) vs Copia (Copy)

La gestione della memoria varia in base a dove sono allocati i dati (Stack o Heap).

### Move Semantics (Heap)
Quando assegniamo una variabile allocata in Heap ad un'altra, Rust sposta la proprietà dell'oggetto. La variabile di partenza cessa di esistere.
```rust
let s1 = String::from("ciao");
let s2 = s1; // La proprietà è stata spostata (MOVE) a s2.

// println!("{s1}"); // ERRORE DI COMPILAZIONE! s1 non è più valida.
```

### Copy Semantics (Stack)
I tipi di dato semplici che risiedono interamente sullo Stack (come gli interi, i decimali, i booleani) implementano il trait `Copy`. In questo caso, il valore viene copiato al momento dell'assegnazione ed entrambe le variabili rimangono valide.
```rust
let x = 5;
let y = x; // Copia del valore
println!("x: {x}, y: {y}"); // OK! Entrambe sono valide
```

---

## 3. Riferimenti e Borrowing (Prestito)

Per utilizzare un valore senza trasferirne l'ownership, si usano i **riferimenti** (indicati con il simbolo `&`). L'atto di creare un riferimento prende il nome di **Borrowing**.

```rust
fn calcola_lunghezza(s: &String) -> usize { // Riceve un riferimento
    s.len()
} // s esce dallo scope, ma non essendo il proprietario della stringa, non viene eliminata in memoria heap

fn main() {
    let s1 = String::from("ciao");
    let len = calcola_lunghezza(&s1); // Passiamo un riferimento a s1
    println!("La lunghezza di '{s1}' è {len}."); // s1 è ancora valida!
}
```

### Le Regole dei Riferimenti
Per prevenire le condizioni di corsa (*data race*) sulla memoria, Rust impone una regola rigida a tempo di compilazione:
> [!IMPORTANT]
> In un qualsiasi scope, per un dato valore è possibile avere:
> *   **O un singolo riferimento mutabile (`&mut T`)**
> *   **O un numero qualsiasi di riferimenti immutabili (`&T`)**
> 
> Non è mai possibile avere entrambi contemporaneamente. Inoltre, i riferimenti devono essere sempre validi (no dangling references).

```rust
let mut s = String::from("ciao");

let r1 = &s; // OK
let r2 = &s; // OK
// let r3 = &mut s; // ERRORE! Non puoi creare un riferimento mutabile se ne esistono già di immutabili

println!("{r1}, {r2}"); // I riferimenti immutabili terminano il loro ciclo qui

let r3 = &mut s; // Adesso è OK perché r1 ed r2 non vengono più usati
```

---

## 4. I Cicli di Vita (Lifetimes)

I cicli di vita sono una forma di annotazione con cui il compilatore si assicura che tutti i riferimenti rimangano validi per tutta la durata del loro utilizzo (evitando puntatori a zone di memoria già deallocate).

### Elisione dei Lifetimes
Molto spesso il compilatore deduce i cicli di vita in autonomia tramite regole predefinite. Ad esempio:
```rust
fn prima_parola(s: &str) -> &str { ... } // Il compilatore deduce che il ritorno vive quanto l'input
```

### Annotazione Esplicita
Quando una funzione riceve più riferimenti in ingresso e ne restituisce uno in uscita, il compilatore non sa a quale dei parametri di input sia legato l'output. È necessario specificare i lifetimes con la sintassi `'a`:

```rust
// Specifica che la stringa restituita vivrà quanto il più corto tra i parametri 'x' e 'y'
fn stringa_piu_lunga<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

---

## 5. I Trait: Definire Comportamenti Condivisi

I **Trait** in Rust equivalgono alle interfacce in altri linguaggi orientati agli oggetti. Definiscono un comportamento che tipi diversi possono implementare.

```rust
// Definizione del Trait
pub trait Descrivibile {
    fn descrivi(&self) -> String;
}

// Struttura che implementerà il Trait
pub struct Articolo {
    pub titolo: String,
    pub autore: String,
}

// Implementazione del Trait per la struttura Articolo
impl Descrivibile for Articolo {
    fn descrivi(&self) -> String {
        format!("'{}' scritto da {}", self.titolo, self.autore)
    }
}
```

### Trait Bounds (Polimorfismo statico)
Possiamo vincolare una funzione ad accettare solo tipi che implementano un determinato Trait a tempo di compilazione:
```rust
pub fn stampa_descrizione<T: Descrivibile>(item: &T) {
    println!("{}", item.descrivi());
}
```

---

## 6. Concorrenza Sicura (*Fearless Concurrency*)

Grazie al sistema di Ownership e Borrowing, il compilatore di Rust blocca i bug di concorrenza prima ancora che l'app venga eseguita.

*   **`Send`**: Questo trait indica che la proprietà del tipo può essere trasferita in modo sicuro tra thread diversi.
*   **`Sync`**: Indica che è sicuro accedere allo stesso tipo di dato da più thread contemporaneamente tramite riferimenti immutabili.
*   **`Arc<T>` (Atomic Reference Counted)**: Un puntatore intelligente per condividere la proprietà di un oggetto in modo sicuro tra più thread.
*   **`Mutex<T>` (Mutual Exclusion)**: Garantisce che solo un thread alla volta possa accedere al valore interno modificabile.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Arc permette la multi-proprietà tra i thread, Mutex garantisce la mutabilità sicura
    let contatore = Arc::new(Mutex::new(0));
    let mut tramezzi = vec![];

    for _ in 0..10 {
        let contatore_condiviso = Arc::clone(&contatore);
        let maniglia = thread::spawn(move || {
            let mut dati = contatore_condiviso.lock().unwrap();
            *dati += 1; // Mutabilità in thread safety
        });
        tramezzi.push(maniglia);
    }

    for maniglia in tramezzi {
        maniglia.join().unwrap();
    }

    println!("Risultato finale: {}", *contatore.lock().unwrap());
}
```

---

[← Fondamenti di Rust](./rust_fondamenti.md) | [Torna all'Hub](../index.md) | [Continua con le Best Practices →](./rust_best_practices.md)

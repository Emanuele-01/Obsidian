# 🚀 Best Practices in Rust

[← Ownership & Sicurezza](./rust_ownership_safety.md) | [Torna all'Hub](./index.md) | [Continua con Server Web & API →](./rust_web_services.md)

---

Scrivere codice idiomatico in Rust (noto come *Rustacean style*) garantisce che i programmi siano non solo sicuri dal punto di vista della memoria, ma anche estremamente veloci, manutenibili e chiari per gli altri sviluppatori. Di seguito sono elencate le principali best practices raccomandate.

---

## 1. Strumenti di Analisi e Formattazione Nativi

Rust include nella sua installazione standard una serie di strumenti per mantenere elevata la qualità del codice.

### Cargo Fmt (Formattazione)
Formatta automaticamente il codice secondo le linee guida ufficiali del linguaggio.
*   **Comando:** `cargo fmt` (eseguibile prima di ogni commit o configurato per l'esecuzione al salvataggio nell'IDE).

### Cargo Clippy (Linting statico)
Analizza il codice alla ricerca di pattern inefficienti, ridondanze o potenziali bug, suggerendo modifiche idiomatiche.
*   **Comando:** `cargo clippy`
*   *Esempio di suggerimento di Clippy:*
    ```rust
    // DA EVITARE:
    if stringa == "" { ... }

    // IDIOMATICO (suggerito da Clippy):
    if stringa.is_empty() { ... }
    ```

---

## 2. Gestione degli Errori: Evitare `unwrap()`

L'uso di `.unwrap()` o `.expect()` causa il crash improvviso dell'applicazione (*panic*) se il valore è `None` o `Err`.

> [!WARNING]
> Nei progetti di produzione (soprattutto in server web o software desktop), l'uso di `unwrap()` è da evitare quasi completamente. L'applicazione deve gestire gli errori in modo aggraziato restituendoli all'utente o provando a recuperare lo stato.

### Gestione Corretta
*   **Per applicazioni (es. CLI, Server Web, App Tauri):** Utilizza il crate **`anyhow`** per una gestione dinamica e flessibile degli errori.
    ```rust
    use anyhow::{Context, Result};

    fn leggi_configurazione() -> Result<String> {
        // Aggiunge contesto all'errore se la lettura fallisce
        std::fs::read_to_string("config.toml")
            .context("Impossibile leggere il file di configurazione")
    }
    ```
*   **Per librerie condivise:** Utilizza il crate **`thiserror`** per definire errori fortemente tipizzati ed esaustivi.

---

## 3. Scrivere Test Unitari e di Integrazione

Rust promuove la scrittura dei test integrando il framework direttamente nel compilatore.

### Test Unitari
Vengono scritti nello stesso file del codice sorgente, all'interno di un modulo dedicato marcato con l'attributo `#[cfg(test)]`. In questo modo i test vengono compilati solo quando si esegue `cargo test`.

```rust
// Codice dell'applicazione
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// Modulo di test integrato
#[cfg(test)]
mod tests {
    use super::*; // Importa le funzioni esterne al modulo

    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }
}
```

### Test di Integrazione
Vengono inseriti in una cartella separata chiamata `tests/` alla radice del progetto. Testano il codice importando il crate come se fosse una libreria esterna.

---

## 4. Ottimizzazione delle Allocazioni in Memoria

*   **Evitare il `.clone()` superfluo**: Clonare oggetti allocati in heap (come `String` o `Vec`) copia i dati fisicamente ed è un'operazione pesante. Cerca di passare riferimenti (`&`) o sfrutta il trasferimento di ownership (move) quando possibile.
*   **Preferire `&str` a `&String`** e **`&[T]` a `&Vec<T>`** come parametri delle funzioni. Questo rende la funzione molto più flessibile (può accettare sia stringhe/vettori allocati in heap, sia fette statiche sullo stack).
    ```rust
    // DA EVITARE:
    fn saluta(nome: &String) { ... }

    // CORRETTO:
    fn saluta(nome: &str) { ... }
    ```

---

## 5. Costrutti Idiomatici

### Iteratori invece di cicli imperativi
Gli iteratori in Rust sono un'astrazione a costo zero (*zero-cost abstraction*). Vengono ottimizzati in assembly efficiente quanto (o più di) un ciclo `for` classico.
```rust
let numeri = vec![1, 2, 3, 4, 5];

// Somma dei soli numeri pari elevati al quadrato
let somma: i32 = numeri.iter()
    .filter(|&&x| x % 2 == 0)
    .map(|&x| x * x)
    .sum();
```

### L'Entry API per le HashMap
Evita di effettuare due ricerche separate nella mappa per inserire o aggiornare un valore.
```rust
use std::collections::HashMap;

let mut conteggi = HashMap::new();
let parola = "rust";

// Cerca la parola; se non esiste la inserisce con valore 0, dopodiché incrementa di 1
*conteggi.entry(parola).or_insert(0) += 1;
```

---

[← Ownership & Sicurezza](./rust_ownership_safety.md) | [Torna all'Hub](./index.md) | [Continua con Server Web & API →](./rust_web_services.md)

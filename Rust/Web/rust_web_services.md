# 🌐 Sviluppo di Server Web & API con Rust

[← Best Practices](../Core/rust_best_practices.md) | [Torna all'Hub](../index.md) | [Continua con Tauri Desktop →](../Tauri/rust_tauri_cross_platform.md)

---

Grazie alle sue prestazioni vicine a quelle del C++ e all'assenza di un Garbage Collector, Rust è diventato uno dei linguaggi preferiti per la realizzazione di servizi web e microservizi ad altissima efficienza e con un consumo minimo di RAM.

In questa guida vedremo come strutturare un server REST asincrono utilizzando il framework **Axum** (sviluppato dal team di Tokio), il runtime asincrono **Tokio** e il database toolkit **SQLx**.

---

## 1. Lo Stack Tecnologico Principale

1.  **Tokio**: Il runtime asincrono standard de facto per Rust. Gestisce il pool di thread e l'I/O non-bloccante (essenziale per elaborare migliaia di connessioni concorrenti).
2.  **Axum**: Framework web flessibile costruito su `tokio`, `hyper` e `tower`. Sfrutta il sistema di tipi di Rust per mappare le richieste HTTP in parametri di funzione.
3.  **Serde**: La libreria di serializzazione e deserializzazione per eccellenza. Converte automaticamente le struct Rust in JSON e viceversa.
4.  **SQLx**: Un toolkit per database completamente asincrono che consente di scrivere query SQL pure controllate dal compilatore a tempo di compilazione.

---

## 2. Codice Completo di un Server Web REST

Ecco l'implementazione pratica di un server web in Rust che gestisce un database di utenti (tramite SQLx e PostgreSQL) e risponde a chiamate JSON.

### Dipendenze (`Cargo.toml`)
```toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
axum = "0.7"
serde = { version = "1.0", features = ["derive"] }
sqlx = { version = "0.7", features = ["runtime-tokio-native-tls", "postgres"] }
anyhow = "1.0"
```

### Codice Sorgente (`main.rs`)
```rust
use axum::{
    routing::{get, post},
    http::StatusCode,
    Json, Router, extract::State,
};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;
use std::net::SocketAddr;

// 1. MODELLO DATI (con derivazione automatica Serde)
#[derive(Serialize, Deserialize, Clone)]
struct Utente {
    id: i32,
    username: String,
    email: String,
}

// Struttura dati per la creazione dell'utente (Input JSON)
#[derive(Deserialize)]
struct CreaUtenteInput {
    username: String,
    email: String,
}

// Stato condiviso dell'applicazione (es. pool del database)
#[derive(Clone)]
struct AppState {
    db_pool: PgPool,
}

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Stringa di connessione a Postgres (solitamente caricata da variabili d'ambiente)
    let db_url = "postgres://postgres:password@localhost/mio_database";

    // 2. CREAZIONE DEL POOL DI CONNESSIONI AL DATABASE
    let pool = PgPool::connect(db_url)
        .await
        .expect("Impossibile connettersi al database");

    let stato_condiviso = AppState { db_pool: pool };

    // 3. DEFINIZIONE DEL ROUTING E DELLO STATO
    let app = Router::new()
        .route("/health", get(health_check))
        .route("/utenti", post(crea_utente).get(lista_utenti))
        .with_state(stato_condiviso); // Condivide il database con gli handler

    // 4. AVVIO DEL SERVER HTTP
    let indirizzo = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server in ascolto su http://{indirizzo}");
    
    let listener = tokio::net::TcpListener::bind(indirizzo).await?;
    axum::serve(listener, app).await?;

    Ok(())
}

// --- HANDLER HTTP ---

// Semplice controllo di salute del server
async fn health_check() -> &'static str {
    "OK"
}

// Handler per creare un utente nel database
async fn crea_utente(
    // Estrae lo stato condiviso (il db pool)
    State(stato): State<AppState>,
    // Deserializza automaticamente il body JSON in CreaUtenteInput
    Json(payload): Json<CreaUtenteInput>,
) -> Result<(StatusCode, Json<Utente>), StatusCode> {
    
    // Esegue una query SQL asincrona e sicura
    let utente_creato = sqlx::query_as!(
        Utente,
        "INSERT INTO utenti (username, email) VALUES ($1, $2) RETURNING id, username, email",
        payload.username,
        payload.email
    )
    .fetch_one(&stato.db_pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?; // Gestione dell'errore (Result)

    // Ritorna lo status 201 (Created) e l'oggetto JSON creato
    Ok((StatusCode::CREATED, Json(utente_creato)))
}

// Handler per listare gli utenti salvati
async fn lista_utenti(
    State(stato): State<AppState>,
) -> Result<Json<Vec<Utente>>, StatusCode> {
    
    let utenti = sqlx::query_as!(
        Utente,
        "SELECT id, username, email FROM utenti"
    )
    .fetch_all(&stato.db_pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(Json(utenti))
}
```

---

## 3. Spiegazione dei Concetti Chiave

### Runtime Asincrono (`#[tokio::main]`)
Rust non possiede una gestione dei thread asincroni built-in nel suo motore runtime per mantenere il binario leggero. Si usa la macro `#[tokio::main]` per convertire la funzione `main` in una coroutine asincrona che gestisce la coda di esecuzione di tutte le funzioni marcate con `.await`.

### Estrazione dello Stato (`State<T>`)
Axum permette di condividere risorse (connessioni database, configurazioni, cache) in modo sicuro tra i vari handler. La struttura `AppState` viene passata a `.with_state(stato)` e gli handler la estraggono tramite il parametro `State(stato)`. Questo processo garantisce la thread safety grazie alle regole di [Sync e Send](../Core/rust_ownership_safety.md#6-concorrenza-sicura-fearless-concurrency).

### Derivazione Serde (`Serialize`/`Deserialize`)
Aggiungendo `#[derive(Serialize)]` a una struct, Serde genera a tempo di compilazione il codice per convertire quella struct in JSON. `Deserialize` fa il processo inverso. Questo evita il ricorso alla reflection a runtime, aumentando le prestazioni e riducendo l'uso di memoria.

### Prevenzione SQL Injection con SQLx
Le query di SQLx sono parametrizzate (usando `$1`, `$2`), il che blocca a monte attacchi di SQL Injection. Inoltre, la macro `query_as!` controlla che le colonne del database corrispondano esattamente ai tipi di dato definiti nelle struct di Rust a tempo di compilazione.

---

[← Best Practices](../Core/rust_best_practices.md) | [Torna all'Hub](../index.md) | [Continua con Tauri Desktop →](../Tauri/rust_tauri_cross_platform.md)

# Approfondimento Tecnico: Algoritmi Crittografici in SSH (RSA, ECDSA, Ed25519)

Nel protocollo **SSH (Secure Shell)**, l'autenticazione tramite chiave si basa sulla **crittografia asimmetrica** (coppia di chiavi pubblica/privata). Nel corso degli anni, diversi algoritmi matematici sono stati introdotti per generare queste chiavi. 

Questo documento analizza in dettaglio i tre algoritmi principali utilizzati in SSH, confrontandone la sicurezza, le performance e la struttura matematica.

---

## 1. RSA (Rivest-Shamir-Adleman)

**RSA** è l'algoritmo di crittografia asimmetrica più vecchio e diffuso. La sua sicurezza si basa sulla **difficoltà matematica della fattorizzazione di grandi numeri interi** in fattori primi:
* Dati due numeri primi giganteschi $p$ e $q$, è facile calcolare il loro prodotto $n = p \times q$.
* Al contrario, dato solo $n$, è estremamente difficile ricavare $p$ e $q$ (problema della fattorizzazione).

### Sicurezza e Dimensioni delle Chiavi
* **Lunghezza minima:** Lo standard industriale considera le chiavi RSA a **1024-bit vulnerabili** e insicure.
* **Standard attuale:** GitHub e i sistemi moderni richiedono una lunghezza minima di **3072-bit** o, preferibilmente, **4096-bit**.
* **Limiti:** Le chiavi RSA molto lunghe richiedono più cicli CPU sia per la generazione che per la firma/verifica dei messaggi, aumentando i tempi di connessione su hardware limitato (es. dispositivi IoT o server embedded).

---

## 2. ECDSA (Elliptic Curve Digital Signature Algorithm)

Per superare la pesantezza delle chiavi RSA, è stata introdotta la **crittografia basata sulle curve ellittiche (ECC)**. ECDSA adatta l'algoritmo DSA classico alle curve ellittiche. La sua sicurezza si basa sulla **difficoltà del logaritmo discreto su curve ellittiche (ECDLP)**.

### Vantaggi e Controversie
* **Efficienza:** Una chiave ECDSA a **256-bit** offre lo stesso livello di sicurezza crittografica di una chiave RSA a **3072-bit**, riducendo drasticamente le dimensioni dei dati trasmessi.
* **Controversie sulla Sicurezza (NSA Backdoor):** ECDSA utilizza curve standardizzate dal **NIST** (come la curva *P-256* o *P-384*). Diversi crittografi (tra cui Bruce Schneier) hanno sollevato sospetti che i parametri di queste curve contengano costanti scelte ad hoc da agenzie governative (come la NSA) per facilitare attacchi di forza bruta invisibili.
* **Sensibilità all'Entropia:** ECDSA richiede un valore casuale ($k$) estremamente puro durante ogni processo di firma. Se il generatore di numeri casuali (RNG) della macchina ha una scarsa entropia, la chiave privata può essere matematicamente ricostruita con pochissime firme registrate da un intercettatore.

---

## 3. Ed25519 (Edwards-curve Digital Signature Algorithm)

Introdotto in OpenSSH nel 2014, **Ed25519** rappresenta oggi il gold standard assoluto per le chiavi SSH. È un'implementazione del protocollo EdDSA che utilizza la curva ellittica ritorta di Edwards denominata **Curve25519**, sviluppata dal crittografo Daniel J. Bernstein.

### Perché Ed25519 è Superiore?

1. **Massima Sicurezza Crittografica:**
   Una chiave Ed25519 occupa solo **256 bit** (generando stringhe cortissime nel file `.pub`), ma offre una resistenza crittografica superiore rispetto a RSA a 4096-bit.

2. **Resistenza ai Side-Channel Attacks:**
   Gli algoritmi RSA ed ECDSA possono essere vulnerabili ad attacchi basati sul tempo (*timing attacks*) o sul consumo energetico della CPU durante i calcoli della firma. Ed25519 è progettato matematicamente per essere eseguito in **tempo costante (Constant-Time)** indipendentemente dai dati di input, neutralizzando queste minacce a livello hardware.

3. **Firme Deterministiche:**
   A differenza di ECDSA, Ed25519 non si affida a un generatore di numeri casuali dinamico per calcolare la firma a ogni connessione, ma genera la costante di firma in modo deterministico combinando un hash della chiave privata con il messaggio. Questo elimina il rischio di furto di chiave derivante da una scarsa entropia di sistema.

4. **Velocità:**
   Ed25519 è significativamente più veloce di RSA e di ECDSA nella generazione delle firme e nella loro validazione, riducendo i tempi di handshake SSH.

---

## Tabella Comparativa

| Algoritmo | Dimensione Chiave Consigliata | Sicurezza Relativa | Velocità Calcolo | Resistenza ad Attacchi Side-Channel |
| :--- | :--- | :--- | :--- | :--- |
| **RSA** | 3072 / 4096 bit | Buona (ma chiavi enormi) | Lento | Bassa |
| **ECDSA** | 256 / 384 bit | Ottima (sospetto backdoor NIST) | Veloce | Bassa (dipende da RNG) |
| **Ed25519** | **256 bit** | **Eccellente (Gold Standard)** | **Ultrarapido** | **Alta (Tempo Costante)** |

---

## Collegamenti Consigliati
* Per vedere come generare una chiave Ed25519 sul proprio terminale per GitHub, vedi [[Come Generare Chiavi SSH per GitHub#Come Generare le Chiavi SSH Localmente]].
* Per implementare la chiave Ed25519 in configurazioni ad account multipli, leggi [[Approfondimento Configurazione Avanzata SSH]].
* Per i concetti crittografici generali, vedi [[Come Generare Chiavi SSH per GitHub#Chiavi Pubbliche e Private]].

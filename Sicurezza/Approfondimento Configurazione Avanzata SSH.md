# Approfondimento Ingegneristico: Configurazione Avanzata del Client SSH e Proxy Tunneling

Il file di configurazione locale di SSH (`~/.ssh/config`) è uno strumento potente che permette di andare ben oltre la semplice memorizzazione di alias per host remoti. Questo documento analizza le configurazioni avanzate del file client, comprese le tecniche di **Proxy Jump** (bastion hosts), i meccanismi di **Keep-Alive** per mantenere attive le connessioni, l'**Agent Forwarding** e le implicazioni di sicurezza del tunneling.

---

## 1. Struttura e Parametri Avanzati del Config File

Il file `~/.ssh/config` viene letto sequenzialmente da OpenSSH ogni volta che si esegue il comando `ssh`. Ecco un esempio di struttura che implementa parametri avanzati:

```bash
# Parametri globali applicati a tutti gli host (devono stare all'inizio o alla fine)
Host *
  AddKeysToAgent yes
  UseKeychain yes # Solo su macOS per memorizzare la passphrase nel Keychain
  ServerAliveInterval 60
  ServerAliveCountMax 3

# Configurazione per il server di produzione
Host prod-server
  HostName 198.51.100.42
  User deployer
  Port 2222 # Porta personalizzata per sicurezza
  IdentityFile ~/.ssh/prod_key_ed25519
  IdentitiesOnly yes
```

### Analisi dei Parametri Critici:
* **`AddKeysToAgent yes`:** Se abilitato, quando si utilizza una chiave per la prima volta inserendo la passphrase, questa viene aggiunta automaticamente all'agente di autenticazione (`ssh-agent`), evitando di dover eseguire `ssh-add` manualmente.
* **`IdentitiesOnly yes`:** Costringe il client a proporre al server esclusivamente la chiave definita nel parametro `IdentityFile` per quel determinato host. Senza questa opzione, OpenSSH propone in sequenza tutte le chiavi caricate in `ssh-agent`. Se si hanno molte chiavi caricate, il server potrebbe rifiutare la connessione con l'errore *Too many authentication failures* prima di raggiungere la chiave corretta.

---

## 2. Proxy Jump: Accedere a Reti Private tramite Bastion Host

Nelle architetture di rete sicure in cloud (es. AWS VPC o server aziendali), le macchine di database o di backend sono posizionate in una sottorete privata non raggiungibile direttamente da Internet. L'unico punto di ingresso è un server gateway esposto pubblicamente, chiamato **Bastion Host** o **Jump Box**.

La direttiva **`ProxyJump`** automatizza la connessione a cascata (tunneling cifrato end-to-end) in modo trasparente.

```
          [ Computer Client ]
                  |
         (Internet, Porta 22)
                  v
         [ Bastion Host (Public IP) ]
                  |
        (Rete Interna, Porta 22)
                  v
       [ Database Server (Private IP) ]
```

### Configurazione nel Config File:
```bash
# Il server gateway intermedio (Bastion)
Host bastion
  HostName bastion.azienda.com
  User gateway-admin
  IdentityFile ~/.ssh/bastion_key

# Il database protetto nella sottorete privata
Host db-private
  HostName 10.0.1.45 # IP privato del database
  User db-operator
  IdentityFile ~/.ssh/db_key
  ProxyJump bastion # Esegue il tunneling attraverso l'host "bastion"
```

* **Funzionamento:** Eseguendo semplicemente `ssh db-private`, OpenSSH stabilisce la connessione con `bastion`, avvia un canale sicuro e lo usa come proxy per connettersi a `10.0.1.45`. Il traffico viene cifrato localmente con la chiave `db_key` prima di passare per il bastion host, garantendo che l'amministratore del bastion non possa decifrare i dati diretti al database.

---

## 3. Port Forwarding (SSH Tunneling)

SSH permette di esporre porte di rete remote in locale (o viceversa) attraverso il canale cifrato SSH.

### A. Local Port Forwarding (`-L`)
Invia il traffico da una porta locale a una determinata porta di un server remoto.
* **Caso d'uso:** Connettersi a un database PostgreSQL remoto (porta 5432) non esposto su Internet:
  ```bash
  ssh -L 5432:localhost:5432 prod-server
  ```
  Ora è possibile puntare un client grafico locale (es. DBeaver) su `localhost:5432` per interagire in sicurezza con il database remoto.

### B. Remote Port Forwarding (`-R`)
Espone un servizio locale sulla rete del server remoto.
* **Caso d'uso:** Mostrare uno sviluppo in corso sulla propria macchina locale (es. porta 3000) a un collega esterno facendolo passare per un server pubblico VPS:
  ```bash
  ssh -R 8080:localhost:3000 vps-pubblico
  ```

---

## 4. SSH Agent Forwarding e Rischi di Sicurezza

La direttiva **`ForwardAgent yes`** permette a un server remoto di accedere al socket del proprio client `ssh-agent` locale per autenticarsi su altre macchine senza dover copiare fisicamente la propria chiave privata sul server remoto.

> [!WARNING]
> **Implicazioni di Sicurezza critiche:** Se un attaccante ottiene i permessi di amministratore (`root`) sul server intermedio su cui è attivo l'Agent Forwarding, può intercettare il socket Unix dell'agente temporaneo in `/tmp/ssh-XXXXXX/agent.XXXX` e utilizzarlo per autenticarsi a nome dell'utente su tutti gli altri server in cui è autorizzata la chiave originale. Abilita `ForwardAgent` solo su server di cui ti fidi ciecamente o prediligi sempre `ProxyJump` (dove la chiave viene decifrata solo localmente).

---

## Collegamenti Consigliati
* Per comprendere le basi delle chiavi SSH e l'uso dell'agente locale, vedi [[Come Generare Chiavi SSH per GitHub#Aggiungere la Chiave SSH a ssh-agent]].
* Per analizzare la robustezza degli algoritmi crittografici da configurare nel file di identità, leggi [[Approfondimento Algoritmi Crittografici SSH]].
* Per implementare i file di configurazione per account multipli, vedi [[Come Generare Chiavi SSH per GitHub#Gestire Più Chiavi SSH per Diversi Account GitHub]].

# Approfondimento Tecnico: Vulnerabilità TOCTOU (Time-of-Check to Time-of-Use) e Race Conditions

Nello sviluppo di software sicuro e nei sistemi operativi multitasking, la corretta sincronizzazione delle risorse è una priorità. La vulnerabilità **TOCTOU (Time-of-Check to Time-of-Use)** è una specifica classe di bug logici di tipo **Race Condition** che si verifica quando uno stato o una risorsa (tipicamente un file) viene modificata da un processo concorrente tra il momento del controllo e il momento dell'uso effettivo.

Questo documento analizza la dinamica della vulnerabilità TOCTOU, fornisce un esempio pratico in C di attacco tramite link simbolico e descrive le tecniche di mitigazione ingegneristiche avanzate.

---

## 1. La Dinamica Temporale del TOCTOU

Il ciclo di vita di un'operazione su file in programmi vulnerabili segue due fasi logiche distinte:
1. **Time of Check (Tempo del Controllo):** Il programma verifica le proprietà di un file o di un percorso (es. permessi di accesso, presenza o tipo di risorsa).
2. **Time of Use (Tempo dell'Uso):** Il programma esegue un'azione basata sul presupposto che lo stato verificato nella fase precedente sia rimasto inalterato.

```
Dinamica temporale di una Race Condition TOCTOU:

PROCESSO VULNERABILE (Privilegiato/Root)       ATTACCANTE (Privilegiati Limitati)
---------------------------------------       ----------------------------------
1. CONTROLLO (Check):
   Verifica se "/tmp/dati" è scrivibile
   dall'utente limitato (access() OK)
                                       ===>  2. INTERVENTO (Finestra di Race):
                                             Rimuove il file "/tmp/dati"
                                             e lo sostituisce con un link simbolico
                                             a "/etc/shadow" o "/etc/passwd".
3. UTILIZZO (Use):
   Scrive dati sensibili su "/tmp/dati"
   (Scrittura dirottata su "/etc/shadow"!)
```

Se l'attaccante riesce ad agire nella **finestra di race** (la frazione di millisecondo che separa il controllo dall'uso), può indurre un'applicazione eseguita con privilegi elevati (es. un demone di sistema o un programma SUID root) a manipolare file sensibili del sistema operativo.

---

## 2. Esempio Pratico: Attacco con Symlink Swap

Si consideri un frammento di codice C vulnerabile in un'applicazione di utilità eseguita come `root`:

```c
#include <unistd.h>
#include <stdio.h>

void scrivi_log_utente(const char* percorso) {
    // 1. TIME OF CHECK: Verifica se il percorso appartiene all'utente reale
    if (access(percorso, W_OK) == 0) {
        
        // --- FINESTRA DI RACE ---
        // L'attaccante esegue: ln -sf /etc/passwd percorso
        
        // 2. TIME OF USE: Apertura e scrittura del file
        FILE *f = fopen(percorso, "w");
        if (f != NULL) {
            fprintf(f, "Log di sistema autorizzato...\n");
            fclose(f);
        }
    } else {
        printf("Accesso negato.\n");
    }
}
```

### Come avviene l'exploit:
1. Il programma controlla `access(percorso, W_OK)`. Poiché `/tmp/user_log` è di proprietà dell'utente normale, il controllo passa con successo.
2. L'attaccante esegue uno script in background che monitora il sistema e scambia `/tmp/user_log` con un link simbolico a `/etc/passwd`.
3. Il programma esegue `fopen(percorso, "w")`. Poiché segue i link simbolici (*resolving links*), apre `/etc/passwd` e sovrascrive i dati utente con privilegi di `root`, permettendo all'attaccante di corrompere i file di autenticazione o iniettare record utente contraffatti per scalare i privilegi.

---

## 3. Tecniche di Mitigazione Ingegneristiche

Per neutralizzare le vulnerabilità TOCTOU è necessario eliminare la finestra temporale tra controllo e utilizzo o rendere l'operazione atomica.

### A. Operazioni Atomiche (Atomic Operations)
Invece di eseguire un controllo prima di un'azione, si esegue direttamente l'azione impostando flag di sistema che garantiscano l'esclusività a livello atomico.
* **Creazione Sicura di File:** Quando si apre un file, si utilizzano i flag combinati `O_CREAT | O_EXCL` con la chiamata di sistema `open()`:
  ```c
  // Fallisce atomicamente a livello di kernel se il file esiste già,
  // impedendo di seguire link simbolici creati al volo.
  int fd = open("/tmp/dati", O_WRONLY | O_CREAT | O_EXCL, 0600);
  ```

### B. Utilizzo di File Descrittori (File Descriptors)
Molti attacchi TOCTOU sfruttano la manipolazione del percorso testuale del file (*filepath*). Una volta aperto un file in sicurezza tramite una chiamata atomica, tutte le operazioni successive (chmod, chown, scrittura) devono avvenire tramite il **file descriptor (FD)** anziché il percorso testuale.
* Si prediligono le chiamate di sistema che accettano un FD:
  * `fchmod()` invece di `chmod()`
  * `fchown()` invece di `chown()`
  * `fstat()` invece di `stat()`
* Una volta ottenuto il file descriptor, il sistema operativo punta direttamente all'inode logico nel file system, rendendo inutile qualsiasi scambio di link o rimozione del percorso da parte di un attaccante concorrente.

### C. Utilizzo dei File Lock per la Sincronizzazione Concorrente
L'acquisizione di un **File Lock** atomico tramite chiamate di sistema (come `flock()` o `fcntl()` su file descriptor) garantisce che una risorsa critica non venga modificata da processi concorrenti cooperativi durante tutto il ciclo di vita delle operazioni di check e use.
* Il blocco impedisce l'accesso in scrittura di altri processi cooperativi fino al rilascio, garantendo l'integrità dei dati e la consistenza delle letture.

---

## Collegamenti Consigliati
* Per comprendere come implementare i file lock a livello di shell e in Go per prevenire race conditions, leggi [[File Lock#2. Come si configura e si implementa?]].
* Per analizzare l'impatto dei file lock nella prevenzione di Denial of Service (DoS) indotti, vedi [[File Lock#B. Prevenzione del Denial of Service (DoS) indotto]].
* Per esplorare la gestione e la sicurezza dei file all'interno di container, consulta [[File Lock#C. Configurazione nei Container (Docker Compose)]].

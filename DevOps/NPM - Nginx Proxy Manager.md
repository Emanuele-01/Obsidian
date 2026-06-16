Per capire a fondo **Nginx Proxy Manager (NPM)** dal punto di vista ingegneristico, dobbiamo analizzare la sua natura architetturale: NPM non è un software server scritto da zero, ma un **wrapper architetturale e gestionale** costruito sopra una distribuzione ufficiale di **Nginx**.

Dal punto di vista tecnico, l'architettura si divide in tre componenti fondamentali: l'infrastruttura di routing, il pannello di controllo (Control Plane) e il ciclo di vita dei certificati.

### 1. A cosa serve (L'architettura Reverse Proxy & TLS Termination)

In un'infrastruttura di micro servizi o di self-hosting, i moduli applicativi (container Docker, runtime Node.js, database, pannelli web) espongono socket TCP su porte non standard (es. `8080`, `9443`, `3000`). Esporle direttamente sul perimetro pubblico della rete è un rischio di sicurezza e un limite architetturale.

NPM si posiziona come **Reverse Proxy** e **Edge Router** (punto di ingresso unico). I suoi scopi tecnici principali sono:

- **Inversion of Control (Routing di Livello 7):** NPM opera al livello Applicazione (L7) dello stack OSI. Legge l'header `Host` delle richieste HTTP/HTTPS entranti (es. `google.it` vs `google.com`) e mappa i flussi di traffico verso i corretti socket interni.
    
- **TLS/SSL Termination:** NPM agisce come l'estremo finale della crittografia asimmetrica verso l'esterno. Gestisce l'handshake TLS (fino a TLS 1.3), decifra il traffico sul perimetro e lo inoltra ai servizi di backend in chiaro (HTTP) o tramite canali cifrati interni, sollevando le applicazioni a valle dal carico computazionale della crittografia.
    
- **Abstraction Layer:** Disaccoppia l'indirizzamento pubblico (DNS) dall'infrastruttura di rete interna (Docker Bridge o porte di localhost).
    

### 2. Come funziona internamente (Sotto il cofano)

NPM è distribuito come un container Docker multi-processo. Se entriamo all'interno del container, troviamo un'architettura divisa nettamente tra la logica di business e il motore di routing:

```
[ Traffico Internet: Porte 80 / 443 ]
                  │
                  ▼
┌─────────────────╪─────────────────────────┐
│ NPM CONTAINER   │                         │
│                 ▼                         │
│       ┌───────────────────┐               │
│       │    Nginx Core     │               │
│       │ (Reverse Proxy)   │               │
│       └─────────┬─────────┘               │
│                 │ (Legge file .conf)      │
│                 ▲                         │
│                 │                         │
│       ┌─────────┴─────────┐               │
│       │  Node.js / Skynet │               │
│       │  (Control Plane)  │               │
│       └─────────▲─────────┘               │
│                 │                         │
│                 ▼                         │
│       ┌───────────────────┐               │
│       │ SQLite / MariaDB  │               │
│       │    (Database)     │               │
│       └───────────────────┘               │
└─────────────────┬─────────────────────────┘
                  │
                  ▼
[ Rete Interna: Porte 8080 / 9443 ]
```

#### A. Il Control Plane (Interfaccia e API)

L'interfaccia grafica che utilizzi sulla porta `81` è un'applicazione **Node.js** (basata su framework backend) che espone delle API REST. Quando interagisci con l'interfaccia (es. aggiungi un Proxy Host):

1. I dati vengono validati e scritti in un database relazionale (di default **SQLite** per istanze singole, o MariaDB/MySQL).
    
2. Un subset del codice di NPM (spesso integrato con script OpenResty/Lua o Node) prende questi record e genera dinamicamente dei file di configurazione Nginx standard validi (file `.conf`).
    
3. Questi file vengono scritti in una directory condivisa (mappata nel volume `/data/nginx/proxy_host/`).
    
4. Viene inviato un segnale di `nginx -s reload` al processo master di Nginx per applicare le modifiche a caldo, a zero-downtime, senza interrompere le connessioni attive.
    

#### B. Il Data Plane (Il Motore Nginx)

Il core binario di Nginx è configurato per includere dinamicamente i file generati dal Control Plane. Un tipico Proxy Host autogenerato da NPM si traduce in un blocco `server` strutturato tecnicamente così:

Nginx

```
server {
    listen 80;
    listen [::]:80;
    server_name home.emanuelepitoni.eu;

    # Certbot / Let's Encrypt ACME Challenge Webroot
    include conf.d/include/letsencrypt-acme-challenge.conf;

    # Redirezione forzata su HTTPS (se attiva)
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name home.emanuelepitoni.eu;

    # Parametri SSL crittografici (Cifrari TLS moderni)
    ssl_certificate /etc/letsencrypt/live/npm-X/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/npm-X/privkey.pem;
    include conf.d/include/ssl-ciphers.conf;

    location / {
        # Configurazione degli header di Proxying (Iniezione Layer 7)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Supporto per i WebSocket (Upgrade dell'HTTP Upgrade/Connection)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Inoltro sul socket di destinazione configurato
        proxy_pass http://172.17.0.1:9443;
    }
}
```

### 3. La gestione automatizzata del protocollo ACME (Let's Encrypt)

Uno dei blocchi architetturali più complessi di NPM è l'automazione dei certificati SSL/TLS tramite il client **Certbot** integrato.

1. **ACME HTTP-01 Challenge:** Quando richiedi un certificato, NPM istruisce Certbot per avviare una richiesta al server di Let's Encrypt. Let's Encrypt risponde inviando un "token" crittografico.
    
2. **Esposizione del Token:** Certbot posiziona questo token in una directory specifica del container. Nginx, tramite una direttiva pre-configurata (la `letsencrypt-acme-challenge.conf` che vedi nel blocco sopra), espone temporaneamente quel token sulla porta `80` all'indirizzo `http://home.emanuelepitoni.eu/.well-known/acme-challenge/token`.
    
3. **Validazione:** I server di Let's Encrypt interrogano il DNS pubblico, bussano all'IP della tua VPS sulla porta 80 cercando quel file. Se lo trovano e il token coincide, l'autorità certificativa valida il possesso del dominio e rilascia i file del certificato (`fullchain.pem` e `privkey.pem`).
    
4. **Automazione Cron-Job:** All'interno del container gira un servizio di pianificazione (un timer interno) che ogni giorno controlla la data di scadenza dei certificati in `/etc/letsencrypt/live/`. Se un certificato ha una validità residua inferiore a 30 giorni, NPM avvia autonomamente il rinnovo in background e ricarica Nginx.
    

### In sintesi

NPM centralizza l'infrastruttura di rete convertendo input logici ad alto livello (da UI web) in direttive di basso livello a basso costo computazionale (file di configurazione Nginx nativi), delegando poi l'effettivo smistamento dei pacchetti TCP/IP e l'elaborazione degli header HTTP alle performance native del motore asincrono a eventi di Nginx.
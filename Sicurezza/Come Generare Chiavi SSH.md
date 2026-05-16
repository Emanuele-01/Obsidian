---
title: "Come Generare Chiavi SSH per GitHub"
source: "https://kinsta.com/it/blog/generare-chiavi-ssh/"
author:
  - "[[Daniel Diaz]]"
published: 2022-02-21
created: 2026-05-16
description: "Ora che il tuo progetto è su GitHub, come fai a mantenerlo sicuro? Ecco tutto quello che devi sapere sulla generazione di chiavi SSH per GitHub."
tags:
  - "clippings"
---
[Git e GitHub](https://kinsta.com/it/blog/cosa-e-github/) sono strumenti essenziali per chiunque lavori nello sviluppo. Sono ampiamente utilizzati in quasi tutti i tipi di progetti di sviluppo software.

Esistono altri servizi di hosting Git come [Gitlab](https://kinsta.com/it/blog/gitlab-vs-github/) e [Bitbucket](https://kinsta.com/it/blog/bitbucket-e-github/), ma GitHub è la scelta più popolare per gli sviluppatori. Potete anche modificare il vostro profilo per renderlo più attraente agli occhi dei recruiter.

Potete usare Git e GitHub per organizzare i vostri progetti, collaborare con altri sviluppatori e sviluppatrici, e ovviamente lo usiamo anche noi di [Kinsta](https://kinsta.com/it/docs/hosting-wordpress/gestione-sito/git/).

Ma poiché [Git e GitHub sono strumenti correlati ma diversi](https://kinsta.com/it/blog/git-contro-github/), dovete aggiornare costantemente il vostro flusso di lavoro con loro.

Raccomandiamo di usare le chiavi [SSH](https://kinsta.com/it/docs/hosting-wordpress/connessione-ssh/) per ciascuna delle vostre macchine. Quindi, in questo tutorial, imparerete cosa sono le chiavi SSH di GitHub, quali sono i loro vantaggi e come generarle e configurarle.

Iniziamo!

## Cosa Sono le Chiavi SSH?

In parole povere, le chiavi SSH sono credenziali utilizzate per il [protocollo SSH (Secure Shell)](https://kinsta.com/it/blog/ssh-ssl/) per consentire un accesso sicuro ai computer remoti su internet. Di solito, l’autenticazione avviene in un ambiente a riga di comando.

Questo protocollo è basato su un’architettura client-server, il che significa che voi come utenti (o “client”) dovete usare un software speciale, chiamato client SSH, per accedere a un server remoto ed eseguire comandi. Questo è fondamentalmente quello che state facendo quando vi autenticate tramite terminale a GitHub.

![Il Terminal mostra due comandi: ](https://kinsta.com/wp-content/uploads/2022/01/Git-push-1024x396.png)

Push su Git.

Ma SSH non è usato solo per GitHub. È usato molto anche da altre piattaforme come Kinsta, Google Cloud e Amazon Web services per creare un canale sicuro per accedere ai loro servizi.

Ora, entrando nel merito di come funzionano realmente le chiavi SSH, dovete capire le differenze tra chiavi pubbliche e private.

### Chiavi Pubbliche e Private

Iniziamo con le basi.

Il protocollo SSH usa una tecnica di crittografia chiamata **crittografia asimmetrica**. Questo termine può sembrare complicato e strano, ma niente potrebbe essere più lontano dalla verità.

Fondamentalmente, la crittografia asimmetrica è un sistema che usa una coppia di chiavi, cioè la chiave **pubblica** e quella **privata**.

Come potete immaginare, la chiave pubblica può essere condivisa con chiunque. Il suo scopo principale è quello di criptare i dati, convertendo il messaggio in un codice segreto o cifrario. Questa chiave viene solitamente inviata ad altri sistemi, per esempio i server, per criptare i dati prima di inviarli su Internet.

D’altra parte, la chiave privata è quella che dovete tenere per voi. Viene usata per decifrare i dati crittografati con la vostra chiave pubblica. Senza di essa, è impossibile decodificare le vostre informazioni criptate.

Questo metodo permette a voi e al server di mantenere un canale di comunicazione sicuro per trasmettere le informazioni.

Ecco cosa succede in background quando vi connettete a un server tramite SSH:

1. Il client invia la chiave pubblica al server.
2. Il server chiede al client di firmare un messaggio casuale criptato con la chiave pubblica usando la chiave privata.
3. Il client firma il messaggio e inoltra il risultato al server.
4. Viene stabilita una connessione sicura tra il client e il server.

È importante tenere le chiavi private al sicuro e non condividerle con nessuno in nessun caso perché sono la chiave di tutte le informazioni che vi vengono inviate.

## Usare le Chiavi SSH con GitHub

Dal 13 agosto 2021, Github non accetta più l’autenticazione tramite password per l’accesso alla riga di comando. Questo significa che ora dovete autenticarvi tramite un token di accesso personale o usare una chiave SSH (un po’ più conveniente).

Ecco cosa succede quando provate ad autenticarvi con la vostra password di GitHub via HTTP in un Terminal:

```markdown
Username for 'https://github.com': yourusername

Password for 'https://yourusername@github.com':

remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.

fatal: Authentication failed for 'https://github.com/yourusername/repository.git/'
```

GitHub ha bisogno della vostra chiave pubblica per autorizzarvi a modificare qualsiasi vostra repo via SSH.

Vediamo come potete generare le chiavi SSH localmente.

### Come Generare le Chiavi SSH Localmente

Ora che avete capito un po’ il protocollo SSH e le differenze tra chiavi pubbliche e private, è il momento di impostare il canale SSH sicuro tra la vostra macchina e i vostri repo GitHub.

Prima di andare avanti, dovreste già avere un [account GitHub](https://github.com/join) e un terminale/punto di comando con [Git](https://git-scm.com/downloads) installato nel vostro sistema. Se state usando Windows, assicuratevi di aver installato [Git bash](https://git-scm.com/download/win), che ha tutti gli strumenti di cui avrete bisogno per seguire questo tutorial.

Il client OpenSSH è il più popolare software open-source utilizzato per connettersi via SSH. Non dovrete preoccuparvi del vostro sistema operativo perché è installato di default su Linux, [macOS](https://kinsta.com/it/blog/fare-uno-screenshot-su-mac/) e [Windows 10](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview).

Per generare chiavi SSH locali dovete avviare un prompt dei comandi su Windows o un terminale sui sistemi basati su Unix. Di solito, potete farlo cercando “terminal”, “cmd”, o “powershell” nel vostro pannello delle applicazioni e facendo clic sull’icona che appare.

![Application finder che mostra diverse applicazioni del Terminal, tra cui ](https://kinsta.com/wp-content/uploads/2022/01/open-terminal-1024x526.png)

Ricerca dell’applicazione da Terminal.

Dopo aver fatto questo, dovreste avere una finestra simile alla seguente immagine.

![Applicazione Terminal semitrasparente che esegue un fish shell.](https://kinsta.com/wp-content/uploads/2022/01/terminal-1024x511.png)

Applicazione del terminal.

Eseguite il seguente comando per generare una coppia di chiavi SSH locali:

```bash
ssh-keygen -t ed25519 -C "kinstauser@kinsta.com"
```

È il momento di svelarvi un segreto: nessuno può davvero ricordare questo comando! La maggior parte degli sviluppatori deve cercarlo su Google ogni volta perché:

1. È un comando molto lungo, con numeri dimenticabili e casuali.
2. Lo usiamo raramente, quindi non vale la pena ricordarlo.

Tuttavia, è importante capire ogni comando che introduciamo nei nostri terminali, quindi vediamo cosa significa ogni sua parte.

- [ssh-keygen](https://linux.die.net/man/1/ssh-keygen): lo strumento a riga di comando utilizzato per creare una nuova coppia di chiavi SSH. Potete vedere i suoi flag con `ssh-keygen help`
- – **t ed25519**: il flag `-t` si usa per indicare l’algoritmo utilizzato per creare la firma digitale della coppia di chiavi. Se il vostro sistema lo supporta, `ed25519` è il miglior algoritmo che potete usare per creare coppie di chiavi SSH.
- **\-C “email”**: il flag `-C` è utilizzato per fornire un commento personalizzato alla fine della chiave pubblica, che di solito è l’email o l’identificazione del creatore della coppia di chiavi.

Dopo aver digitato il comando nel vostro terminale, dovrete inserire il file in cui vorreste salvare le chiavi. Per impostazione predefinita, si trova nella vostra home directory, in una cartella nascosta chiamata “.ssh”, ma potete cambiarla in quello che volete.

Poi vi verrà chiesta una passphrase da aggiungere alla vostra coppia di chiavi. Questo aggiunge un ulteriore livello di sicurezza se, in qualsiasi momento, il vostro dispositivo viene compromesso. Non è obbligatorio aggiungere una passphrase, ma è sempre raccomandato.

Questo è l’aspetto dell’intero processo:

![Il comando ssh-keygen con vari messaggi che con vari messaggi tra cui il file in cui verranno salvate le chiavi ](https://kinsta.com/wp-content/uploads/2022/01/ssh-keygen-1024x540.png)

Comando ssh-keygen.

Come potete vedere, questo comando genera due file nella directory che avete selezionato (comunemente **~/.ssh**): la chiave pubblica con l’estensione `.pub` e quella privata senza estensione.

Vi mostreremo come aggiungere la chiave pubblica al vostro account GitHub più tardi.

### Aggiungere la Chiave SSH a ssh-agent

Il programma **ssh-agent** viene eseguito in background, mantiene le vostre chiavi private e le frasi di accesso al sicuro e le tiene pronte per l’uso da parte di ssh. È una grande utility che vi evita di digitare la vostra passphrase ogni volta che volete connettervi a un server.

Per questo motivo, aggiungerete la vostra nuova chiave privata a questo agente. Ecco come:

1. Assicuratevi che ssh-agent sia in esecuzione in background.
	```bash
	eval \`ssh-agent\`
	# Agent pid 334065
	```
	Se ricevete un messaggio simile a questo tutto va bene: significa che ssh-agent è in esecuzione sotto un particolare id di processo (PID).
2. Aggiungete la vostra chiave privata SSH (quella senza estensione) a ssh-agent.
	```bash
	ssh-add ~/.ssh/kinsta_keys
	```
	Sostituite **kinsta\_keys** con il nome della vostra chiave SSH. Se questa è la prima chiave che avete creato, dovrebbe essere chiamata “id\_algorithm\_used”, per esempio, **id\_ed25519**.

### Aggiungere la Chiave SSH all’Account GitHub

Il passo finale è aggiungere la vostra chiave pubblica al vostro account GitHub. Seguite queste istruzioni:

1. Copiate la vostra chiave pubblica SSH nei vostri appunti. Potete aprire il file in cui si trova con un editor di testo e copiarlo, o usare il terminale per mostrare il suo contenuto.
	```bash
	cat ~/.ssh/kinsta_keys.pub
	# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJl3dIeudNqd0DPMRD6OIh65tjkxFNOtwGcWB2gCgPhk kinstauser@kinsta.com
	```
2. [Accedete a GitHub](https://github.com/login) e andate nella sezione in alto a destra della pagina, fate clic sulla foto del vostro profilo e selezionate ****Settings****.![Pannello in alto a destra di GitHub che mostra diverse sezioni con una freccia che punta alla sezione Impostazioni.](https://kinsta.com/wp-content/uploads/2022/01/GitHub-settings.png)
	Impostazioni GitHub.
3. Poi, nelle impostazioni del profilo, fate clic su **SSH and GPG keys**.![Pannello delle impostazioni del profilo che mostra l'opzione chiavi SSH e GPG.](https://kinsta.com/wp-content/uploads/2022/01/GitHub-ssh-gpg-keys.png)
	Chiavi SSH e GPG.
4. Fate clic sul pulsante **New SSH key**.![Sezione chiavi SSH con una freccia che punta al pulsante New SSH key.](https://kinsta.com/wp-content/uploads/2022/01/GitHub-new-ssh-key-1024x340.png)
	Pulsante New SSH key.
5. Date un **titolo** alla vostra nuova chiave SSH su GitHub: di solito, il dispositivo da cui userete quella chiave. E poi incollate la chiave nell’area **Key**.![Aggiungere un nuovo modulo di chiave SSH con i campi ](https://kinsta.com/wp-content/uploads/2022/01/title-key-field-1024x587.png)
	Aggiungere una nuova forma di chiave SSH.
6. Aggiungete la chiave SSH.![Pulsante Add SSH key.](https://kinsta.com/wp-content/uploads/2022/01/add-ssh-key-button.png)
	Pulsante per aggiungere la chiave SSH.

### Provare la Connessione SSH con un Repo Push

È il momento di testare tutto quello che avete fatto finora. Farete modifiche, commit e push a uno dei tuoi repo esistenti usando SSH per assicurarvi che la vostra connessione sia impostata correttamente.

Per il nostro esempio, modificheremo il semplice sito HTML che abbiamo creato nel nostro [tutorial Git per lo sviluppo web](https://kinsta.com/it/blog/git-per-lo-sviluppo-web/).

Per prima cosa, avremo bisogno di clonare il repository nella nostra macchina locale. Possiamo andare alla pagina del repo su GitHub e copiare l’indirizzo SSH che fornisce.

![Pagina GitHub che mostra il comando clone SSH.](https://kinsta.com/wp-content/uploads/2022/01/clone-ssh-1024x498.png)

Comando clone SSH.

Poi clonate il repo usando un terminale:

```bash
git clone git@github.com:DaniDiazTech/HTML-site.git
```

Ora, aggiungiamo un semplice tag `<h1>` nel file **index.html**:

```html
...
<div class="container my-2">
    <h1 class="text-center">A new title!<h1>
</div>

<div class="container my-3">
...
```
![Semplice sito HTML con il titolo ](https://kinsta.com/wp-content/uploads/2022/01/new-title-1024x351.png)

Un semplice sito HTML.

Non toccheremo né JavaScript né CSS per mantenere questa modifica semplice. Ma se sapete usare JavaScript, potreste trovare un posto nel team di Kinsta. Date un’occhiata alle [abilità di programmazione che richiediamo per far parte del team di Kinsta](https://kinsta.com/developer-roles/coding-skills-at-kinsta/).

Dopo aver fatto questo, fate un commit per queste modifiche:

```bash
git commit -am "Added a simple title"
```

E fate un push su GitHub come fareste normalmente.

```bash
git push
```

Se tutto è andato bene, congratulazioni! Avete appena impostato una connessione SSH tra la vostra macchina e GitHub.

### Gestire Più Chiavi SSH per Diversi Account GitHub

Se avete più account GitHub, diciamo uno per i vostri progetti personali e uno per lavoro, è difficile usare SSH per entrambi. Di solito avreste bisogno di macchine separate per autenticarvi sui diversi account GitHub.

Ma questo può essere risolto facilmente configurando il file di configurazione di SSH.

Entriamo nel merito.

1. Create un’altra coppia di chiavi SSH e aggiungetela al vostro altro account GitHub. Tenete a mente il nome del file a cui state assegnando la nuova chiave.
	```bash
	ssh-keygen -t ed25519 -C "work@email.com"
	```
2. Create il file di configurazione SSH. Il file di configurazione dice al programma ssh come dovrebbe comportarsi. Per impostazione predefinita, il file di configurazione potrebbe non esistere, quindi createlo all’interno della cartella.ssh/:
	```bash
	touch ~/.ssh/config
	```
3. Modificate il file di configurazione di SSH. Aprite il file di configurazione e incollate il codice qui sotto:
	```bash
	#Your day-to-day GitHub account
	Host github.com
	  HostName github.com
	  IdentityFile ~/.ssh/id_ed25519
	  IdentitiesOnly yes
	# Work account
	Host github-work
	  HostName github.com
	  IdentityFile ~/.ssh/work_key_file
	  IdentitiesOnly yes
	```

Ora, ogni volta che avete bisogno di autenticarti via SSH usando il vostro account di lavoro o secondario, modificate un po’ l’indirizzo SSH della repo. Da:

```bash
git@github.com:workaccount/project.git
```

… arrivate a:

```bash
git@github-work:workaccount/project.git
```

## Riepilogo

Congratulazioni – avete imparato la maggior parte delle conoscenze pratiche necessarie per connettervi a GitHub via SSH!

In questo tutorial abbiamo discusso la necessità del protocollo SSH, le differenze tra chiavi pubbliche e private, come generare le chiavi, aggiungerle a GitHub e anche gestire più chiavi SSH per diversi account GitHub. Tenete presente che, a meno che non vogliate perdere l’accesso a tutto, la vostra chiave privata deve rimanere tale: privata.

Con queste conoscenze, ora potete sviluppare un [flusso di lavoro impeccabile con Git e GitHub](https://kinsta.com/it/blog/git-per-lo-sviluppo-web/). Continuate a programmare!
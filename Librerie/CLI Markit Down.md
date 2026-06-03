La CLI (Command Line Interface) di MarkItDown di Microsoft è volutamente minimale. Essendo uno strumento nato per fare una sola cosa e farla bene, non ti bombarda con decine di opzioni astruse.

La sintassi base del comando è questa:

Bash

```
markitdown <percorso_del_file> [opzioni]
```

Ecco una panoramica completa dei comandi, dei flag disponibili e di come usarli con esempi pratici sul campo.

## I Comandi e i Flag Principali

Dal terminale, i flag che userai nell'80% dei casi sono principalmente due:

|**Flag / Sintassi**|**Descrizione**|
|---|---|
|`<file>`|Il file che vuoi convertire (obbligatorio).|
|`-o`, `--output`|Specifica il percorso del file `.md` in cui salvare il risultato.|
|`>` (Reindirizzamento)|Alternativa classica dei terminali per salvare l'output in un file.|
|`--version`|Mostra la versione attuale di MarkItDown installata.|
|`--help`|Mostra la guida rapida ai comandi.|

## Esempi Pratici di Utilizzo

Vediamo come si comporta nei vari scenari reali quando sei dentro il tuo terminale (con l'ambiente virtuale attivato).

### 1. Visualizzazione rapida (Anteprima)

Se vuoi solo dare un'occhiata veloce a come verrebbe convertito un documento senza salvarlo, inserisci solo il nome del file. Il testo formattato apparirà direttamente nel terminale.

Bash

```
markitdown Relazione_Trimestrale.docx
```

### 2. Convertire e salvare un PDF (Il comando classico)

Per salvare il risultato in un file Markdown pulito pronto per essere letto, usa il flag `-o`:

Bash

```
markitdown Manuale_Utente.pdf -o Manuale_Utente.md
```

### 3. Estrarre tabelle da Excel

Se passi un file Excel, MarkItDown fa una magia: converte i fogli di calcolo in tabelle Markdown con i separatori `|`. Puoi usare la tecnica del reindirizzamento (`>`), che funziona esattamente come il flag `-o`:

Bash

```
markitdown Budget_2026.xlsx > tabelle_budget.md
```

### 4. Gestione dei file ZIP (Batch automatico)

Se hai una cartella compressa piena di documenti misti (un PDF, un file Word, un HTML), non devi spacchettarli uno a uno. Passa direttamente lo ZIP a MarkItDown:

Bash

```
markitdown archivio_documenti.zip -o riassunto_contenuti.md
```

> **Cosa succede dietro le quinte?** MarkItDown aprirà lo ZIP, convertirà ogni file supportato che trova all'interno e unirà tutto il testo in un unico, grande file Markdown strutturato.

### 5. Ripulire pagine Web (HTML) direttamente da terminale

Se hai salvato una pagina web sul computer in formato `.html` e vuoi eliminare i tag `<div>`, i fogli di stile e i JavaScript per tenere solo il testo formattato:

Bash

```
markitdown articolo_blog.html -o articolo_pulito.md
```

## Piccolo "Trucco" per l'utilizzo avanzato

Se ti trovi a dover automatizzare molti file su Linux o Mac, puoi combinare MarkItDown con i comandi nativi del terminale (come `find` o `xargs`) per convertire un'intera cartella in un colpo solo.

Ad esempio, per convertire **tutti i file .docx** di una cartella:

Bash

```
for file in *.docx; do markitdown "$file" -o "${file%.docx}.md"; done
```

Se provi a lanciare uno di questi comandi sui tuoi file e qualcosa non ti torna nel formato finale, dimmi pure che tipo di documento stai convertendo!
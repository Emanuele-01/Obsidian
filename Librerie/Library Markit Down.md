La libreria Python di `markitdown` è progettata per essere estremamente snella e intuitiva. Non ci sono decine di metodi diversi; il cuore della libreria si basa su **un unico metodo principale** (`convert`), affiancato da un paio di utility e dalla gestione delle configurazioni avanzate (come l'integrazione con gli LLM).

Ecco la panoramica completa dei metodi della classe principale `MarkItDown` e di come utilizzarli con esempi di codice.

## 1. Il metodo fondamentale: `convert()`

Questo è il metodo principale che userai nel 95% dei casi. Prende in input il percorso di un file locale (o un oggetto file-like) e restituisce un oggetto risultato che contiene il testo in Markdown.

### Sintassi:

Python

```
convert(source, bytes=None, file_ext=None, **kwargs)
```

- **`source`**: Il percorso del file (stringa) o un file aperto.
    
- **`bytes`**: (Opzionale) I dati del file in formato binario, se non stai leggendo da un percorso locale.
    
- **`file_ext`**: (Opzionale) L'estensione del file (es. `".pdf"`), fondamentale se usi il parametro `bytes` per far capire a MarkItDown come interpretare i dati.
    

### Esempio 1: Conversione standard di un file locale

Python

```
from markitdown import MarkItDown

md = MarkItDown()

# Converte un file Word (.docx)
result = md.convert("piano_marketing.docx")

# Il risultato è un oggetto. Per accedere al testo Markdown usi l'attributo .text_content
print(result.text_content)
```

### Esempio 2: Conversione da stream di byte (Memory Stream)

Utile se stai creando un'applicazione web (ad esempio con FastAPI o Flask) e l'utente carica un file che non vuoi salvare sul disco.

Python

```
from markitdown import MarkItDown

md = MarkItDown()

# Simuliamo di avere i byte di un file PDF in memoria
with open("documento.pdf", "rb") as f:
    file_bytes = f.read()

# Convertiamo passando i byte e specificando l'estensione originale
result = md.convert(source=None, bytes=file_bytes, file_ext=".pdf")

print(result.text_content)
```

## 2. Metodi di supporto per estensioni specifiche

Internamente, la classe `MarkItDown` mappa le estensioni dei file a dei metodi di conversione specifici. Sebbene sia sempre consigliato usare il metodo generico `convert()`, la libreria espone (o permette di registrare) convertitori dedicati.

I metodi interni standard includono la gestione di:

- `convert_pdf`
    
- `convert_docx` / `convert_pptx` / `convert_xlsx`
    
- `convert_html`
    
- `convert_images` / `convert_audio`
    

Nella pratica, **non è necessario chiamarli direttamente**, poiché `convert()` capisce da solo l'estensione e reindirizza il file al metodo corretto.

## 3. Configurazione avanzata tramite il Costruttore (`__init__`)

Più che una lista di metodi diversi, la potenza di MarkItDown in Python si esprime tramite i parametri che puoi passare quando crei l'istanza della classe `MarkItDown()`.

Ecco i parametri principali accettati dal costruttore:

Python

```
MarkItDown(llm_client=None, llm_model=None, adapter=None)
```

### Esempio 3: Utilizzo con un modello di Intelligenza Artificiale (OpenAI)

Se passi un client LLM, MarkItDown sbloccherà automaticamente le funzionalità di descrizione delle immagini e trascrizione/analisi dell'audio durante la conversione.

Python

```
from markitdown import MarkItDown
from openai import OpenAI

# Inizializza il client OpenAI ufficiale
openai_client = OpenAI(api_key="la_tua_api_key")

# Passa il client e il modello a MarkItDown
md = MarkItDown(llm_client=openai_client, llm_model="gpt-4o")

# Ora, se converti un PDF che contiene immagini, o un'immagine singola,
# l'AI genererà una descrizione dettagliata integrata nel Markdown.
result = md.convert("ricevuta_fiscale.jpg")
print(result.text_content)
```

### Esempio 4: Gestione degli errori durante la conversione

È sempre buona norma inserire il metodo `convert` all'interno di un blocco `try-except`, in quanto la conversione potrebbe fallire se il file è corrotto o se il formato non è supportato.

Python

```
from markitdown import MarkItDown

md = MarkItDown()

try:
    result = md.convert("file_sconosciuto.xyz")
    print(result.text_content)
except UnsupportedFormatException:
    print("Errore: Questo formato di file non è supportato da MarkItDown.")
except Exception as e:
    print(f"Si è verificato un errore imprevisto: {e}")
```

## Ricapitolando: Cosa restituisce il metodo `convert()`?

Il metodo non restituisce una semplice stringa, ma un oggetto di tipo `DocumentConverterResult`. Gli attributi principali che puoi estrarre da questo oggetto sono:

- `result.text_content`: Stringa contenente tutto il testo convertito in puro Markdown.
    
- `result.title`: (Se disponibile) Il titolo del documento estratto dai metadati.
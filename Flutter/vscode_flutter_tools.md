# 🔌 Strumenti dell'Estensione VS Code per Flutter

[← Flutter CLI & FVM](./flutter_cli_fvm.md) | [Torna all'Hub](./index.md) | [Continua con Scorciatoie Widget Snippets →](./vscode_widget_snippets.md)

---

L'estensione ufficiale **Flutter** per Visual Studio Code (sviluppata in collaborazione con il team di Dart) trasforma l'editor in un vero e proprio IDE integrato. Di seguito analizziamo tutti gli strumenti integrati e come sfruttarli al massimo.

---

## 1. Comandi Rapidi (Command Palette)

Premendo `Ctrl + Shift + P` (Windows/Linux) o `Cmd + Shift + P` (macOS), puoi accedere alla barra dei comandi di VS Code. Cerca i seguenti comandi digitando `Flutter` o `Dart`:

*   **`Flutter: New Project`**: Crea un nuovo progetto guidato chiedendo la cartella e il nome.
*   **`Flutter: Select Device`**: Permette di selezionare o avviare un emulatore/dispositivo fisico direttamente dalla barra inferiore di VS Code.
*   **`Flutter: Run Flutter Doctor`**: Esegue la diagnostica senza aprire il terminale esterno.
*   **`Flutter: Launch Emulator`**: Avvia velocemente un emulatore configurato.
*   **`Dart: Add Dependency`**: Cerca un pacchetto su *pub.dev* e lo aggiunge a `pubspec.yaml` inserendo automaticamente l'ultima versione stabile.
*   **`Flutter: Open DevTools`**: Apre la suite di debugging esterna (vedi sotto).

---

## 2. Refactoring Rapido con il Quick Fix (`Ctrl + .` / `Cmd + .`)

Uno degli strumenti più potenti in assoluto. Posizionando il cursore su un widget e premendo la combinazione di tasti **`Ctrl + .`** (o cliccando sull'icona della lampadina gialla), si apre il menu contestuale di refactoring:

*   **Wrap with Widget / Wrap with Column / Row / Padding / Center / Container**: Avvolge il widget corrente all'interno di un altro widget di layout senza dover spostare le parentesi a mano.
*   **Remove this Widget**: Elimina il widget corrente e collega il suo figlio direttamente al widget genitore.
*   **Convert to StatefulWidget / StatelessWidget**: Converte la classe del widget e riscrive automaticamente lo stato o il metodo `build()`.
*   **Extract Widget**: Estrae la porzione di codice selezionata creando una nuova classe `StatelessWidget` separata (fondamentale per le performance, vedi [Ottimizzazione delle Prestazioni](./sviluppo_ios_android.md#5-ottimizzazione-delle-prestazioni)).
*   **Move Widget Up / Down**: Sposta l'ordine di dichiarazione di un widget all'interno di un array di figli (`children`).

---

## 3. Flutter DevTools & Widget Inspector

Durante l'esecuzione dell'app in modalità debug (`F5`), l'estensione attiva la barra degli strumenti di debug e consente di lanciare **Flutter DevTools** (una suite di diagnostica web).

I controlli principali disponibili sono:

### Flutter Widget Inspector
Consente di ispezionare visivamente la struttura ad albero dei widget dell'applicazione.
*   **Select Widget Mode**: Cliccando su questo pulsante e poi su un elemento dello schermo del simulatore, l'IDE evidenzierà la riga di codice esatta che ha generato quell'elemento.
*   **Debug Paint**: Disegna dei bordi e delle linee guida colorate attorno a tutti i widget a schermo. È utilissimo per capire perché un elemento non si allinea o per trovare le cause degli errori di *layout overflow* (pixel che escono dallo schermo).
*   **Repaint Rainbow**: Assegna bordi di colori diversi a tutti i widget che vengono ridisegnati a schermo. Se vedi continui cambi di colore su un widget che dovrebbe essere statico, significa che stai sprecando cicli di calcolo ricostruendolo inutilmente.
*   **Slow Animations**: Rallenta tutte le transizioni e le animazioni (es. transizioni [Hero](./schermata_dettaglio.md#2-spiegazione-dei-widget-utilizzati)) per valutarne la fluidità visiva.

### Performance & Memory (DevTools)
*   **Performance View**: Mostra un grafico in tempo reale dei frame al secondo (FPS). Se un frame supera i 16ms (per schermi a 60Hz), viene segnalato in rosso (jank).
*   **Memory View**: Permette di monitorare l'allocazione di memoria RAM e individuare eventuali leak (es. dovuti a `TextEditingController` non chiusi correttamente in `dispose`).

---

[← Flutter CLI & FVM](./flutter_cli_fvm.md) | [Torna all'Hub](./index.md) | [Continua con Scorciatoie Widget Snippets →](./vscode_widget_snippets.md)

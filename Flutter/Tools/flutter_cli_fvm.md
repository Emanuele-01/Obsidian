# 🛠️ Flutter CLI & FVM (Flutter Version Manager)

[← Torna all'Hub](../index.md) | [Continua con gli Strumenti VS Code →](./vscode_flutter_tools.md)

---

Per sviluppare in modo efficiente con Flutter, è fondamentale saper utilizzare l'interfaccia a riga di comando (**Flutter CLI**) e saper gestire le diverse versioni dell'SDK tramite **FVM**.

---

## 1. Guida ai Comandi della Flutter CLI

La CLI di Flutter fornisce tutti gli strumenti necessari per creare, diagnosticare, eseguire e compilare le applicazioni.

### Diagnostica del Sistema
*   **`flutter doctor`**: Scansiona il sistema locale alla ricerca di dipendenze mancanti (es. Android Studio, Xcode, CocoaPods) e fornisce un report dettagliato per risolvere i problemi di configurazione.
*   **`flutter doctor -v`**: Versione logorroica (verbose) per visualizzare i percorsi esatti e i dettagli di errore.

### Gestione del Progetto
*   **`flutter create <nome_progetto>`**: Genera un nuovo progetto Flutter nella cartella specificata con una codebase preconfigurata.
*   **`flutter devices`**: Elenca tutti i dispositivi fisici, simulatori o browser web attualmente connessi e disponibili per l'esecuzione.
*   **`flutter emulators`**: Mostra la lista degli emulatori configurati sul PC e permette di avviarli con `--launch <id_emulatore>`.

### Esecuzione e Debugging
*   **`flutter run`**: Avvia l'applicazione sul dispositivo attivo o apre un menu di scelta se sono presenti più dispositivi.
    *   `flutter run -d <id_dispositivo>`: Forza l'avvio su un dispositivo specifico.
    *   `flutter run --release`: Compila ed esegue in modalità ottimizzata per il rilascio (disabilita Hot Reload e debugger).
    *   `flutter run --profile`: Avvia in modalità profilo, utile per misurare le performance dell'app.
*   *Comandi interattivi in console durante l'esecuzione:*
    *   `r`: Esegue l'**Hot Reload** (aggiorna la UI mantenendo lo stato dell'app).
    *   `R`: Esegue l'**Hot Restart** (riavvia l'app da zero ripristinando lo stato iniziale).
    *   `q`: Chiude l'app e termina il processo di debug.

### Gestione delle Dipendenze
*   **`flutter pub get`**: Scarica e installa i pacchetti e le librerie definiti nel file `pubspec.yaml`.
*   **`flutter pub upgrade`**: Aggiorna i pacchetti esistenti alle ultime versioni compatibili.
*   **`flutter pub add <nome_pacchetto>`**: Aggiunge automaticamente un pacchetto al file `pubspec.yaml` e lo scarica.

### Manutenzione e Build
*   **`flutter clean`**: Cancella la cartella `build/` e i file temporanei dell'SDK. È il primo comando da eseguire in caso di errori di compilazione apparentemente inspiegabili.
*   **`flutter build apk`**: Compila il pacchetto d'installazione per Android in formato APK.
*   **`flutter build appbundle`**: Compila il pacchetto Android in formato AAB, richiesto da Google Play.
*   **`flutter build ipa` / `flutter build ios`**: Compila i pacchetti di distribuzione e archivio nativi per iOS (richiede macOS).

---

## 2. FVM (Flutter Version Manager)

### Perché usare FVM?
Spesso ci si ritrova a lavorare su più progetti contemporaneamente: un progetto legacy potrebbe richiedere una versione precedente di Flutter (es. `3.10.x`), mentre un nuovo progetto potrebbe richiedere l'ultima versione stabile (`3.22.x`). 
FVM consente di installare più versioni di Flutter sul proprio PC e di associare ad ogni progetto la propria versione specifica senza conflitti globali.

### Installazione Veloce
Puoi installare FVM globalmente tramite la CLI di Dart:
```bash
dart pub global activate fvm
```

### Comandi Principali di FVM
*   **`fvm install <versione>`**: Scarica e installa una versione specifica dell'SDK (es. `fvm install 3.19.0` o `fvm install stable`).
*   **`fvm use <versione>`**: Configura il progetto corrente (nella cartella in cui ci si trova) per utilizzare la versione selezionata dell'SDK. Genera una cartella locale `.fvm/`.
*   **`fvm list`**: Mostra la lista di tutte le versioni di Flutter installate localmente sul PC.
*   **`fvm releases`**: Mostra tutte le release storiche di Flutter disponibili per il download.
*   **`fvm flutter <comando>`**: Esegue un comando Flutter standard utilizzando l'SDK specifico del progetto (es. `fvm flutter run`).

### Configurazione di VS Code per FVM
Per far sì che l'estensione di VS Code utilizzi la versione locale di FVM anziché quella globale del sistema, crea (o modifica) il file `.vscode/settings.json` alla radice del progetto:

```json
{
  "dart.flutterSdkPath": ".fvm/flutter_sdk",
  // Impedisce a VS Code di cercare l'SDK globale
  "search.exclude": {
    "**/.fvm": true
  },
  "watcher.exclude": {
    "**/.fvm": true
  }
}
```

---

[← Torna all'Hub](../index.md) | [Continua con gli Strumenti VS Code →](./vscode_flutter_tools.md)

# Hub di Sviluppo Flutter & Dart

Benvenuto nella wiki dedicata allo sviluppo di applicazioni cross-platform per iOS e Android utilizzando **Flutter** e il linguaggio **Dart**. Questa raccolta di note è strutturata per essere interamente navigabile all'interno di Obsidian.

---

## 🗺️ Mappa dei Contenuti

Seleziona un argomento per iniziare l'esplorazione:

```mermaid
graph TD
    Index[Hub Flutter & Dart] --> Dart[1. Fondamenti di Dart]
    Index --> Widgets[2. Guida ai Widget]
    Index --> Techniques[3. Sviluppo iOS & Android]
    Index --> Screens[4. Schermate Pratiche & Logica]
    
    Dart -.->|Sintassi Asincrona| Techniques
    Widgets -.->|Gestione dello Stato| Techniques
    Dart -.->|Costruttori e OOP| Widgets
    
    Screens --> Login[🔐 Login Screen]
    Screens --> Home[🏠 Home Screen]
    Screens --> Detail[🛍️ Detail Screen]
    Screens --> Profile[👤 Profile Screen]
    
    Login -.->|Naviga a| Home
    Home -.->|Visualizza| Detail
    Home -.->|Contiene| Profile
```

### 1. 🎯 [Fondamenti di Dart](./dart_fondamenti.md)
*Impara le basi del linguaggio prima di costruire la tua interfaccia utente.*
*   Variabili, costanti (`final`, `const`) e tipi di dato.
*   **Null Safety** rigorosa e operatori associati.
*   Funzioni (parametri nominati e posizionali).
*   Programmazione Orientata agli Oggetti (OOP: classi, ereditarietà, mixins).
*   **Asincronia** (`Future` e `Stream`).

### 2. 🧱 [Widget Principali in Flutter](./widget_principali.md)
*Il cuore dell'interfaccia utente: scopri come comporre le schermate.*
*   Differenze tra `StatelessWidget` e `StatefulWidget` con relativi cicli di vita.
*   Widget strutturali (`Scaffold`, `AppBar`).
*   Dimensionamento e spaziamento (`Container`, `Padding`, `SizedBox`).
*   Layout complessi (`Row`, `Column`, `Stack`).
*   Liste performanti (`ListView.builder`).
*   Interazioni fisiche (`GestureDetector` e pulsanti).

### 3. 📱 [Tecniche di Sviluppo iOS e Android](./sviluppo_ios_android.md)
*Best practice e architetture per la pubblicazione su store reali.*
*   Pattern di **State Management** (Riverpod, BLoC, Provider).
*   UI Adattiva per iOS/Android e layout responsivi.
*   Integrazione di codice nativo con i **Method Channels**.
*   Ottimizzazioni delle performance per evitare il *jank*.
*   Preparazione e deployment sugli store (App Store e Google Play).

### 4. 💻 [Schermate Esempio con Logica Pratica](./schermata_login.md)
*Codice Flutter completo e commentato per comprendere come unire layout e logica.*
*   🔐 **[Schermata di Login](./schermata_login.md)**: Validazione dei form, controller di testo, stati di caricamento ed eccezioni grafiche.
*   🏠 **[Schermata Home / Dashboard](./schermata_home.md)**: Menu di navigazione inferiore, griglie dinamiche (`GridView.builder`) e logica di filtraggio dei dati.
*   🛍️ **[Schermata di Dettaglio](./schermata_dettaglio.md)**: Transizioni fluide con `Hero` widget, passaggio di dati e calcolo del carrello.
*   👤 **[Schermata Profilo / Impostazioni](./schermata_profile.md)**: Gestione delle preferenze (Switch) e toggle dello stato tra modalità di lettura e modifica dati.

---

## 🚀 Percorso di Apprendimento Consigliato

1.  Comprendi a fondo la sintassi del linguaggio in [Fondamenti di Dart](./dart_fondamenti.md).
2.  Impara a comporre la UI studiando i [Widget Principali](./widget_principali.md).
3.  Vedi la teoria in pratica studiando le **[Schermate Esempio](./schermata_login.md)**.
4.  Approfondisci l'architettura professionale e il rilascio in [Tecniche di Sviluppo](./sviluppo_ios_android.md).

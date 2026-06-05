# Guida a integrare OneSignal SDK in Flutter

OneSignal mette a disposizione un SDK Flutter ufficiale per integrare push notification e funzionalità collegate in applicazioni Flutter per iOS e Android.[cite:58][cite:71] La documentazione ufficiale presenta il setup Flutter come parte del flusso standard di integrazione mobile, con configurazione dedicata per progetto, piattaforme e credenziali necessarie.[cite:58][cite:72]

## Panoramica architetturale

In una configurazione tipica, l'app Flutter integra il package `onesignal_flutter`, mentre il pannello OneSignal gestisce registrazione del dispositivo, invio dei messaggi e segmentazione utenti.[cite:71][cite:79] Su Android, OneSignal richiede la configurazione del canale Firebase/FCM nel dashboard perché la consegna delle notifiche passa da Firebase Cloud Messaging.[cite:58][cite:59]

## Prerequisiti

Prima dell'integrazione servono questi elementi:

- Un progetto Flutter funzionante.[cite:58]
- Un'app configurata su OneSignal con relativo **App ID**.[cite:58][cite:79]
- Per Android, un progetto Firebase con credenziali FCM caricate in OneSignal.[cite:58][cite:59]
- Per iOS, configurazione Apple Push Notification service e setup iOS previsto dalla documentazione OneSignal.[cite:58][cite:72]

## Installazione del package

Il package pubblicato su pub.dev è `onesignal_flutter`.[cite:71] Il setup ufficiale rimanda alla guida di integrazione per le istruzioni complete di piattaforma.[cite:71][cite:58]

Aggiunta rapida da terminale:

```bash
flutter pub add onesignal_flutter
```

In alternativa nel `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  onesignal_flutter: ^latest
```

Dopo l'aggiunta del package, eseguire:

```bash
flutter pub get
```

## Configurazione nel dashboard OneSignal

Il flusso consigliato parte dalla creazione dell'app nel dashboard OneSignal e dalla scelta della piattaforma da configurare.[cite:58][cite:79] Per Android, la documentazione specifica che vanno caricate le credenziali Firebase nelle impostazioni della piattaforma Google Android (FCM).[cite:59]

Per Android, in pratica occorre:

1. Creare o selezionare il progetto Firebase.[cite:59]
2. Recuperare le credenziali richieste da OneSignal per FCM.[cite:59]
3. Aprire il dashboard OneSignal e configurare la piattaforma Android caricando le credenziali Firebase.[cite:59]

## Inizializzazione base in Flutter

Una volta installato il package, OneSignal va inizializzato all'avvio dell'app con il proprio App ID.[cite:58][cite:71] La documentazione del package conferma che l'SDK Flutter è pensato proprio per semplificare l'integrazione tra app Flutter e OneSignal.[cite:71]

Esempio minimo:

```dart
import 'package:flutter/material.dart';
import 'package:onesignal_flutter/onesignal_flutter.dart';

const String oneSignalAppId = 'YOUR-ONESIGNAL-APP-ID';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  OneSignal.initialize(oneSignalAppId);

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: Text('OneSignal inizializzato')),
    );
  }
}
```

## Gestione dei permessi

La richiesta del permesso notifiche deve essere gestita in modo esplicito, soprattutto su iOS e sulle versioni Android più recenti in cui l'autorizzazione è runtime.[cite:58][cite:72] Una buona pratica è non chiedere il permesso appena al primo frame, ma dopo aver spiegato all'utente il valore delle notifiche con una schermata o un prompt contestuale.[cite:58]

Esempio di richiesta permesso:

```dart
Future<void> requestNotificationPermission() async {
  final accepted = await OneSignal.Notifications.requestPermission(true);
  debugPrint('Permesso notifiche: $accepted');
}
```

Best practice consigliate:

- Chiedere il permesso dopo una breve spiegazione UX, non immediatamente all'avvio.[cite:58]
- Fare la richiesta in un punto del funnel in cui l'utente capisce il beneficio, per esempio dopo login o attivazione di una funzione rilevante.[cite:58]
- Evitare richieste ripetute o aggressive se il permesso è già stato negato.[cite:58][cite:72]

## Listener per click e foreground

OneSignal mette a disposizione listener per intercettare eventi come click sulla notifica e visualizzazione in foreground.[cite:79] In Flutter è utile registrarli subito dopo l'inizializzazione, così l'app può reagire correttamente quando una notifica viene aperta o ricevuta mentre l'app è attiva.[cite:75][cite:78]

Esempio:

```dart
void setupOneSignalListeners() {
  OneSignal.Notifications.addClickListener((event) {
    final data = event.notification.additionalData;
    debugPrint('Notifica cliccata: $data');
  });

  OneSignal.Notifications.addForegroundWillDisplayListener((event) {
    debugPrint('Notifica ricevuta in foreground');
    event.notification.display();
  });
}
```

Uso nell'avvio:

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  OneSignal.initialize(oneSignalAppId);
  setupOneSignalListeners();

  runApp(const MyApp());
}
```

## Navigazione da notifica

Una pratica molto comune è usare `additionalData` nel payload per decidere quale schermata aprire quando l'utente tocca la notifica.[cite:75][cite:78] Questo approccio rende le notifiche più utili perché possono portare direttamente a una chat, un ordine, un contenuto o una sezione specifica dell'app.[cite:79]

Esempio semplice:

```dart
final navigatorKey = GlobalKey<NavigatorState>();

void setupOneSignalListeners() {
  OneSignal.Notifications.addClickListener((event) {
    final data = event.notification.additionalData;
    final screen = data?['screen'];
    final id = data?['id'];

    if (screen == 'order_details' && id != null) {
      navigatorKey.currentState?.push(
        MaterialPageRoute(
          builder: (_) => OrderDetailsPage(orderId: id.toString()),
        ),
      );
    }
  });
}
```

Nel `MaterialApp`:

```dart
MaterialApp(
  navigatorKey: navigatorKey,
  home: const HomePage(),
)
```

## Gestione utente, alias e identificazione

Le versioni recenti dell'SDK OneSignal si basano su un modello utente centrico, come indicato nella migration guide ufficiale.[cite:76][cite:80] Per questo motivo è importante collegare il dispositivo all'identità applicativa dell'utente solo dopo autenticazione o quando l'identità è sufficientemente affidabile.[cite:76]

Best practice:

- Associare l'utente dopo login, non prima.[cite:76]
- Tenere separati i dati anonimi iniziali dai dati dell'utente autenticato.[cite:76]
- In caso di logout, rimuovere o aggiornare l'associazione in modo coerente con il modello utente dell'app.[cite:76]

Esempio concettuale:

```dart
Future<void> onUserLoggedIn(String userId) async {
  await OneSignal.login(userId);
}

Future<void> onUserLoggedOut() async {
  await OneSignal.logout();
}
```

## Uso dei tag

OneSignal documenta l'uso dei **tag** per associare metadati all'utente, come piano, lingua, ruolo o preferenze.[cite:83] I tag sono molto utili per segmentare gli invii, ma funzionano bene solo se vengono mantenuti puliti, coerenti e semanticamente stabili nel tempo.[cite:83]

Esempio:

```dart
Future<void> setUserTags() async {
  await OneSignal.User.addTagWithKey('plan', 'pro');
  await OneSignal.User.addTagWithKey('language', 'it');
  await OneSignal.User.addTagWithKey('role', 'customer');
}
```

Best practice per i tag:

- Usare chiavi brevi e stabili, per esempio `plan`, `lang`, `role`.[cite:83]
- Evitare dati sensibili nei tag.[cite:83]
- Non duplicare informazioni che appartengono già al backend se non servono alla segmentazione push.[cite:83]
- Aggiornare i tag quando cambia il profilo utente, per evitare segmentazioni errate.[cite:83]

## Privacy e consenso

Se l'app opera in contesti con requisiti privacy più stringenti, OneSignal supporta flussi di consenso esplicito per la privacy prima dell'uso pieno dell'SDK.[cite:77] Le issue e la documentazione mostrano che l'ordine corretto tra inizializzazione, consenso e richiesta permessi è importante per evitare comportamenti inattesi.[cite:74][cite:77]

Esempio concettuale di flusso con consenso:

```dart
Future<void> setupOneSignalWithConsent() async {
  OneSignal.consentRequired(true);
  OneSignal.initialize(oneSignalAppId);

  // Dopo che l'utente ha accettato l'informativa privacy:
  OneSignal.consentGiven(true);

  await OneSignal.Notifications.requestPermission(true);
}
```

Best practice privacy:

- Separare consenso privacy da consenso notifiche, perché non sono sempre la stessa cosa.[cite:77]
- Non tentare di leggere identificativi push prima che l'SDK sia inizializzato e che il consenso richiesto sia stato concesso.[cite:77]
- Testare con particolare attenzione i flussi iOS se si usa il gating del consenso.[cite:77]

## Errori comuni da evitare

Ci sono alcuni errori ricorrenti nelle integrazioni Flutter con OneSignal:

- Inizializzare l'SDK troppo tardi o in punti non deterministici dell'app lifecycle, causando listener non registrati correttamente.[cite:75][cite:78]
- Chiedere il permesso notifiche senza spiegare il valore all'utente, ottenendo tassi di opt-in più bassi.[cite:58]
- Mescolare OneSignal e gestione FCM diretta senza un disegno chiaro dell'architettura notifiche.[cite:58][cite:59]
- Gestire male click e deep link, perdendo il contesto della notifica aperta.[cite:75][cite:78]
- Usare tag disordinati, inconsistenti o con semantica variabile nel tempo.[cite:83]

## Sequenza consigliata di integrazione

Una sequenza ordinata per integrare bene OneSignal in Flutter è questa:

1. Creare il progetto/app nel dashboard OneSignal e recuperare l'App ID.[cite:58][cite:79]
2. Configurare Android con Firebase/FCM nel dashboard OneSignal.[cite:59]
3. Installare `onesignal_flutter` nel progetto Flutter.[cite:71]
4. Inizializzare OneSignal all'avvio dell'app.[cite:58][cite:71]
5. Registrare i listener di click e foreground subito dopo l'inizializzazione.[cite:75][cite:78]
6. Chiedere il permesso notifiche in un punto UX corretto.[cite:58]
7. Associare utente, alias e tag solo quando servono davvero e con dati puliti.[cite:76][cite:83]
8. Testare su dispositivo reale per Android e iOS, inclusi tap, foreground e apertura schermate.[cite:58][cite:72]

## Mini esempio completo

Questo esempio unisce inizializzazione, listener, permesso e login utente in una forma semplice da adattare a un progetto reale.[cite:58][cite:71][cite:76]

```dart
import 'package:flutter/material.dart';
import 'package:onesignal_flutter/onesignal_flutter.dart';

const String oneSignalAppId = 'YOUR-ONESIGNAL-APP-ID';
final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  OneSignal.initialize(oneSignalAppId);

  OneSignal.Notifications.addClickListener((event) {
    final data = event.notification.additionalData;
    final screen = data?['screen'];

    if (screen == 'promo') {
      navigatorKey.currentState?.push(
        MaterialPageRoute(builder: (_) => const PromoPage()),
      );
    }
  });

  OneSignal.Notifications.addForegroundWillDisplayListener((event) {
    event.notification.display();
  });

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      navigatorKey: navigatorKey,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  Future<void> enableNotifications() async {
    await OneSignal.Notifications.requestPermission(true);
  }

  Future<void> attachUser() async {
    await OneSignal.login('user-123');
    await OneSignal.User.addTagWithKey('plan', 'premium');
    await OneSignal.User.addTagWithKey('language', 'it');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('OneSignal Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: enableNotifications,
              child: const Text('Abilita notifiche'),
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: attachUser,
              child: const Text('Collega utente'),
            ),
          ],
        ),
      ),
    );
  }
}

class PromoPage extends StatelessWidget {
  const PromoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: Text('Pagina promo')),
    );
  }
}
```

## Best practice finali

- Centralizzare l'inizializzazione di OneSignal in un service dedicato, evitando setup sparsi in widget diversi.[cite:58][cite:75]
- Usare una strategia chiara per il routing da notifica, basata su `additionalData` e non su stringhe hardcoded in molti punti dell'app.[cite:75][cite:78]
- Richiedere i permessi solo quando l'utente ha contesto sufficiente per capire il beneficio.[cite:58]
- Allineare login/logout dell'app con login/logout lato OneSignal, così segmentazione e targeting restano coerenti.[cite:76]
- Trattare i tag come metadati di targeting, non come database applicativo.[cite:83]
- Testare sempre i casi di app in foreground, background e cold start su device reali.[cite:58][cite:72]

## In sintesi operativa

L'integrazione di OneSignal in Flutter è relativamente lineare se viene impostata bene fin dall'inizio: setup piattaforme, inizializzazione pulita, listener registrati subito, richiesta permessi nel momento giusto e uso disciplinato di utente e tag.[cite:58][cite:71][cite:83] Le migliori integrazioni non si limitano a ricevere notifiche, ma costruiscono anche una buona UX di opt-in, una navigazione affidabile da notifica e una segmentazione ordinata lato prodotto.[cite:58][cite:76][cite:83]

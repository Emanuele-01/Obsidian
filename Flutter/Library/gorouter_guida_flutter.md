# Guida a GoRouter in Flutter

GoRouter è un package per Flutter che offre un sistema di routing dichiarativo basato sulla Router API, con un approccio centrato sugli URL e pensato per semplificare la navigazione tra schermate diverse.[# Navigation and routing] La documentazione Flutter indica che `Navigator` e `Router` possono convivere, e suggerisce l'uso di un package come GoRouter quando servono esigenze più avanzate, come deep link o configurazioni di navigazione più articolate.[cite:39]

## Quando usarlo

Per app semplici, Flutter può già gestire la navigazione con `Navigator`, `push`, `pop` e rotte nominate.[cite:39] Quando però il progetto cresce, GoRouter diventa utile perché permette di definire rotte, parametri URL, redirect e percorsi annidati in modo più ordinato rispetto alla navigazione imperativa classica.[cite:25][cite:30][cite:33]

## Installazione

Il package è pubblicato su pub.dev come `go_router`.[cite:25] Il modo più rapido per aggiungerlo al progetto è questo:

```bash
flutter pub add go_router
```

L'installazione tramite `flutter pub add go_router` è mostrata anche in esempi pratici dedicati a Flutter e GoRouter.[cite:30]

In alternativa, si può aggiungere manualmente nel file `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^latest
```

Dopo la modifica del file, basta eseguire:

```bash
flutter pub get
```

## Configurazione base

Per usare GoRouter, l'app deve usare `MaterialApp.router` invece di `MaterialApp`, passando una configurazione di routing.[cite:39] Il package si presenta infatti come un routing package dichiarativo che usa la Router API per fornire una navigazione più comoda e basata sugli URL.[cite:25]

Esempio minimo:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MyApp());
}

final GoRouter router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/details',
      builder: (context, state) => const DetailsPage(),
    ),
  ],
);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router,
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.go('/details'),
          child: const Text('Apri dettagli'),
        ),
      ),
    );
  }
}

class DetailsPage extends StatelessWidget {
  const DetailsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dettagli')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.pop(),
          child: const Text('Indietro'),
        ),
      ),
    );
  }
}
```

## `go()` e `push()`

GoRouter espone metodi come `context.go()` e `context.push()` per navigare tra le schermate.[cite:39][cite:36] In generale, `go()` cambia la location corrente e riallinea lo stack alla nuova rotta, mentre `push()` aggiunge una nuova schermata sopra quella corrente mantenendo la possibilità di tornare indietro con il back button.[cite:33][cite:36]

Esempio pratico:

```dart
ElevatedButton(
  onPressed: () => context.go('/profilo'),
  child: const Text('Vai al profilo'),
)

ElevatedButton(
  onPressed: () => context.push('/profilo'),
  child: const Text('Apri profilo sopra la pagina corrente'),
)
```

Regola pratica:

- Usare `go()` per cambiare sezione dell'app, ad esempio Home, Profilo o Impostazioni.[cite:33]
- Usare `push()` per schermate temporanee o di dettaglio, ad esempio il dettaglio di un prodotto aperto sopra una lista.[cite:33][cite:36]

## Parametri nel percorso

Uno dei vantaggi principali di GoRouter è la possibilità di definire parametri nel path, ad esempio `/user/:id` oppure `/prodotto/:slug`.[cite:33] Gli esempi pratici mostrano che questi valori possono essere letti da `state.pathParameters` nel `builder` della rotta.[cite:30]

Esempio:

```dart
final GoRouter router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'user/:id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return UserPage(userId: id);
          },
        ),
      ],
    ),
  ],
);

class UserPage extends StatelessWidget {
  final String userId;

  const UserPage({super.key, required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Utente $userId')),
      body: const Center(child: Text('Pagina utente')),
    );
  }
}
```

Navigazione verso quella rotta:

```dart
context.go('/user/42');
```

## Query parameters

GoRouter lavora con location in formato URI e quindi si presta bene anche alla gestione dei query parameter, soprattutto in contesti web o con filtri di ricerca.[cite:25][cite:33] Quando serve costruire URL come `/search?tab=recenti`, i parametri si possono leggere dallo stato della rotta tramite le informazioni dell'URI.[cite:33]

Esempio:

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final tab = state.uri.queryParameters['tab'] ?? 'tutti';
    return SearchPage(tab: tab);
  },
)
```

Navigazione:

```dart
context.go('/search?tab=recenti');
```

## Redirect

GoRouter supporta i redirect, utili per casi come autenticazione, onboarding obbligatorio o protezione di aree riservate.[cite:30][cite:33] La documentazione Flutter segnala che un package di routing avanzato è particolarmente adatto quando l'app deve riconfigurare il `Navigator` in risposta a deep link e percorsi più complessi.[cite:39]

Esempio semplificato di redirect verso login:

```dart
bool isLoggedIn = false;

final GoRouter router = GoRouter(
  redirect: (context, state) {
    final isGoingToLogin = state.matchedLocation == '/login';

    if (!isLoggedIn && !isGoingToLogin) {
      return '/login';
    }

    if (isLoggedIn && isGoingToLogin) {
      return '/';
    }

    return null;
  },
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginPage(),
    ),
  ],
);
```

In questo schema, l'utente non autenticato viene reindirizzato a `/login`, mentre un utente già autenticato non resta nella schermata di login.[cite:33]

## Rotte annidate

GoRouter permette anche di definire rotte figlie dentro una rotta padre, mantenendo una struttura più chiara quando le pagine condividono una gerarchia logica.[cite:30] Questo approccio è utile, per esempio, per avere una Home con sotto-pagine come dettaglio utente, dettaglio ordine o impostazioni.[cite:30][cite:33]

Esempio:

```dart
final GoRouter router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'settings',
          builder: (context, state) => const SettingsPage(),
        ),
      ],
    ),
  ],
);
```

La rotta finale sarà `/settings`, ma la definizione resta organizzata sotto la schermata principale.[cite:30]

## Esempio completo

Questo esempio unisce installazione concettuale, rotte base, parametri e redirect in una mini app didattica costruita con l'approccio raccomandato da GoRouter e dalla documentazione Flutter per `MaterialApp.router`.[cite:25][cite:39]

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MyApp());
}

bool isLoggedIn = false;

final GoRouter router = GoRouter(
  redirect: (context, state) {
    final goingToLogin = state.matchedLocation == '/login';

    if (!isLoggedIn && !goingToLogin) return '/login';
    if (isLoggedIn && goingToLogin) return '/';

    return null;
  },
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginPage(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'product/:id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return ProductPage(productId: id);
          },
        ),
      ],
    ),
  ],
);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router,
    );
  }
}

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            isLoggedIn = true;
            context.go('/');
          },
          child: const Text('Accedi'),
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Center(
            child: ElevatedButton(
              onPressed: () => context.go('/product/7'),
              child: const Text('Apri prodotto 7'),
            ),
          ),
          const SizedBox(height: 16),
          Center(
            child: ElevatedButton(
              onPressed: () {
                isLoggedIn = false;
                context.go('/login');
              },
              child: const Text('Logout'),
            ),
          ),
        ],
      ),
    );
  }
}

class ProductPage extends StatelessWidget {
  final String productId;

  const ProductPage({super.key, required this.productId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Prodotto $productId')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.pop(),
          child: const Text('Torna indietro'),
        ),
      ),
    );
  }
}
```

## Errori comuni

- Usare `MaterialApp` invece di `MaterialApp.router`, impedendo a GoRouter di gestire correttamente il routing.[cite:39]
- Confondere `go()` e `push()`, ottenendo uno stack di navigazione diverso da quello previsto.[cite:33][cite:36]
- Definire parametri nel path ma dimenticare di leggerli da `state.pathParameters`.[cite:30]
- Aspettarsi che i deep link funzionino senza configurare anche la parte piattaforma, come Android e iOS, quando necessario.[cite:33]

## In pratica

GoRouter è una scelta solida quando l'app Flutter ha bisogno di navigazione moderna, URL chiari, deep link, redirect e struttura più dichiarativa.[cite:25][cite:33][cite:39] Per un'app piccola si può ancora partire dal `Navigator`, ma per progetti medio-grandi GoRouter tende a rendere il codice di navigazione più leggibile e più semplice da mantenere.[cite:39][cite:33]

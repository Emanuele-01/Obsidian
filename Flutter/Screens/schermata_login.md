# 🔐 Schermata di Login

[← Torna all'Hub](../index.md) | [Continua con la Schermata Home →](./schermata_home.md)

---

La schermata di login è fondamentale per quasi ogni applicazione. In questa guida vedremo come costruirne una che comprenda la validazione dell'input, la gestione dello stato di caricamento e la logica di transizione verso un'altra schermata.

---

## 1. Architettura ed Elementi Chiave

Per realizzare un login interattivo con validazione, abbiamo bisogno di:
1.  Un **`StatefulWidget`** per tracciare lo stato (se l'utente ha cliccato sul pulsante, se stiamo caricando, ecc.). Approfondisci la struttura in [StatefulWidget](../Widgets/widget_principali.md#1-statelesswidget-vs-statefulwidget).
2.  Una **`GlobalKey<FormState>`** per identificare univocamente il form e validare i campi in blocco.
3.  Dei **`TextEditingController`** per leggere il testo digitato dall'utente.
4.  L'asincronia per simulare una richiesta API ad un server (vedi [Future e Async/Await](../Dart/dart_fondamenti.md#future-e-asyncawait)).

---

## 2. Codice Completo Commentato

Ecco l'implementazione di una schermata di login completa. Puoi usarla come base o studiarne i commenti per capirne il funzionamento:

```dart
import 'package:flutter/material.dart';
// Importiamo la schermata home per poterci navigare dopo il login
import 'schermata_home.dart'; 

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  // Chiave globale per accedere allo stato del Form
  final _formKey = GlobalKey<FormState>();

  // Controller per gestire e leggere i testi inseriti
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  // Variabili di stato (Dart fundamentals)
  bool _isLoading = false;
  bool _obscurePassword = true; // Nasconde la password per default

  // Eseguito quando il widget viene rimosso: liberiamo la memoria dei controller
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  // Funzione asincrona che simula la chiamata di login
  Future<void> _submitLogin() async {
    // 1. Controlla se i campi del form passano la validazione
    if (_formKey.currentState!.validate()) {
      // Imposta lo stato di caricamento a true per mostrare lo spinner
      setState(() {
        _isLoading = true;
      });

      // Simula una chiamata di rete (es. API REST) di 2 secondi
      await Future.delayed(const Duration(seconds: 2));

      setState(() {
        _isLoading = false;
      });

      // Logica fittizia di successo
      if (_emailController.text == "user@example.com" && _passwordController.text == "password123") {
        // Navigazione: sostituisce la schermata attuale con la Home
        if (mounted) {
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(builder: (context) => const HomeScreen()),
          );
        }
      } else {
        // Mostra un messaggio di errore se le credenziali sono errate
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(
              content: Text('Credenziali non valide! Usa: user@example.com / password123'),
              backgroundColor: Colors.red,
            ),
          );
        }
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // Usiamo SafeArea per evitare che la UI si sovrapponga a notch/barre di sistema
      body: SafeArea(
        child: Center(
          // SingleChildScrollView evita errori di pixel overflow quando appare la tastiera
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24.0),
            child: Form(
              key: _formKey, // Colleghiamo la chiave al form
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  // Logo / Icona
                  const Icon(
                    Icons.lock_outline,
                    size: 80,
                    color: Colors.blue,
                  ),
                  const SizedBox(height: 20),
                  
                  // Titolo
                  const Text(
                    'Benvenuto',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 30),

                  // Campo Email
                  TextFormField(
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: const InputDecoration(
                      labelText: 'Email',
                      prefixIcon: Icon(Icons.email),
                      border: OutlineInputBorder(),
                    ),
                    // Logica di validazione tramite funzione anonima (Dart)
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Inserisci la tua email';
                      }
                      if (!value.contains('@')) {
                        return 'Inserisci un indirizzo email valido';
                      }
                      return null; // Ritorna null se il campo è corretto
                    },
                  ),
                  const SizedBox(height: 16),

                  // Campo Password
                  TextFormField(
                    controller: _passwordController,
                    obscureText: _obscurePassword, // Nasconde il testo digitato
                    decoration: InputDecoration(
                      labelText: 'Password',
                      prefixIcon: const Icon(Icons.lock),
                      border: const OutlineInputBorder(),
                      // Icona a destra per mostrare/nascondere la password
                      suffixIcon: IconButton(
                        icon: Icon(
                          _obscurePassword ? Icons.visibility : Icons.visibility_off,
                        ),
                        onPressed: () {
                          setState(() {
                            _obscurePassword = !_obscurePassword;
                          });
                        },
                      ),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Inserisci la tua password';
                      }
                      if (value.length < 6) {
                        return 'La password deve avere almeno 6 caratteri';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 24),

                  // Pulsante Login o Indicatore Caricamento
                  _isLoading
                      ? const Center(child: CircularProgressIndicator())
                      : ElevatedButton(
                          onPressed: _submitLogin,
                          style: ElevatedButton.styleFrom(
                            padding: const EdgeInsets.symmetric(vertical: 16),
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(8),
                            ),
                          ),
                          child: const Text('Accedi', style: TextStyle(fontSize: 16)),
                        ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 3. Spiegazione dei Widget Utilizzati

*   **`Form`**: Contenitore che raggruppa più campi di testo. Fornisce metodi per validare, resettare e salvare lo stato dei campi figli.
*   **`TextFormField`**: Rispetto a un semplice `TextField`, si integra con `Form` e supporta la proprietà `validator`, che viene invocata quando chiami `_formKey.currentState!.validate()`.
*   **`TextEditingController`**: Permette di leggere il testo (`controller.text`) o di modificarlo programmaticamente. Deve sempre essere rilasciato nel metodo `dispose()` per evitare memory leak.
*   **`SingleChildScrollView`**: Avvolge il contenuto per renderlo scorrevole. Evita crash grafici dovuti alla tastiera che riduce lo spazio verticale disponibile.
*   **`ScaffoldMessenger`**: Utilizzato per mostrare messaggi pop-up temporanei (chiamati `SnackBar`) nella parte inferiore dello schermo.

---

[← Torna all'Hub](../index.md) | [Continua con la Schermata Home →](./schermata_home.md)

# 👤 Schermata Profilo e Impostazioni

[← Torna alla Schermata Home](./schermata_home.md) | [Continua con la Schermata Dettaglio →](./schermata_dettaglio.md)

---

La schermata Profilo serve a mostrare i dati dell'utente loggato, permetterne la modifica in tempo reale e gestire preferenze dell'app (come la modalità scura o il logout).

In questa guida vedremo come gestire l'interazione tra **modalità di visualizzazione** e **modalità di modifica** (edit mode) utilizzando variabili booleane e la ricostruzione dello stato tramite `setState`.

---

## 1. Codice Completo Commentato

Ecco l'implementazione pratica della pagina Profilo:

```dart
import 'package:flutter/material.dart';
// Importiamo il login per consentire il logout
import 'schermata_login.dart';

class ProfileScreen extends StatefulWidget {
  const ProfileScreen({super.key});

  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  // Variabile booleana per alternare la UI tra visualizzazione e modifica
  bool _isEditing = false;

  // Preferenza dell'app (es. Tema scuro)
  bool _isDarkMode = false;

  // Dati utente salvati nello stato (Dart variables)
  String _userName = 'Mario Rossi';
  String _userBio = 'Sviluppatore Flutter appassionato di tecnologia e UI/UX.';

  // Controller per gestire la modifica dei campi di testo
  late TextEditingController _nameController;
  late TextEditingController _bioController;

  @override
  void initState() {
    super.initState();
    // Inizializziamo i controller con i valori correnti dello stato
    _nameController = TextEditingController(text: _userName);
    _bioController = TextEditingController(text: _userBio);
  }

  @override
  void dispose() {
    _nameController.dispose();
    _bioController.dispose();
    super.dispose();
  }

  // Funzione per salvare le modifiche apportate
  void _saveProfile() {
    setState(() {
      _userName = _nameController.text;
      _userBio = _bioController.text;
      _isEditing = false; // Disattiva la modalità modifica
    });
    
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Profilo salvato con successo!')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Il mio Profilo'),
        actions: [
          // Cambia l'icona dell'app bar in base allo stato di modifica
          IconButton(
            icon: Icon(_isEditing ? Icons.save : Icons.edit),
            onPressed: () {
              if (_isEditing) {
                _saveProfile();
              } else {
                setState(() {
                  _isEditing = true; // Attiva modalità modifica
                });
              }
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          children: [
            const SizedBox(height: 20),
            
            // 1. AVATAR UTENTE (Cerchio con immagine/iniziali)
            const CircleAvatar(
              radius: 60,
              backgroundColor: Colors.blue,
              child: Text(
                'MR',
                style: TextStyle(fontSize: 40, color: Colors.white, fontWeight: FontWeight.bold),
              ),
            ),
            const SizedBox(height: 24),

            // 2. CAMPI DATI (Dinamici in base a _isEditing)
            if (!_isEditing) ...[
              // Vista di sola lettura
              Text(
                _userName,
                style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Text(
                'mario.rossi@example.com',
                style: TextStyle(color: Colors.grey[600], fontSize: 16),
              ),
              const SizedBox(height: 16),
              Text(
                _userBio,
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 14, fontStyle: FontStyle.italic),
              ),
            ] else ...[
              // Vista di input/modifica
              TextField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Nome e Cognome',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 16),
              TextField(
                controller: _bioController,
                maxLines: 3, // Consente l'inserimento di più righe per la bio
                decoration: const InputDecoration(
                  labelText: 'Biografia',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 16),
              ElevatedButton(
                onPressed: _saveProfile,
                child: const Text('Salva Modifiche'),
              ),
            ],

            const SizedBox(height: 30),
            const Divider(), // Linea divisoria
            const SizedBox(height: 10),

            // 3. SEZIONE IMPOSTAZIONI (Lista di opzioni tramite ListTile)
            const Align(
              alignment: Alignment.centerLeft,
              child: Text(
                'Impostazioni App',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.blue),
              ),
            ),
            const SizedBox(height: 10),

            // Toggle Dark Mode
            SwitchListTile(
              title: const Text('Modalità Scura'),
              subtitle: const Text('Cambia il tema visivo dell\'applicazione'),
              secondary: const Icon(Icons.brightness_6),
              value: _isDarkMode,
              onChanged: (bool value) {
                setState(() {
                  _isDarkMode = value;
                });
              },
            ),

            // Pulsante Informazioni
            ListTile(
              leading: const Icon(Icons.info_outline),
              title: const Text('Info Applicazione'),
              trailing: const Icon(Icons.chevron_right),
              onTap: () {
                showAboutDialog(
                  context: context,
                  applicationName: 'Store Demo App',
                  applicationVersion: '1.0.0',
                  applicationIcon: const Icon(Icons.store, color: Colors.blue),
                );
              },
            ),

            // Pulsante Esci (Logout)
            ListTile(
              leading: const Icon(Icons.exit_to_app, color: Colors.red),
              title: const Text('Logout', style: TextStyle(color: Colors.red)),
              onTap: () {
                // Reindirizza l'utente alla schermata di login pulendo la pila di navigazione
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(builder: (context) => const LoginScreen()),
                  (route) => false, // Rimuove tutte le rotte precedenti
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 2. Spiegazione dei Widget Utilizzati

*   **`CircleAvatar`**: Widget circolare preconfigurato che ospita un'immagine o delle iniziali di testo. Molto utilizzato nei layout dei profili o dei commenti.
*   **`SwitchListTile`**: Unione tra una voce di menu (`ListTile`) e uno switch on/off (`Switch`). Ottimizza lo spazio per le impostazioni e gestisce l'allineamento automatico dei componenti.
*   **`Divider`**: Semplice linea orizzontale di separazione estetica tra i contenuti della pagina.
*   **`showAboutDialog()`**: Funzione integrata in Flutter che apre una finestra modale preconfigurata contenente informazioni sull'app (nome, licenze, versione).
*   **`Navigator.pushAndRemoveUntil`**: Metodo di navigazione usato per forzare il logout. Sostituisce la schermata corrente e svuota la cronologia delle pagine salvate, impedendo all'utente di tornare indietro con il tasto di sistema del dispositivo.

---

[← Torna alla Schermata Home](./schermata_home.md) | [Continua con la Schermata Dettaglio →](./schermata_dettaglio.md)

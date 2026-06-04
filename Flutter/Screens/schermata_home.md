# 🏠 Schermata Home (Dashboard)

[← Torna alla Schermata Login](./schermata_login.md) | [Continua con la Schermata Profilo →](./schermata_profile.md)

---

La schermata Home è spesso un pannello di controllo o "Dashboard". In questa guida vedremo come creare una struttura classica con:
1.  Una barra di navigazione inferiore (**`BottomNavigationBar`**).
2.  Una lista orizzontale di categorie cliccabili che filtrano una griglia sottostante (logica Dart in tempo reale).
3.  Una griglia dinamica (**`GridView.builder`**) che mostra una serie di elementi con immagini e dettagli.
4.  L'uso di widget di ottimizzazione come `ListView.builder` (vedi [Widget per Liste](../Widgets/widget_principali.md#5-widget-per-liste-e-scorrimento)).

---

## 1. Codice Completo Commentato

Ecco l'implementazione pratica della schermata Home:

```dart
import 'package:flutter/material.dart';
// Importiamo le schermate per la navigazione
import 'schermata_profile.dart';
import 'schermata_dettaglio.dart';

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  // Indice corrente della barra di navigazione inferiore
  int _currentIndex = 0;

  // Lista delle categorie disponibili (Dart List)
  final List<String> _categories = ['Tutti', 'Tech', 'Moda', 'Casa', 'Sport'];
  String _selectedCategory = 'Tutti';

  // Dati fittizi: una lista di mappe contenenti i prodotti (Mock Data)
  final List<Map<String, dynamic>> _allProducts = [
    {'id': '1', 'title': 'Smartphone Pro', 'category': 'Tech', 'price': '999€', 'image': 'phone_icon'},
    {'id': '2', 'title': 'Laptop Ultra', 'category': 'Tech', 'price': '1499€', 'image': 'laptop_icon'},
    {'id': '3', 'title': 'Giacca Autunnale', 'category': 'Moda', 'price': '120€', 'image': 'jacket_icon'},
    {'id': '4', 'title': 'Lampada LED Smart', 'category': 'Casa', 'price': '45€', 'image': 'light_icon'},
    {'id': '5', 'title': 'Scarpe Running', 'category': 'Sport', 'price': '85€', 'image': 'shoe_icon'},
    {'id': '6', 'title': 'Cuffie Bluetooth', 'category': 'Tech', 'price': '199€', 'image': 'headphones_icon'},
  ];

  // Lista filtrata dei prodotti basata sulla categoria selezionata
  List<Map<String, dynamic>> get _filteredProducts {
    if (_selectedCategory == 'Tutti') {
      return _allProducts;
    }
    // Utilizziamo il metodo where() delle liste di Dart per filtrare
    return _allProducts.where((p) => p['category'] == _selectedCategory).toList();
  }

  @override
  Widget build(BuildContext context) {
    // Definizione delle schermate principali collegate alla BottomNavigationBar
    // Se l'indice è 1, mostriamo la pagina del profilo, altrimenti la dashboard
    if (_currentIndex == 1) {
      return const ProfileScreen();
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard Store'),
        actions: [
          IconButton(
            icon: const Icon(Icons.notifications),
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Nessuna nuova notifica')),
              );
            },
          ),
        ],
      ),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Sezione di Benvenuto
          const Padding(
            padding: EdgeInsets.all(16.0),
            child: Text(
              'Esplora i prodotti',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
          ),

          // 1. LISTA ORIZZONTALE DELLE CATEGORIE
          SizedBox(
            height: 50,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              itemCount: _categories.length,
              padding: const EdgeInsets.symmetric(horizontal: 12),
              itemBuilder: (context, index) {
                final category = _categories[index];
                final isSelected = category == _selectedCategory;
                
                return Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 4),
                  child: ChoiceChip(
                    label: Text(category),
                    selected: isSelected,
                    selectedColor: Colors.blue,
                    labelStyle: TextStyle(
                      color: isSelected ? Colors.white : Colors.black,
                    ),
                    onSelected: (bool selected) {
                      setState(() {
                        // Aggiorna lo stato e filtra i prodotti
                        _selectedCategory = category;
                      });
                    },
                  ),
                );
              },
            ),
          ),

          const SizedBox(height: 10),

          // 2. GRIGLIA VERTICALE DEI PRODOTTI (FILTRATA)
          Expanded(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              // GridView.builder ottimizza le prestazioni caricando solo gli elementi a schermo
              child: GridView.builder(
                itemCount: _filteredProducts.length,
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 2, // 2 colonne
                  crossAxisSpacing: 12,
                  mainAxisSpacing: 12,
                  childAspectRatio: 0.8, // Rapporto tra larghezza e altezza del box
                ),
                itemBuilder: (context, index) {
                  final product = _filteredProducts[index];
                  
                  return GestureDetector(
                    // Al tap sul prodotto, navighiamo verso il dettaglio
                    onTap: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (context) => DetailScreen(product: product),
                        ),
                      );
                    },
                    child: Card(
                      elevation: 4,
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          // Immagine/Icona placeholder del prodotto
                          Expanded(
                            child: Container(
                              decoration: BoxDecoration(
                                color: Colors.grey[200],
                                borderRadius: const BorderRadius.vertical(top: Radius.circular(12)),
                              ),
                              child: Center(
                                child: Icon(
                                  _getProductIcon(product['image']),
                                  size: 50,
                                  color: Colors.grey[600],
                                ),
                              ),
                            ),
                          ),
                          Padding(
                            padding: const EdgeInsets.all(8.0),
                            child: Column(
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                Text(
                                  product['title'],
                                  style: const TextStyle(fontWeight: FontWeight.bold),
                                  maxLines: 1,
                                  overflow: TextOverflow.ellipsis,
                                ),
                                const SizedBox(height: 4),
                                Row(
                                  mainAxisAlignment: MainAxisAlignment.between,
                                  children: [
                                    Text(
                                      product['price'],
                                      style: const TextStyle(color: Colors.blue, fontWeight: FontWeight.bold),
                                    ),
                                    Text(
                                      product['category'],
                                      style: TextStyle(fontSize: 10, color: Colors.grey[600]),
                                    ),
                                  ],
                                ),
                              ],
                            ),
                          ),
                        ],
                      ),
                    ),
                  );
                },
              ),
            ),
          ),
        ],
      ),

      // 3. BARRA DI NAVIGAZIONE INFERIORE
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index; // Cambia pagina (es. Home -> Profilo)
          });
        },
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'Profilo',
          ),
        ],
      ),
    );
  }

  // Helper metod per ritornare l'icona in base alla stringa salvata nei dati
  IconData _getProductIcon(String imageName) {
    switch (imageName) {
      case 'phone_icon': return Icons.phone_android;
      case 'laptop_icon': return Icons.laptop;
      case 'jacket_icon': return Icons.checkroom;
      case 'light_icon': return Icons.lightbulb;
      case 'shoe_icon': return Icons.directions_run;
      case 'headphones_icon': return Icons.headphones;
      default: return Icons.device_unknown;
    }
  }
}
```

---

## 2. Spiegazione dei Widget Utilizzati

*   **`ChoiceChip`**: Un chip Material Design che supporta lo stato selezionato/non selezionato, ideale per filtri veloci o tag di categoria.
*   **`GridView.builder`**: Funziona in modo analogo a `ListView.builder` ma organizza gli elementi in una griglia bidimensionale. Richiede un `gridDelegate` per definire il numero di colonne e le distanze.
*   **`Card`**: Un pannello con angoli arrotondati e una leggera ombra, usato per dare tridimensionalità ed evidenziare i singoli prodotti.
*   **`BottomNavigationBar`**: Una barra inferiore che rende immediato il cambio tra le sezioni principali dell'applicazione tramite un indice numerico (`currentIndex`).
*   **`GestureDetector`**: Avvolge l'intera `Card` del prodotto per intercettare il tocco dell'utente (`onTap`) ed eseguire la transizione verso la schermata di dettaglio.

---

[← Torna alla Schermata Login](./schermata_login.md) | [Continua con la Schermata Profilo →](./schermata_profile.md)

# 🛍️ Schermata di Dettaglio

[← Torna alla Schermata Home](./schermata_home.md) | [Torna all'Hub](../index.md)

---

La schermata di dettaglio mostra le informazioni complete di un singolo elemento selezionato (in questo caso, un prodotto dallo store). 

In questa guida vedremo come:
1.  **Ricevere dati nel costruttore** da una schermata precedente (passaggio dati).
2.  Utilizzare il widget **`Hero`** per creare una transizione grafica fluida dell'immagine del prodotto tra la schermata Home e la schermata Dettaglio.
3.  Aggiungere **logica di calcolo interattiva** (moltiplicazione quantità * prezzo) e aggiornare dinamicamente il valore a schermo.

---

## 1. Codice Completo Commentato

Ecco l'implementazione pratica della schermata Dettaglio:

```dart
import 'package:flutter/material.dart';

class DetailScreen extends StatefulWidget {
  // Riceviamo la mappa del prodotto tramite il costruttore (Dart OOP)
  // vedi approfondimenti sui costruttori in: Fondamenti di Dart (OOP)
  final Map<String, dynamic> product;

  const DetailScreen({super.key, required this.product});

  @override
  State<DetailScreen> createState() => _DetailScreenState();
}

class _DetailScreenState extends State<DetailScreen> {
  // Quantità selezionata dall'utente (Stato locale)
  int _quantity = 1;

  // Funzione per incrementare la quantità
  void _increment() {
    setState(() {
      _quantity++;
    });
  }

  // Funzione per decrementare la quantità (minimo 1)
  void _decrement() {
    if (_quantity > 1) {
      setState(() {
        _quantity--;
      });
    }
  }

  // Logica Dart per calcolare il prezzo totale dinamico
  double _calculateTotalPrice() {
    // Puliamo la stringa del prezzo (rimuoviamo '€') e la convertiamo in double
    final priceString = widget.product['price'].replaceAll('€', '');
    final price = double.tryParse(priceString) ?? 0.0;
    return price * _quantity;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.product['title']),
      ),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. WIDGET HERO PER L'ANIMAZIONE DI TRANSIZIONE
            // Il tag deve essere identico a quello definito nella schermata Home
            Hero(
              tag: 'product-image-${widget.product['id']}',
              child: Container(
                width: double.infinity,
                height: 300,
                color: Colors.grey[200],
                child: Center(
                  child: Icon(
                    _getProductIcon(widget.product['image']),
                    size: 150,
                    color: Colors.grey[600],
                  ),
                ),
              ),
            ),
            
            Padding(
              padding: const EdgeInsets.all(20.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Categoria e Badge
                  Chip(
                    label: Text(widget.product['category']),
                    backgroundColor: Colors.blue[50],
                  ),
                  const SizedBox(height: 10),

                  // Titolo Prodotto
                  Text(
                    widget.product['title'],
                    style: const TextStyle(fontSize: 26, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),

                  // Prezzo unitario
                  Text(
                    'Prezzo unitario: ${widget.product['price']}',
                    style: const TextStyle(fontSize: 18, color: Colors.blue, fontWeight: FontWeight.w600),
                  ),
                  const SizedBox(height: 20),

                  // Descrizione di esempio
                  const Text(
                    'Descrizione del Prodotto',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'Questo è un prodotto demo di alta qualità. Perfetto per le tue attività quotidiane. '
                    'Realizzato con materiali resistenti e dal design moderno.',
                    style: TextStyle(color: Colors.grey[700], height: 1.5),
                  ),
                  
                  const SizedBox(height: 30),
                  const Divider(),
                  const SizedBox(height: 20),

                  // 2. CONTROLLO DELLA QUANTITÀ (Pulsanti + e -)
                  Row(
                    mainAxisAlignment: MainAxisAlignment.between,
                    children: [
                      const Text(
                        'Seleziona Quantità:',
                        style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                      ),
                      Row(
                        children: [
                          IconButton(
                            onPressed: _decrement,
                            icon: const Icon(Icons.remove_circle_outline),
                            color: Colors.blue,
                            iconSize: 30,
                          ),
                          Text(
                            '$_quantity',
                            style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                          ),
                          IconButton(
                            onPressed: _increment,
                            icon: const Icon(Icons.add_circle_outline),
                            color: Colors.blue,
                            iconSize: 30,
                          ),
                        ],
                      )
                    ],
                  ),
                  
                  const SizedBox(height: 30),

                  // 3. RIEPILOGO PREZZO TOTALE E PULSANTE D'ACQUISTO
                  Row(
                    mainAxisAlignment: MainAxisAlignment.between,
                    children: [
                      Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          const Text(
                            'Totale da pagare:',
                            style: TextStyle(color: Colors.grey, fontSize: 14),
                          ),
                          Text(
                            '${_calculateTotalPrice().toStringAsFixed(2)}€',
                            style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Colors.green),
                          ),
                        ],
                      ),
                      ElevatedButton.icon(
                        onPressed: () {
                          // Simula l'aggiunta al carrello
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text('Aggiunto/i $_quantity ${widget.product['title']} al carrello!'),
                              backgroundColor: Colors.green,
                            ),
                          );
                        },
                        icon: const Icon(Icons.shopping_cart),
                        label: const Text('Acquista'),
                        style: ElevatedButton.styleFrom(
                          backgroundColor: Colors.green,
                          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
                          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
                        ),
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
  }

  // Restituisce l'icona corrispondente al tipo salvato
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

*   **`Hero`**: Permette l'animazione di transizione di un widget grafico tra due pagine. Flutter si occupa di calcolare lo spostamento e il ridimensionamento del widget. L'unica condizione è che la proprietà `tag` sia identica sia nella pagina di partenza che in quella di arrivo.
*   **`Chip`**: Un piccolo elemento visuale compatto che racchiude testo o immagini, perfetto per indicare categorie o filtri applicati.
*   **`IconButton`**: Pulsante circolare che racchiude un'icona. È comodo per azioni incrementali (+/-) o controlli multimediali.
*   **`ElevatedButton.icon`**: Costruttore speciale del pulsante in rilievo che permette di allineare facilmente un'icona a fianco del testo all'interno dell'etichetta del pulsante.

---

[← Torna alla Schermata Home](./schermata_home.md) | [Torna all'Hub](../index.md)

# -// lib/main.dart
import 'package:flutter/material.dart';

void main() => runApp(RestaurantApp());

class RestaurantApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'مَطابخنا',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        fontFamily: 'Tahoma',
        primarySwatch: Colors.deepOrange,
      ),
      home: Directionality(
        textDirection: TextDirection.rtl,
        child: RestaurantsListPage(),
      ),
    );
  }
}

/* نموذج بيانات بسيط */
class MenuItem {
  final String id;
  final String name;
  final String description;
  final double price;
  final String image;
  MenuItem({required this.id, required this.name, required this.description, required this.price, required this.image});
}

class Restaurant {
  final String id;
  final String name;
  final String city;
  final String image;
  final double rating;
  final List<MenuItem> menu;
  Restaurant({required this.id, required this.name, required this.city, required this.image, required this.rating, required this.menu});
}

/* بيانات تجريبية */
final sampleRestaurants = <Restaurant>[
  Restaurant(
    id: 'r1',
    name: 'مطعم السندباد',
    city: 'بغداد',
    image: 'https://via.placeholder.com/400x200.png?text=سندباد',
    rating: 4.5,
    menu: [
      MenuItem(id: 'm1', name: 'شاورما دجاج', description: 'صوص خاص وخبز طازج', price: 25000, image: 'https://via.placeholder.com/200.png?text=شاورما'),
      MenuItem(id: 'm2', name: 'بيتزا مارجريتا', description: 'جبنة موزاريلا وطماطم', price: 40000, image: 'https://via.placeholder.com/200.png?text=بيتزا'),
    ],
  ),
  Restaurant(
    id: 'r2',
    name: 'مشويات الفجر',
    city: 'البصرة',
    image: 'https://via.placeholder.com/400x200.png?text=مشويات',
    rating: 4.2,
    menu: [
      MenuItem(id: 'm3', name: 'كباب لحم', description: 'مشوي على الفحم', price: 60000, image: 'https://via.placeholder.com/200.png?text=كباب'),
      MenuItem(id: 'm4', name: 'أرز بخاري', description: 'رز مع بهارات خاصة', price: 20000, image: 'https://via.placeholder.com/200.png?text=أرز'),
    ],
  ),
];

/// صفحة القائمة الرئيسية للمطاعم
class RestaurantsListPage extends StatefulWidget {
  @override
  _RestaurantsListPageState createState() => _RestaurantsListPageState();
}

class _RestaurantsListPageState extends State<RestaurantsListPage> {
  String selectedCity = 'الكل';

  List<Restaurant> get filtered {
    if (selectedCity == 'الكل') return sampleRestaurants;
    return sampleRestaurants.where((r) => r.city == selectedCity).toList();
  }

  @override
  Widget build(BuildContext context) {
    final cities = ['الكل', ...{for (var r in sampleRestaurants) r.city}];
    return Scaffold(
      appBar: AppBar(
        title: Text('المطاعم'),
        actions: [
          IconButton(icon: Icon(Icons.shopping_cart), onPressed: () {
            Navigator.push(context, MaterialPageRoute(builder: (_) => CartPage()));
          })
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(12),
            child: DropdownButtonFormField<String>(
              value: selectedCity,
              items: cities.map((c) => DropdownMenuItem(value: c, child: Text(c))).toList(),
              onChanged: (v) => setState(() { selectedCity = v ?? 'الكل'; }),
              decoration: InputDecoration(labelText: 'اختر المدينة', border: OutlineInputBorder()),
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: filtered.length,
              itemBuilder: (_, i) {
                final r = filtered[i];
                return Card(
                  margin: EdgeInsets.symmetric(horizontal: 12, vertical: 8),
                  child: ListTile(
                    leading: Image.network(r.image, width: 90, fit: BoxFit.cover),
                    title: Text(r.name),
                    subtitle: Text('${r.rating} ⭐ • ${r.city}'),
                    trailing: Icon(Icons.arrow_forward_ios),
                    onTap: () => Navigator.push(context, MaterialPageRoute(builder: (_) => RestaurantPage(restaurant: r))),
                  ),
                );
              },
            ),
          )
        ],
      ),
    );
  }
}

/// صفحة عرض مطعم وقائمة المطعم
class RestaurantPage extends StatelessWidget {
  final Restaurant restaurant;
  RestaurantPage({required this.restaurant});
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(restaurant.name),
        actions: [IconButton(icon: Icon(Icons.shopping_cart), onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => CartPage())) )],
      ),
      body: ListView(
        children: [
          Image.network(restaurant.image, height: 180, fit: BoxFit.cover, width: double.infinity),
          Padding(padding: EdgeInsets.all(12), child: Text('القائمة', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold))),
          ...restaurant.menu.map((m) => MenuCard(item: m)).toList(),
        ],
      ),
    );
  }
}

class MenuCard extends StatelessWidget {
  final MenuItem item;
  MenuCard({required this.item});
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      child: ListTile(
        leading: Image.network(item.image, width: 80, fit: BoxFit.cover),
        title: Text(item.name),
        subtitle: Text(item.description),
        trailing: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('${item.price.toStringAsFixed(0)} د.ع'),
            SizedBox(height: 6),
            ElevatedButton(
              child: Text('أضف'),
              onPressed: () {
                CartModel.of(context).add(item);
                final snack = SnackBar(content: Text('أضيفت ${item.name} إلى السلة'));
                ScaffoldMessenger.of(context).showSnackBar(snack);
              },
            )
          ],
        ),
        onTap: () => Navigator.push(context, MaterialPageRoute(builder: (_) => MenuItemPage(item: item))),
      ),
    );
  }
}

/// صفحة تفاصيل الوجبة
class MenuItemPage extends StatelessWidget {
  final MenuItem item;
  MenuItemPage({required this.item});
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(item.name)),
      body: ListView(
        padding: EdgeInsets.all(12),
        children: [
          Image.network(item.image, height: 220, fit: BoxFit.cover),
          SizedBox(height: 12),
          Text(item.name, style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
          SizedBox(height: 8),
          Text(item.description),
          SizedBox(height: 12),
          Text('السعر: ${item.price.toStringAsFixed(0)} د.ع', style: TextStyle(fontSize: 18)),
          SizedBox(height: 12),
          ElevatedButton(
            child: Text('أضف إلى السلة'),
            onPressed: () {
              CartModel.of(context).add(item);
              ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('أضيفت ${item.name}')));
            },
          )
        ],
      ),
    );
  }
}

/* نموذج سلة بسيطة كموديل (Singleton عبر InheritedWidget) */
class CartModel extends InheritedWidget {
  final _CartState data;
  CartModel({required Widget child, required this.data}): super(child: child);

  static _CartState of(BuildContext context) {
    final cm = context.dependOnInheritedWidgetOfExactType<CartModel>();
    if (cm == null) throw Exception('CartModel not found');
    return cm.data;
  }

  @override
  bool updateShouldNotify(covariant CartModel oldWidget) => true;
}

class CartPage extends StatefulWidget {
  @override
  _CartState createState() => _CartState();
}

class _CartState extends State<CartPage> {
  final Map<String, int> _items = {}; // itemId -> qty

  void add(MenuItem item) {
    setState(() {
      _items[item.id] = (_items[item.id] ?? 0) + 1;
    });
  }

  void removeOne(MenuItem item) {
    setState(() {
      final q = (_items[item.id] ?? 0) - 1;
      if (q <= 0) _items.remove(item.id);
      else _items[item.id] = q;
    });
  }

  double get total {
    double t = 0;
    for (var r in sampleRestaurants) {
      for (var m in r.menu) {
        final q = _items[m.id] ?? 0;
        t += q * m.price;
      }
    }
    return t;
  }

  @override
  Widget build(BuildContext context) {
    return CartModel(
      data: this,
      child: Scaffold(
        appBar: AppBar(title: Text('السلة')),
        body: _items.isEmpty
            ? Center(child: Text('السلة فارغة'))
            : Column(
                children: [
                  Expanded(
                    child: ListView(
                      children: _items.entries.map((e) {
                        final id = e.key;
                        final qty = e.value;
                        final item = sampleRestaurants.expand((r) => r.menu).firstWhere((m) => m.id == id);
                        return ListTile(
                          leading: Image.network(item.image, width: 70, fit: BoxFit.cover),
                          title: Text(item.name),
                          subtitle: Text('الكمية: $qty'),
                          trailing: Column(
                            mainAxisAlignment: MainAxisAlignment.center,
                            children: [
                              Text('${(qty * item.price).toStringAsFixed(0)} د.ع'),
                              Row(
                                mainAxisSize: MainAxisSize.min,
                                children: [
                                  IconButton(icon: Icon(Icons.remove_circle_outline), onPressed: () => removeOne(item)),
                                  IconButton(icon: Icon(Icons.add_circle_outline), onPressed: () => add(item)),
                                ],
                              )
                            ],
                          ),
                        );
                      }).toList(),
                    ),
                  ),
                  Padding(
                    padding: EdgeInsets.all(12),
                    child: Column(
                      children: [
                        Text('الإجمالي: ${total.toStringAsFixed(0)} د.ع', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                        SizedBox(height: 8),
                        ElevatedButton(
                          child: Text('تأكيد الطلب'),
                          onPressed: () {
                            showDialog(context: context, builder: (_) => AlertDialog(
                              title: Text('تأكيد'),
                              content: Text('هل تريد تأكيد الطلب؟'),
                              actions: [
                                TextButton(child: Text('إلغاء'), onPressed: () => Navigator.pop(context)),
                                ElevatedButton(child: Text('تأكيد الطلب'), onPressed: () {
                                  setState(() { _items.clear(); });
                                  Navigator.pop(context);
                                  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('تم إرسال الطلب')));
                                })
                              ],
                            ));
                          },
                        )
                      ],
                    ),
                  )
                ],
              ),
      ),
    );
  }
}

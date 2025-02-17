import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: Text('Кнопки Приложение'),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              ElevatedButton(
                onPressed: () {
                  print('Первая кнопка нажата');
                },
                child: Text('Кнопка 1'),
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  print('Вторая кнопка нажата');
                },
                child: Text('Кнопка 2'),
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  print('Третья кнопка нажата');
                },
                child: Text('Кнопка 3'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

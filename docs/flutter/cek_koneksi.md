# Check Koneksi Internet

Untuk melakukan check koneksi internet kita menggunakan [connectivity plus](https://pub.dev/packages/connectivity_plus), pada saat dokumentasi ini di buat example yang betebaran kita harus implementasi pada setiap screen. Disini kita hanya deklarasi sekali saja di main pada `Materialapp`, tapi sudah mengcover semua screen

## Usage/Examples

```dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:flutter/material.dart';

MaterialApp(
    routes: {
        '/home': (context) => HomeScreen(),
        '/about': (context) => AboutScreen(),
    },
    initialRoute: '/home',
    builder: (ctxm, widget) {
        return StreamBuilder(
            stream: Connectivity().onConnectivityChanged,
            builder: (ctxb, AsyncSnapshot<ConnectivityResult> snapshot) {
            return snapshot.data == ConnectivityResult.none
                ? const NoInternetScreen()
                : widget!;
            },
        );
    }),
)
```

## Reference

- [https://pub.dev/packages/connectivity_plus/example](https://pub.dev/packages/connectivity_plus/example)
- [https://stackoverflow.com/questions/68030497/check-connectivity-in-flutter-and-change-state-according-connection-status](https://stackoverflow.com/questions/68030497/check-connectivity-in-flutter-and-change-state-according-connection-status)

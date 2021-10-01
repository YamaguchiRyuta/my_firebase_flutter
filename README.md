[![Deploy to Firebase Hosting on merge](https://github.com/YamaguchiRyuta/my_firebase_flutter/actions/workflows/firebase-hosting-merge.yml/badge.svg)](https://github.com/YamaguchiRyuta/my_firebase_flutter/actions/workflows/firebase-hosting-merge.yml)
[![Deploy to Firebase Hosting on PR](https://github.com/YamaguchiRyuta/my_firebase_flutter/actions/workflows/firebase-hosting-pull-request.yml/badge.svg)](https://github.com/YamaguchiRyuta/my_firebase_flutter/actions/workflows/firebase-hosting-pull-request.yml)

# my_firebase

A new Flutter project.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://flutter.dev/docs/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://flutter.dev/docs/cookbook)

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials, samples, guidance on
mobile development, and a full API reference.

# 環境構築でのメモ

- [vulkan Runtime](https://vulkan.lunarg.com/sdk/home) が別途必要だった。
- Hyper-V と HAXM の共存はできない。
- Intel Virtualization Technology も有効にする。

# 開発手順

- Android Studio で Flutter プロジェクトを新規作成
- [pubspec.yaml に firebase.core を追加](https://firebase.flutter.dev/docs/overview/#installation)

  ```bash
  # Packageの追加
  flutter pub add firebase_core
  flutter pub add firebase_analytics
  flutter pub add firebase_auth
  flutter pub add cloud_firestore
  # Packageの取得(上記コマンドを実行した場合は不要)
  #  flutter pub get
  ```

  ```dart
  import 'package:firebase_core/firebase_core.dart';
  import 'package:firebase_auth/firebase_auth.dart';
  import 'package:cloud_firestore/cloud_firestore.dart';
  ```

- [web/index.html に SDK を設定](https://firebase.flutter.dev/docs/installation/web)

追記内容は Firebase プロジェクトのマイアプリにウェブアプリを追加し、

「プロジェクトの設定」から確認できる。

```html
<!-- The core Firebase JS SDK is always required and must be listed first -->
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-app.js"></script>

<!-- TODO: Add SDKs for Firebase products that you want to use
   https://firebase.google.com/docs/web/setup#available-libraries -->
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-analytics.js"></script>

<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-firestore.js"></script>

<script>
  // Your web app's Firebase configuration
  // For Firebase JS SDK v7.20.0 and later, measurementId is optional
  // 以下の値は適切にセットすること。
  var firebaseConfig = {
    apiKey: "API_KEY",
    authDomain: "PROJECT_ID.firebaseapp.com",
    databaseURL: "https://PROJECT_ID.firebaseio.com",
    projectId: "PROJECT_ID",
    storageBucket: "PROJECT_ID.appspot.com",
    messagingSenderId: "SENDER_ID",
    appId: "APP_ID",
    measurementId: "G-MEASUREMENT_ID",
  };

  // Initialize Firebase
  firebase.initializeApp(firebaseConfig);
  firebase.analytics();
</script>
```

- [FlutterFire を初期化するよう main.dart を編集](https://firebase.flutter.dev/docs/overview/#initializing-flutterfire)

FutureBuilder 経由で MyApp を渡す。StatefulWidget を使う方法も紹介されていた。

firebase の auth, store などを実装。

- Firebase\*\*\*\*.instance で auth や store のインスタンスを参照できる。これを使うのがメイン
- 認証エラー時は例外が投げられる。例外の Stacktrace を見れば原因が分かる
- Authentication を使うには Firebase 側で Sign-in method を設定する必要がある。
- 本番環境モードでの Firestore にはルールの設定が必要。
- Firestore は普通の NoSQL。MongoDB とかのような。

# デプロイ手順

- Firebase CLI をインストール

  ```bash
  npm install -g firebase-tools

  firebase login
  ```

- Firebase プロジェクトとして初期化

  public ディレクトリの指定では build/web を指定すること

  色々聞かれるが、CLI のメッセージを見ればなんとかなる

  ```bash
  firebase init
  ```

- Flutter で Web を有効化していないならする

  ```bash
  flutter channel stable
  flutter upgrade
  flutter config --enable-web
  # com.exampleから変更していれば--orgオプションは省略可
  flutter create --org package_name .
  ```

- Flutter を Web 用にビルド

  通常は Canvas 要素としてレンダリングされる。

  フォントを追加しないと漢字が中国語っぽいフォントになってしまう

  ```bash
  flutter build web
  # flutter build web --web-renderer canvaskit
  ```

  html でビルドする場合はこっち。

  漢字のフォントは直るが、Firefox だと微妙にレイアウトがズレたりするっぽい。

  (FloatingActionButton に hover した時など)

  ```bash
  flutter build web --web-renderer html
  ```

  デプロイ実行

  ```bash
  firebase deploy
  ```

  デプロイ結果  
  [FirebaseHosting](https://fir-first-a8330.web.app)

  こっちも見ておこう
  [Firebase ドキュメント](https://firebase.google.com/docs/hosting?hl=ja)

  GitHub Actions
  [with GitHub Actions](https://zenn.dev/pressedkonbu/articles/deploy-flutter-web-app-with-firebase-hosting)

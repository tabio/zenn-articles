---
title: "Software Design 「はじめてのFlutter」 の備忘録"
emoji: "🍹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "xcode", "android studio"]
published: true
---

Software DesignのFlutterの記事の備忘録
## Flutterの特徴

- 独自のレンダリングシステム
  - Skiaという描画ライブラリが組み込まれた独自のレンダリングシステムがある
  - このおかげでネイティブでの実装と比較しても遜色ないパフォーマンスを実現
- 各プラットフォーム対応状況が進んでおり、1つのコードベースで複数のプラットフォームで動作する価値が高まっている
  - Android, iOS以外にもWeb, Windowsは安定版になっている
  - macOS, Linuxもベータ版として開発中
  - iOS/Androidエンジニア区別なく1つにまとまって少人数開発できる
  - プラットフォーム間の実装の差が生まれにくい
- Dart言語
  - Googleが開発している言語
  - Java, JavaScriptと似ている
  - Null Safety
    - Dart 2.12で入った
    - 予期せぬNullへの参照をビルド前にチェックできる
    - 非Null許容型とNull許容型を明確に区別している
      - Null型の変数を非Null型の変数に入れる時はNullチェックしないと代入できない
      ```dart
      String value = ''; // 非Null型
      String? nullableValue = null;

      if (nullableValue == null) {
        return
      }

      value = nullableValue;
      ```
- 他のクロスプラットフォーム(cordova, xamarin)に対する強み
  - ホットリロードによる圧倒的な生産性の高さ
  - DevTools
  - 初学者でも学習しやすい。公式Youtubeチャンネルもある
  - pub.devに高品質なパッケージがそろっている
  - iOS, Android固有の実装(ファイルシステム、加速度センサ、カメラなど)を呼び出せる
    - プラットフォーム固有の機能を呼び出すための機構を `Plugin` と呼んでいる
- UIの構築するあらゆる要素は `Widget` として表現される
  - Widgetツリーという構造
    - Widgetの下層に別のWidget(子Widget)を渡し、さらの孫WidgetにもWidgetを渡していくという流れで、アプリ全体のレイアウトが管理されている
    - 厳密にはツリー構造の本体はWidgetから生成されるElementというオブジェクト
      - FlutterはElementを上手く隠蔽して開発者がWidgetに集中できるようにしてくれている

## Flutter SDKのインストール

[こちら](https://docs.flutter.dev/get-started/install/macos)から最新版をDLてPathを通す
or homebrewで入れる

```
cd ~/development
unzip ~/Downloads/flutter_macos_arm64_3.0.5-stable.zip
```

or

```
brew install flutter
```

インストール完了したらFluter Dockerで現状確認
環境構築のために不足してくれる部分を指摘してくれる
現時点でAndroid SDKは対応していなかったり

```
🕙 22:52✗ flutter doctor
Running "flutter pub get" in flutter_tools...                       9.8s
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.0.5, on macOS 12.4 21F79 darwin-arm, locale ja-JP)
[✗] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from: https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK components.
      (or visit https://flutter.dev/docs/get-started/install/macos#android-setup for detailed instructions).
      If the Android SDK has been installed to a custom location, please use
      `flutter config --android-sdk` to update to that location.

[✗] Xcode - develop for iOS and macOS
    ✗ Xcode installation is incomplete; a full installation is necessary for iOS development.
      Download at: https://developer.apple.com/xcode/download/
      Or install Xcode via the App Store.
      Once installed, run:
        sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
        sudo xcodebuild -runFirstLaunch
    ✗ CocoaPods not installed.
        CocoaPods is used to retrieve the iOS and macOS platform side's plugin code that responds to your plugin usage
        on the Dart side.
        Without CocoaPods, plugins will not work on iOS or macOS.
        For more info, see https://flutter.dev/platform-plugins
      To install see https://guides.cocoapods.org/using/getting-started.html#installation for instructions.
[✗] Chrome - develop for the web (Cannot find Chrome executable at /Applications/Google
    Chrome.app/Contents/MacOS/Google Chrome)
    ! Cannot find Chrome. Try setting CHROME_EXECUTABLE to a Chrome executable.
[!] Android Studio (not installed)
[✓] VS Code (version 1.68.1)
[✓] Connected device (1 available)
[✓] HTTP Host Availability
```

## Androidのセットアップ

android studioもbrewで入れる

```
brew install android-studio
```

Android Studioを起動させてセットアップウィザードを進め、再度flutter doctorを試す

```
🕙 22:57❯ flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[!] Android toolchain - develop for Android devices (Android SDK version 33.0.0)
    ✗ cmdline-tools component is missing
      Run `path/to/sdkmanager --install "cmdline-tools;latest"`
      See https://developer.android.com/studio/command-line for more details.
    ✗ Android license status unknown.
      Run `flutter doctor --android-licenses` to accept the SDK licenses.
      See https://flutter.dev/docs/get-started/install/macos#android-setup for more details.
[✓] Android Studio (version 2021.2)
```

android toolchainに注意マークが出てるので対応する
- androidのcommand-line toolが入ってないから入れろと言っているので、[こちら](https://qiita.com/ShortArrow/items/46ca3717384039419605#android-sdk%E3%81%95%E3%82%93%E3%81%AB%E9%A0%BC%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B)を参考にしてAndroidStudioのPreferenceから入れる
- 上記対応後にライセンス許諾するコマンド実行する
  ```
  flutter doctor --android-licenses
  ```

## Xcodeのインストール

- AppStoreからXcodeをダウンロードしてインストール
- Xcode付属のコマンドラインツールを最新版にしてライセンスに同意
  ```
  sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
  sudo xcodebuild -runFirstLaunch
  sudo xcodebuild -license
  ```
- flutter doctorで確認
  ```
  [!] Xcode - develop for iOS and macOS (Xcode 13.4.1)
    ✗ CocoaPods not installed.
        CocoaPods is used to retrieve the iOS and macOS platform side's plugin code that responds to your plugin usage
        on the Dart side.
        Without CocoaPods, plugins will not work on iOS or macOS.
        For more info, see https://flutter.dev/platform-plugins
  ```
  cocapodが無いと言われるのでbrewで入れる
  ```
  brew install cocoapods
  [✓] Xcode - develop for iOS and macOS (Xcode 13.4.1)
  ```

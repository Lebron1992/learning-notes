# How to Setup Flutter & Firebase with Multiple Flavors using the FlutterFire CLI

## Prerequisites

1. [create flavors in flutter](https://docs.flutter.dev/deployment/flavors#launching-your-app-flavors)

Assume you have already created the following flavors in your flutter project.

```bash
flutter run --flavor dev
flutter run --flavor stg
flutter run --flavor prod
```

## Installing the Firebase and FlutterFire CLI

### Firebase CLI

Install the Firebase CLI:

```bash
npm install -g firebase-tools
```

To verify that the Firebase CLI has been installed:

```bash
firebase --version
```

Login to Firebase:

```bash
firebase login
```

### FlutterFire CLI

Install the FlutterFire CLI:

```bash
dart pub global activate flutterfire_cli
```

You will see the warning message:

```bash
Warning: Pub installs executables into $HOME/.pub-cache/bin, which is not on your path.
You can fix that by adding this to your shell's config file (.zshrc, .bashrc, .bash_profile, etc.):

  export PATH="$PATH":"$HOME/.pub-cache/bin"
```

Run `export PATH="$PATH":"$HOME/.pub-cache/bin"` in your terminal to add the FlutterFire CLI to your path.

## FlutterFire Config with Multiple Flavors

create `flutterfire-config.sh` file in the root of your project:

```bash
#!/bin/bash
# Script to generate Firebase configuration files for different flavors

if [[ $# -eq 0 ]]; then
  echo "Error: No environment specified. Use 'dev', 'stg', or 'prod'."
  exit 1
fi

case $1 in
  dev)
    flutterfire config \
      --project=flutter-ship-dev \
      --out=lib/firebase_options_dev.dart \
      --ios-bundle-id=com.codewithandrea.flutterShipApp.dev \
      --ios-out=ios/flavors/dev/GoogleService-Info.plist \
      --android-package-name=com.codewithandrea.flutter_ship_app.dev \
      --android-out=android/app/src/dev/google-services.json
    ;;
  stg)
    flutterfire config \
      --project=flutter-ship-stg \
      --out=lib/firebase_options_stg.dart \
      --ios-bundle-id=com.codewithandrea.flutterShipApp.stg \
      --ios-out=ios/flavors/stg/GoogleService-Info.plist \
      --android-package-name=com.codewithandrea.flutter_ship_app.stg \
      --android-out=android/app/src/stg/google-services.json
    ;;
  prod)
    flutterfire config \
      --project=flutter-ship-prod \
      --out=lib/firebase_options_prod.dart \
      --ios-bundle-id=com.codewithandrea.flutterShipApp \
      --ios-out=ios/flavors/prod/GoogleService-Info.plist \
      --android-package-name=com.codewithandrea.flutter_ship_app \
      --android-out=android/app/src/prod/google-services.json
    ;;
  *)
    echo "Error: Invalid environment specified. Use 'dev', 'stg', or 'prod'."
    exit 1
    ;;
esac
```

update the `project`(the firebase project id) and `ios-bundle-id` and `android-package-name` with your own values.

## Running the FlutterFire Config Script for each Flavor

Before running the script, run the following command to make the script executable:

```bash
chmod +x ./flutterfire-config.sh
```

Take the `dev` flavor as an example:

```bash
./flutterfire-config.sh dev
```

When prompted, select `Build configuration`:

```bash
? You have to choose a configuration type. Either build configuration (most likely choice) or a target set up. ›
❯ Build configuration
  Target  
```

Then choose the `Debug-dev` build configuration:

```bash
? Please choose one of the following build configurations ›
  Debug                                        
  Release                                        
  Profile                                        
❯ Debug-dev                                      
  Profile-dev                                        
  Release-dev                                        
  Debug-stg                                        
  Profile-stg                                        
  Release-stg                                        
  Debug-prod                                        
  Profile-prod                                        
  Release-prod
```

Next, choose the platforms you want to configure:

```bash
? Which platforms should your configuration support (use arrow keys & space to select)? ›
✔ android                                      
✔ ios                                      
  macos                                        
  web                                      
  windows
```

This step may take a few minutes to complete. 

Done! You could config the `stg` and `prod` flavors in the same way.

## Installing the firebase_core package

Run the following command to install the core plugin:

```bash
flutter pub add firebase_core
flutter pub get
```

## Initializing Firebase 

### Option 1: Centralize the Firebase Initialization logic

Create a new file `firebase.dart` in the `lib` directory:

```dart
// firebase.dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart';
import 'package:flutter_ship_app/firebase_options_prod.dart' as prod;
import 'package:flutter_ship_app/firebase_options_stg.dart' as stg;
import 'package:flutter_ship_app/firebase_options_dev.dart' as dev;

Future<void> initializeFirebaseApp() async {
  // Determine which Firebase options to use based on the flavor
  final firebaseOptions = switch (appFlavor) {
    'prod' => prod.DefaultFirebaseOptions.currentPlatform,
    'stg' => stg.DefaultFirebaseOptions.currentPlatform,
    'dev' => dev.DefaultFirebaseOptions.currentPlatform,
    _ => throw UnsupportedError('Invalid flavor: $flavor'),
  };
  await Firebase.initializeApp(options: firebaseOptions);
}
```

In the `main.dart` file, call the `initializeFirebaseApp` function before calling `runApp`:

```dart
// main.dart
import 'firebase.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeFirebaseApp();
  runApp(const MyApp());
}
```

### Option 2: Use Multiple Entry Points

A more secure approach is to create three separate entry points: `main_dev.dart`, `main_stg.dart`, and `main_prod.dart` which look like this:

```dart
// main_dev.dart
import 'package:flutter_ship_app/firebase_options_dev.dart';
import 'main.dart';

void main() async {
  runMainApp(DefaultFirebaseOptions.currentPlatform);
}

// main.dart
void runMainApp(FirebaseOptions firebaseOptions) async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: firebaseOptions);
  runApp(const MainApp());
}
```

## Running the App

### Option 1: Terminal

```bash
flutter run --flavor dev
flutter run --flavor stg
flutter run --flavor prod
```

### Option 2: Visual Studio Code

- Open the `launch.json` file
- Add the following configurations:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
			"name": "dev",
			"request": "launch",
			"type": "dart",
			"program": "lib/main_dev.dart",
			"args": ["--flavor", "dev", "--target", "lib/main_dev.dart"]
		},
    {
      "name": "stg",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_stg.dart",
      "args": ["--flavor", "stg", "--target", "lib/main_stg.dart"]
    },
    {
      "name": "prod",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_prod.dart",
      "args": ["--flavor", "prod", "--target", "lib/main_prod.dart"]
    }
  ]
}
```

- Click on the `Run and Debug` button in the left sidebar
- Select the desired flavor
- Click on the `Run` button

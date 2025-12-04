# Project Sample Code

## 1. Architecture Overview

### Clean Architecture Layers

```
lib/
├── main.dart                           # App entry point
├── core/
│   ├── constants/                     # App-wide constants
│   ├── theme/                         # Theme configuration
│   ├── utils/                         # Helper functions
│   └── router/                        # Navigation routing
├── presentation/                      # UI Layer
│   ├── screens/
│   │   ├── auth/
│   │   │   └── sign_in_screen.dart
│   │   ├── chat/
│   │   │   ├── chat_screen.dart
│   │   │   └── widgets/
│   │   │       ├── message_composer.dart
│   │   │       └── message_bubble.dart
│   │   ├── history/
│   │   │   └── history_screen.dart
│   │   └── profile/
│   │       └── profile_screen.dart
│   └── providers/                     # Riverpod providers
│       ├── auth_provider.dart
│       └── chat_provider.dart
├── domain/                            # Business Logic Layer
│   ├── entities/
│   │   ├── user.dart
│   │   ├── conversation.dart
│   │   └── message.dart
│   ├── repositories/                  # Abstract interfaces
│   │   ├── auth_repository.dart
│   │   ├── chat_repository.dart
│   │   └── storage_repository.dart
│   └── usecases/
│       ├── send_message_usecase.dart
│       └── stream_response_usecase.dart
└── data/                              # Data Layer
    ├── models/                        # DTOs with fromJson/toJson
    │   ├── user_model.dart
    │   ├── conversation_model.dart
    │   └── message_model.dart
    ├── repositories/                  # Concrete implementations
    │   ├── auth_repository_impl.dart
    │   └── chat_repository_impl.dart
    └── datasources/
        ├── firebase_ai_datasource.dart
        └── firebase_auth_datasource.dart
```
## 2. Setup Project

### Create flutter app
```bash
  flutter create <app-name>
```

### pubspec.yaml

```yaml
name: chat_app
description: A production-ready Flutter chat app powered by Firebase AI Logic (Gemini) with streaming responses, clean architecture, and comprehensive testing.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.8.1

dependencies:
  flutter:
    sdk: flutter
  
  # Firebase Core
  firebase_core: ^4.2.1
  
  # Firebase Services
  firebase_auth: ^6.1.2
  cloud_firestore: ^6.1.0
  firebase_storage: ^13.0.4
  firebase_ai: ^3.6.0
  firebase_analytics: ^12.0.4
  firebase_crashlytics: ^5.0.5
  
  # Google Sign-In
  google_sign_in: ^7.2.0
  
  # State Management
  flutter_riverpod: ^3.0.3
  riverpod_annotation: ^3.0.3
  
  # Navigation
  go_router: ^17.0.0
  
  # UI Components
  cupertino_icons: ^1.0.8
  cached_network_image: ^3.4.1
  image_picker: ^1.2.1
  flutter_markdown_plus: ^1.0.5
  
  # Utilities
  intl: ^0.20.2
  uuid: ^4.5.2
  logger: ^2.6.2
  path_provider: ^2.1.5
  share_plus: ^12.0.1
  
  # Image Processing
  image: ^4.5.4
  mocktail: ^1.0.4

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  # Linting & Code Quality
  flutter_lints: ^5.0.0
  
  # Code Generation
  build_runner: ^2.7.1
  riverpod_generator: ^3.0.3
  
  # Testing
  fake_cloud_firestore: ^4.0.0
  firebase_auth_mocks: ^0.15.1
  integration_test:
    sdk: flutter
  
  # Coverage
  coverage: ^1.7.2

flutter:
  uses-material-design: true
  
  # Assets
  assets:
    # - assets/images/
    # - assets/icons/
  
  # Fonts (optional custom fonts)
  # fonts:
  #   - family: CustomFont
  #     fonts:
  #       - asset: assets/fonts/CustomFont-Regular.ttf
  #       - asset: assets/fonts/CustomFont-Bold.ttf
  #         weight: 700
```


### main.dart
```dart

import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/router/app_router.dart';
import 'core/theme/app_theme.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);

  // Setup Crashlytics for error reporting
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const ProviderScope(child: ChatApp()));
}

/// Root application widget
class ChatApp extends ConsumerWidget {
  const ChatApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);

    return MaterialApp.router(
      title: 'Chat App',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      themeMode: ThemeMode.system,
      routerConfig: router,
    );
  }
}


```

### core/app_router.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

import '../../presentation/providers/auth_provider.dart';
import '../../presentation/screens/auth/sign_in_screen.dart';
import '../../presentation/screens/chat/chat_screen.dart';
import '../../presentation/screens/history/history_screen.dart';
import '../../presentation/screens/profile/profile_screen.dart';

/// Main navigation shell with bottom navigation bar
class MainNavigationShell extends StatefulWidget {
  final Widget child;
  final int currentIndex;

  const MainNavigationShell({
    super.key,
    required this.child,
    required this.currentIndex,
  });

  @override
  State<MainNavigationShell> createState() => _MainNavigationShellState();
}

class _MainNavigationShellState extends State<MainNavigationShell> {
  void _onTabTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/chat');
        break;
      case 1:
        context.go('/history');
        break;
      case 2:
        context.go('/profile');
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: widget.child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: widget.currentIndex,
        onTap: (index) => _onTabTapped(index, context),
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.chat_bubble_outline),
            activeIcon: Icon(Icons.chat_bubble),
            label: 'Chat',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.history_outlined),
            activeIcon: Icon(Icons.history),
            label: 'History',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person_outline),
            activeIcon: Icon(Icons.person),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}

/// App router provider with authentication-aware routing
final appRouterProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authStateProvider);

  return GoRouter(
    initialLocation: '/chat',
    redirect: (context, state) {
      final isAuthenticated = authState.asData?.value != null;
      final isAuthRoute = state.matchedLocation == '/sign-in';

      // Redirect to sign-in if not authenticated
      if (!isAuthenticated && !isAuthRoute) {
        return '/sign-in';
      }

      // Redirect to chat if already authenticated and on sign-in page
      if (isAuthenticated && isAuthRoute) {
        return '/chat';
      }

      return null; // No redirect needed
    },
    routes: [
      // Auth routes
      GoRoute(
        path: '/sign-in',
        builder: (context, state) => const SignInScreen(),
      ),

      // Main app routes with bottom navigation
      ShellRoute(
        builder: (context, state, child) {
          final currentIndex = _getTabIndex(state.matchedLocation);
          return MainNavigationShell(currentIndex: currentIndex, child: child);
        },
        routes: [
          GoRoute(
            path: '/chat',
            builder: (context, state) => const ChatScreen(),
          ),
          GoRoute(
            path: '/history',
            builder: (context, state) => const HistoryScreen(),
          ),
          GoRoute(
            path: '/profile',
            builder: (context, state) => const ProfileScreen(),
          ),
        ],
      ),
    ],
    errorBuilder: (context, state) => Scaffold(
      body: Center(child: Text('Page not found: ${state.matchedLocation}')),
    ),
  );
});

int _getTabIndex(String location) {
  if (location.startsWith('/chat')) return 0;
  if (location.startsWith('/history')) return 1;
  if (location.startsWith('/profile')) return 2;
  return 0;
}



```


### core/theme.dart

```dart

import 'package:flutter/material.dart';

class AppTheme {
  // Brand colors
  static const Color primaryColor = Color(0xFF4285F4); // Google Blue
  static const Color secondaryColor = Color(0xFF34A853); // Google Green
  static const Color errorColor = Color(0xFFEA4335); // Google Red
  static const Color warningColor = Color(0xFFFBBC04); // Google Yellow

  // Message colors
  static const Color userMessageColor = Color(0xFF4285F4);
  static const Color assistantMessageColor = Color(0xFFF1F3F4);

  /// Light theme configuration
  static ThemeData get lightTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: primaryColor,
        brightness: Brightness.light,
        error: errorColor,
      ),
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
        scrolledUnderElevation: 2,
      ),
      cardTheme: CardThemeData(
        elevation: 1,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
      inputDecorationTheme: InputDecorationTheme(
        border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
        filled: true,
        contentPadding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 12,
        ),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
      floatingActionButtonTheme: FloatingActionButtonThemeData(
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      ),
    );
  }

  /// Dark theme configuration
  static ThemeData get darkTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: primaryColor,
        brightness: Brightness.dark,
        error: errorColor,
      ),
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
        scrolledUnderElevation: 2,
      ),
      cardTheme: CardThemeData(
        elevation: 1,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
      inputDecorationTheme: InputDecorationTheme(
        border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
        filled: true,
        contentPadding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 12,
        ),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
      floatingActionButtonTheme: FloatingActionButtonThemeData(
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      ),
    );
  }
}


```


### domain/entities



### Configure Google Sign-In

#### Android

1. Get SHA-1 certificate fingerprint:

### iOS
```
  <key>GIDClientID</key>
	<string><...>.apps.googleusercontent.com</string>
	<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleTypeRole</key>
			<string>Editor</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>com.googleusercontent.apps.<...></string>
			</array>
		</dict>
	</array>
```

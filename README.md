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
  # assets:
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


### Firestore rules

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    
    // Users collection
    match /users/{userId} {
      // Users can only read/write their own profile
      allow read: if isOwner(userId);
      
      // Allow create/update if authenticated and it's their own profile
      allow create, update: if isAuthenticated() 
        && request.auth.uid == userId;
      
      // Prevent deletion
      allow delete: if false;
    }
    
    // Conversations collection
    match /conversations/{conversationId} {
      // Users can only access their own conversations
      allow read: if isAuthenticated() 
        && resource.data.userId == request.auth.uid;
      
      // Allow create only if authenticated and setting own userId
      allow create: if isAuthenticated()
        && request.resource.data.userId == request.auth.uid;
      
      // Allow update only if owner
      allow update: if isAuthenticated()
        && resource.data.userId == request.auth.uid;
      
      // Allow delete only if owner
      allow delete: if isAuthenticated()
        && resource.data.userId == request.auth.uid;
      
      // Messages subcollection - SIMPLIFIED RULES
      match /messages/{messageId} {
        // Users can read messages from their own conversations
        allow read: if isAuthenticated()
          && get(/databases/$(database)/documents/conversations/$(conversationId)).data.userId == request.auth.uid;
        
        // Allow create if parent conversation belongs to user
        allow create: if isAuthenticated()
          && get(/databases/$(database)/documents/conversations/$(conversationId)).data.userId == request.auth.uid;
        
        // Allow update if parent conversation belongs to user
        allow update: if isAuthenticated()
          && get(/databases/$(database)/documents/conversations/$(conversationId)).data.userId == request.auth.uid;
        
        // Prevent deletion
        allow delete: if false;
      }
    }
  }
}
```

## domain/

### domain/entities/conversation.dart

```dart
class Conversation {
  final String id;
  final String userId;
  final String title;
  final DateTime createdAt;
  final DateTime updatedAt;
  final int messageCount;

  const Conversation({
    required this.id,
    required this.userId,
    required this.title,
    required this.createdAt,
    required this.updatedAt,
    this.messageCount = 0,
  });

  Conversation copyWith({
    String? id,
    String? userId,
    String? title,
    DateTime? createdAt,
    DateTime? updatedAt,
    int? messageCount,
  }) {
    return Conversation(
      id: id ?? this.id,
      userId: userId ?? this.userId,
      title: title ?? this.title,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
      messageCount: messageCount ?? this.messageCount,
    );
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Conversation &&
          runtimeType == other.runtimeType &&
          id == other.id;

  @override
  int get hashCode => id.hashCode;
}

```

### domain/entities/message.dart
```dart
/// Message role (user or AI assistant)
enum MessageRole {
  user,
  assistant;

  String toJson() => name;

  static MessageRole fromJson(String json) {
    return MessageRole.values.firstWhere((e) => e.name == json);
  }
}

/// Message status for tracking delivery
enum MessageStatus {
  sending,
  sent,
  error;

  String toJson() => name;

  static MessageStatus fromJson(String json) {
    return MessageStatus.values.firstWhere((e) => e.name == json);
  }
}

/// Domain entity representing a chat message
class Message {
  final String id;
  final String conversationId;
  final MessageRole role;
  final String content;
  final List<String> attachments; // Cloud Storage URLs
  final MessageStatus status;
  final DateTime createdAt;
  final bool isStreaming; // True while receiving chunks from AI

  const Message({
    required this.id,
    required this.conversationId,
    required this.role,
    required this.content,
    this.attachments = const [],
    this.status = MessageStatus.sent,
    required this.createdAt,
    this.isStreaming = false,
  });

  Message copyWith({
    String? id,
    String? conversationId,
    MessageRole? role,
    String? content,
    List<String>? attachments,
    MessageStatus? status,
    DateTime? createdAt,
    bool? isStreaming,
  }) {
    return Message(
      id: id ?? this.id,
      conversationId: conversationId ?? this.conversationId,
      role: role ?? this.role,
      content: content ?? this.content,
      attachments: attachments ?? this.attachments,
      status: status ?? this.status,
      createdAt: createdAt ?? this.createdAt,
      isStreaming: isStreaming ?? this.isStreaming,
    );
  }

  /// Helper to append content during streaming
  Message appendContent(String chunk) {
    return copyWith(content: content + chunk, isStreaming: true);
  }

  /// Mark streaming as complete
  Message completeStreaming() {
    return copyWith(isStreaming: false, status: MessageStatus.sent);
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Message && runtimeType == other.runtimeType && id == other.id;

  @override
  int get hashCode => id.hashCode;
}

```


### domain/entities/user.dart

```dart
// lib/domain/entities/user.dart

/// Domain entity representing an authenticated user
class User {
  final String uid;
  final String email;
  final String displayName;
  final String? photoURL;

  const User({
    required this.uid,
    required this.email,
    required this.displayName,
    this.photoURL,
  });

  User copyWith({
    String? uid,
    String? email,
    String? displayName,
    String? photoURL,
  }) {
    return User(
      uid: uid ?? this.uid,
      email: email ?? this.email,
      displayName: displayName ?? this.displayName,
      photoURL: photoURL ?? this.photoURL,
    );
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is User && runtimeType == other.runtimeType && uid == other.uid;

  @override
  int get hashCode => uid.hashCode;
}

```

### domain/repositories/auth_repository.dart

```dart
import '../entities/user.dart';

abstract class AuthRepository {
  /// Stream of current authentication state
  Stream<User?> get authStateChanges;

  /// Get current authenticated user
  User? get currentUser;

  /// Sign in with Google OAuth
  Future<User> signInWithGoogle();

  /// Sign out current user
  Future<void> signOut();
}


```


### domain/repositories/chat_repository.dart

```dart
import '../entities/conversation.dart';
import '../entities/message.dart';

/// Abstract repository for chat operations
abstract class ChatRepository {
  /// Create a new conversation
  Future<Conversation> createConversation({
    required String userId,
    String title = 'New Chat',
  });

  /// Get all conversations for a user
  Stream<List<Conversation>> watchConversations(String userId);

  /// Get a single conversation
  Future<Conversation?> getConversation(String conversationId);

  /// Update conversation (e.g., rename)
  Future<void> updateConversation(Conversation conversation);

  /// Delete a conversation and all its messages
  Future<void> deleteConversation(String conversationId);

  /// Send a user message and get streaming response from AI
  /// Returns a stream of message chunks as they arrive
  Stream<Message> sendMessageWithStreaming({
    required String conversationId,
    required String content,
    List<String> attachments = const [],
  });

  /// Get messages for a conversation
  Stream<List<Message>> watchMessages(String conversationId);

  /// Save a message to Firestore
  Future<void> saveMessage(Message message);

  /// Rename a conversation
  Future<void> renameConversation(String conversationId, String newTitle);
}

```

### domain/repositories/storage_repository.dart


```dart
import 'dart:io';

/// Abstract repository for cloud storage operations
abstract class StorageRepository {
  /// Upload user avatar
  Future<String> uploadAvatar({required String userId, required File file});

  /// Upload chat attachment (image)
  Future<String> uploadAttachment({
    required String conversationId,
    required File file,
  });

  /// Delete a file from storage
  Future<void> deleteFile(String url);
}

```

## data/

### data/models/conversation_model.dart

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/conversation.dart';

/// Data transfer object for Conversation with Firestore serialization
class ConversationModel extends Conversation {
  const ConversationModel({
    required super.id,
    required super.userId,
    required super.title,
    required super.createdAt,
    required super.updatedAt,
    super.messageCount,
  });

  /// Create from domain entity
  factory ConversationModel.fromEntity(Conversation conversation) {
    return ConversationModel(
      id: conversation.id,
      userId: conversation.userId,
      title: conversation.title,
      createdAt: conversation.createdAt,
      updatedAt: conversation.updatedAt,
      messageCount: conversation.messageCount,
    );
  }

  /// From Firestore → DTO
  factory ConversationModel.fromFirestore(
    DocumentSnapshot<Map<String, dynamic>> doc,
  ) {
    final data = doc.data() ?? {};

    return ConversationModel(
      id: doc.id,
      userId: data['userId'] as String? ?? '',
      title: data['title'] as String? ?? 'New Chat',
      createdAt: (data['createdAt'] as Timestamp?)?.toDate() ?? DateTime.now(),
      updatedAt: (data['updatedAt'] as Timestamp?)?.toDate() ?? DateTime.now(),
      messageCount: data['messageCount'] as int? ?? 0,
    );
  }

  /// DTO → Firestore
  Map<String, dynamic> toFirestore() {
    return {
      'userId': userId,
      'title': title,
      'createdAt': Timestamp.fromDate(createdAt),
      'updatedAt': Timestamp.fromDate(updatedAt),
      'messageCount': messageCount,
    };
  }

  /// Convert to domain entity
  Conversation toEntity() {
    return Conversation(
      id: id,
      userId: userId,
      title: title,
      createdAt: createdAt,
      updatedAt: updatedAt,
      messageCount: messageCount,
    );
  }
}

```

### data/models/message_model.dart

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/message.dart';

/// Data transfer object for Message with Firestore serialization
class MessageModel extends Message {
  const MessageModel({
    required super.id,
    required super.conversationId,
    required super.role,
    required super.content,
    super.attachments,
    super.status,
    required super.createdAt,
    super.isStreaming,
  });

  /// Create from domain entity
  factory MessageModel.fromEntity(Message message) {
    return MessageModel(
      id: message.id,
      conversationId: message.conversationId,
      role: message.role,
      content: message.content,
      attachments: message.attachments,
      status: message.status,
      createdAt: message.createdAt,
      isStreaming: message.isStreaming,
    );
  }

  /// Create from Firestore document
  factory MessageModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return MessageModel(
      id: doc.id,
      conversationId: data['conversationId'] as String,
      role: MessageRole.fromJson(data['role'] as String),
      content: data['content'] as String,
      attachments: List<String>.from(data['attachments'] ?? []),
      status: MessageStatus.fromJson(data['status'] as String? ?? 'sent'),
      createdAt: (data['createdAt'] as Timestamp).toDate(),
      isStreaming: false, // Never streaming when loaded from Firestore
    );
  }

  /// Convert to Firestore document
  Map<String, dynamic> toFirestore() {
    return {
      'conversationId': conversationId,
      'role': role.toJson(),
      'content': content,
      'attachments': attachments,
      'status': status.toJson(),
      'createdAt': Timestamp.fromDate(createdAt),
    };
  }

  /// Convert to domain entity
  Message toEntity() {
    return Message(
      id: id,
      conversationId: conversationId,
      role: role,
      content: content,
      attachments: attachments,
      status: status,
      createdAt: createdAt,
      isStreaming: isStreaming,
    );
  }
}
```

### data/model/user_model.dart
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/user.dart';

/// Data transfer object for User with Firestore serialization
class UserModel extends User {
  const UserModel({
    required super.uid,
    required super.email,
    required super.displayName,
    super.photoURL,
  });

  /// Create from domain entity
  factory UserModel.fromEntity(User user) {
    return UserModel(
      uid: user.uid,
      email: user.email,
      displayName: user.displayName,
      photoURL: user.photoURL,
    );
  }

  /// Create from Firestore document
  factory UserModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return UserModel(
      uid: data['uid'] as String,
      email: data['email'] as String,
      displayName: data['displayName'] as String,
      photoURL: data['photoURL'] as String?,
    );
  }

  /// Convert to Firestore document
  Map<String, dynamic> toFirestore() {
    return {
      'uid': uid,
      'email': email,
      'displayName': displayName,
      'photoURL': photoURL,
      'createdAt': FieldValue.serverTimestamp(),
    };
  }

  /// Convert to domain entity
  User toEntity() {
    return User(
      uid: uid,
      email: email,
      displayName: displayName,
      photoURL: photoURL,
    );
  }
}

```

### data/datasources/firebase_ai_datasource.dart

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_ai/firebase_ai.dart';

/// Configuration for Gemini model
class GeminiConfig {
  static const String defaultModel = 'gemini-2.5-flash';

  static GenerationConfig get generationConfig => GenerationConfig(
    temperature: 0.7, // Balanced creativity
    topK: 40,
    topP: 0.95,
    maxOutputTokens: 2048, // Limit response length
  );

  static List<SafetySetting> get safetySettings => [
    SafetySetting(HarmCategory.harassment, HarmBlockThreshold.medium, null),
    SafetySetting(HarmCategory.hateSpeech, HarmBlockThreshold.medium, null),
    SafetySetting(
      HarmCategory.sexuallyExplicit,
      HarmBlockThreshold.medium,
      null,
    ),
    SafetySetting(
      HarmCategory.dangerousContent,
      HarmBlockThreshold.medium,
      null,
    ),
  ];
}

/// Data source for Firebase AI Logic (Gemini) operations
class FirebaseAiDataSource {
  final GenerativeModel _model;

  FirebaseAiDataSource({GenerativeModel? model})
    : _model =
          model ??
          FirebaseAI.googleAI(auth: FirebaseAuth.instance).generativeModel(
            model: GeminiConfig.defaultModel,
            generationConfig: GeminiConfig.generationConfig,
            safetySettings: GeminiConfig.safetySettings,
          );

  /// Generate streaming response from Gemini
  /// Returns a stream of text chunks as they arrive
  Stream<String> generateStreamingResponse({
    required String prompt,
    List<String>?
    conversationHistory, // Optional: previous messages for context
  }) async* {
    try {
      // Build content with conversation history if provided
      final content = _buildContent(prompt, conversationHistory);

      // Start streaming response
      final stream = _model.generateContentStream([content]);

      // Yield each text chunk as it arrives
      await for (final chunk in stream) {
        final text = chunk.text;
        if (text != null && text.isNotEmpty) {
          yield text;
        }
      }
    } catch (e) {
      // Handle API errors gracefully
      if (e.toString().contains('quota')) {
        throw GeminiQuotaExceededException(
          'API quota exceeded. Please try again later.',
        );
      } else if (e.toString().contains('safety')) {
        throw GeminiSafetyException('Response blocked by safety filters.');
      } else {
        throw GeminiApiException('Failed to generate response: $e');
      }
    }
  }

  /// Generate complete response (non-streaming) - for simple queries
  Future<String> generateResponse({
    required String prompt,
    List<String>? conversationHistory,
  }) async {
    try {
      final content = _buildContent(prompt, conversationHistory);
      final response = await _model.generateContent([content]);

      return response.text ?? '';
    } catch (e) {
      if (e.toString().contains('quota')) {
        throw GeminiQuotaExceededException(
          'API quota exceeded. Please try again later.',
        );
      } else if (e.toString().contains('safety')) {
        throw GeminiSafetyException('Response blocked by safety filters.');
      } else {
        throw GeminiApiException('Failed to generate response: $e');
      }
    }
  }

  /// Build content with optional conversation history for context
  Content _buildContent(String prompt, List<String>? history) {
    // For now, just send the current prompt
    // TODO: Add conversation history for multi-turn context
    return Content.text(prompt);
  }
}

/// Custom exceptions for Gemini API errors
class GeminiApiException implements Exception {
  final String message;
  GeminiApiException(this.message);

  @override
  String toString() => message;
}

class GeminiQuotaExceededException extends GeminiApiException {
  GeminiQuotaExceededException(super.message);
}

class GeminiSafetyException extends GeminiApiException {
  GeminiSafetyException(super.message);
}

/// Provider for FirebaseAiDataSource
final firebaseAiDataSourceProvider = Provider<FirebaseAiDataSource>((ref) {
  return FirebaseAiDataSource();
});

```

### data/datasources/firebase_auth_datasource.dart

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart' as firebase_auth;
import 'package:google_sign_in/google_sign_in.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../domain/entities/user.dart';
import '../models/user_model.dart';

/// Provider for FirebaseAuth instance
final firebaseAuthProvider = Provider<firebase_auth.FirebaseAuth>((ref) {
  return firebase_auth.FirebaseAuth.instance;
});

/// Provider for GoogleSignIn instance
final googleSignInProvider = Provider<GoogleSignIn>((ref) {
  return GoogleSignIn.instance;
});

/// Provider for Firestore instance
final firestoreProvider = Provider<FirebaseFirestore>((ref) {
  return FirebaseFirestore.instance;
});

/// Data source for Firebase Authentication operations
class FirebaseAuthDataSource {
  final firebase_auth.FirebaseAuth _firebaseAuth;
  final GoogleSignIn _googleSignIn;
  final FirebaseFirestore _firestore;

  FirebaseAuthDataSource({
    required firebase_auth.FirebaseAuth firebaseAuth,
    required GoogleSignIn googleSignIn,
    required FirebaseFirestore firestore,
  }) : _firebaseAuth = firebaseAuth,
       _googleSignIn = googleSignIn,
       _firestore = firestore;

  /// Stream of authentication state changes
  Stream<User?> get authStateChanges {
    return _firebaseAuth.authStateChanges().map((firebaseUser) {
      if (firebaseUser == null) return null;
      return User(
        uid: firebaseUser.uid,
        email: firebaseUser.email ?? '',
        displayName: firebaseUser.displayName ?? 'Anonymous',
        photoURL: firebaseUser.photoURL,
      );
    });
  }

  /// Get current authenticated user
  User? get currentUser {
    final firebaseUser = _firebaseAuth.currentUser;
    if (firebaseUser == null) return null;

    return User(
      uid: firebaseUser.uid,
      email: firebaseUser.email ?? '',
      displayName: firebaseUser.displayName ?? 'Anonymous',
      photoURL: firebaseUser.photoURL,
    );
  }

  /// Sign in with Google OAuth
  Future<User> signInWithGoogle() async {
    try {
      await _googleSignIn.initialize(
        serverClientId:
            "691293837772-o73gk47cg30rp91reh13rreilcl6eo65.apps.googleusercontent.com",
      );
      final GoogleSignInAccount googleUser = await _googleSignIn.authenticate();
      final idToken = googleUser.authentication.idToken;
      final authorizationClient = googleUser.authorizationClient;
      GoogleSignInClientAuthorization? authorization = await authorizationClient
          .authorizationForScopes(['email', 'profile']);
      final accessToken = authorization?.accessToken;

      final credential = firebase_auth.GoogleAuthProvider.credential(
        accessToken: accessToken,
        idToken: idToken,
      );

      // Sign in to Firebase with the Google credential
      final userCredential = await _firebaseAuth.signInWithCredential(
        credential,
      );

      final firebaseUser = userCredential.user;
      if (firebaseUser == null) {
        throw Exception('Failed to sign in with Google');
      }

      // Create user entity
      final user = User(
        uid: firebaseUser.uid,
        email: firebaseUser.email ?? '',
        displayName: firebaseUser.displayName ?? 'Anonymous',
        photoURL: firebaseUser.photoURL,
      );

      // Save/update user profile in Firestore
      await _saveUserProfile(user);

      return user;
    } catch (e) {
      throw Exception('Google Sign-In failed: $e');
    }
  }

  /// Sign out current user
  Future<void> signOut() async {
    await Future.wait([_firebaseAuth.signOut(), _googleSignIn.signOut()]);
  }

  /// Save or update user profile in Firestore
  Future<void> _saveUserProfile(User user) async {
    final userModel = UserModel.fromEntity(user);
    final userDoc = _firestore.collection('users').doc(user.uid);

    // Check if user document exists
    final docSnapshot = await userDoc.get();

    if (docSnapshot.exists) {
      // Update existing user (preserve createdAt)
      await userDoc.update({
        'email': user.email,
        'displayName': user.displayName,
        'photoURL': user.photoURL,
      });
    } else {
      // Create new user document
      await userDoc.set(userModel.toFirestore());
    }
  }
}

/// Provider for FirebaseAuthDataSource
final firebaseAuthDataSourceProvider = Provider<FirebaseAuthDataSource>((ref) {
  return FirebaseAuthDataSource(
    firebaseAuth: ref.watch(firebaseAuthProvider),
    googleSignIn: ref.watch(googleSignInProvider),
    firestore: ref.watch(firestoreProvider),
  );
});
```

### data/repositories/auth_repository_impl.dart

```dart
import '../../domain/entities/user.dart';
import '../../domain/repositories/auth_repository.dart';
import '../datasources/firebase_auth_datasource.dart';

/// Implementation of AuthRepository using Firebase Auth
class AuthRepositoryImpl implements AuthRepository {
  final FirebaseAuthDataSource _dataSource;

  AuthRepositoryImpl({required FirebaseAuthDataSource dataSource})
    : _dataSource = dataSource;

  @override
  Stream<User?> get authStateChanges => _dataSource.authStateChanges;

  @override
  User? get currentUser => _dataSource.currentUser;

  @override
  Future<User> signInWithGoogle() => _dataSource.signInWithGoogle();

  @override
  Future<void> signOut() => _dataSource.signOut();
}
```


### data/repositories/chat_repository_impl.dart

```dart
import 'package:chat_app/data/datasources/firebase_auth_datasource.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:uuid/uuid.dart';

import '../../domain/entities/conversation.dart';
import '../../domain/entities/message.dart';
import '../../domain/repositories/chat_repository.dart';
import '../datasources/firebase_ai_datasource.dart';
import '../models/conversation_model.dart';
import '../models/message_model.dart';

class ChatRepositoryImpl implements ChatRepository {
  final FirebaseFirestore _firestore;
  final FirebaseAiDataSource _aiDataSource;
  final Uuid _uuid;

  ChatRepositoryImpl({
    required FirebaseFirestore firestore,
    required FirebaseAiDataSource aiDataSource,
    Uuid? uuid,
  }) : _firestore = firestore,
       _aiDataSource = aiDataSource,
       _uuid = uuid ?? const Uuid();

  @override
  Future<Conversation> createConversation({
    required String userId,
    String title = 'New Chat',
  }) async {
    final now = DateTime.now();
    final docRef = _firestore.collection('conversations').doc();

    final conversation = Conversation(
      id: docRef.id,
      userId: userId,
      title: title,
      createdAt: now,
      updatedAt: now,
      messageCount: 0,
    );

    await docRef.set(
      ConversationModel.fromEntity(conversation).toFirestore()
        ..addAll({'updatedAt': FieldValue.serverTimestamp()}),
    );

    return conversation;
  }

  @override
  Stream<List<Conversation>> watchConversations(String userId) {
    return _firestore
        .collection('conversations')
        .where('userId', isEqualTo: userId)
        .orderBy('updatedAt', descending: true)
        .snapshots()
        .map(
          (snapshot) => snapshot.docs
              .map((doc) => ConversationModel.fromFirestore(doc).toEntity())
              .toList(),
        );
  }

  @override
  Future<Conversation?> getConversation(String conversationId) async {
    final doc = await _firestore
        .collection('conversations')
        .doc(conversationId)
        .get();
    if (!doc.exists) return null;
    return ConversationModel.fromFirestore(doc).toEntity();
  }

  @override
  Future<void> updateConversation(Conversation conversation) async {
    await _firestore
        .collection('conversations')
        .doc(conversation.id)
        .update(
          ConversationModel.fromEntity(conversation).toFirestore()
            ..addAll({'updatedAt': FieldValue.serverTimestamp()}),
        );
  }

  @override
  Future<void> deleteConversation(String conversationId) async {
    try {
      // Use a batch to delete the conversation and all its messages
      final batch = _firestore.batch();

      // Delete all messages in this conversation
      final messagesSnapshot = await _firestore
          .collection('conversations')
          .doc(conversationId)
          .collection('messages')
          .get();

      for (final doc in messagesSnapshot.docs) {
        batch.delete(doc.reference);
      }

      // Delete the conversation document
      batch.delete(_firestore.collection('conversations').doc(conversationId));

      await batch.commit();
    } catch (e) {
      throw Exception('Failed to delete conversation: $e');
    }
  }

  // ---------------------------------------------------------------------------
  // Stream Assistant Response
  // ---------------------------------------------------------------------------
  @override
  Stream<Message> sendMessageWithStreaming({
    required String conversationId,
    required String content,
    List<String> attachments = const [],
  }) async* {
    // ---------------- USER MESSAGE ----------------
    final userMsgId = _uuid.v4();
    final userMessage = Message(
      id: userMsgId,
      conversationId: conversationId,
      role: MessageRole.user,
      content: content,
      attachments: attachments,
      status: MessageStatus.sent,
      createdAt: DateTime.now(),
    );

    await saveMessage(userMessage);
    yield userMessage;

    // ---------------- UPDATE TITLE IF FIRST MESSAGE ----------------
    final convoRef = _firestore.collection('conversations').doc(conversationId);
    final convoSnap = await convoRef.get();
    if (convoSnap.exists) {
      final convoData = convoSnap.data()!;
      final currentTitle = convoData['title'] as String? ?? 'New Chat';

      if (currentTitle == 'New Chat') {
        // Use first line of user message (max 50 chars) as title
        final snippet = content.trim().split('\n').first;
        final newTitle = snippet.length > 50
            ? snippet.substring(0, 50)
            : snippet;

        await convoRef.update({'title': newTitle});
      }
    }

    // ---------------- ASSISTANT MESSAGE ----------------
    final assistantId = _uuid.v4();
    var assistant = Message(
      id: assistantId,
      conversationId: conversationId,
      role: MessageRole.assistant,
      content: '',
      status: MessageStatus.sending,
      createdAt: DateTime.now(),
      isStreaming: true,
    );

    yield assistant;

    try {
      final stream = _aiDataSource.generateStreamingResponse(prompt: content);

      await for (final chunk in stream) {
        assistant = assistant.appendContent(chunk);
        yield assistant;
      }

      assistant = assistant.completeStreaming();
      await saveMessage(assistant);
      yield assistant;

      // Update conversation metadata
      await convoRef.update({
        'updatedAt': FieldValue.serverTimestamp(),
        'messageCount': FieldValue.increment(2), // user + assistant
      });
    } catch (e) {
      assistant = assistant.copyWith(
        status: MessageStatus.error,
        isStreaming: false,
        content: assistant.content.isEmpty ? 'Error: $e' : assistant.content,
      );

      await saveMessage(assistant);
      yield assistant;
    }
  }

  // ---------------------------------------------------------------------------
  // Watch Messages in a Conversation
  // ---------------------------------------------------------------------------
  @override
  Stream<List<Message>> watchMessages(String conversationId) {
    return _firestore
        .collection('conversations')
        .doc(conversationId)
        .collection('messages')
        .orderBy('createdAt')
        .snapshots()
        .map(
          (snapshot) => snapshot.docs
              .map((doc) => MessageModel.fromFirestore(doc).toEntity())
              .toList(),
        );
  }

  // ---------------------------------------------------------------------------
  // Save Message
  // ---------------------------------------------------------------------------
  @override
  Future<void> saveMessage(Message message) async {
    final ref = _firestore
        .collection('conversations')
        .doc(message.conversationId)
        .collection('messages')
        .doc(message.id);

    await ref.set(MessageModel.fromEntity(message).toFirestore());
  }

  @override
  Future<void> renameConversation(
    String conversationId,
    String newTitle,
  ) async {
    try {
      await _firestore.collection('conversations').doc(conversationId).update({
        'title': newTitle,
        'updatedAt': FieldValue.serverTimestamp(),
      });
    } catch (e) {
      throw Exception('Failed to rename conversation: $e');
    }
  }
}

final chatRepositoryProvider = Provider<ChatRepository>((ref) {
  return ChatRepositoryImpl(
    firestore: ref.watch(firestoreProvider),
    aiDataSource: ref.watch(firebaseAiDataSourceProvider),
  );
});

```

## presentation

### presentation/providers/auth_provider.dart

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../data/datasources/firebase_auth_datasource.dart';
import '../../data/repositories/auth_repository_impl.dart';
import '../../domain/entities/user.dart';
import '../../domain/repositories/auth_repository.dart';

/// Repository Provider
final authRepositoryProvider = Provider<AuthRepository>((ref) {
  return AuthRepositoryImpl(
    dataSource: ref.watch(firebaseAuthDataSourceProvider),
  );
});

/// Auth State Stream (Firebase Auth state listener)
final authStateProvider = StreamProvider<User?>((ref) {
  final repository = ref.watch(authRepositoryProvider);
  return repository.authStateChanges;
});

/// Current user provider (sync access)
final currentUserProvider = Provider<User?>((ref) {
  final repository = ref.watch(authRepositoryProvider);
  return repository.currentUser;
});

class AuthNotifier extends Notifier<AuthState> {
  /// Safe repository getter (no late final needed)
  AuthRepository get _repository => ref.read(authRepositoryProvider);

  @override
  AuthState build() {
    return const AuthState.initial();
  }

  /// Sign in with Google
  Future<void> signInWithGoogle() async {
    state = const AuthState.loading();
    try {
      final user = await _repository.signInWithGoogle();

      // When successful, emit authenticated state
      state = AuthState.authenticated(user);
    } catch (e) {
      state = AuthState.error(e.toString());
    }
  }

  /// Sign out
  Future<void> signOut() async {
    state = const AuthState.loading();
    try {
      await _repository.signOut();

      // After Firebase signs out, authStateProvider will emit null
      state = const AuthState.unauthenticated();
    } catch (e) {
      state = AuthState.error(e.toString());
    }
  }
}


sealed class AuthState {
  const AuthState();

  const factory AuthState.initial() = AuthStateInitial;
  const factory AuthState.loading() = AuthStateLoading;
  const factory AuthState.authenticated(User user) = AuthStateAuthenticated;
  const factory AuthState.unauthenticated() = AuthStateUnauthenticated;
  const factory AuthState.error(String message) = AuthStateError;
}

class AuthStateInitial extends AuthState {
  const AuthStateInitial();
}

class AuthStateLoading extends AuthState {
  const AuthStateLoading();
}

class AuthStateAuthenticated extends AuthState {
  final User user;
  const AuthStateAuthenticated(this.user);
}

class AuthStateUnauthenticated extends AuthState {
  const AuthStateUnauthenticated();
}

class AuthStateError extends AuthState {
  final String message;
  const AuthStateError(this.message);
}

/// Notifier provider
final authNotifierProvider = NotifierProvider<AuthNotifier, AuthState>(
  () => AuthNotifier(),
);

```

### presentation/providers/chat_provider.dart

```dart
import 'dart:async';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../data/repositories/chat_repository_impl.dart';
import '../../domain/entities/conversation.dart';
import '../../domain/entities/message.dart';
import '../../domain/repositories/chat_repository.dart';
import 'auth_provider.dart';

/// Chat state model
class ChatState {
  final List<Message> messages;
  final bool isStreaming;
  final String? error;
  final Conversation? currentConversation;

  const ChatState({
    this.messages = const [],
    this.isStreaming = false,
    this.error,
    this.currentConversation,
  });

  ChatState copyWith({
    List<Message>? messages,
    bool? isStreaming,
    String? error,
    Conversation? currentConversation,
  }) {
    return ChatState(
      messages: messages ?? this.messages,
      isStreaming: isStreaming ?? this.isStreaming,
      error: error,
      currentConversation: currentConversation ?? this.currentConversation,
    );
  }
}

class ChatNotifier extends Notifier<ChatState> {
  ChatRepository get _repo => ref.watch(chatRepositoryProvider);

  StreamSubscription<List<Message>>? _messagesSub;

  String? get _userId => ref.read(currentUserProvider)?.uid;

  @override
  ChatState build() {
    ref.onDispose(() {
      _messagesSub?.cancel();
    });
    return const ChatState();
  }

  Future<void> loadConversation([String? conversationId]) async {
    if (_userId == null) return;

    try {
      Conversation? conversation;

      if (conversationId != null) {
        conversation = await _repo.getConversation(conversationId);
        state = state.copyWith(
          currentConversation: conversation,
          messages: [],
          error: null,
        );
        if (conversation != null) {
          _watchMessages(conversation.id);
        }
      }
    } catch (e) {
      state = state.copyWith(error: e.toString());
    }
  }

  void reset() {
    state = ChatState(
      messages: [],
      isStreaming: false,
      error: null,
      currentConversation: null,
    );
  }

  void _watchMessages(String conversationId) {
    _messagesSub?.cancel();

    _messagesSub = _repo
        .watchMessages(conversationId)
        .listen(
          (messages) {
            if (!state.isStreaming) {
              state = state.copyWith(messages: messages, error: null);
            }
          },
          onError: (err) {
            state = state.copyWith(error: err.toString());
          },
        );
  }

  /// ---------------------------------------------
  /// SEND MESSAGE + STREAM AI RESPONSE
  /// ---------------------------------------------
  Future<void> sendMessage(
    String content, {
    List<String> attachments = const [],
  }) async {
    Conversation? conversation = state.currentConversation;

    // Create a conversation only when sending the FIRST message
    if (conversation == null) {
      if (_userId == null) {
        state = state.copyWith(error: "Not logged in");
        return;
      }

      try {
        conversation = await _repo.createConversation(
          userId: _userId!,
          title: content.trim(), // First message becomes title
        );

        state = state.copyWith(currentConversation: conversation, error: null);

        _watchMessages(conversation.id);
      } catch (e) {
        state = state.copyWith(error: "Failed to create conversation: $e");
        return;
      }
    }

    // Continue sending message normally
    state = state.copyWith(isStreaming: true, error: null);

    try {
      final stream = _repo.sendMessageWithStreaming(
        conversationId: conversation.id,
        content: content,
        attachments: attachments,
      );

      await for (final msg in stream) {
        final updatedList = _patchMessage(state.messages, msg);

        state = state.copyWith(
          messages: updatedList,
          isStreaming: msg.isStreaming,
        );
      }

      state = state.copyWith(isStreaming: false);
    } catch (e) {
      state = state.copyWith(
        isStreaming: false,
        error: "Message send failed: $e",
      );
    }
  }

  /// ---------------------------------------------
  /// PATCH STREAMING MESSAGE INTO LIST
  /// ---------------------------------------------
  List<Message> _patchMessage(List<Message> messages, Message newMsg) {
    final index = messages.indexWhere((m) => m.id == newMsg.id);

    if (index >= 0) {
      final list = [...messages];
      list[index] = newMsg;
      return list;
    }

    // Add new message
    return [...messages, newMsg];
  }

  /// ---------------------------------------------
  /// CLEAR ERROR
  /// ---------------------------------------------
  void clearError() {
    state = state.copyWith(error: null);
  }
}

final chatNotifierProvider = NotifierProvider<ChatNotifier, ChatState>(
  () => ChatNotifier(),
);

/// Conversations list (history screen)
final conversationsProvider = StreamProvider<List<Conversation>>((ref) {
  final user = ref.watch(currentUserProvider);
  if (user == null) return const Stream.empty();

  final repo = ref.watch(chatRepositoryProvider);
  return repo.watchConversations(user.uid);
});

```

### presentation/screens/auth/sign_in_screen.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../providers/auth_provider.dart';

/// Sign-in screen with Google OAuth
class SignInScreen extends ConsumerWidget {
  const SignInScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authNotifierProvider);

    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // App logo/icon
              const Icon(
                Icons.chat_bubble_outline,
                size: 80,
                color: Color(0xFF4285F4),
              ),
              const SizedBox(height: 24),

              // App title
              Text(
                'Demo Chat',
                style: Theme.of(context).textTheme.headlineLarge?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 8),

              // Subtitle
              Text(
                'Powered by Firebase AI Logic',
                style: Theme.of(
                  context,
                ).textTheme.bodyMedium?.copyWith(color: Colors.grey[600]),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 48),

              // Sign in button
              authState is AuthStateLoading
                  ? const Center(child: CircularProgressIndicator())
                  : ElevatedButton.icon(
                      onPressed: () {
                        ref
                            .read(authNotifierProvider.notifier)
                            .signInWithGoogle();
                      },
                      icon: Image.asset(
                        'assets/images/google_logo.png',
                        height: 24,
                        errorBuilder: (_, __, ___) => const Icon(Icons.login),
                      ),
                      label: const Text('Sign in with Google'),
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.all(16),
                      ),
                    ),

              // Error message
              if (authState is AuthStateError) ...[
                const SizedBox(height: 16),
                Card(
                  color: Colors.red[50],
                  child: Padding(
                    padding: const EdgeInsets.all(12.0),
                    child: Row(
                      children: [
                        const Icon(Icons.error_outline, color: Colors.red),
                        const SizedBox(width: 8),
                        Expanded(
                          child: Text(
                            authState.message,
                            style: const TextStyle(color: Colors.red),
                          ),
                        ),
                      ],
                    ),
                  ),
                ),
              ],

              const SizedBox(height: 24),

              // Terms and privacy
              Text(
                'By signing in, you agree to our Terms of Service and Privacy Policy',
                style: Theme.of(
                  context,
                ).textTheme.bodySmall?.copyWith(color: Colors.grey[600]),
                textAlign: TextAlign.center,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

```

### presentation/screens/chat/chat_screen.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:image_picker/image_picker.dart';

import '../../providers/chat_provider.dart';
import 'widgets/message_bubble.dart';
import 'widgets/message_composer.dart';

/// Main chat screen with streaming message display
class ChatScreen extends ConsumerStatefulWidget {
  const ChatScreen({super.key});

  @override
  ConsumerState<ChatScreen> createState() => _ChatScreenState();
}

class _ChatScreenState extends ConsumerState<ChatScreen> {
  final ScrollController _scrollController = ScrollController();
  final TextEditingController _textController = TextEditingController();
  final ImagePicker _imagePicker = ImagePicker();

  @override
  void dispose() {
    _scrollController.dispose();
    _textController.dispose();
    super.dispose();
  }

  void _scrollToBottom() {
    if (_scrollController.hasClients) {
      _scrollController.animateTo(
        _scrollController.position.maxScrollExtent,
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeOut,
      );
    }
  }

  Future<void> _pickImage() async {
    final XFile? image = await _imagePicker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );

    if (image != null) {
      // TODO: Upload to Cloud Storage and get URL
      // For now, just show a placeholder
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Image attachments coming soon!')),
      );
    }
  }

  Future<void> _sendMessage() async {
    final text = _textController.text.trim();
    if (text.isEmpty) return;

    _textController.clear();

    await ref.read(chatNotifierProvider.notifier).sendMessage(text);

    // Scroll to bottom after sending
    WidgetsBinding.instance.addPostFrameCallback((_) {
      _scrollToBottom();
    });
  }

  @override
  Widget build(BuildContext context) {
    final chatState = ref.watch(chatNotifierProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text(chatState.currentConversation?.title ?? 'New Chat'),
        actions: [
          IconButton(
            icon: const Icon(Icons.more_vert),
            onPressed: () {
              // TODO: Show conversation options (rename, delete)
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // Messages list
          Expanded(
            child: chatState.messages.isEmpty
                ? _buildEmptyState()
                : ListView.builder(
                    controller: _scrollController,
                    padding: const EdgeInsets.all(16),
                    itemCount: chatState.messages.length,
                    itemBuilder: (context, index) {
                      final message = chatState.messages[index];
                      return MessageBubble(
                        message: message,
                        isStreaming: message.isStreaming,
                      );
                    },
                  ),
          ),

          // Error display
          if (chatState.error != null)
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.red[50],
              child: Row(
                children: [
                  const Icon(Icons.error_outline, color: Colors.red),
                  const SizedBox(width: 8),
                  Expanded(
                    child: Text(
                      chatState.error!,
                      style: const TextStyle(color: Colors.red),
                    ),
                  ),
                  TextButton(
                    onPressed: () {
                      ref.read(chatNotifierProvider.notifier).clearError();
                    },
                    child: const Text('Dismiss'),
                  ),
                ],
              ),
            ),

          // Message composer
          MessageComposer(
            controller: _textController,
            onSend: _sendMessage,
            onPickImage: _pickImage,
            isEnabled: !chatState.isStreaming,
          ),
        ],
      ),
    );
  }

  Widget _buildEmptyState() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.chat_bubble_outline, size: 64, color: Colors.grey[400]),
          const SizedBox(height: 16),
          Text(
            'Start a conversation',
            style: TextStyle(fontSize: 18, color: Colors.grey[600]),
          ),
          const SizedBox(height: 8),
          Text(
            'Ask me anything!',
            style: TextStyle(fontSize: 14, color: Colors.grey[500]),
          ),
        ],
      ),
    );
  }
}

```

 
### presentation/screens/chat/widgets/

#### message_bubble.dart


```dart
// lib/presentation/screens/chat/widgets/message_bubble.dart

import 'package:flutter/material.dart';
import 'package:flutter_markdown_plus/flutter_markdown_plus.dart';

import '../../../../domain/entities/message.dart';

/// Message bubble widget with streaming indicator
class MessageBubble extends StatelessWidget {
  final Message message;
  final bool isStreaming;

  const MessageBubble({
    super.key,
    required this.message,
    this.isStreaming = false,
  });

  @override
  Widget build(BuildContext context) {
    final isUser = message.role == MessageRole.user;

    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: isUser
            ? MainAxisAlignment.end
            : MainAxisAlignment.start,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          if (!isUser) ...[
            CircleAvatar(
              radius: 16,
              backgroundColor: Colors.grey[300],
              child: const Icon(Icons.smart_toy, size: 18),
            ),
            const SizedBox(width: 8),
          ],
          Flexible(
            child: Column(
              crossAxisAlignment: isUser
                  ? CrossAxisAlignment.end
                  : CrossAxisAlignment.start,
              children: [
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: isUser
                        ? Theme.of(context).colorScheme.primary
                        : Colors.grey[200],
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      // Message content
                      isUser
                          ? Text(
                              message.content,
                              style: TextStyle(
                                color: isUser ? Colors.white : Colors.black87,
                              ),
                            )
                          : MarkdownBody(
                              data: message.content,
                              styleSheet: MarkdownStyleSheet(
                                p: TextStyle(
                                  color: isUser ? Colors.white : Colors.black87,
                                ),
                              ),
                            ),

                      // Streaming indicator
                      if (isStreaming) ...[
                        const SizedBox(height: 8),
                        const Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            SizedBox(
                              width: 12,
                              height: 12,
                              child: CircularProgressIndicator(strokeWidth: 2),
                            ),
                            SizedBox(width: 8),
                            Text(
                              'Streaming...',
                              style: TextStyle(
                                fontSize: 12,
                                fontStyle: FontStyle.italic,
                              ),
                            ),
                          ],
                        ),
                      ],

                      // Error indicator
                      if (message.status == MessageStatus.error) ...[
                        const SizedBox(height: 8),
                        Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            Icon(
                              Icons.error_outline,
                              size: 14,
                              color: isUser ? Colors.white70 : Colors.red,
                            ),
                            const SizedBox(width: 4),
                            Text(
                              'Failed to send',
                              style: TextStyle(
                                fontSize: 12,
                                color: isUser ? Colors.white70 : Colors.red,
                              ),
                            ),
                          ],
                        ),
                      ],
                    ],
                  ),
                ),

                // Timestamp
                const SizedBox(height: 4),
                Text(
                  _formatTime(message.createdAt),
                  style: TextStyle(fontSize: 11, color: Colors.grey[600]),
                ),
              ],
            ),
          ),
          if (isUser) ...[
            const SizedBox(width: 8),
            CircleAvatar(
              radius: 16,
              backgroundColor: Theme.of(context).colorScheme.primary,
              child: const Icon(Icons.person, size: 18, color: Colors.white),
            ),
          ],
        ],
      ),
    );
  }

  String _formatTime(DateTime dateTime) {
    final now = DateTime.now();
    final difference = now.difference(dateTime);

    if (difference.inDays > 0) {
      return '${difference.inDays}d ago';
    } else if (difference.inHours > 0) {
      return '${difference.inHours}h ago';
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes}m ago';
    } else {
      return 'Just now';
    }
  }
}

```

#### message_composer.dart

```dart
// lib/presentation/screens/chat/widgets/message_composer.dart

import 'package:flutter/material.dart';

/// Message input composer with send button and attachment option
class MessageComposer extends StatelessWidget {
  final TextEditingController controller;
  final VoidCallback onSend;
  final VoidCallback onPickImage;
  final bool isEnabled;

  const MessageComposer({
    super.key,
    required this.controller,
    required this.onSend,
    required this.onPickImage,
    this.isEnabled = true,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Theme.of(context).cardColor,
        boxShadow: [
          BoxShadow(
            color: Colors.black.withValues(alpha: 0.05),
            blurRadius: 10,
            offset: const Offset(0, -2),
          ),
        ],
      ),
      child: SafeArea(
        child: Row(
          children: [
            // Attachment button
            IconButton(
              icon: const Icon(Icons.attach_file),
              onPressed: isEnabled ? onPickImage : null,
              tooltip: 'Attach image',
            ),

            // Text input
            Expanded(
              child: TextField(
                controller: controller,
                enabled: isEnabled,
                decoration: InputDecoration(
                  hintText: 'Message...',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(24),
                    borderSide: BorderSide.none,
                  ),
                  filled: true,
                  contentPadding: const EdgeInsets.symmetric(
                    horizontal: 16,
                    vertical: 8,
                  ),
                ),
                maxLines: null,
                textInputAction: TextInputAction.send,
                onSubmitted: (_) => isEnabled ? onSend() : null,
              ),
            ),

            const SizedBox(width: 8),

            // Send button
            FloatingActionButton.small(
              onPressed: isEnabled ? onSend : null,
              child: const Icon(Icons.send),
            ),
          ],
        ),
      ),
    );
  }
}

```

### presentation/screens/history/history_screen.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import 'package:intl/intl.dart';

import '../../providers/chat_provider.dart';

/// History screen showing list of past conversations
class HistoryScreen extends ConsumerWidget {
  const HistoryScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final conversationsAsync = ref.watch(conversationsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Chat History')),
      body: Builder(
        builder: (context) {
          final async = conversationsAsync;

          // Show loading ONLY when no data has been received yet
          if (async.isLoading && !async.hasValue) {
            return const Center(child: CircularProgressIndicator());
          }

          // If stream delivered data
          final conversations = async.value ?? [];

          if (conversations.isEmpty) {
            return _buildEmptyState();
          }

          return ListView.builder(
            itemCount: conversations.length,
            itemBuilder: (context, index) {
              final conversation = conversations[index];

              return Card(
                margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                child: ListTile(
                  leading: const CircleAvatar(child: Icon(Icons.chat)),
                  title: Text(
                    conversation.title,
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  subtitle: Text(
                    '${conversation.messageCount} messages • ${_formatDate(conversation.updatedAt)}',
                    style: Theme.of(context).textTheme.bodySmall,
                  ),
                  trailing: PopupMenuButton(
                    itemBuilder: (context) => [
                      const PopupMenuItem(
                        value: 'rename',
                        child: Row(
                          children: [
                            Icon(Icons.edit, size: 20),
                            SizedBox(width: 8),
                            Text('Rename'),
                          ],
                        ),
                      ),
                      const PopupMenuItem(
                        value: 'delete',
                        child: Row(
                          children: [
                            Icon(Icons.delete, size: 20, color: Colors.red),
                            SizedBox(width: 8),
                            Text('Delete', style: TextStyle(color: Colors.red)),
                          ],
                        ),
                      ),
                    ],
                    onSelected: (value) async {
                      if (value == 'rename') {
                        _showRenameDialog(context, ref, conversation);
                      } else if (value == 'delete') {
                        _showDeleteDialog(context, ref, conversation.id);
                      }
                    },
                  ),
                  onTap: () {
                    ref
                        .read(chatNotifierProvider.notifier)
                        .loadConversation(conversation.id);

                    context.go('/chat');
                  },
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(chatNotifierProvider.notifier).reset();
          context.go('/chat');
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildEmptyState() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.history, size: 64, color: Colors.grey[400]),
          const SizedBox(height: 16),
          Text(
            'No conversations yet',
            style: TextStyle(fontSize: 18, color: Colors.grey[600]),
          ),
          const SizedBox(height: 8),
          Text(
            'Start chatting to see your history',
            style: TextStyle(fontSize: 14, color: Colors.grey[500]),
          ),
        ],
      ),
    );
  }

  String _formatDate(DateTime dateTime) {
    final now = DateTime.now();
    final difference = now.difference(dateTime);

    if (difference.inDays == 0) {
      return 'Today ${DateFormat.jm().format(dateTime)}';
    } else if (difference.inDays == 1) {
      return 'Yesterday';
    } else if (difference.inDays < 7) {
      return '${difference.inDays} days ago';
    } else {
      return DateFormat.yMMMd().format(dateTime);
    }
  }

  void _showRenameDialog(BuildContext context, WidgetRef ref, conversation) {
    final controller = TextEditingController(text: conversation.title);

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Rename Conversation'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(
            labelText: 'Title',
            border: OutlineInputBorder(),
          ),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () async {
              final newTitle = controller.text.trim();
              if (newTitle.isNotEmpty) {
                // TODO: Implement rename in ChatRepository
                Navigator.pop(context);
              }
            },
            child: const Text('Save'),
          ),
        ],
      ),
    );
  }

  void _showDeleteDialog(
    BuildContext context,
    WidgetRef ref,
    String conversationId,
  ) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Delete Conversation'),
        content: const Text(
          'Are you sure? This will permanently delete this conversation and all its messages.',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () async {
              // TODO: Implement delete in ChatRepository
              Navigator.pop(context);
            },
            style: FilledButton.styleFrom(backgroundColor: Colors.red),
            child: const Text('Delete'),
          ),
        ],
      ),
    );
  }
}

```

### presentation/screens/profile/profile_screen.dart

```dart
import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../providers/auth_provider.dart';

/// Profile screen with user info and sign-out
class ProfileScreen extends ConsumerWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(authStateProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: userAsync.when(
        data: (user) {
          if (user == null) {
            return const Center(child: Text('Not signed in'));
          }

          return SingleChildScrollView(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                const SizedBox(height: 24),

                // User avatar
                Stack(
                  children: [
                    CircleAvatar(
                      radius: 60,
                      backgroundImage: user.photoURL != null
                          ? CachedNetworkImageProvider(user.photoURL!)
                          : null,
                      child: user.photoURL == null
                          ? const Icon(Icons.person, size: 60)
                          : null,
                    ),
                    Positioned(
                      bottom: 0,
                      right: 0,
                      child: CircleAvatar(
                        radius: 20,
                        backgroundColor: Theme.of(context).colorScheme.primary,
                        child: IconButton(
                          icon: const Icon(Icons.camera_alt, size: 18),
                          color: Colors.white,
                          onPressed: () {
                            // TODO: Implement avatar upload
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(
                                content: Text('Avatar upload coming soon!'),
                              ),
                            );
                          },
                        ),
                      ),
                    ),
                  ],
                ),

                const SizedBox(height: 24),

                // User name
                Text(
                  user.displayName,
                  style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
                ),

                const SizedBox(height: 8),

                // User email
                Text(
                  user.email,
                  style: Theme.of(
                    context,
                  ).textTheme.bodyLarge?.copyWith(color: Colors.grey[600]),
                ),

                const SizedBox(height: 32),

                // Profile options
                Card(
                  child: Column(
                    children: [
                      ListTile(
                        leading: const Icon(Icons.settings),
                        title: const Text('Settings'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () {
                          // TODO: Navigate to settings
                        },
                      ),
                      const Divider(height: 1),
                      ListTile(
                        leading: const Icon(Icons.help_outline),
                        title: const Text('Help & Support'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () {
                          // TODO: Navigate to help
                        },
                      ),
                      const Divider(height: 1),
                      ListTile(
                        leading: const Icon(Icons.info_outline),
                        title: const Text('About'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () {
                          _showAboutDialog(context);
                        },
                      ),
                    ],
                  ),
                ),

                const SizedBox(height: 16),

                // Sign out button
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton.icon(
                    onPressed: () {
                      _showSignOutDialog(context, ref);
                    },
                    icon: const Icon(Icons.logout),
                    label: const Text('Sign Out'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.red,
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.all(16),
                    ),
                  ),
                ),

                const SizedBox(height: 24),

                // App version
                Text(
                  'Version 1.0.0',
                  style: Theme.of(
                    context,
                  ).textTheme.bodySmall?.copyWith(color: Colors.grey[500]),
                ),
              ],
            ),
          );
        },
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(child: Text('Error: $error')),
      ),
    );
  }

  void _showSignOutDialog(BuildContext context, WidgetRef ref) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Sign Out'),
        content: const Text('Are you sure you want to sign out?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () {
              ref.read(authNotifierProvider.notifier).signOut();
              Navigator.pop(context);
            },
            style: FilledButton.styleFrom(backgroundColor: Colors.red),
            child: const Text('Sign Out'),
          ),
        ],
      ),
    );
  }

  void _showAboutDialog(BuildContext context) {
    showAboutDialog(
      context: context,
      applicationName: 'Demo Chat',
      applicationVersion: '1.0.0',
      applicationLegalese: '© 2025 Your Company\nPowered by Firebase AI Logic',
      children: [
        const SizedBox(height: 16),
        const Text(
          'A production-ready Flutter chat app with real-time streaming '
          'responses powered by Google Gemini AI.',
        ),
      ],
    );
  }
}

```

### Refactoring of code

- message_bubble.dart (_formatTime)
- states in providers






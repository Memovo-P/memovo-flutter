# Memovo Flutter Initialization Specification

The project should be initialized as a **native Android+iOS Flutter application** using feature-first clean architecture, Bloc/Cubit state management, `go_router` navigation, and backend-ready repository interfaces. No API endpoints, localization, `.env` configuration, or final design system will be assumed at this stage.

## 1. Final package list

Use the latest versions compatible with the selected stable Flutter/Dart release. Do not hardcode package versions until the project is created and dependency compatibility is checked.

### Core architecture

| Package | Purpose | Status |
| --- | --- | --- |
| `flutter_bloc` | Bloc and Cubit state management | Required |
| `equatable` | Value equality for events, states, and domain objects | Required |
| `go_router` | Declarative routing, route guards, nested navigation, and deep links | Required |
| `get_it` | Lightweight dependency injection and service registration | Required |
| `freezed_annotation` | Immutable model annotations and union/sealed-state annotations | Required |
| `json_annotation` | JSON serialization annotations | Required |
| `freezed` | Generates immutable models and unions | Development only |
| `json_serializable` | Generates `fromJson` and `toJson` implementations | Development only |
| `build_runner` | Runs Freezed and JSON serialization generators | Development only |

### Networking, persistence, and security

| Package | Purpose | Status |
| --- | --- | --- |
| `dio` | REST API client, interceptors, request cancellation, upload progress, and centralized error handling | Required |
| `hydrated_bloc` | Persists selected Bloc/Cubit state, such as onboarding completion and lightweight session/UI state | Required |
| `shared_preferences` | Simple local flags and preferences | Required |
| `flutter_secure_storage` | Secure storage for access/refresh tokens or other sensitive session data | Required |
| `connectivity_plus` | Detects likely connectivity changes and informs offline UX | Required |
| `logger` | Structured development logging for capture, navigation, upload, and API diagnostics | Recommended |

`connectivity_plus` should only inform the user interface that connectivity may be unavailable. It should not be treated as proof that a request will succeed; the API request itself remains the final authority.

### Capture, files, and content editing

| Package | Purpose | Status |
| --- | --- | --- |
| `receive_sharing_intent` | Receive shared text, URLs, files, images, and videos from Android and iOS | Required for capture preparation |
| `flutter_quill` | Rich-text editor for Add Note and About This Link, including bold, italic, lists, links, and Delta JSON | Required according to wireframes |
| `file_picker` | Optional manual selection of PDFs and general files | Recommended |
| `url_launcher` | Open the original saved link in the browser or another supported application | Required |
| `flutter_svg` | Render SVG logo and interface assets when final design assets are provided | Recommended |

The share package must be wrapped behind a project-owned abstraction such as `PlatformCaptureAdapter`. The rest of the Flutter application must consume a normalized `CapturePayload`, not the package’s raw platform-specific model.

### Testing and quality

| Package | Purpose | Status |
| --- | --- | --- |
| `flutter_lints` | Baseline Dart and Flutter lint rules | Required |
| `bloc_test` | Unit testing for Bloc and Cubit behavior | Recommended |
| `mocktail` | Mocking repositories, API clients, and platform adapters in tests | Recommended |

No UI component library, responsive-layout package, icon package, authentication SDK, or `.env` package should be added at initialization. Flutter Material widgets, `MediaQuery`, `LayoutBuilder`, and built-in Material icons are sufficient until the design system arrives.

## 2. Recommended project structure

```text
lib/
├── app/
│   ├── app.dart
│   ├── app_router.dart
│   ├── app_theme.dart
│   ├── app_dependencies.dart
│   └── app_shell.dart
│
├── core/
│   ├── constants/
│   ├── errors/
│   ├── extensions/
│   ├── logging/
│   ├── network/
│   │   ├── api_client.dart
│   │   ├── api_result.dart
│   │   └── interceptors/
│   ├── platform/
│   │   ├── capture/
│   │   ├── deep_links/
│   │   └── file_handling/
│   ├── routing/
│   ├── storage/
│   ├── theme/
│   ├── utils/
│   └── widgets/
│
├── features/
│   ├── splash/
│   ├── onboarding/
│   ├── auth/
│   ├── home/
│   ├── capture/
│   ├── notes/
│   ├── library/
│   ├── memory_detail/
│   ├── search/
│   ├── assistant/
│   ├── tags/
│   ├── settings/
│   └── help_support/
│
└── main.dart
```

Each feature should follow the same internal structure:

```text
feature_name/
├── data/
│   ├── datasources/
│   ├── models/
│   └── repositories/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
└── presentation/
    ├── bloc/
    ├── pages/
    └── widgets/
```

For simple features such as splash or onboarding, a smaller structure is acceptable. For core features such as capture, library, search, and assistant, keep the full separation because these features will evolve as backend functionality becomes available.

## 3. Feature responsibilities

| Feature | Initial Flutter responsibility |
| --- | --- |
| `splash` | Display startup state and decide whether to go to onboarding, authentication, or the protected app shell |
| `onboarding` | Show the three onboarding screens, support pagination, Skip, Next, and Get Started actions, and persist completion locally |
| `auth` | Present welcome, sign in, register, verification, forgot password, and reset password screens; call backend-owned authentication interfaces |
| `home` | Display the authenticated home/library summary and provide the primary navigation shell |
| `capture` | Receive normalized shared content, show quick capture UI, accept optional context, validate attachments, and submit the capture |
| `notes` | Add and edit rich-text notes using `flutter_quill` |
| `library` | Display saved memories, filters, tabs, empty states, loading states, and pagination |
| `memory_detail` | Display title, source, summary, tags, context, original link, edit, archive, and delete actions |
| `search` | Submit natural-language queries and filters to the backend search API and render ranked results |
| `assistant` | Display the AI chat interface and linked memories returned by the backend RAG service |
| `tags` | Display and filter by tags; editing can initially remain within item/collection editing |
| `settings` | Provide profile, privacy, export, session, and application settings entry points |
| `help_support` | Provide placeholder support/help navigation until the final requirements are defined |

## 4. Navigation plan from the wireframes

### Public route group

```text
/splash
/onboarding
/auth/welcome
/auth/sign-in
/auth/register
/auth/verify
/auth/forgot-password
/auth/reset-password
```

The splash route should be the initial route. Its routing decision should be based on three conditions: whether onboarding has been completed, whether the user has a valid authenticated session, and whether a pending shared capture is waiting to be processed.

The onboarding flow should contain three pages based on the wireframes: saving anything from anywhere, finding memories instantly, and understanding notes with AI. The final page leads to Get Started, while Skip leads to authentication.

### Authentication route group

```text
/auth/welcome
/auth/sign-in
/auth/register
/auth/verify
/auth/forgot-password
/auth/reset-password
```

The backend owns the actual authentication behavior. Flutter should define authentication states such as unauthenticated, authenticating, authenticated, verification-required, password-reset-requested, and authentication-failure. The route guard should redirect unauthenticated users away from protected routes.

### Protected application route group

```text
/app/home
/app/library
/app/search
/app/assistant
/app/profile
/app/tags
/app/settings
/app/help
/app/memory/:memoryId
/app/add-note
/app/save-link
/app/capture/incoming
```

The authenticated area should use a shared application shell. The wireframes show bottom navigation for the primary destinations and a side menu for secondary destinations.

### Primary bottom navigation

The wireframes indicate these main destinations:

| Navigation position | Destination |
| --- | --- |
| Home icon | `/app/home` |
| Profile icon | `/app/profile` |
| Plus action | Opens capture options rather than navigating directly |
| Library/documents icon | `/app/library` |
| Chat icon | `/app/assistant` |

The plus action should open a modal or bottom sheet with two choices:

```text
Add content
├── Save Link → /app/save-link
└── New Note  → /app/add-note
```

An incoming share should bypass the normal plus menu and route directly to `/app/capture/incoming` after the shared payload has been normalized.

## 5. Route guard behavior

The router should have one centralized redirect policy rather than individual checks scattered across screens.

| Condition | Redirect behavior |
| --- | --- |
| App is starting | Stay on `/splash` until initialization completes |
| First launch and onboarding incomplete | Redirect to `/onboarding` |
| Onboarding complete but session absent | Redirect to `/auth/welcome` |
| Session expired | Redirect to `/auth/sign-in` while preserving the intended destination when appropriate |
| Session authenticated | Allow `/app/*` routes |
| Shared payload received while unauthenticated | Store a pending capture locally and redirect to authentication; resume capture after login |
| Shared payload received while authenticated | Navigate to `/app/capture/incoming` |
| Authenticated user opens public auth route | Redirect to `/app/home` |
| Unknown route | Show a controlled not-found/fallback page |

The pending capture case is important. If the user shares a link before signing in, the app should not discard it. The native receiver should persist a minimal pending payload locally, then the router should resume the capture flow once authentication succeeds.

## 6. Share Intent integration boundary

The initialization should prepare the following structure without implementing the complete capture feature yet:

```text
core/platform/capture/
├── capture_payload.dart
├── platform_capture_adapter.dart
├── android_capture_adapter.dart
├── ios_capture_adapter.dart
└── pending_capture_storage.dart
```

The native layer receives platform-specific content. The adapter normalizes it into one application model:

```text
Android Intent / iOS Share Extension
                ↓
      PlatformCaptureAdapter
                ↓
          CapturePayload
                ↓
        CaptureCubit/Bloc
                ↓
        /app/capture/incoming
```

The payload should support nullable URL, title, text, source platform, and a list of attachments. The initialization should account for both cold-start and warm-app share events. Actual Android manifest filters and iOS Share Extension target configuration can be completed as part of the capture feature implementation, but the project boundaries and route must exist from the beginning.

## 7. Theme initialization

Because the final design system has not been received, initialize a neutral theme foundation rather than attempting to finalize the visual identity. The theme layer should centralize color scheme, typography, spacing, shape definitions, input decoration, button styles, navigation styles, and light/dark mode placeholders.

The initial theme should avoid hardcoding styling directly inside feature widgets. Once the design system arrives, the visual update should primarily affect `app_theme.dart` and shared components rather than requiring a feature-wide rewrite.

## 8. Dependency registration boundaries

Register dependencies by abstraction, not by concrete UI usage. The initial dependency categories should be:

| Category | Examples |
| --- | --- |
| Infrastructure | Dio API client, secure storage, shared preferences, logger |
| Platform | Share receiver adapter, file handler, URL launcher abstraction |
| Data | Auth repository, capture repository, library repository, search repository, assistant repository |
| Domain | Use cases for authentication, capture, library, search, and assistant |
| Presentation | Bloc/Cubit factories and route-level dependencies |

Since backend endpoints are not available yet, repositories should be represented by interfaces and temporary placeholder implementations. The UI must not call Dio directly.

## 9. Initialization acceptance criteria

The initialization task is complete when the following conditions are true:

| Area | Acceptance criterion |
| --- | --- |
| Project | Android and iOS Flutter project opens and runs successfully |
| Architecture | Feature-first clean architecture folders exist and follow one consistent convention |
| State | Bloc/Cubit and Equatable are configured for feature state management |
| Models | Freezed and JSON serialization generation are configured for future backend models |
| Routing | `go_router` contains public, authentication, and protected route groups |
| Guards | Onboarding and authentication redirect decisions have defined locations in the architecture |
| Capture | Share receiver abstraction and incoming-capture route are prepared for Android/iOS integration |
| Theme | Centralized placeholder theme exists and can be replaced when the design system arrives |
| Packages | Required dependencies are categorized and installed without unnecessary UI or auth SDK packages |
| Backend boundary | Repository interfaces exist conceptually without assuming endpoint URLs or response implementations |
| Localization | No localization package or RTL implementation is included yet, as requested |
| Environment | No `.env` package or environment configuration is included yet, as requested |

This initialization gives the project a stable foundation for feature development while avoiding premature decisions about backend APIs, AI implementation, localization, or the final visual design system.

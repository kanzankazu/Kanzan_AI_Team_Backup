---
title: Android Code Review Rules — BRI brimod-monorepo
created: 2026-10-26
updated: 2026-10-26
type: sop
tags: [android, code-review, bri, brimod-monorepo, standards, architecture]
sources: []
confidence: high
---

# Android Code Review Rules — BRI brimod-monorepo

Comprehensive code review guidelines untuk Android projects di BRI (Kotlin, Jetpack Compose, Clean Architecture, MVI pattern).

## Quick Links
- [Part A: Core Review Rules](#part-a-core-rules) — shared dengan semua Android
- [Part B: Project-Specific (brimod-monorepo)](#part-b-project-specific) — BRI-only
- [Part C: Team Collaboration](#part-c-collaboration) — proposal mechanism

## Severity Taxonomy

| Level | Impact | Action |
|---|---|---|
| 🔴 Critical | Bug, security, crash, memory leak | MUST fix before merge |
| 🟡 Medium | Stability, perf, maintainability risk | SHOULD fix |
| 🟢 Minor | Style, code smell, dead code | NICE to fix |
| 💙 Appreciation | Good practices spotted | Acknowledge |

## Key Sections

### Verifikasi Ulang Finding
- 🔴 Critical → 3-pass verification + full file read
- 🟡 Medium → 2-pass verification + full file read
- 🟢 Minor → 1-2 pass, pattern check di codebase

### Output Requirements
- Header: tanggal, reviewer, version, file stats
- Findings table: severity, file, line, issue, fix
- Fix suggestions: BEFORE/AFTER code untuk 🔴 Critical
- Test scenarios: precondition, steps, expected results
- Ringkasan: tabel severity + checklist urutan pengerjaan

### Architecture Checklists

#### New (Compose + MVVM + Clean)
✅ ViewModel: custom scope cancel, state thread-safe, business logic → UseCase, no Context/Activity, no hardcoded delay
✅ Composable: no object without remember, LaunchedEffect key benar, recomposition minimized, collectAsStateWithLifecycle
✅ Repository: error handling konsisten, no !!, resource closed, mapping safe
✅ DI: semua UseCase provided, scope benar (Singleton vs ViewModelScoped)
✅ Navigation: NavArg match consume, no large data di NavArg

#### Old (XML + MVP/MVVM)
✅ Activity/Fragment: no memory leak inner class, onDestroy cleanup, no getActivity()!!
✅ Adapter: object creation di onBind, DiffUtil, image lifecycle-aware
✅ XML Layout: flat structure, no hardcoded values, no wrap_content di ScrollView
✅ Presenter/VM: RxJava dispose, callback after destroy, threading, LiveData with lifecycle
✅ Java: no NPE, no raw type, no deadlock, no resource leak

#### Shared
✅ UseCase: single responsibility, no side effects, suspend vs blocking
✅ Mapper: no force unwrap, sensible defaults, no duplication
✅ Gradle: versions pinned, no duplicates, version catalog, no beta in prod
✅ Strings: via strings.xml, feature prefix, no duplicate, skip values-en
✅ Analytics: naming convention, required params, tracker placement
✅ Resources: check BRIIcon/BRIIllustration first, prefix (ic_, bg_, img_, shape_, sel_)
✅ Tests: BehaviorSpec for VM, FunSpec for util, advanceUntilIdle, clearMocks, coVerify assertions

### BRI-Specific Rules

#### Design System (WAJIB pakai BRITheme)
- Color: `BRITheme.color.*`
- Typography: `BRITheme.typography.*`
- Radius: `BRITheme.radius.*`
- Screen wrapper: `BRIGeneralScreen` / `BRICollapseScreen`
- Button: `BRIButton`
- Input: `BRIForm` / `BRIFormSearch`
- Bottom sheet: `BRIBottomSheet` / `BRIBaseBottomSheet`
- Import: `id.co.bri.brimons.ds.compose.*` (NOT androidx.material3)

#### Architecture (MVI + Clean)
- Error from ErrorCode, NOT message.contains()
- ExceptionHandler: predicate-based error matching
- SESSION_END handled by GeneralApiException (don't handle manual)
- UseCase WAJIB jika ada: validation, business logic, combine repos; SKIP jika passthrough

#### DI & Hilt
- @InstallIn(SingletonComponent) → repository, navigation binding
- @HiltViewModel → ViewModel (JANGAN @Singleton)
- @Binds (bukan @Provides), interface → impl, fun internal
- JANGAN inject Activity/Fragment ke repo/use case — pakai @ApplicationContext

#### Coroutines & Threading
- JANGAN withContext(IO) di repository — EncryptedCallExecutor sudah handle
- JANGAN GlobalScope / CoroutineScope(...) di ViewModel — hanya viewModelScope
- JANGAN runBlocking di production
- CPU-intensive → withContext(Dispatchers.Default)
- JANGAN swallow CancellationException

#### Serialization
- kotlinx.serialization (bukan Gson) untuk module baru
- @Serializable WAJIB di Request, Response, Result
- JANGAN mix @SerializedName (Gson) + @SerialName (kotlinx) di class yang sama
- @Serializable di Result hanya jika travel sebagai NavArg JSON

#### Navigation & NavArgs
- Route → @Serializable data object/class (bukan string)
- Complex object → toJsonString() kirim, fromJsonString() terima
- Sheet state di Navigation file (bukan Screen)
- ≥ 3 sheet → enum, bukan Boolean flags
- Back nav via BackHandler di Navigation file
- Screen TIDAK BOLEH hold navController
- collectLatest (bukan collect), LaunchedEffect(viewModel.effect)

#### Recomposition (Zero Recompose)
- @Immutable di semua *State class
- List field → ImmutableList via toPersistentList()
- TIDAK BOLEH List/Set/Map sebagai parameter
- rememberUpdatedState(onEvent) di top Screen
- LazyColumn items: key + contentType
- mutableStateOf WAJIB wrapped remember { }
- ./gradlew qittaErangelDebugComposeCompilerCheck → exit 0

#### Performance
- JankStats.createAndTrack(window) di BaseComposeActivity.onResume() — zero JANK
- jankStats = null di onDestroy() — prevent leak
- No Activity/Context field di ViewModel
- Second build: Reusing configuration cache
- No println/file I/O di build-logic, no afterEvaluate

#### Naming Convention
| Artefact | Pattern |
|---|---|
| Error code | XxxErrorCode enum |
| Exception | XxxException (object + handler) |
| Custom exception | XxxNotFoundException |
| Validation error | XxxValidationError (sealed interface) |
| URL constants | XxxUrlConstant (internal object) |
| Request/Response/Result | XxxRequest / XxxResponse / XxxResult |
| Repository | XxxRepository / XxxRepositoryImpl |
| DI module | XxxDataModule |
| State/Event/Effect | XxxState : UiState / XxxEvent : UiEvent / XxxEffect : UiEffect |
| ViewModel/Navigation/Route | XxxViewModel / XxxNavigation.kt / @Serializable XxxRoute |
| Screen composable | fun XxxScreen(state, onEvent) — STATELESS |
| Presentation mapper | XxxValidationError.asErrorText() |
| NavGraphBuilder ext | NavGraphBuilder.xxxScreen(navController, onFinish) |

### Standards Workspace References

| Area | File |
|---|---|
| Module structure, MVI, template | `.kiro/specs/standards/architecture.md` |
| Coroutines, dispatcher, Hilt scope | `.kiro/specs/standards/coroutines-di.md` |
| BRITheme DS components | `.kiro/specs/standards/design-system.md` |
| Routes, NavArgs, back nav | `.kiro/specs/standards/navigation-navargs.md` |
| @Immutable, recomposition | `.kiro/specs/standards/recomposition-rules.md` |
| Unit test pattern | `.kiro/specs/standards/testing-rules.md` |
| Full PR checklist | `.kiro/specs/standards/review-checklist.md` |
| Jank root causes | `.kiro/specs/standards/perf-jankstats.md` |
| Memory leak pattern | `.kiro/specs/standards/perf-memory-leak.md` |
| Build performance | `.kiro/specs/standards/perf-gradle-profiler.md` |

---

## Version History
- **v2.0.0** (2026-10-26) — Initial BRI ruleset, generic + project-specific split
- Part A (Core) reusable untuk semua Android
- Part B (Project-Specific) untuk brimod-monorepo
- Part C (Collaboration) untuk AI reviewer proposals

---

## Related Pages
[[Android Review Rules — Generic Template]]
[[Kanzan AI Team — Code Review Workflow]]
[[BRI Design System Guide]]

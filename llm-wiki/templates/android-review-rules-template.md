---
title: Android Code Review Rules — [PROJECT_NAME]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: template
tags: [android, code-review, template, standards]
sources: []
confidence: high
---

# Android Code Review Rules — [PROJECT_NAME]

Generic template untuk Android code review. Copy, customize, gunakan.

## Project Metadata

| Field | Value |
|---|---|
| Project Name | [PROJECT_NAME] |
| Platform | Android (Kotlin, Jetpack Compose / XML) |
| Architecture | [MVVM / MVI / MVP] |
| Min SDK | [API XX] |
| Target SDK | [API XX] |
| Key Dependencies | [Hilt / Koin, kotlinx-serialization / Gson, Coroutines, ...] |
| Review Owner | [Name/Team] |
| Last Updated | YYYY-MM-DD |

---

## Part A: Core Review Rules (UNIVERSAL — Shared with all Android)

### Severity Taxonomy

| Level | Impact | Action |
|---|---|---|
| 🔴 Critical | Bug, security, crash, memory leak, data loss | MUST fix before merge |
| 🟡 Medium | Stability risk, perf issue, threading bug, lifecycle leak | SHOULD fix |
| 🟢 Minor | Style, code smell, dead code, naming | NICE to fix |
| 💙 Appreciation | Good practices, clever solution | Acknowledge |

### Verification Workflow

#### 🔴 Critical → 3-pass verification + full file read
1. Pass 1: Baca diff + full file. Identifikasi issue.
2. Pass 2: Telusuri caller, callee, code path. Ada guard lain? Override condition?
3. Pass 3: Re-evaluate verdict. Mitigasi terlewat?

#### 🟡 Medium → 2-pass verification + full file read
1. Pass 1: Baca diff + context. Identifikasi issue.
2. Pass 2: Verifikasi handling di file lain, parent class, base class. Re-evaluate severity.

#### 🟢 Minor → 2-pass (pattern check)
1. Pass 1: Baca diff, identifikasi issue.
2. Pass 2: Cek apakah pattern sama sudah ada di codebase. Jangan flag jika konsisten.

### Output Format (MANDATORY)

**Header:**
```
Tanggal: YYYY-MM-DD | Reviewer: [Name] | Version: v1
Files Changed: N | Insertions: +N | Deletions: -N
Commit List: {ID} - {MESSAGE}
```

**Findings Table:**
```
| # | Done | Severity | File | Line | Issue | Fix Suggestion |
|---|------|----------|------|------|-------|----------------|
| 1 | ⬜ | 🔴 | XxxFile.kt | 45 | [Issue] | [Fix 1 line] |
```

**Sections:**
- Appreciation 💙 (optional)
- Findings — Code Quality
- 🔧 Fix Suggestion (Full Code) — WAJIB untuk 🔴 Critical
- 🧪 Test Scenario — WAJIB untuk 🔴 Critical
- Ringkasan (tabel severity + checklist urutan pengerjaan)

### Architecture Checklist

#### New (Compose + [ARCHITECTURE])
✅ State: @Immutable, no mutable field
✅ ViewModel: no Context/Activity, business logic → UseCase, threading OK
✅ Composable: no object without remember, LaunchedEffect key benar, recomposition minimal
✅ Repository: error handling konsisten, no !!, resource cleanup
✅ DI: semua dependency provided, scope benar
✅ Navigation: NavArg match, no large data in args
✅ Tests: pass Green, coverage ≥ X%

#### Old (XML + [ARCHITECTURE])
✅ Activity/Fragment: no memory leak, onDestroy cleanup
✅ Adapter: object creation outside onBind, DiffUtil, lifecycle-aware
✅ Layout: flat hierarchy, no hardcoded values
✅ Presenter/VM: no swallow exception, threading safe
✅ Java: no NPE, no raw type, no resource leak

#### Shared
✅ Strings: via strings.xml, no hardcoded
✅ Resources: no duplicate, correct prefix (ic_, bg_, img_)
✅ Dependencies: versions pinned, no conflict, no beta in prod
✅ Error handling: [Custom pattern]
✅ Serialization: [Gson / kotlinx / other]
✅ DI framework: [Hilt / Koin / other]

---

## Part B: Project-Specific Rules

### 1. Design System

**Colors, Typography, Components:**
```
[ ] Warna dari [DesignSystemName].color.*
[ ] Typography dari [DesignSystemName].typography.*
[ ] Button: [ButtonComponentName]
[ ] Input: [InputComponentName]
[ ] Layout wrapper: [ScreenComponentName]
[ ] Import: from [package name] (BUKAN androidx.material3.*)
```

**If using custom DS:**
- Color palette file: [path]
- Typography spec: [path]
- Component library: [path]
- Figma link: [URL]

### 2. Architecture Pattern

**Pattern:** [MVVM / MVI / MVP / Other]

**Key principles:**
```
[ ] Principle 1: [Description]
[ ] Principle 2: [Description]
[ ] Principle 3: [Description]
```

**Naming convention for pattern:**
| Artefact | Pattern |
|---|---|
| State | [Xxx]State : [BaseState] |
| Event | [Xxx]Event : [BaseEvent] |
| Effect | [Xxx]Effect : [BaseEffect] |
| ViewModel | [Xxx]ViewModel |
| Screen | fun [Xxx]Screen() |

### 3. Dependency Injection

**Framework:** [Hilt / Koin / Other]

**Key rules:**
```
[ ] DI rule 1: [e.g., @InstallIn(SingletonComponent) untuk repository]
[ ] DI rule 2: [e.g., @HiltViewModel untuk ViewModel]
[ ] DI rule 3: [e.g., No Context injection ke repo]
```

**Scope mapping:**
| Component | Scope |
|---|---|
| Repository | Singleton |
| UseCase | Singleton |
| ViewModel | ViewModelScoped |
| [Custom] | [Custom] |

### 4. Error Handling

**Pattern:** [Exception object / Result type / ErrorCode enum]

**Key rules:**
```
[ ] Error rule 1: [e.g., dari ErrorCode enum, bukan message.contains()]
[ ] Error rule 2: [e.g., ExceptionHandler dengan predicate]
[ ] Error rule 3: [e.g., JANGAN swallow exception tanpa logging]
```

**Example:**
```kotlin
// ❌ Wrong
if (error.message?.contains("Session End") == true) { ... }

// ✅ Right
if (error is SessionEndException) { ... }
```

### 5. Threading & Coroutines

**Key rules:**
```
[ ] Threading rule 1: [e.g., JANGAN GlobalScope]
[ ] Threading rule 2: [e.g., viewModelScope untuk ViewModel]
[ ] Threading rule 3: [e.g., withContext untuk dispatcher]
[ ] Threading rule 4: [e.g., JANGAN runBlocking]
```

### 6. Navigation

**Pattern:** [String route / Type-safe Compose Navigation / Jetpack Navigation]

**Key rules:**
```
[ ] Nav rule 1: [e.g., NavArg should be serializable]
[ ] Nav rule 2: [e.g., No large data in args]
[ ] Nav rule 3: [e.g., Back navigation via BackHandler]
```

### 7. Serialization

**Framework:** [Gson / kotlinx-serialization / Other]

**Key rules:**
```
[ ] Serialization rule 1: [e.g., @Serializable di Request/Response]
[ ] Serialization rule 2: [e.g., JANGAN mix @SerializedName + @SerialName]
[ ] Serialization rule 3: [e.g., Custom serializers untuk complex types]
```

### 8. Performance & Memory

**Key rules:**
```
[ ] Perf rule 1: [e.g., No memory leak di listeners]
[ ] Perf rule 2: [e.g., Bitmap handling lifecycle-aware]
[ ] Perf rule 3: [e.g., Coroutines di heavy computation]
[ ] Perf rule 4: [e.g., Minimize recomposition di Compose]
```

### 9. Security

**Key rules:**
```
[ ] Security rule 1: [e.g., Secure storage untuk token]
[ ] Security rule 2: [e.g., HTTPS only]
[ ] Security rule 3: [e.g., Input validation]
[ ] Security rule 4: [e.g., No hardcoded API keys]
```

### 10. Testing

**Framework:** [JUnit / Kotest / Spek / Other]

**Coverage target:** [XX%]

**Key rules:**
```
[ ] Test rule 1: [e.g., ViewModel → BehaviorSpec]
[ ] Test rule 2: [e.g., UseCase → FunSpec]
[ ] Test rule 3: [e.g., Mock dependencies, not APIs]
[ ] Test rule 4: [e.g., Test both happy path + error cases]
```

---

## Part C: Team Collaboration & Governance

### Review Workflow

**Before PR:**
1. Run linter & formatter
2. Run local tests — semua hijau
3. Self-review code
4. git fetch --all

**During Review:**
1. Fetch latest
2. Get diff + full file context
3. Follow checklist + verify findings per severity
4. Gunakan template comments

**After Review:**
- Diskusikan findings tenang
- Author update PR
- Re-review jika diperlukan

### Proposal Mechanism

**New findings jadi rule?** Propose di "Active Proposals" section
Format:
- Problem: [Issue description]
- Current behavior: [What happens now]
- Proposed solution: [Rule change]
- Affected sections: [Mana yang berubah]
- Example: [Code sample]
- Confidence: [HIGH / MEDIUM / LOW]

### Escalation Path

1. Question → Ask in PR comment
2. Disagreement → Discuss dengan team lead
3. Architectural decision → Refer ke architecture guide
4. Security concern → Escalate ke security owner

---

## Standards & References

**Project Standards:**
- Architecture: [path or link]
- Design System: [path or link]
- Build: [path or link]
- Security: [path or link]

**External References:**
- Android Best Practices: https://developer.android.com/guide/topics/large-screen/
- Kotlin Coroutines: https://kotlinlang.org/docs/coroutines-overview.html
- Jetpack Compose: https://developer.android.com/jetpack/compose

---

## Troubleshooting

**Review stuck?** → Set deadline + involve tech lead
**Disagreement?** → Refer to architecture guide + get 3rd opinion
**PR too large?** → Request split per feature/layer
**Quality low?** → Use template comments + pair review

---

## Version Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | YYYY-MM-DD | [Name] | Initial version |
| | | | |

---

## Related Documents

- [[Android Review Rules — BRI brimod-monorepo]] (project-specific reference)
- [[Testing Rules — [PROJECT_NAME]]]
- [[Design System — [PROJECT_NAME]]]
- [[Architecture Guide — [PROJECT_NAME]]]

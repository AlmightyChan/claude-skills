# Observation Categories

This file defines the 8 categories of issues the iterate skill proactively scans for during sessions. Observations are surfaced once with severity context, logged to ISSUES.md, and deferred to user direction.

**Behavior:** The skill does not fix observations automatically. It mentions each observation once, explains why it matters, and asks the user whether to address it now, log it for later, or dismiss it.

**Aspirational note:** These heuristics are v1 approximations. Some patterns (e.g., missing accessibility) require judgment beyond grep. Focus on grep-checkable patterns first; refine based on dogfooding.

---

## Categories

### 1. Dead Code / Stubs

**Description:** Functions that are never called, empty implementations, or placeholder code that was intended to be replaced.

**Default Severity:** medium

**Scan Heuristics:**
- `// TODO` or `// FIXME` inside function bodies (indicates placeholder)
- `fatalError("Not implemented")` or `fatalError("TODO")`
- Functions whose name appears only at the declaration site (defined but never called)
- `return nil` or `return []` as the only line in a non-trivial function
- Methods that only contain `print()` or `debugPrint()` statements

**Example Observation:**
"`Engine.runScan()` appears to be stubbed -- it sleeps for 200ms and returns without calling `generateSuggestions()`. This is the core scan function. [BUG, priority/critical]"

---

### 2. Unhandled Error Paths

**Description:** Code that silently swallows errors, uses force unwraps in non-trivial contexts, or ignores failure cases.

**Default Severity:** high

**Scan Heuristics:**
- `try?` without subsequent `nil` check or fallback handling
- `try!` in non-test, non-playground code
- `catch { }` or `catch { _ in }` (empty catch blocks)
- `catch` blocks that only contain `print()` or `debugPrint()`
- Force unwraps (`!`) on optionals from external data (network, database, user input)
- `as!` force casts outside of test code
- `.get()` on Result types without error handling

**Example Observation:**
"Paywall purchase uses `try?` which silently swallows StoreKit errors. If a purchase fails, the user sees no feedback. [BUG, priority/high]"

---

### 3. Missing Accessibility

**Description:** Custom views or interactive elements that lack accessibility labels, traits, or Dynamic Type support.

**Default Severity:** medium

**Scan Heuristics:**
- `Button` or `NavigationLink` without `.accessibilityLabel()` in the same view body
- Custom `View` structs with `onTapGesture` but no `.accessibilityAddTraits(.isButton)`
- `.font(.system(size:` with hardcoded numeric sizes (not using `.body`, `.title`, etc.)
- `Image` without `.accessibilityLabel()` (unless decorative with `.accessibilityHidden(true)`)
- `HStack` or `VStack` containing multiple text elements without `.accessibilityElement(children: .combine)`
- Fixed `.frame(width:, height:)` on text-containing views

**Example Observation:**
"SuggestionCardView uses `onTapGesture` for its primary interaction but has no `.accessibilityAddTraits(.isButton)`. Screen reader users cannot identify it as tappable. [POLISH, priority/medium]"

---

### 4. Hardcoded Values

**Description:** Literal strings, numbers, or URLs that should be configurable, localized, or extracted to constants.

**Default Severity:** medium

**Scan Heuristics:**
- URL strings containing `http://` or `https://` outside of configuration files
- Strings matching email patterns (`*@*.*`) in source code
- `"id000000000"` or similar placeholder IDs
- Numeric literals used as timeouts, retry counts, or thresholds (e.g., `sleep(3)`, `maxRetries = 5`)
- Price strings (e.g., `"$2.99"`, `"$19.99"`) hardcoded in views
- API keys or tokens as string literals (anything matching `[A-Za-z0-9]{32,}`)

**Example Observation:**
"OnboardingPaywallView has hardcoded `\"$2.99/month\"` and `\"$19.99/year\"` instead of pulling prices from SubscriptionManager. A/B pricing variants will not display correctly. [BUG, priority/medium]"

---

### 5. TODOs / FIXMEs

**Description:** Comment markers indicating incomplete or known-broken code.

**Default Severity:** low (elevated to high if in a critical code path)

**Scan Heuristics:**
- `// TODO:` or `// TODO` (case-insensitive)
- `// FIXME:` or `// FIXME` (case-insensitive)
- `// HACK:` or `// HACK` (case-insensitive)
- `// XXX:` or `// XXX`
- `// TEMP:` or `// TEMPORARY`
- Comments containing "will be connected during", "placeholder", "stub", "not yet implemented"

**Severity Elevation:** If the TODO/FIXME appears in a file listed in PROJECT.md's Key Files, or in a file with `priority/critical` or `priority/high` issues, elevate to high.

**Example Observation:**
"NotificationManager line 364 has comment: `\"This will be connected during app integration.\"` The notification action handlers are not wired to `NotificationActions.handleAction()`. [BUG, priority/critical if notifications are a core feature]"

---

### 6. Build Warnings

**Description:** Compiler warnings, deprecation notices, or analyzer warnings that indicate potential problems.

**Default Severity:** medium

**Scan Heuristics:**
- Xcode build output containing `warning:` lines
- `@available(*, deprecated` annotations in project code (not framework code)
- `#warning("` compiler directives
- Unused variable warnings: variables assigned but never read
- Unreachable code warnings
- Implicit conversion warnings

**Example Observation:**
"Build produces 3 deprecation warnings for `UIApplication.shared.keyWindow` usage. This API was deprecated in iOS 13 and may be removed in a future release. [POLISH, priority/medium]"

---

### 7. Privacy / Security Flags

**Description:** Patterns that suggest potential privacy violations, insecure storage, or missing security controls.

**Default Severity:** high

**Scan Heuristics:**
- `UserDefaults` storing values with key names containing "token", "secret", "password", "key", "credential"
- `print()` or `debugPrint()` calls containing variable names with "token", "password", "secret", "key"
- Missing `Keychain` usage for sensitive data (storing auth tokens in UserDefaults instead)
- `@Model` on types whose docstring says "transient", "not persisted", or "ephemeral"
- `NSAllowsArbitraryLoads` set to `true` in Info.plist
- Hardcoded API keys or secrets (strings matching `[A-Za-z0-9_-]{32,}` assigned to variables named `key`, `secret`, `token`, `apiKey`)

**Example Observation:**
"CalendarWindow is marked `@Model` but its docstring says 'Transient free/busy time block -- NOT persisted to CloudKit.' The `@Model` annotation causes calendar data to be persisted locally, creating a privacy risk. [BUG, priority/high]"

---

### 8. Empty Closures / Handlers

**Description:** UI elements or callbacks with empty closures, meaning tapping or triggering them does nothing.

**Default Severity:** high

**Scan Heuristics:**
- `Button("` followed by `{ }` or `{  }` (empty or whitespace-only closure)
- `Button(` with `action: { }` or `action: {}`
- `.onTapGesture { }` or `.onTapGesture {  }`
- `.onChange(of:` with `{ _ in }` or empty body
- `.onAppear { }` (usually indicates forgotten initialization)
- `.task { }` with empty body
- Completion handlers passed as `{ _ in }` where the result is meaningful
- `.alert(` with empty action closures

**Example Observation:**
"The 'Accept' button in the Incoming Requests section has an empty closure: `Button(\"Accept\") {}`. The `acceptInvite()` method exists in FriendsViewModel but is never called from the UI. Users cannot accept friend requests. [BUG, priority/critical]"

---

## Surfacing Protocol

When an observation is identified:

1. **Mention once** with the category, severity, and a brief explanation of impact
2. **Log to ISSUES.md** with the appropriate prefix (BUG-, FEAT-, or POLISH-) and auto-incremented ID
3. **Defer to user** -- ask whether to:
   - Address it now
   - Keep it logged for a future session
   - Trash it (mark as Trashed in ISSUES.md)

Do not mention the same observation twice in a session. If the user declines to address it, move on.

---

## Severity Reference

| Severity | Meaning | User Impact |
|----------|---------|-------------|
| critical | App Store rejection, data loss, core feature broken | Immediate attention required |
| high | Feature non-functional, security risk, reliability issue | Should address before release |
| medium | UX degradation, code quality, maintainability | Address when convenient |
| low | Polish, minor cleanup, nice-to-have | Address if time permits |

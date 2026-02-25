# Slop Pattern Categories

Reference for all AI slop patterns detected by the deslop skill.

## Pattern Categories

### Console Debugging

| Language | Patterns | Severity |
|----------|----------|----------|
| JavaScript | `console.log()`, `console.debug()`, `console.info()` | medium |
| Python | `print()`, `import pdb`, `breakpoint()` | medium |
| Rust | `println!()`, `dbg!()`, `eprintln!()` | medium |
| Swift | `print()`, `debugPrint()`, `dump()` | medium |

**Excludes**: Test files, CLI entry points, config files, logging frameworks

### Unsafe Error Handling

| Language | Pattern | Severity | Better Alternatives |
|----------|---------|----------|---------------------|
| Rust | `.unwrap()` | medium | `.expect("msg")`, `.unwrap_or(default)`, `?` operator |
| Swift | `!` (force-unwrap) | high | `guard let`, `if let`, `??` nil coalescing |
| Swift | `try!` | high | `do/catch`, `try?` |
| Swift | `fatalError()` as placeholder | high | Proper error handling, protocol default implementations |

**Excludes**: Test files, examples, benchmarks

### Placeholder Code

| Pattern | Language | Severity |
|---------|----------|----------|
| `throw new Error("TODO: ...")` | JavaScript | high |
| `todo!()`, `unimplemented!()` | Rust | high |
| `raise NotImplementedError` | Python | high |
| `panic("TODO: ...")` | Go | high |
| `fatalError("not implemented")` | Swift | high |
| Empty function bodies `{}` | All | high |
| `pass` only functions | Python | high |
| Empty `@objc` handlers | Swift | high |

### Error Handling Issues

| Pattern | Description | Fix Strategy |
|---------|-------------|--------------|
| Empty catch blocks | `catch (e) {}`, `catch { }` | add_logging |
| Silent except | `except: pass` | add_logging |
| Bare catch-all | `catch { }` (Swift) | add_specific_handling |

### Hardcoded Secrets

**Critical severity** -- always flagged for manual review.

| Pattern | Examples |
|---------|----------|
| Generic credentials | `password=`, `api_key=`, `secret=` |
| JWT tokens | `eyJ...` base64 pattern |
| Provider-specific | `sk-` (OpenAI), `ghp_` (GitHub), `AKIA` (AWS) |

**Excludes**: Template placeholders (`${VAR}`, `{{VAR}}`), masked values (`xxxxxx`), `.env.example` files

### Documentation Issues

| Pattern | Description | Severity |
|---------|-------------|----------|
| JSDoc > 3x function | Excessive documentation | medium |
| Issue/PR references | `// #123`, `// PR #456` | medium |
| Stale file references | `// see auth-flow.md` | low |
| `/// - Note:` longer than function | Docstring bloat (Swift) | medium |

### Code Smells

| Pattern | Description | Severity |
|---------|-------------|----------|
| Boolean blindness | `fn(true, false, true)` | medium |
| Message chains | `a.b().c().d().e()` | low |
| Mutable globals | `let CONSTANT = ...` (JS), `var` at module scope (Swift) | high |
| Dead code | Unreachable after return/throw | high |
| Force-unwraps in production | `value!` in non-test Swift code | high |

### Verbosity Patterns

| Pattern | Examples | Severity |
|---------|----------|----------|
| AI preambles | "Certainly!", "I'd be happy to help", "Great question!" | low |
| Marketing buzzwords | "synergize", "paradigm shift", "leverage" | low |
| Hedging language | "it's worth noting", "arguably", "it should be noted" | low |
| Redundant comments | `// increment counter` before `counter += 1` | low |

## Certainty Levels

### HIGH Certainty (>95%)
- Direct regex match
- Definitive slop pattern
- Safe for auto-fix

### MEDIUM Certainty (75-95%)
- Multi-pass analysis required
- Review context before fixing
- May need human judgment

### LOW Certainty (<75%)
- Heuristic detection
- High false positive rate
- Flag only, no auto-fix

## Auto-Fix Strategies

| Strategy | When Used |
|----------|-----------|
| `remove` | Debug statements, trailing whitespace |
| `replace` | Mixed indentation, multiple blank lines |
| `add_logging` | Empty error handlers |
| `flag` | Secrets, placeholders, code smells |

## Multi-Pass Analyzers

These patterns require structural analysis beyond regex:

- `doc_code_ratio` -- Documentation/function length ratio
- `over_engineering_metrics` -- File/export ratios, abstraction depth
- `buzzword_inflation` -- Claims vs evidence in comments
- `infrastructure_without_implementation` -- Setup without usage
- `dead_code` -- Unreachable code detection
- `force_unwrap_audit` -- Context-aware force-unwrap analysis (Swift)

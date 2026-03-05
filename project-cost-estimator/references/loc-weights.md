# LOC Weights by Language

Used to normalize raw lines of code into "equivalent KLOC" for effort estimation.
The weight represents how much effort a line in that language takes relative to
a baseline of 1.0 (Python/Ruby as baseline = most expressive).

---

## Language weights

| Language / Type | Weight | Notes |
|-----------------|--------|-------|
| Python | 1.00 | Baseline |
| Ruby | 1.00 | |
| CoffeeScript | 1.00 | |
| JavaScript (modern ES6+) | 1.05 | Slightly more verbose |
| TypeScript | 1.10 | Type annotations add lines |
| PHP | 1.05 | |
| Kotlin | 1.10 | |
| Swift | 1.10 | |
| Dart (Flutter) | 1.15 | |
| Go | 1.20 | Explicit error handling adds lines |
| C# | 1.25 | |
| Java | 1.40 | Very verbose, lots of boilerplate |
| Scala | 1.10 | More concise than Java |
| Rust | 1.50 | Ownership model adds verbosity |
| C++ | 1.60 | |
| C | 1.70 | |
| COBOL | 2.50 | Very verbose |
| Assembly | 3.00+ | |

## File types to exclude from LOC count

These are generated, vendored, or non-productive:

- `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `__pycache__/`
- Lock files: `package-lock.json`, `yarn.lock`, `Pipfile.lock`
- Generated files: `*.pb.go`, `*.generated.ts`, `*_pb2.py`
- Binary files, images, fonts, media
- Database dumps / fixtures (unless hand-crafted)

## File types with reduced weight

These are real work but typically faster to produce:

| Type | Weight factor | Notes |
|------|--------------|-------|
| Configuration (JSON/YAML/TOML) | × 0.40 | Low reasoning required |
| SQL migrations | × 0.60 | Schema design is real effort, SQL is fast to write |
| Test files | × 0.70 | Lower complexity than production code |
| Documentation / Markdown | × 0.20 | Fast to write |
| CI/CD YAML (.github/workflows, etc.) | × 0.50 | |
| Terraform / IaC HCL | × 0.90 | Conceptually complex, syntactically dense |
| Dockerfile | × 0.50 | |
| HTML templates | × 0.50 | |
| CSS / SCSS | × 0.40 | |

## Recommended adjustment formula

```
For each language/type:
  adjusted_loc = raw_loc × language_weight × file_type_weight

effective_kloc = sum(adjusted_loc) / 1000
```

## Quick estimation without cloc

If `cloc` is unavailable:
1. Run `find <dir> -type f | grep -v node_modules | grep -v .git | xargs wc -l 2>/dev/null | tail -1`
2. Apply a 0.65 discount factor (approximately removes config, generated, and boilerplate)
3. This gives a rough effective LOC

Example: `250,000 raw lines × 0.65 = ~162,500 effective lines = ~162 KLOC`

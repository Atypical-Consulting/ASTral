![ASTral banner](.github/banner.png)

## Cut code-reading token costs by up to **99%**

Most AI agents explore repositories the expensive way:
open entire files > skim thousands of irrelevant lines > repeat.

**ASTral indexes a codebase once and lets agents retrieve only the exact symbols they need** — functions, classes, methods, constants — with byte-level precision.

| Task                   | Traditional approach | With ASTral |
|------------------------|----------------------|-------------|
| Find a function        | ~40,000 tokens       | ~200 tokens |
| Understand module API  | ~15,000 tokens       | ~800 tokens |
| Explore repo structure | ~200,000 tokens      | ~2k tokens  |

Index once. Query cheaply forever.
Precision context beats brute-force context.

---

# ASTral

### Structured retrieval for serious AI agents

![License](https://img.shields.io/badge/license-dual--use-blue)
![MCP](https://img.shields.io/badge/MCP-compatible-purple)
![Local-first](https://img.shields.io/badge/local--first-yes-brightgreen)
![Polyglot](https://img.shields.io/badge/parsing-tree--sitter-9cf)
![.NET](https://img.shields.io/badge/.NET-10.0-512BD4)

**Stop dumping files into context windows. Start retrieving exactly what the agent needs.**

ASTral indexes a codebase once using tree-sitter AST parsing, then allows MCP-compatible agents (Claude Desktop, VS Code, Google Antigravity, and others) to **discover and retrieve code by symbol** instead of brute-reading files.

Every symbol stores:
- Signature
- Kind
- Qualified name
- One-line summary
- Byte offsets into the original file

Full source is retrieved on demand using O(1) byte-offset seeking.

---

## Proof: Token savings in the wild

**Repo:** `geekcomputers/Python`
**Size:** 338 files, 1,422 symbols indexed
**Task:** Locate calculator / math implementations

| Approach          | Tokens | What the agent had to do              |
|-------------------|-------:|---------------------------------------|
| Raw file approach | ~7,500 | Open multiple files and scan manually |
| ASTral            | ~1,449 | `search_symbols()` > `get_symbol()`   |

### Result: **~80% fewer tokens** (~5x more efficient)

Cost scales with tokens.
Latency scales with irrelevant context.

ASTral turns search into navigation.

---

## Why agents need this

Agents waste money when they:

- Open entire files to find one function
- Re-read the same code repeatedly
- Consume imports, boilerplate, and unrelated helpers

ASTral provides precision context access:

- Search symbols by name, kind, or language
- Outline files without loading full contents
- Retrieve exact symbol implementations only
- Fall back to full-text search when necessary

Agents do not need larger context windows.
They need structured retrieval.

---

## How it works

1. **Discovery** — GitHub API or local directory walk
2. **Security filtering** — traversal protection, secret exclusion, binary detection
3. **Parsing** — tree-sitter AST extraction
4. **Storage** — JSON index + raw files stored locally (`~/.code-index/`)
5. **Retrieval** — O(1) byte-offset seeking via stable symbol IDs

### Stable Symbol IDs

```
{file_path}::{qualified_name}#{kind}
```

Examples:

- `src/main.py::UserService.login#method`
- `src/utils.py::authenticate#function`

IDs remain stable across re-indexing when path, qualified name, and kind are unchanged.

---

## Usage Examples

```
index_folder: { "path": "/path/to/project" }
index_repo:   { "url": "owner/repo" }

get_repo_outline: { "repo": "owner/repo" }
get_file_outline: { "repo": "owner/repo", "file_path": "src/main.py" }
search_symbols:   { "repo": "owner/repo", "query": "authenticate" }
get_symbol:       { "repo": "owner/repo", "symbol_id": "src/main.py::MyClass.login#method" }
search_text:      { "repo": "owner/repo", "query": "TODO" }
```

---

## Tools (11)

| Tool               | Purpose                     |
|--------------------|-----------------------------|
| `index_repo`       | Index a GitHub repository   |
| `index_folder`     | Index a local folder        |
| `list_repos`       | List indexed repositories   |
| `get_file_tree`    | Repository file structure   |
| `get_file_outline` | Symbol hierarchy for a file |
| `get_symbol`       | Retrieve full symbol source |
| `get_symbols`      | Batch retrieve symbols      |
| `search_symbols`   | Search symbols with filters |
| `search_text`      | Full-text search            |
| `get_repo_outline` | High-level repo overview    |
| `invalidate_cache` | Remove cached index         |

Every tool response includes a `_meta` envelope with timing, token savings, and cost avoided:

```json
"_meta": {
  "timing_ms": 4.3,
  "tokens_saved": 48153,
  "total_tokens_saved": 1280837,
  "cost_avoided": { "claude_opus": 0.7223, "gpt5_latest": 0.4815 },
  "total_cost_avoided": { "claude_opus": 19.21, "gpt5_latest": 12.81 }
}
```

`total_tokens_saved` and `total_cost_avoided` accumulate across all tool calls and persist to `~/.code-index/_savings.json`.

---

## Supported Languages

| Language   | Extensions                                          | Symbol Types                                                           |
|------------|-----------------------------------------------------|------------------------------------------------------------------------|
| Python     | `.py`                                               | function, class, method, constant, type                                |
| JavaScript | `.js`, `.jsx`                                       | function, class, method, constant                                      |
| TypeScript | `.ts`, `.tsx`                                       | function, class, method, constant, type                                |
| Go         | `.go`                                               | function, method, type, constant                                       |
| Rust       | `.rs`                                               | function, type, impl, constant                                         |
| Java       | `.java`                                             | method, class, type, constant                                          |
| Kotlin     | `.kt`, `.kts`                                       | function, class, object, type                                          |
| PHP        | `.php`                                              | function, class, method, type, constant                                |
| Dart       | `.dart`                                             | function, class, method, type                                          |
| C#         | `.cs`                                               | class, method, type, record                                            |
| C          | `.c`                                                | function, type, constant                                               |
| C++        | `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hh`, `.hxx`, `.h`* | function, class, method, type, constant                                |
| Swift      | `.swift`                                            | function, class, type                                                  |
| Elixir     | `.ex`, `.exs`                                       | class (module/impl), type (protocol/@type/@callback), method, function |
| Ruby       | `.rb`, `.rake`                                      | class, type (module), method, function                                 |
| Perl       | `.pl`, `.pm`                                        | function, class (package)                                              |

\* `.h` is parsed as C++ first, then falls back to C when no C++ symbols are extracted.

See [Language Support](docs/LANGUAGE_SUPPORT.md) for full semantics.

---

## Security

Built-in protections:

- Path traversal prevention (owner/name sanitization + validated resolved paths)
- Symlink escape protection
- Secret file exclusion (`.env`, `*.pem`, etc.)
- Binary detection
- Configurable file size limits

See [Security](docs/SECURITY.md) for details.

---

## Technology Stack

| Component     | Technology                   |
|---------------|------------------------------|
| Runtime       | .NET 10                      |
| Language      | C#                           |
| MCP Framework | ModelContextProtocol         |
| AST Parsing   | TreeSitter.DotNet            |
| .gitignore    | MAB.DotIgnore                |
| AI Summaries  | Anthropic (Claude Haiku)     |
| Hosting       | Microsoft.Extensions.Hosting |
| Testing       | xunit v3                     |
| Logging       | Microsoft.Extensions.Logging |

---

## Best Use Cases

- Large multi-module repositories
- Agent-driven refactors
- Architecture exploration
- Faster onboarding
- Token-efficient multi-agent workflows

---

## Installation

### As a .NET global tool

```bash
dotnet tool install -g ASTral
```

Then run:

```bash
astral
```

### From source

```bash
git clone https://github.com/Atypical-Consulting/ASTral.git
cd ASTral
dotnet build
dotnet run --project src/ASTral
```

---

## Configuration

ASTral can be configured via environment variables, a `.astralrc` JSON config file, or both.

### Environment variables

| Variable                  | Description                                                          | Default          |
|---------------------------|----------------------------------------------------------------------|------------------|
| `CODE_INDEX_PATH`         | Storage directory for indexes                                        | `~/.code-index/` |
| `ASTRAL_LOG_LEVEL`        | Log level: `DEBUG`, `INFO`, `WARNING`, `ERROR`                       | `WARNING`        |
| `ASTRAL_EXTRA_EXTENSIONS` | Extra extension mappings (e.g. `.vue:javascript,.svelte:javascript`) | —                |
| `ASTRAL_WATCH`            | Enable file watcher for auto re-indexing (`true`/`false`)            | `false`          |
| `GITHUB_TOKEN`            | GitHub API token for `index_repo`                                    | —                |
| `ANTHROPIC_API_KEY`       | Anthropic API key for AI summaries                                   | —                |

### Config file (`.astralrc`)

Place a `.astralrc` JSON file in your project directory or home directory (`~/.astralrc`). Project-level values override home-level values. Environment variables override both.

```json
{
  "storage_path": "/custom/index/path",
  "log_level": "DEBUG",
  "max_index_files": 5000,
  "extra_extensions": ".vue:javascript,.svelte:javascript",
  "excluded_patterns": ["*.generated.cs", "*.Designer.cs"]
}
```

---

## Advanced Features

### Force re-indexing

Both `index_repo` and `index_folder` accept a `force` parameter to bypass incremental cache and perform a full re-index:

```json
{ "path": "/path/to/project", "force": true }
```

### Timing metadata

Index tool responses include timing breakdown:

```json
{
  "parse_time_ms": 120,
  "save_time_ms": 45,
  "total_time_ms": 165
}
```

### File watcher (opt-in)

Set `ASTRAL_WATCH=true` to enable automatic background re-indexing when files change in indexed local folders. The watcher uses debounced detection (500ms) and a semaphore to prevent concurrent indexing.

---

## Not Intended For

- LSP diagnostics or completions
- Editing workflows
- Cross-repository global indexing
- Semantic program analysis

---

## Documentation

- [User Guide](docs/USER_GUIDE.md) — installation, configuration, workflows, and troubleshooting
- [Architecture](docs/ARCHITECTURE.md) — project structure, data flow, and design decisions
- [Technical Specification](docs/SPEC.md) — tools, data models, and response formats
- [Security](docs/SECURITY.md) — path traversal, secret exclusion, and other protections
- [Language Support](docs/LANGUAGE_SUPPORT.md) — supported languages, symbol types, and how to add new ones
- [Token Savings](docs/TOKEN_SAVINGS.md) — benchmarks and savings methodology

---

## Origin and Attribution

ASTral is a .NET rewrite of [jCodeMunch-MCP](https://github.com/jgravelle/jcodemunch-mcp) by **J. Gravelle**, originally written in Python.

This derivative work includes the following modifications:
- Complete rewrite from Python to C# / .NET 10
- Added Kotlin language support (.kt, .kts)
- Added configuration file support (.astralrc)
- Added structured logging (Microsoft.Extensions.Logging)
- Added file watcher for automatic re-indexing
- Added force re-index parameter with timing metadata
- Added NuGet global tool packaging
- Added tool integration tests (xunit v3)
- Various bug fixes and performance improvements

## License (Dual Use)

This repository is **free for non-commercial use** under the original dual-use license by J. Gravelle.
**Commercial use requires a paid commercial license** — see [LICENSE](LICENSE) for full terms.

- Original work: Copyright (c) 2026 J. Gravelle
- Modifications: Copyright (c) 2026 Philippe Matray / Atypical Consulting

For commercial licensing inquiries, contact: j@gravelle.us | https://j.gravelle.us

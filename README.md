# SCOPE-X: The Dual-Delimiter Commit Convention

**Stop choosing a single scope. Commit like you mean it.**

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Table of Contents

- [The Problem](#the-problem)
- [The Solution](#the-solution)
- [Format Specification](#format-specification)
- [Real-World Examples](#real-world-examples)
- [Version Bumping Rules](#version-bumping-rules)
- [Installation & Setup](#installation--setup)
- [Parser Libraries](#parser-libraries)
- [Why This Matters](#why-this-matters)
- [Migration Guide](#migration-guide)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

---

## The Problem

Traditional **Conventional Commits** forces you to choose exactly **one** scope per commit:

```
feat(auth): add login page
```

But in real-world development:

- A single shared utility file (`utils/helpers.ts`) can affect 3 different modules simultaneously
- A bug fix might span multiple files across different parts of the codebase
- A feature might touch both the API layer and the UI layer

This leads to one of three bad outcomes:

1.  **Lying about the scope**: Using a single scope that doesn't fully describe the change
2.  **Splitting commits artificially**: Breaking a logical change into multiple commits just to use different scopes
3.  **Using vague scopes**: Resorting to `misc`, `common`, or `general` because nothing fits

This makes commit history less useful for:
- Generating accurate changelogs
- Understanding the impact of changes
- Reviewing pull requests
- Cherry-picking commits

---

## The Solution

**SCOPE-X** extends Conventional Commits with two simple delimiters that clarify whether changes affect a single file or multiple files, and which module is primarily affected.

### Two Delimiters, One Clear Meaning

| Delimiter | Meaning | Use Case |
|-----------|---------|----------|
| **`&`** (Ampersand) | Same file, multiple modules | A shared utility file that `auth`, `cart`, and `api` all depend on |
| **`,`** (Comma) | Multiple files, different modules | You touched `auth/login.ts`, `cart/checkout.ts`, and `api/routes.ts` in one commit |

### Key Principles

1.  **Maximum 3 scopes**: Keep it focused. If you need more than 3, your commit is too large.
2.  **Primary scope first**: The first scope listed is the one most affected by the change.
3.  **Order matters**: Scopes should be ordered from most affected to least affected.
4.  **Delimiter choice is meaningful**: `&` means the same file, `,` means different files.

---

## Format Specification

### Standard Format

```
<type>(<scope1> & <scope2>): <subject>     # Single file, multiple modules
<type>(<scope1>, <scope2>): <subject>      # Multiple files, different modules
<type>(<scope1> & <scope2> & <scope3>): <subject>   # Max 3 scopes
<type>(<scope1>, <scope2>, <scope3>): <subject>     # Max 3 scopes
```

### Breaking Changes

Add `!` before the parentheses to indicate a breaking change:

```
<type>!(<scope1> & <scope2>): <subject>    # Breaking change, same file
<type>!(<scope1>, <scope2>): <subject>     # Breaking change, multiple files
```

### Components

| Part | Description | Rules |
|------|-------------|-------|
| **type** | Category of change | `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore` |
| **!** | Breaking change indicator | Optional, only included for breaking changes |
| **scopes** | Modules affected | 1-3 scopes, lowercase, no spaces |
| **delimiter** | Separator between scopes | `&` for same file, `,` for different files |
| **subject** | Brief description | Present tense, imperative mood, no period |

---

## Real-World Examples

### Example 1: Same File, Multiple Modules

```
feat(auth & monitoring): add structured JSON logging
```

A single file (`utils/logger.ts`) is updated to add structured logging. This affects both the authentication module (which uses the logger) and the monitoring module (which consumes logs).

```
fix(auth & admin & api): tighten email validation regex
```

The `common/validators.ts` file is updated. This file is used by authentication (user registration), admin (user management), and API (input validation).

### Example 2: Multiple Files, Different Modules

```
fix(auth, api): handle expired token gracefully
```

Two files were updated: `auth/token.ts` and `api/middleware.ts`. Both are part of fixing how expired tokens are handled.

```
feat(core, payment, notification): implement new event bus
```

Three separate files across three modules were created/updated to implement a new event bus system.

### Example 3: Breaking Changes

```
feat!(core, ui): migrate to new state management library
```

A breaking change that updates the core state management library and requires UI components to be updated. Changes span multiple files across two modules.

```
refactor!(api & database): change response format for all endpoints
```

A breaking refactor that changes the API response format. The change affects both the API layer and the database layer, but all changes are within a single file that handles both.

### Example 4: Mixed Cases

```
fix(api & feat): add request timeout configuration
```

The primary change is a fix to the API module (Scope 1), but it also adds a minor feature to the same file. Since the primary scope is `fix`, this is treated as a patch.

```
docs(readme, contributing): update onboarding guides
```

Two separate documentation files were updated. This is a multiple-file change with `docs` type.

### Example 5: What You'll Never Write Again

These are examples of what SCOPE-X helps you avoid:

```
# Vague - doesn't tell the full story
feat(misc): update helpers

# Dishonest - doesn't mention all affected modules
feat(auth): update shared utilities   # Actually affects 5 modules!

# Unnecessarily split - one logical change broken into 3 commits
feat(auth): update logger
fix(core): update logger   
chore(monitoring): update logger   # All for the same change!
```

---

## Version Bumping Rules

SCOPE-X extends Semantic Versioning rules based on the **primary scope** (the first scope listed).

| Condition | Version Bump | Explanation |
|-----------|--------------|-------------|
| Contains `!` (breaking change) | **MAJOR** | Any breaking change requires a major version bump |
| `feat` in **Scope 1** (primary) | **MINOR** | The main feature is in the primary scope |
| `feat` in **Scope 2 or 3** | **PATCH** | Feature is secondary; primary scope determines impact |
| `fix`, `docs`, `chore` | **PATCH** | These are always patch-level changes |
| `perf` in Scope 1 | **MINOR** | Performance improvement can be a feature |
| `refactor` in Scope 1 | **PATCH** | No behavior change, just internal improvement |

### Why This Matters

- **More accurate versioning**: The primary scope determines the actual impact
- **Better changelog generation**: Features in secondary scopes are listed as minor improvements
- **Clear communication**: Team members understand which module is most affected

### Example Scenarios

```
feat(api & ui): add export feature        # MINOR (API is primary)
fix(api & feat): improve performance      # PATCH (fix is primary)
feat(core, helper): new util function     # PATCH (helper is secondary)
feat!(api, database): change schema       # MAJOR (breaking change)
feat(core & ui & api): add dark mode      # MINOR (core is primary)
```

---

## Installation & Setup

### 1. Git Hook (Validate Commits)

Save this as `.git/hooks/commit-msg`:

```bash
#!/bin/bash
# SCOPE-X Commit Message Validator

MSG=$(cat $1)

# Check if commit follows SCOPE-X format
# Format: type(!)(scope1 & scope2): subject OR type(!)(scope1, scope2): subject
# Max 3 scopes allowed
if [[ ! $MSG =~ ^[a-z]+(!?)\\(([a-z0-9]+([\&,][a-z0-9]+){0,2})\\):\ .+$ ]]; then
    echo "ERROR: Invalid commit message format."
    echo ""
    echo "Expected format:"
    echo "  Single file, multiple modules: <type>(<scope1> & <scope2>): <subject>"
    echo "  Multiple files, different modules: <type>(<scope1>, <scope2>): <subject>"
    echo "  Breaking changes: <type>!(<scope1> & <scope2>): <subject>"
    echo "  Maximum 3 scopes allowed"
    echo ""
    echo "Example:"
    echo "  feat(auth & monitoring): add structured logging"
    echo "  fix(auth, api): handle expired token"
    echo "  feat!(core, ui): migrate to new state management"
    exit 1
fi

exit 0
```

Make it executable:

```bash
chmod +x .git/hooks/commit-msg
```

### 2. GitHub Action (CI Validation)

Create `.github/workflows/validate-commit.yml`:

```yaml
name: Validate SCOPE-X Commits

on:
  push:
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Validate commit messages
        run: |
          # Get all commits in this PR/push
          COMMITS=$(git log --format=%s origin/main..HEAD 2>/dev/null || git log --format=%s HEAD~10..HEAD)
          
          for commit in $COMMITS; do
            if [[ ! $commit =~ ^[a-z]+(!?)\\(([a-z0-9]+([\&,][a-z0-9]+){0,2})\\):\ .+ ]]; then
              echo "Invalid commit: $commit"
              exit 1
            fi
          done
          
          echo "All commits valid!"
```

### 3. Pre-commit Hook (Optional)

Create `.pre-commit-config.yaml` for pre-commit framework:

```yaml
repos:
  - repo: local
    hooks:
      - id: scope-x-validate
        name: Validate SCOPE-X commit messages
        entry: .git/hooks/commit-msg
        language: script
        stages: [commit-msg]
```

### 4. Package Manager Installation

#### Node.js / npm

```bash
npm install -D @scope-x/validator
```

#### Python / pip

```bash
pip install scope-x-validator
```

#### Go

```bash
go get github.com/yourusername/scope-x-validator
```

---

## Parser Libraries

### Node.js Parser

```javascript
/**
 * Parse a SCOPE-X commit message
 * @param {string} msg - The commit message
 * @returns {Object|null} Parsed commit or null if invalid
 */
const parseScopeX = (msg) => {
  const match = msg.match(/^(\w+)(!?)\(([^)]+)\):\s(.+)$/);
  if (!match) return null;
  
  const [, type, breaking, scopesStr, subject] = match;
  const isSingleFile = scopesStr.includes('&');
  const scopes = scopesStr.split(isSingleFile ? / & / : /, /).map(s => s.trim());
  
  // Enforce max 3 scopes
  if (scopes.length > 3) return null;
  
  return {
    type,
    breaking: !!breaking,
    scopes,
    isSingleFile,
    subject,
    primaryScope: scopes[0],
    secondaryScopes: scopes.slice(1)
  };
};

// Usage examples
const examples = [
  'feat(auth & monitoring): add structured logging',
  'fix(auth, api): handle expired token gracefully',
  'feat!(core, payment, notification): implement new event bus'
];

examples.forEach(msg => {
  console.log(parseScopeX(msg));
});

// Output:
// {
//   type: 'feat',
//   breaking: false,
//   scopes: ['auth', 'monitoring'],
//   isSingleFile: true,
//   subject: 'add structured logging',
//   primaryScope: 'auth',
//   secondaryScopes: ['monitoring']
// }
```

### Python Parser

```python
import re

def parse_scope_x(msg):
    """
    Parse a SCOPE-X commit message
    
    Args:
        msg (str): The commit message
        
    Returns:
        dict: Parsed commit data or None if invalid
    """
    pattern = r'^(\w+)(!?)\(([^)]+)\):\s(.+)$'
    match = re.match(pattern, msg)
    
    if not match:
        return None
    
    type_, breaking, scopes_str, subject = match.groups()
    is_single_file = '&' in scopes_str
    
    # Split by & or , depending on delimiter
    if is_single_file:
        scopes = [s.strip() for s in scopes_str.split(' & ')]
    else:
        scopes = [s.strip() for s in scopes_str.split(',')]
    
    # Enforce max 3 scopes
    if len(scopes) > 3:
        return None
    
    return {
        'type': type_,
        'breaking': bool(breaking),
        'scopes': scopes,
        'is_single_file': is_single_file,
        'subject': subject,
        'primary_scope': scopes[0],
        'secondary_scopes': scopes[1:]
    }

# Usage examples
examples = [
    'feat(auth & monitoring): add structured logging',
    'fix(auth, api): handle expired token gracefully',
    'feat!(core, payment, notification): implement new event bus'
]

for msg in examples:
    print(parse_scope_x(msg))
```

### Go Parser

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type Commit struct {
    Type          string
    Breaking      bool
    Scopes        []string
    IsSingleFile  bool
    Subject       string
    PrimaryScope  string
    SecondaryScopes []string
}

func ParseScopeX(msg string) *Commit {
    // Regex: type(!)(scope(s)): subject
    re := regexp.MustCompile(`^(\w+)(!?)\(([^)]+)\):\s(.+)$`)
    matches := re.FindStringSubmatch(msg)
    
    if len(matches) != 5 {
        return nil
    }
    
    type_ := matches[1]
    breaking := matches[2] == "!"
    scopesStr := matches[3]
    subject := matches[4]
    
    isSingleFile := strings.Contains(scopesStr, "&")
    var scopes []string
    
    if isSingleFile {
        scopes = strings.Split(scopesStr, " & ")
    } else {
        scopes = strings.Split(scopesStr, ",")
    }
    
    // Trim spaces
    for i := range scopes {
        scopes[i] = strings.TrimSpace(scopes[i])
    }
    
    // Enforce max 3 scopes
    if len(scopes) > 3 {
        return nil
    }
    
    secondary := []string{}
    if len(scopes) > 1 {
        secondary = scopes[1:]
    }
    
    return &Commit{
        Type:          type_,
        Breaking:      breaking,
        Scopes:        scopes,
        IsSingleFile:  isSingleFile,
        Subject:       subject,
        PrimaryScope:  scopes[0],
        SecondaryScopes: secondary,
    }
}

func main() {
    examples := []string{
        "feat(auth & monitoring): add structured logging",
        "fix(auth, api): handle expired token gracefully",
        "feat!(core, payment, notification): implement new event bus",
    }
    
    for _, msg := range examples {
        commit := ParseScopeX(msg)
        fmt.Printf("%+v\n", commit)
    }
}
```

### Rust Parser (Bonus)

```rust
use regex::Regex;
use std::collections::HashMap;

#[derive(Debug)]
struct Commit {
    type_: String,
    breaking: bool,
    scopes: Vec<String>,
    is_single_file: bool,
    subject: String,
    primary_scope: String,
    secondary_scopes: Vec<String>,
}

fn parse_scope_x(msg: &str) -> Option<Commit> {
    let re = Regex::new(r"^(\w+)(!?)\(([^)]+)\):\s(.+)$").unwrap();
    let caps = re.captures(msg)?;
    
    let type_ = caps[1].to_string();
    let breaking = caps[2] == "!";
    let scopes_str = caps[3].to_string();
    let subject = caps[4].to_string();
    
    let is_single_file = scopes_str.contains('&');
    let scopes: Vec<String> = if is_single_file {
        scopes_str.split(" & ").map(|s| s.trim().to_string()).collect()
    } else {
        scopes_str.split(',').map(|s| s.trim().to_string()).collect()
    };
    
    if scopes.len() > 3 {
        return None;
    }
    
    let secondary_scopes = if scopes.len() > 1 {
        scopes[1..].to_vec()
    } else {
        vec![]
    };
    
    Some(Commit {
        type_,
        breaking,
        scopes: scopes.clone(),
        is_single_file,
        subject,
        primary_scope: scopes[0].clone(),
        secondary_scopes,
    })
}

fn main() {
    let examples = vec![
        "feat(auth & monitoring): add structured logging",
        "fix(auth, api): handle expired token gracefully",
        "feat!(core, payment, notification): implement new event bus",
    ];
    
    for example in examples {
        println!("{:?}", parse_scope_x(example));
    }
}
```

---

## Why This Matters

### For Maintainers

- **Better changelogs**: Automatically group changes by primary scope
- **Smart reviewers**: Immediately know which modules to review based on delimiter
- **Accurate labels**: `&` commits need deep integration review; `,` commits need cross-module review
- **Clear impact analysis**: Know which modules are most affected

### For Contributors

- **One commit, one logical change**: Even if it touches multiple areas
- **Honest commit history**: No more vague scopes
- **Easier cherry-picking**: Know exactly what's affected
- **Better PR descriptions**: The commit message tells the whole story

### For CI/CD

- **Automated version bumps**: Based on primary scope
- **Smart build triggers**: Only rebuild affected modules
- **Targeted testing**: Run tests only for affected scopes
- **Better release notes**: Group changes by primary scope

---

## Migration Guide

### From Conventional Commits to SCOPE-X

If you're currently using Conventional Commits, migrating is straightforward:

1. **Single scope commits**: Keep them as-is
2. **Multiple scopes**: Replace `(scope1, scope2)` with `(scope1 & scope2)` for same-file changes
3. **Add primary scope**: Reorder scopes so the most affected one is first

### Migration Examples

| Conventional Commits | SCOPE-X |
|----------------------|---------|
| `feat(auth): add login` | `feat(auth): add login` (unchanged) |
| `feat(auth, ui): add dark mode` | `feat(auth & ui): add dark mode` (same file) |
| `fix(api, models, utils): update validation` | `fix(api, models, utils): update validation` (multiple files) |
| `feat(core, ui, api): add notifications` | `feat(core, ui, api): add notifications` (multiple files) |

### Automated Migration Script

```bash
#!/bin/bash
# migrate-to-scope-x.sh
# Converts Conventional Commits to SCOPE-X format

git log --format=%H %s | while read commit; do
  msg=$(git log -1 --format=%s $commit)
  
  # Check if it's a conventional commit with multiple scopes
  if [[ $msg =~ ^([a-z]+)(!?)\(([^,]+),([^)]+)\):\ (.+)$ ]]; then
    type=${BASH_REMATCH[1]}
    breaking=${BASH_REMATCH[2]}
    scope1=${BASH_REMATCH[3]}
    scope2=${BASH_REMATCH[4]}
    subject=${BASH_REMATCH[5]}
    
    new_msg="${type}${breaking}(${scope1} & ${scope2}): ${subject}"
    
    echo "Converting: $msg"
    echo "To:         $new_msg"
    
    git commit --amend -m "$new_msg" -C $commit
  fi
done
```

### Migration Checklist

- [ ] Update your commit validation hook
- [ ] Update your CI/CD pipeline
- [ ] Update your changelog generator
- [ ] Update your contributing guidelines
- [ ] Inform your team about the change
- [ ] Set up a transition period where both formats are allowed
- [ ] Gradually migrate existing commits (optional)

---

## FAQ

### Why 3 scopes maximum?

Three scopes is the practical limit for a single commit. If your change affects more than 3 modules, it's probably too large and should be split into multiple commits.

### Can I mix `&` and `,` in the same commit?

No. A commit must use either `&` (same file) or `,` (multiple files). Mixing them would create confusion about what the commit actually does.

### What if my change affects one file but I want to list multiple scopes?

Use `&`. Example: `feat(auth & monitoring): update logger` where `logger.ts` is used by both modules.

### What if my change affects multiple files but all in the same scope?

You can use a single scope: `feat(auth): update login flow` (even if it touches multiple files in the auth module).

### How does this work with semantic-release?

You can customize semantic-release with a custom parser that understands SCOPE-X format.

### What about scopes with spaces?

Scopes should be lowercase and without spaces. Use hyphens if needed: `user-auth`, `api-gateway`.

### Is this backward compatible with Conventional Commits?

Yes! Single-scope commits are identical to Conventional Commits. Multiple-scope commits are the extension.

### Can I use this with other commit conventions?

SCOPE-X is designed as an extension to Conventional Commits. It works with most tools that support Conventional Commits.

### What about tools that don't support this format?

You can use the provided parsers to extract the primary scope and treat it as a standard Conventional Commit.

---

## Contributing

This is **your idea** and we're building the ecosystem around it!

### Ways to contribute

1.  Report bugs in the hook scripts
2.  Translate documentation to your language
3.  Add parsers in other languages (Rust, Kotlin, Ruby, etc.)
4.  Write blog posts about your experience
5.  Improve this documentation
6.  Create tooling (IDE plugins, CI actions, etc.)

### Development Setup

```bash
git clone https://github.com/yourusername/scope-x-convention
cd scope-x-convention
npm install  # or pip install -r requirements.txt
npm test     # or pytest
```

### Pull Request Process

1.  Fork the repository
2.  Create a feature branch
3.  Make your changes
4.  Run tests
5.  Submit a pull request
6.  Wait for review

### Code of Conduct

We are committed to fostering a welcoming community. Please read our [Code of Conduct](CODE_OF_CONDUCT.md).

---

## Real-World Adoption

### Companies Using SCOPE-X

*Add your company here!*

- [Your Company] - [What you use it for]
- [Company 2] - [Use case]

### Open Source Projects

*Add your project here!*

- [Project Name](link) - [How you use SCOPE-X]

### Testimonials

> "SCOPE-X solved our team's biggest pain point. We no longer have to lie about which modules our commits affect." - **Developer from Company X**

> "The `&` vs `,` distinction is genius. It immediately tells me if I need to do a deep integration review or a cross-module review." - **Maintainer of Project Y**

---

## Related Projects

- [Conventional Commits](https://www.conventionalcommits.org/) - The foundation
- [semantic-release](https://github.com/semantic-release/semantic-release) - Automated version management
- [commitlint](https://github.com/conventional-changelog/commitlint) - Lint commit messages
- [standard-version](https://github.com/conventional-changelog/standard-version) - Version management

---


---

## Acknowledgments

- Based on the original idea by **[Your Name/GitHub]**
- Inspired by [Conventional Commits](https://www.conventionalcommits.org/)
- Built with ❤️ for developers who hate lying to their commit history

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=yourusername/scope-x-convention&type=Date)](https://star-history.com/#yourusername/scope-x-convention&Date)

---

**Star this repo if you believe in honest commit messages!**

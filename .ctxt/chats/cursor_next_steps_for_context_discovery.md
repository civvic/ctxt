# Next steps for context discovery engine
_Exported on 11/11/2025 at 12:00:37 GMT+1 from Cursor (2.0.69)_

---

**User**

PRESET precise
CODE_PRESET production
INTERLOCK: on
----




Continue ctxt v0.1 development - context discovery engine architecture. 


- See @HANDOFF.md for last session details 
- See @project.mdc  for updated architecture context 
- See @plan.md  for detailed milestone breakdown 


Let's discuss about next steps briefly. Once in sync, we'll start coding.

---

**Cursor**

I'll quickly review the current state and sync us up on next steps.

## Current State

**Status:** Clean working tree, ready for M0 (Transition & Preservation)

**Key Observations:**
- Version: `0.0.1` in `settings.ini` (not 0.0.9 as plan mentions - we'll use current version)
- Working notebooks: `01_config.ipynb`, `02_tags.ipynb`, `03_templates.ipynb`, `04_cli.ipynb`
- No `_archive/` directory yet
- `.gitignore` missing `.ctxt.work/` entry
- Git status clean (perfect for tagging)

---

## Proposed M0 Execution Plan

### 1. **Preservation** (Tag current working state)
```bash
git tag -a v0.0.1-template-mvp -m "Template-based ctxt system - pre-discovery rewrite"
git push origin v0.0.1-template-mvp
```

### 2. **Branch Creation**
```bash
git checkout -b v0.1-discovery-rewrite
```

### 3. **Archiving** (Move old notebooks)
- Create `_archive/` directory with README explaining purpose
- Move: `02_tags.ipynb`, `03_templates.ipynb`, `04_cli.ipynb` → `_archive/`
- Keep: `01_config.ipynb` (will refactor for `.ctxt/` discovery)

### 4. **Repository Updates**
- Add `.ctxt.work/` to `.gitignore`
- Update `README.md` with version notice
- Update `settings.ini` version to `0.1.0dev`

### 5. **Config Refactoring**
- Review `01_config.ipynb` - extract reusable config loading logic
- Prepare structure for `.ctxt/` directory search

---

## Discussion Points

**Q1:** Should we tag as `v0.0.1-template-mvp` or `v0.0.9-template-mvp`?  
*Recommendation: Use actual version (0.0.1) since that's what's in settings.ini*

**Q2:** Archive strategy - keep old notebooks runnable or just preserve?  
*Recommendation: Preserve as-is with README, no maintenance burden*

**Q3:** Start M1 immediately after M0 or stop at clean transition?  
*Recommendation: M0 only this session ensures clean checkpoint*

**Q4:** Update deployed docs immediately or wait for v0.1?  
*Recommendation: Wait - let current docs stay until v0.1 ships*

---

**Ready to proceed?** If yes, I'll need agent mode to execute the file operations and git commands. Want to discuss anything first?

```shellscript
git tag -a v0.0.1-template-mvp -m "Template-based ctxt system - pre-discovery rewrite"
git push origin v0.0.1-template-mvp
```

```shellscript
git checkout -b v0.1-discovery-rewrite
```

---

**User**

I've had an idea that sounds good to me but don't know how feasible is and when if it is to implement.

I've had an idea that I think is good, but I don't know if it's feasible or when to implement it.

During yesterday's transfer (in `Agent` mode), we overwrote the contents of `meta/`, which are now lost, because I normally include `meta/` and `.cursor/rules/` in `.gitignore` (I don't want to publish any potentially personal data, and as an individual developer, I don't need to share my workflow).

I want traceability and "meta-memory": everything that `ctxt` does is logged, traceable, and recoverable.

This is a complex and multifaceted problem, and you'll probably suggest postponing it to a later version. You're probably right, but I want to consider it at least now.

Why not implement at least part of the solution with the best tool we already have in programming: `git`? It has everything we need. We could have `.ctxt/` folder under `git` control, as a `git` project. The parent folder of `.ctxt/` can  also be under `git`, obviously, but is a different git project: is the user project (thatshould always have `.ctxt/` ignored, it's to personal to share.

It seems like a good, reasonable, and consistent idea to me.

---

**Cursor**

## Excellent Idea - Let's Analyze Feasibility

Your instinct is right: this is **elegant and architecturally sound**. Let me break down why and when.

---

## Why This Works

### 1. **Nested Git is Standard Practice**
```bash
project/               # User's repo
├── .git/             # Project version control
├── .gitignore        # Contains: .ctxt/
└── .ctxt/            # ctxt's repo (independent)
    ├── .git/         # Separate version control
    └── .gitignore    # Protects sensitive logs
```

- Parent git ignores `.ctxt/` entirely
- No submodule complexity needed
- Tools handle this fine (common pattern for vendor dirs, caches, etc.)

### 2. **Solves Your Exact Problems**

| Problem | Git Solution |
|---------|--------------|
| Lost `meta/` content | `git log`, `git diff`, `git checkout` |
| Traceability | Full commit history with messages |
| Personal data leakage | Parent `.gitignore` prevents accidental sharing |
| Meta-memory | Branches, tags, reflog for exploration |
| Recovery | Native `git revert`, `git restore` |

### 3. **Architectural Fit**

Looking at your plan's directory structure:

```
.ctxt/
├── config.yml         # Version controlled ✓
├── sources/           # Discovery history ✓
├── context/           # Generated context versions ✓
├── outputs/           # Formatter results ✓
├── history.json       # Could simplify or eliminate
└── logs/              # Maybe .gitignore sensitive ones
```

**Every file benefits from versioning** - you want to see:
- "What did `python_env` discover last Tuesday?"
- "When did my git config change?"
- "Revert to context before LLM mangled it"

---

## Implementation Timing

### Option A: **M1 (Foundation)** ✅ Recommended
**Rationale:** Directory structure is established in M1. Initialize git repo then.

**Tasks:**
- `.ctxt/` finder also checks if it's a git repo
- `ctxt init` creates `.ctxt/` as git repo with initial commit
- Add `.ctxt/` to parent project's `.gitignore` (with user permission)
- Document in M1 that `.ctxt/` is versioned

**Pros:** Clean from start, no migration needed  
**Cons:** Slightly more M1 complexity

### Option B: **M3 (Aggregation)** - When we start writing heavily
**Rationale:** Defer until we actually produce significant content

**Pros:** Simpler M1/M2  
**Cons:** Migration burden, loses early history

### Option C: **v0.2** - Post-launch enhancement
**Rationale:** Ship v0.1 simpler, add sophistication later

**Pros:** Minimal v0.1 scope  
**Cons:** Users have non-git `.ctxt/` to migrate

---

## Recommended Design

### Automatic Initialization
```python
def ensure_ctxt_dir(path: Path) -> Path:
    """Find or create nearest .ctxt/ directory"""
    ctxt_dir = path / ".ctxt"
    
    if not ctxt_dir.exists():
        ctxt_dir.mkdir()
        _init_git_repo(ctxt_dir)  # <-- New
        _create_gitignore(ctxt_dir)
    
    return ctxt_dir

def _init_git_repo(ctxt_dir: Path):
    """Initialize .ctxt/ as git repository"""
    subprocess.run(["git", "init"], cwd=ctxt_dir, capture_output=True)
    subprocess.run(["git", "add", "."], cwd=ctxt_dir)
    subprocess.run([
        "git", "commit", "-m", 
        "Initialize ctxt discovery workspace"
    ], cwd=ctxt_dir)
```

### Smart Gitignore
`.ctxt/.gitignore`:
```gitignore
# Ignore logs with potential secrets
logs/*_sensitive.log

# Ephemeral work (though this goes in .ctxt.work/ now)
*.tmp
```

### Optional CLI Sugar
```bash
ctxt log        # git log in .ctxt/
ctxt diff       # git diff in .ctxt/
ctxt commit -m  # git commit in .ctxt/
```

---

## Tradeoffs

### Benefits
✅ Robust traceability (mature tool)  
✅ No custom history system needed  
✅ Privacy preserved (parent ignores `.ctxt/`)  
✅ Optional remote backup (push to private repo)  
✅ Familiar workflow for developers  
✅ Diff/blame/bisect for debugging  

### Concerns
⚠️ Users might forget `.ctxt/` is separate repo  
⚠️ Adds git dependency (already needed for `GitSource`)  
⚠️ Slightly more complex initialization  
⚠️ Auto-commits might feel "magical"  

---

## My Recommendation

**Implement in M1 (Foundation)** for these reasons:

1. **Minimal added complexity** - We're already creating `.ctxt/` structure
2. **Prevents history loss** - Protects against your exact pain point
3. **Aligns with architecture** - "Git-like search" already in plan
4. **Solves real problem** - Not premature optimization, addresses actual need
5. **Clean semantics** - `.ctxt/` = personal workspace = private repo

**Addition to M1 tasks:**
```markdown
- [ ] Initialize `.ctxt/` as git repository on creation
- [ ] Create `.ctxt/.gitignore` for sensitive logs
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (with permission)
- [ ] Auto-commit on major operations (source collection, aggregation)
```

**Commit strategy:**
- After source discovery: `git commit -m "Discover: python_env, git, nbdev"`
- After aggregation: `git commit -m "Aggregate context (4 fragments)"`
- After formatting: `git commit -m "Format: cursor, generic"`

---

## Question for You

Do you want:

**A) Full auto-commit** - `ctxt` commits after each phase automatically  
**B) Manual control** - User runs `git commit` in `.ctxt/` when desired  
**C) Hybrid** - Auto-commit with `--no-commit` flag to disable

My vote: **A (full auto-commit)** - The whole point is effortless traceability. Make it disappear.

---

**Your call:** Should we add this to M1, or would you prefer to see v0.1 ship simpler and add this in v0.2?

```shellscript
project/               # User's repo
├── .git/             # Project version control
├── .gitignore        # Contains: .ctxt/
└── .ctxt/            # ctxt's repo (independent)
    ├── .git/         # Separate version control
    └── .gitignore    # Protects sensitive logs
```

```plaintext
.ctxt/
├── config.yml         # Version controlled ✓
├── sources/           # Discovery history ✓
├── context/           # Generated context versions ✓
├── outputs/           # Formatter results ✓
├── history.json       # Could simplify or eliminate
└── logs/              # Maybe .gitignore sensitive ones
```

```python
def ensure_ctxt_dir(path: Path) -> Path:
    """Find or create nearest .ctxt/ directory"""
    ctxt_dir = path / ".ctxt"
    
    if not ctxt_dir.exists():
        ctxt_dir.mkdir()
        _init_git_repo(ctxt_dir)  # <-- New
        _create_gitignore(ctxt_dir)
    
    return ctxt_dir

def _init_git_repo(ctxt_dir: Path):
    """Initialize .ctxt/ as git repository"""
    subprocess.run(["git", "init"], cwd=ctxt_dir, capture_output=True)
    subprocess.run(["git", "add", "."], cwd=ctxt_dir)
    subprocess.run([
        "git", "commit", "-m", 
        "Initialize ctxt discovery workspace"
    ], cwd=ctxt_dir)
```

```plaintext
# Ignore logs with potential secrets
logs/*_sensitive.log

# Ephemeral work (though this goes in .ctxt.work/ now)
*.tmp
```

```shellscript
ctxt log        # git log in .ctxt/
ctxt diff       # git diff in .ctxt/
ctxt commit -m  # git commit in .ctxt/
```

```markdown
- [ ] Initialize `.ctxt/` as git repository on creation
- [ ] Create `.ctxt/.gitignore` for sensitive logs
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (with permission)
- [ ] Auto-commit on major operations (source collection, aggregation)
```

---

**User**

Full auto-commit, full `ctxt` responsability. The contents of `.ctxt/` are managed by `ctxt`. Of course, it's completely transparent (everything is "text") and the user can view and modify it, but isn't best practice. It he wants to change something, should use the CLI.



Future versions will consider the Hybrid approach after we discuss semantics.

---

**Cursor**

## Perfect - Clean Separation of Concerns

**Principle:** `.ctxt/` is ctxt's managed workspace. Git is the implementation, CLI is the interface.

---

## Updated M1 Design

### New Component: Git Operations Module

Add to `05_core.ipynb`:

```python
class CtxtRepo:
    """Manages .ctxt/ as a git repository"""
    
    def __init__(self, ctxt_path: Path):
        self.path = ctxt_path
        self.git_dir = ctxt_path / ".git"
    
    def is_initialized(self) -> bool:
        """Check if .ctxt/ is a git repo"""
        return self.git_dir.exists()
    
    def initialize(self) -> None:
        """Initialize .ctxt/ as git repository"""
        if self.is_initialized():
            return
        
        subprocess.run(["git", "init"], cwd=self.path, check=True, 
                      capture_output=True)
        
        # Create .gitignore
        gitignore = self.path / ".gitignore"
        gitignore.write_text("*.tmp\n.DS_Store\n")
        
        # Initial commit
        self.commit("Initialize ctxt workspace", allow_empty=True)
    
    def commit(self, message: str, allow_empty: bool = False) -> None:
        """Auto-commit current state"""
        subprocess.run(["git", "add", "-A"], cwd=self.path, check=True,
                      capture_output=True)
        
        cmd = ["git", "commit", "-m", message]
        if allow_empty:
            cmd.append("--allow-empty")
        
        result = subprocess.run(cmd, cwd=self.path, capture_output=True)
        # Don't check=True: "nothing to commit" is acceptable
    
    def log(self, n: int = 10) -> list[str]:
        """Get recent commit messages"""
        result = subprocess.run(
            ["git", "log", f"-{n}", "--oneline"],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout.strip().split("\n") if result.returncode == 0 else []
    
    def diff(self, ref: str = "HEAD") -> str:
        """Show changes since ref"""
        result = subprocess.run(
            ["git", "diff", ref],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout if result.returncode == 0 else ""
```

---

## Commit Point Strategy

### Automatic Commits Triggered By:

```python
# M2: After source collection
repo.commit("Discover: python_env, git, nbdev (3 sources, 847 tokens)")

# M3: After aggregation  
repo.commit("Aggregate: 3 sources → unified context (12 fragments merged)")

# M4: After formatting
repo.commit("Format: cursor, generic (2 outputs)")

# M5: On config changes
repo.commit("Update config: enable source 'docker'")
```

### Message Format
```
<operation>: <summary> (<metrics>)

Examples:
- "Discover: python_env, git (2 sources, 423 tokens)"
- "Aggregate: 4 sources → context (sensitivity: low)"
- "Format: cursor (1 output, 3 files)"
- "Update config: disable formatter 'generic'"
```

---

## Impact on Architecture

### M1 Tasks (Updated)

```markdown
**M1: Foundation**

- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on first use
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes (ContextSource, Aggregator, Formatter, LLM)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton
```

### M5: History (Simplified)

**Before:** Custom `history.json` with fingerprints, permissions, change detection  
**After:** Lightweight metadata, git provides history

`history.json` becomes:
```json
{
  "last_run": "2025-11-10T14:23:00Z",
  "permissions": {
    "sources": ["python_env", "git", "nbdev"],
    "formatters": ["cursor", "generic"]
  },
  "user_preferences": {
    "auto_discover": true,
    "vendor_default": "cursor"
  }
}
```

**Git provides:**
- Change detection (`git diff`)
- Discovery history (`git log sources/`)
- Content rollback (`git checkout <ref>`)
- Fingerprinting (commit SHAs)

---

## CLI Additions (M5)

### For Advanced Users (Optional)

```bash
ctxt history           # Show git log with pretty formatting
ctxt diff              # Show changes since last discovery
ctxt show <ref>        # Show context state at commit

# Future (v0.2):
ctxt rollback          # Interactive: pick commit to restore
ctxt --no-commit       # Disable auto-commit for this run
```

---

## Safety Considerations

### Git Dependency
```python
def check_git_available() -> bool:
    """Verify git is installed"""
    try:
        subprocess.run(["git", "--version"], capture_output=True, check=True)
        return True
    except (FileNotFoundError, subprocess.CalledProcessError):
        return False
```

**Failure Mode:** If git unavailable:
- Log warning to `.ctxt/logs/warning.log`
- Continue without versioning
- Inform user: "Install git for full traceability"

### Parent .gitignore Update

```python
def suggest_ignore_ctxt(project_root: Path) -> None:
    """Prompt to add .ctxt/ to parent .gitignore"""
    gitignore = project_root / ".gitignore"
    
    if not gitignore.exists():
        return  # No git repo in parent
    
    content = gitignore.read_text()
    if ".ctxt/" in content or ".ctxt" in content:
        return  # Already ignored
    
    # Only prompt once (tracked in .ctxt/history.json)
    print("⚠️  Add .ctxt/ to your project's .gitignore? (recommended)")
    print("   This keeps your personal context workspace private.")
    # ... interactive prompt ...
```

---

## Benefits of This Approach

### 1. **Solves Your Pain Point**
```bash
# Yesterday's loss would have been prevented:
cd .ctxt
git log --oneline
git show HEAD~3:context/SUMMARY.md  # View 3 commits ago
```

### 2. **Enables Future Features**
- **Sync:** Optional `git remote` for backup/sync across machines
- **Branching:** Experiment with different discovery configs
- **Collaboration:** Share `.ctxt/` between team members (rare, but possible)

### 3. **Developer-Friendly**
```bash
# Power users can still use git directly:
cd .ctxt
git log --stat            # See what changed when
git blame config.yml      # When was this setting added?
git bisect                # Find when discovery broke
```

### 4. **Audit Trail**
Every change is timestamped, attributed, and reversible.

---

## Updated Milestone Estimate

| Milestone | Original | With Git | Delta |
|-----------|----------|----------|-------|
| M1        | 1 session | 1 session | +30min for CtxtRepo |
| M5        | 1 session | 0.5 session | -30min (simpler history) |

**Net: Zero time cost** - Complexity shifts from custom history to git wrapper.

---

## Next Steps (When Ready for Agent Mode)

1. **Execute M0** as originally planned
2. **Start M1** with updated design:
   - `05_core.ipynb` includes `CtxtRepo` class
   - `.ctxt/` auto-initializes as git repo
   - First test: `ctxt init` creates versioned workspace

---

**Confirmation:** This matches your vision? Full auto-commit, ctxt manages `.ctxt/` completely, git is transparent implementation detail?

If yes, we're ready to start coding when you switch to agent mode. M0 first (preservation), then M1 with git integration.

```python
class CtxtRepo:
    """Manages .ctxt/ as a git repository"""
    
    def __init__(self, ctxt_path: Path):
        self.path = ctxt_path
        self.git_dir = ctxt_path / ".git"
    
    def is_initialized(self) -> bool:
        """Check if .ctxt/ is a git repo"""
        return self.git_dir.exists()
    
    def initialize(self) -> None:
        """Initialize .ctxt/ as git repository"""
        if self.is_initialized():
            return
        
        subprocess.run(["git", "init"], cwd=self.path, check=True, 
                      capture_output=True)
        
        # Create .gitignore
        gitignore = self.path / ".gitignore"
        gitignore.write_text("*.tmp\n.DS_Store\n")
        
        # Initial commit
        self.commit("Initialize ctxt workspace", allow_empty=True)
    
    def commit(self, message: str, allow_empty: bool = False) -> None:
        """Auto-commit current state"""
        subprocess.run(["git", "add", "-A"], cwd=self.path, check=True,
                      capture_output=True)
        
        cmd = ["git", "commit", "-m", message]
        if allow_empty:
            cmd.append("--allow-empty")
        
        result = subprocess.run(cmd, cwd=self.path, capture_output=True)
        # Don't check=True: "nothing to commit" is acceptable
    
    def log(self, n: int = 10) -> list[str]:
        """Get recent commit messages"""
        result = subprocess.run(
            ["git", "log", f"-{n}", "--oneline"],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout.strip().split("\n") if result.returncode == 0 else []
    
    def diff(self, ref: str = "HEAD") -> str:
        """Show changes since ref"""
        result = subprocess.run(
            ["git", "diff", ref],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout if result.returncode == 0 else ""
```

```python
# M2: After source collection
repo.commit("Discover: python_env, git, nbdev (3 sources, 847 tokens)")

# M3: After aggregation  
repo.commit("Aggregate: 3 sources → unified context (12 fragments merged)")

# M4: After formatting
repo.commit("Format: cursor, generic (2 outputs)")

# M5: On config changes
repo.commit("Update config: enable source 'docker'")
```

```plaintext
<operation>: <summary> (<metrics>)

Examples:
- "Discover: python_env, git (2 sources, 423 tokens)"
- "Aggregate: 4 sources → context (sensitivity: low)"
- "Format: cursor (1 output, 3 files)"
- "Update config: disable formatter 'generic'"
```

```markdown
**M1: Foundation**

- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on first use
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes (ContextSource, Aggregator, Formatter, LLM)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton
```

```json
{
  "last_run": "2025-11-10T14:23:00Z",
  "permissions": {
    "sources": ["python_env", "git", "nbdev"],
    "formatters": ["cursor", "generic"]
  },
  "user_preferences": {
    "auto_discover": true,
    "vendor_default": "cursor"
  }
}
```

```shellscript
ctxt history           # Show git log with pretty formatting
ctxt diff              # Show changes since last discovery
ctxt show <ref>        # Show context state at commit

# Future (v0.2):
ctxt rollback          # Interactive: pick commit to restore
ctxt --no-commit       # Disable auto-commit for this run
```

```python
def check_git_available() -> bool:
    """Verify git is installed"""
    try:
        subprocess.run(["git", "--version"], capture_output=True, check=True)
        return True
    except (FileNotFoundError, subprocess.CalledProcessError):
        return False
```

```python
def suggest_ignore_ctxt(project_root: Path) -> None:
    """Prompt to add .ctxt/ to parent .gitignore"""
    gitignore = project_root / ".gitignore"
    
    if not gitignore.exists():
        return  # No git repo in parent
    
    content = gitignore.read_text()
    if ".ctxt/" in content or ".ctxt" in content:
        return  # Already ignored
    
    # Only prompt once (tracked in .ctxt/history.json)
    print("⚠️  Add .ctxt/ to your project's .gitignore? (recommended)")
    print("   This keeps your personal context workspace private.")
    # ... interactive prompt ...
```

```shellscript
# Yesterday's loss would have been prevented:
cd .ctxt
git log --oneline
git show HEAD~3:context/SUMMARY.md  # View 3 commits ago
```

```shellscript
# Power users can still use git directly:
cd .ctxt
git log --stat            # See what changed when
git blame config.yml      # When was this setting added?
git bisect                # Find when discovery broke
```

---

**User**

Would it be necessary to add all of this to the plan and the supplementary metadata documents? Remember that `Cursor` discards the context at any moment without warning, and we could lose this very important discussion.



I, messy human, won't remember most of this soon, but for you, my highly competent fellow programmer with a perfect memory (that is, context), to forget it would be a tragedy.



And this smoothly connect with another point I will want to addres after handling this one.

---

**Cursor**

## Absolutely Critical - Let's Capture This Now

You're completely right. This is **architectural decision #6** and must be documented immediately.

I need to update:
1. `meta/plan.md` - Add git integration to architecture + M1
2. `meta/HANDOFF.md` - Capture decision in session summary
3. Create `meta/decisions.md` - Architecture decision record (new)

---

## Required Updates

### 1. `meta/plan.md` Changes

**Section: Core Architecture** (add after "Components:")
```markdown
**Git Integration:**
- `.ctxt/` is a git repository managed by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern
```

**Section: Directory Structure** (update comment):
```markdown
.ctxt/                        # Git repo - managed by ctxt
├── .git/                     # Version control (auto-managed)
├── .gitignore                # Protects sensitive files
├── config.yml                # Merged configuration (write to nearest)
```

**M1 Tasks** (add):
```markdown
- [ ] CtxtRepo class - git operations wrapper (init, commit, log, diff)
- [ ] Auto-initialize `.ctxt/` as git repo on creation
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] Auto-commit after major operations with descriptive messages
```

**M5 Tasks** (simplify):
```markdown
- [ ] `History` class - lightweight metadata (permissions, preferences)
- [ ] Change detection via `git diff` wrapper
- [ ] CLI: `ctxt history`, `ctxt diff`, `ctxt show`
```

**Key Decisions** (add new #6):
```markdown
**6. Git-Based Traceability:** `.ctxt/` is a git repository. Auto-commit after operations. Full history, recovery, and audit trail. Parent repo ignores `.ctxt/`.
```

---

### 2. `meta/HANDOFF.md` Changes

**Last Session Summary** (add):
```markdown
**Accomplished:**
- [previous items...]
- **CRITICAL DECISION:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository
  - Auto-commit after discover/aggregate/format operations
  - Provides traceability and prevents data loss
  - Simplifies M5 (history via git, not custom JSON)
  - Added to M1 tasks: CtxtRepo class

**Design Rationale:**
- Solves actual pain point (lost meta/ content yesterday)
- Git provides mature traceability/recovery for free
- Nested repo pattern (no submodules) - common practice
- Parent .gitignore ensures privacy
- Net zero time cost (M1 +30min, M5 -30min)
```

**Next Session** (update M1):
```markdown
2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure
```

---

### 3. `meta/decisions.md` (NEW FILE)

```markdown
# Architecture Decision Records

## ADR-001: Git-Based Workspace Management

**Date:** 2025-11-10  
**Status:** Accepted  
**Context:** v0.1 discovery architecture rewrite

### Problem

1. Lost `meta/` content during development (no version control)
2. Need traceability: "What did ctxt discover last Tuesday?"
3. Need recovery: "Revert context before LLM mangled it"
4. `.ctxt/` contains personal/sensitive data - must not leak to project repo

### Decision

**`.ctxt/` is a git repository managed entirely by ctxt.**

- Auto-initialize on first use (`git init`)
- Auto-commit after operations with descriptive messages
- Parent project `.gitignore` must include `.ctxt/` (prompt user once)
- Nested repo pattern (no submodules)
- User can inspect but should not manually edit
- CLI provides `ctxt history`, `ctxt diff`, `ctxt show` for access

### Consequences

**Positive:**
- Robust traceability (mature tool)
- No custom history system needed (simplifies M5)
- Optional remote backup (user can add git remote)
- Familiar git workflow for advanced users
- Prevents data loss from context resets

**Negative:**
- Requires git installed (already needed for GitSource)
- Slightly more complex initialization (+30min M1)
- Auto-commits might surprise users (but documented)

**Neutral:**
- M1 complexity +30min, M5 complexity -30min (net zero)

### Implementation

**M1:** Add `CtxtRepo` class to `05_core.ipynb`
```python
class CtxtRepo:
    def initialize() -> None
    def commit(message: str) -> None
    def log(n: int) -> list[str]
    def diff(ref: str) -> str
```

**Commit Strategy:**
- After discovery: `"Discover: python_env, git (2 sources, 423 tokens)"`
- After aggregation: `"Aggregate: 3 sources → context (12 fragments)"`
- After formatting: `"Format: cursor, generic (2 outputs)"`

**M5:** Simplify `history.json` to preferences only, delegate versioning to git
```

---

## What I Need From You

**Switch to agent mode** so I can:

1. Update `meta/plan.md` (3 sections)
2. Update `meta/HANDOFF.md` (2 sections)  
3. Create `meta/decisions.md` (new file)

**Or** copy the markdown above and update manually (your choice).

---

## Your Second Point?

You mentioned this connects to another issue you want to address. I suspect it's about **documentation persistence strategy** itself - how we ensure critical decisions like this don't get lost.

Is it:
- **A)** Where to capture design discussions (plan vs decisions vs notes)?
- **B)** How to sync meta/ across sessions (git + remote)?
- **C)** What belongs in Cursor rules vs meta/ docs?
- **D)** Something else entirely?

I'm ready to discuss once we've captured this git decision. Should I wait for agent mode, or continue discussion first?

```markdown
**Git Integration:**
- `.ctxt/` is a git repository managed by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern
```

```markdown
.ctxt/                        # Git repo - managed by ctxt
├── .git/                     # Version control (auto-managed)
├── .gitignore                # Protects sensitive files
├── config.yml                # Merged configuration (write to nearest)
```

```markdown
- [ ] CtxtRepo class - git operations wrapper (init, commit, log, diff)
- [ ] Auto-initialize `.ctxt/` as git repo on creation
- [ ] Suggest adding `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] Auto-commit after major operations with descriptive messages
```

```markdown
- [ ] `History` class - lightweight metadata (permissions, preferences)
- [ ] Change detection via `git diff` wrapper
- [ ] CLI: `ctxt history`, `ctxt diff`, `ctxt show`
```

```markdown
**6. Git-Based Traceability:** `.ctxt/` is a git repository. Auto-commit after operations. Full history, recovery, and audit trail. Parent repo ignores `.ctxt/`.
```

```markdown
**Accomplished:**
- [previous items...]
- **CRITICAL DECISION:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository
  - Auto-commit after discover/aggregate/format operations
  - Provides traceability and prevents data loss
  - Simplifies M5 (history via git, not custom JSON)
  - Added to M1 tasks: CtxtRepo class

**Design Rationale:**
- Solves actual pain point (lost meta/ content yesterday)
- Git provides mature traceability/recovery for free
- Nested repo pattern (no submodules) - common practice
- Parent .gitignore ensures privacy
- Net zero time cost (M1 +30min, M5 -30min)
```

```markdown
2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure
```

```markdown
# Architecture Decision Records

## ADR-001: Git-Based Workspace Management

**Date:** 2025-11-10  
**Status:** Accepted  
**Context:** v0.1 discovery architecture rewrite

### Problem

1. Lost `meta/` content during development (no version control)
2. Need traceability: "What did ctxt discover last Tuesday?"
3. Need recovery: "Revert context before LLM mangled it"
4. `.ctxt/` contains personal/sensitive data - must not leak to project repo

### Decision

**`.ctxt/` is a git repository managed entirely by ctxt.**

- Auto-initialize on first use (`git init`)
- Auto-commit after operations with descriptive messages
- Parent project `.gitignore` must include `.ctxt/` (prompt user once)
- Nested repo pattern (no submodules)
- User can inspect but should not manually edit
- CLI provides `ctxt history`, `ctxt diff`, `ctxt show` for access

### Consequences

**Positive:**
- Robust traceability (mature tool)
- No custom history system needed (simplifies M5)
- Optional remote backup (user can add git remote)
- Familiar git workflow for advanced users
- Prevents data loss from context resets

**Negative:**
- Requires git installed (already needed for GitSource)
- Slightly more complex initialization (+30min M1)
- Auto-commits might surprise users (but documented)

**Neutral:**
- M1 complexity +30min, M5 complexity -30min (net zero)

### Implementation

**M1:** Add `CtxtRepo` class to `05_core.ipynb`
class CtxtRepo:
    def initialize() -> None
    def commit(message: str) -> None
    def log(n: int) -> list[str]
    def diff(ref: str) -> str**Commit Strategy:**
- After discovery: `"Discover: python_env, git (2 sources, 423 tokens)"`
- After aggregation: `"Aggregate: 3 sources → context (12 fragments)"`
- After formatting: `"Format: cursor, generic (2 outputs)"`

**M5:** Simplify `history.json` to preferences only, delegate versioning to git
```

```plaintext

---

## What I Need From You

**Switch to agent mode** so I can:

1. Update `meta/plan.md` (3 sections)
2. Update `meta/HANDOFF.md` (2 sections)  
3. Create `meta/decisions.md` (new file)

**Or** copy the markdown above and update manually (your choice).

---

## Your Second Point?

You mentioned this connects to another issue you want to address. I suspect it's about **documentation persistence strategy** itself - how we ensure critical decisions like this don't get lost.

Is it:
- **A)** Where to capture design discussions (plan vs decisions vs notes)?
- **B)** How to sync meta/ across sessions (git + remote)?
- **C)** What belongs in Cursor rules vs meta/ docs?
- **D)** Something else entirely?

I'm ready to discuss once we've captured this git decision. Should I wait for agent mode, or continue discussion first?
```

---

**User**

`Agent` mode, let's secure this decission. Additional discussion later.

---

**Cursor**

I'll secure this architectural decision by updating the documentation files now.

Now I'll update all three documentation files to capture this critical architectural decision.

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git-like: search CWD → ~/, aggregate configs
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata, fingerprints, permissions
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, base classes)

**Deliverable:** Can find `.ctxt/` dirs, log output, has plugin contracts defined

**Time:** 1 session

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - load/save `history.json`
- [ ] Change detection - diff against previous run
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git repo - managed by ctxt
├── .git/                    # Version control (auto-managed)
├── .gitignore               # Protects sensitive files
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata (lightweight - git provides versioning)
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, base classes)

**Deliverable:** Can find `.ctxt/` dirs, log output, has plugin contracts defined

**Time:** 1 session

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - load/save `history.json`
- [ ] Change detection - diff against previous run
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git repo - managed by ctxt
├── .git/                    # Version control (auto-managed)
├── .gitignore               # Protects sensitive files
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata (lightweight - git provides versioning)
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on creation
- [ ] **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] **Auto-commit** after major operations with descriptive messages
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, CtxtRepo, base classes)

**Deliverable:** Can find `.ctxt/` dirs, initialize as git repos, log output, has plugin contracts defined

**Time:** 1 session (+30min for git integration)

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - load/save `history.json`
- [ ] Change detection - diff against previous run
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git repo - managed by ctxt
├── .git/                    # Version control (auto-managed)
├── .gitignore               # Protects sensitive files
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata (lightweight - git provides versioning)
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on creation
- [ ] **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] **Auto-commit** after major operations with descriptive messages
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, CtxtRepo, base classes)

**Deliverable:** Can find `.ctxt/` dirs, initialize as git repos, log output, has plugin contracts defined

**Time:** 1 session (+30min for git integration)

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - lightweight metadata (permissions, preferences only)
- [ ] Change detection via `git diff` wrapper
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
  - `ctxt history` - show git log with formatting
  - `ctxt diff` - show changes since last discovery
  - `ctxt show` - show context state at commit
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence, git operations

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session (-30min, simplified by git integration)

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git repo - managed by ctxt
├── .git/                    # Version control (auto-managed)
├── .gitignore               # Protects sensitive files
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata (lightweight - git provides versioning)
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on creation
- [ ] **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] **Auto-commit** after major operations with descriptive messages
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, CtxtRepo, base classes)

**Deliverable:** Can find `.ctxt/` dirs, initialize as git repos, log output, has plugin contracts defined

**Time:** 1 session (+30min for git integration)

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - lightweight metadata (permissions, preferences only)
- [ ] Change detection via `git diff` wrapper
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
  - `ctxt history` - show git log with formatting
  - `ctxt diff` - show changes since last discovery
  - `ctxt show` - show context state at commit
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence, git operations

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session (-30min, simplified by git integration)

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

**6. Git-Based Traceability:** `.ctxt/` is a git repository managed by ctxt. Auto-commit after operations (discover, aggregate, format) with descriptive messages. Provides full history, recovery, audit trail. Parent repo ignores `.ctxt/` for privacy. Simplifies M5 (git replaces custom history system).

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

```markdown
## ctxt v0.1 - Plan

### Core Architecture

**Pipeline:** Discovery → Aggregation → Formatting → History

**Components:**
- **Context Sources:** Pluggable modules, black-box API, discover one aspect of environment
- **Aggregator:** Merges fragments, resolves conflicts, produces model-independent context
- **Formatters:** Adapt unified context to vendor formats (Cursor, generic markdown)
- **History:** Track what was found, enable incremental runs
- **LLM:** Noop hooks throughout (identity function), ready for future enhancement

**Git Integration:**
- `.ctxt/` is a git repository managed entirely by ctxt
- Auto-commit after each operation (discover, aggregate, format)
- Provides traceability, recovery, and audit trail
- Parent project ignores `.ctxt/` (personal workspace)
- No submodules - simple nested repo pattern

---

### Directory Structure

```
.ctxt/                        # Git repo - managed by ctxt
├── .git/                    # Version control (auto-managed)
├── .gitignore               # Protects sensitive files
├── config.yml               # Merged configuration (write to nearest)
├── sources/                 # Raw discovered fragments (for debugging)
│   ├── python_env.json
│   ├── git.json
│   └── nbdev.json
├── context/                 # Aggregated, model-independent
│   ├── context.json         # Structured
│   └── SUMMARY.md           # Human-readable
├── outputs/                 # Formatted for vendors
│   ├── Cursor/              # User copies/symlinks as needed
│   └── generic/
├── history.json             # Last run metadata (lightweight - git provides versioning)
└── logs/                    # Warnings, audit trails

.ctxt.work/                  # Ephemeral (gitignored, Model 03's idea)
└── snapshot/                # Quick context dumps for one-off use
```

**Write rule:** Only write to nearest `.ctxt/` (create if absent). Parent/home `.ctxt/` are read-only.

---

### v0.1 Sources

**Ship with 4:**
1. **python_env** - interpreter, venv type, top-N packages
2. **git** - repo root, branch, remotes (redacted), user name/email
3. **nbdev** - detect `settings.ini`, notebook structure
4. **ctxt** - existing `.ctxt.yml`, roll up from search path

**Safety:** Read broadly, redact values. Detect env var names only (e.g., "ANTHROPIC_API_KEY exists" not the value).

---

### Plugin API (Minimal Contracts)

```python
# ContextSource
class ContextSource:
    def probe(ctx: Context) -> bool:
        """Quick check if source applies"""
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        """Discover and return fragments"""

# Fragment
@dataclass
class Fragment:
    source: str              # "python_env"
    kind: str                # "config" | "env" | "docs"
    content: dict[str, Any]  # Structured data
    confidence: float        # 0.0-1.0
    sensitivity: str         # "low" | "medium" | "high"
    provenance: dict         # path, timestamp, hash
    tokens_est: int

# Aggregator
class Aggregator:
    def assemble(fragments: list[Fragment], llm: LLM) -> UnifiedContext:
        """Merge, dedupe, resolve conflicts"""

# Formatter
class Formatter:
    def format(context: UnifiedContext) -> dict[Path, str]:
        """Return {filepath: content} to write"""

# LLM (noop default)
class LLM:
    def enhance(input: str) -> str:
        """Default: return input unchanged"""
```

---

### Milestones

**M0: Transition & Preservation**

**Goal:** Archive current working version, prepare clean slate

**Tasks:**
- [ ] Commit all current work with clear message
- [ ] Tag current state as `v0.0.9-template-mvp`
- [ ] Create branch `v0.1-discovery-rewrite`
- [ ] Archive old notebooks:
  - Move `02_tags.ipynb` → `_archive/02_tags.ipynb`
  - Move `03_templates.ipynb` → `_archive/03_templates.ipynb`
  - Move `04_cli.ipynb` → `_archive/04_cli_old.ipynb`
- [ ] Update README with transition note:
  ```markdown
  ## Version Notice
  **Current:** v0.1 (discovery-based architecture)
  **Previous:** v0.0.9 template-based version available at tag `v0.0.9-template-mvp`
  
  To use old version: `pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp`
  ```
- [ ] Update `.gitignore` to include `.ctxt.work/`
- [ ] Create `_archive/README.md` explaining what's there and why
- [ ] Refactor `01_config.ipynb` to support `.ctxt/` discovery (keep reusable parts)

**Deliverable:** Clean working tree ready for new development, old version preserved and accessible

**Time:** 30 minutes

---

**M1: Foundation**

**Goal:** Core infrastructure for discovery architecture

**Tasks:**
- [ ] `.ctxt/` finder - walk up tree (CWD → ~/), collect all `.ctxt/` directories
- [ ] **CtxtRepo class** - git operations wrapper (init, commit, log, diff)
- [ ] **Auto-initialize** `.ctxt/` as git repo on creation
- [ ] **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
- [ ] **Auto-commit** after major operations with descriptive messages
- [ ] Config merger - aggregate configs by precedence (nearest wins)
- [ ] Base classes:
  - `ContextSource` (ABC with `probe()`, `collect()`)
  - `Aggregator` (ABC with `assemble()`)
  - `Formatter` (ABC with `format()`)
  - `LLM` (ABC with `enhance()`, default `NoopLLM`)
- [ ] Logging infrastructure - write to `.ctxt/logs/`
- [ ] Silent-by-default CLI skeleton

**Notebook:** `05_core.ipynb` (Fragment, Context, CtxtRepo, base classes)

**Deliverable:** Can find `.ctxt/` dirs, initialize as git repos, log output, has plugin contracts defined

**Time:** 1 session (+30min for git integration)

---

**M2: Sources**

**Goal:** Implement 4 core context sources

**Tasks:**
- [ ] `PythonEnvSource` - sys.version, venv detection, top-20 packages
- [ ] `GitSource` - repo root, branch, user.name/email (local+global), remotes (redacted)
- [ ] `NbdevSource` - detect settings.ini, count notebooks, extract lib_name
- [ ] `CtxtSource` - read existing `.ctxt.yml`, merge from discovered paths
- [ ] Redaction utilities - strip sensitive values, keep names only
- [ ] Token estimation - rough character count heuristics

**Notebook:** `06_sources.ipynb`

**Tests:** Mock filesystems, verify each source works independently

**Deliverable:** 4 working sources that return `Fragment` objects

**Time:** 1-2 sessions

---

**M3: Aggregation**

**Goal:** Merge fragments into unified context

**Tasks:**
- [ ] `SimpleAggregator` - merge by source priority, no LLM
- [ ] Conflict resolution - last-wins or annotate conflicts
- [ ] Deduplication - hash-based fragment matching
- [ ] Fingerprinting - content hashing for change detection
- [ ] Produce outputs:
  - `context.json` (structured, with provenance)
  - `SUMMARY.md` (human-readable)
- [ ] LLM hook points (call but get noop response)

**Notebook:** `07_aggregator.ipynb`

**Tests:** Multiple fragments → single context, conflicts handled

**Deliverable:** Working aggregator that produces `context.json` + `SUMMARY.md`

**Time:** 1 session

---

**M4: Formatting**

**Goal:** Adapt unified context to vendor formats

**Tasks:**
- [ ] `GenericFormatter` - plain markdown output
- [ ] `CursorFormatter` - `.mdc` structure for Cursor rules
- [ ] Write to `.ctxt/outputs/{vendor}/`
- [ ] Ownership headers - mark files as generated
- [ ] Template rendering for formatters

**Notebook:** `08_formatters.ipynb`

**Tests:** Same context → different vendor formats

**Deliverable:** 2 working formatters writing to `.ctxt/outputs/`

**Time:** 1 session

---

**M5: History & CLI**

**Goal:** Track discoveries, build user interface

**Tasks:**
- [ ] `History` class - lightweight metadata (permissions, preferences only)
- [ ] Change detection via `git diff` wrapper
- [ ] Permission tracking - record what sources ran
- [ ] CLI commands:
  - `ctxt` - main workflow
  - `ctxt audit` - dry-run, show what would be discovered
  - `ctxt where` - print write-target path
  - `ctxt history` - show git log with formatting
  - `ctxt diff` - show changes since last discovery
  - `ctxt show` - show context state at commit
- [ ] Flags: `--verbose`, `--vendor`, `--force`
- [ ] Silent default - only log to files

**Notebook:** `09_cli.ipynb` (new)

**Tests:** CLI invocations, history persistence, git operations

**Deliverable:** Working CLI, end-to-end discovery pipeline

**Time:** 1 session (-30min, simplified by git integration)

---

**M6: Integration & Polish**

**Goal:** Connect all pieces, validate nbdev workflow

**Tasks:**
- [ ] Integration tests - full pipeline in test project
- [ ] Update `index.ipynb` with new documentation
- [ ] `nbdev_prepare` - regenerate docs
- [ ] Test in multiple project types (nbdev, plain Python, empty dir)
- [ ] Verify `.ctxt/` vs `.ctxt.work/` separation
- [ ] Error handling polish
- [ ] Performance check (should be fast <1s)

**Deliverable:** Shippable v0.1

**Time:** 1 session

---

### What We're NOT Doing (v0.1)

❌ Rulepacks registry (v2)  
❌ CI integration (v2)  
❌ Watch mode (v2)  
❌ Token budgets (v2)  
❌ Actual LLM integration (hooks only)  
❌ Writing outside `.ctxt/` (user manually copies if desired)  
❌ TUI/wizard (silent by default)  
❌ Advanced drift detection (basic fingerprinting only)  

---

### Key Decisions

**1. Silent Mode:** No output unless `--verbose`. Warnings to logs, not stdout.

**2. Failure Mode:** Any source can fail. Aggregator continues. "Generate nothing" is acceptable outcome.

**3. Safety:** Transparent about what's read (log everything). No network. No secrets exfiltration.

**4. Config Merge:** Nearest `.ctxt/` wins. Parents contribute but don't override.

**5. Ephemeral vs Persistent:** `.ctxt/` is persistent, `.ctxt.work/` is gitignored scratch space.

**6. Git-Based Traceability:** `.ctxt/` is a git repository managed by ctxt. Auto-commit after operations (discover, aggregate, format) with descriptive messages. Provides full history, recovery, audit trail. Parent repo ignores `.ctxt/` for privacy. Simplifies M5 (git replaces custom history system).

---

### CLI v0.1

```bash
ctxt                     # Discover → aggregate → format to .ctxt/
ctxt audit               # Dry-run: show what would be discovered
ctxt where               # Print write-target .ctxt/ path
ctxt history             # Show discovery history (git log)
ctxt diff                # Show changes since last run (git diff)
ctxt show <ref>          # Show context at specific commit
ctxt --verbose           # Enable output
ctxt --vendor cursor     # Only generate Cursor formatter
```

---

### Acceptance Criteria

- [ ] Run `ctxt` in any folder, produces `.ctxt/context/SUMMARY.md`
- [ ] Discovers Python env, git config, nbdev (if present)
- [ ] Merges configs from CWD → ~/
- [ ] Writes only to nearest `.ctxt/`
- [ ] No secrets in output
- [ ] LLM hooks present but noop
- [ ] Silent by default, detailed logs in `.ctxt/logs/`
- [ ] All code in notebooks (nbdev)
- [ ] Old version accessible via git tag

---

**Total estimated time:** 6-8 sessions (M0: 0.5h, M1-M5: 1-2 sessions each, M6: 1 session)

```

Now updating `HANDOFF.md`:

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Files Modified:**
- `meta/plan.md` (created) - Complete v0.1 implementation plan
- `meta/HANDOFF.md` (updated) - This file
- `meta/STARTING_PROMPT_TEMPLATE.md`
- `.cursor/rules/project.mdc`

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment and Context dataclasses
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-10)

**Accomplished:**
- **CRITICAL ARCHITECTURAL DECISION:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository (nested repo, not submodule)
  - Auto-commit after discover/aggregate/format operations with descriptive messages
  - Provides traceability, recovery, and prevents data loss
  - Simplifies M5 (history via git, not custom JSON system)
  - Parent project `.gitignore` must include `.ctxt/` (user prompted once)
  - Added `CtxtRepo` class to M1 tasks
- Updated documentation to capture git decision:
  - `meta/plan.md` - Architecture, M1/M5 tasks, Key Decision #6
  - `meta/decisions.md` - Created ADR-001 (Architecture Decision Record)
  - `meta/HANDOFF.md` - This file

**Design Rationale:**
- Solves actual pain point (lost `meta/` content in previous session)
- Git provides mature traceability/recovery infrastructure for free
- Nested repo pattern (common practice, no complexity)
- Full auto-commit: ctxt manages `.ctxt/` entirely, CLI is user interface
- Net zero time cost (M1 +30min, M5 -30min simplified)
- Enables future: remote sync, branching experiments, team collaboration

**Commit Strategy:**
- `"Discover: python_env, git (2 sources, 423 tokens)"`
- `"Aggregate: 3 sources → context (12 fragments)"`
- `"Format: cursor, generic (2 outputs)"`

## Previous Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment and Context dataclasses
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-10)

**Accomplished:**
- **CRITICAL ARCHITECTURAL DECISION:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository (nested repo, not submodule)
  - Auto-commit after discover/aggregate/format operations with descriptive messages
  - Provides traceability, recovery, and prevents data loss
  - Simplifies M5 (history via git, not custom JSON system)
  - Parent project `.gitignore` must include `.ctxt/` (user prompted once)
  - Added `CtxtRepo` class to M1 tasks
- Updated documentation to capture git decision:
  - `meta/plan.md` - Architecture, M1/M5 tasks, Key Decision #6
  - `meta/decisions.md` - Created ADR-001 (Architecture Decision Record)
  - `meta/HANDOFF.md` - This file

**Design Rationale:**
- Solves actual pain point (lost `meta/` content in previous session)
- Git provides mature traceability/recovery infrastructure for free
- Nested repo pattern (common practice, no complexity)
- Full auto-commit: ctxt manages `.ctxt/` entirely, CLI is user interface
- Net zero time cost (M1 +30min, M5 -30min simplified)
- Enables future: remote sync, branching experiments, team collaboration

**Commit Strategy:**
- `"Discover: python_env, git (2 sources, 423 tokens)"`
- `"Aggregate: 3 sources → context (12 fragments)"`
- `"Format: cursor, generic (2 outputs)"`

## Previous Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-10)

**Accomplished:**
- **CRITICAL ARCHITECTURAL DECISION:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository (nested repo, not submodule)
  - Auto-commit after discover/aggregate/format operations with descriptive messages
  - Provides traceability, recovery, and prevents data loss
  - Simplifies M5 (history via git, not custom JSON system)
  - Parent project `.gitignore` must include `.ctxt/` (user prompted once)
  - Added `CtxtRepo` class to M1 tasks
- Updated documentation to capture git decision:
  - `meta/plan.md` - Architecture, M1/M5 tasks, Key Decision #6
  - `meta/decisions.md` - Created ADR-001 (Architecture Decision Record)
  - `meta/HANDOFF.md` - This file

**Design Rationale:**
- Solves actual pain point (lost `meta/` content in previous session)
- Git provides mature traceability/recovery infrastructure for free
- Nested repo pattern (common practice, no complexity)
- Full auto-commit: ctxt manages `.ctxt/` entirely, CLI is user interface
- Net zero time cost (M1 +30min, M5 -30min simplified)
- Enables future: remote sync, branching experiments, team collaboration

**Commit Strategy:**
- `"Discover: python_env, git (2 sources, 423 tokens)"`
- `"Aggregate: 3 sources → context (12 fragments)"`
- `"Format: cursor, generic (2 outputs)"`

## Previous Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-10: Git Integration Architecture Decision
- **Critical decision:** `.ctxt/` as auto-managed git repository
- Documented ADR-001 in `meta/decisions.md`
- Updated M1 (add CtxtRepo) and M5 (simplify via git) in plan
- Rationale: Prevents data loss, provides traceability, enables recovery
- Strategy: Full auto-commit, nested repo pattern, parent ignores `.ctxt/`

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

Now creating the Architecture Decision Record:

```markdown
# Architecture Decision Records

This document captures significant architectural decisions made during ctxt development. Each record includes context, decision, rationale, and consequences.

---

## ADR-001: Git-Based Workspace Management

**Date:** 2025-11-10  
**Status:** Accepted  
**Context:** v0.1 discovery architecture rewrite  
**Participants:** vic, Claude (Sonnet 4.5)

### Problem

During v0.1 development planning, we encountered several critical requirements:

1. **Data Loss Prevention:** Lost `meta/` content during development session (no version control in gitignored directories)
2. **Traceability Need:** Must answer questions like "What did ctxt discover last Tuesday?"
3. **Recovery Capability:** Need ability to revert context before LLM modifications
4. **Privacy Concern:** `.ctxt/` contains personal/sensitive data - must not leak to project repository
5. **Audit Trail:** Regulatory/personal need to track what was discovered and when

### Decision

**`.ctxt/` is a git repository managed entirely by ctxt.**

Implementation details:
- Auto-initialize on first use (`git init` in `.ctxt/`)
- Auto-commit after major operations with descriptive messages
- Parent project `.gitignore` must include `.ctxt/` (prompt user once, remember preference)
- Nested repo pattern (no git submodules)
- User can inspect (read-only observation) but should not manually edit
- CLI provides `ctxt history`, `ctxt diff`, `ctxt show <ref>` for controlled access
- Commit messages follow format: `"<operation>: <summary> (<metrics>)"`

### Rationale

**Why git specifically:**
- Mature, ubiquitous tool (already required for GitSource)
- Proven traceability/recovery infrastructure
- Native diff, blame, bisect for debugging
- Optional remote backup (user can add `git remote`)
- Familiar workflow for developers
- Zero custom history implementation needed

**Why auto-commit:**
- Effortless traceability (core goal)
- Prevents "forgot to save" scenarios
- Consistent snapshot discipline
- User focuses on work, not bookkeeping

**Why nested repo (not submodule):**
- Simpler mental model
- No parent repo coupling
- Common pattern (vendor dirs, caches)
- Works even if parent isn't git-tracked

**Why parent ignores `.ctxt/`:**
- Personal workspace (like `.vscode/`, IDE configs)
- May contain sensitive discovery data
- Not appropriate for project version control
- Each developer has own context needs

### Consequences

**Positive:**
- ✅ Robust traceability using mature tool
- ✅ No custom history system needed (simplifies M5: -30min)
- ✅ Optional remote sync/backup capability
- ✅ Familiar git workflow for advanced users
- ✅ Prevents data loss from context resets
- ✅ Built-in audit trail with timestamps
- ✅ Enables future: branching experiments, team collaboration

**Negative:**
- ⚠️ Requires git installed (already dependency for GitSource)
- ⚠️ Slightly more complex initialization (M1: +30min)
- ⚠️ Auto-commits might surprise users initially (mitigated by docs)
- ⚠️ Disk space for `.git/` history (negligible for text files)

**Neutral:**
- ↔️ M1 complexity +30min, M5 complexity -30min (net zero time cost)
- ↔️ User *can* use git directly in `.ctxt/` but discouraged

### Implementation

**M1 Tasks (Foundation):**

Add `CtxtRepo` class to `05_core.ipynb`:

```python
class CtxtRepo:
    """Manages .ctxt/ as a git repository"""
    
    def __init__(self, ctxt_path: Path):
        self.path = ctxt_path
        self.git_dir = ctxt_path / ".git"
    
    def is_initialized(self) -> bool:
        """Check if .ctxt/ is a git repo"""
        return self.git_dir.exists()
    
    def initialize(self) -> None:
        """Initialize .ctxt/ as git repository"""
        if self.is_initialized():
            return
        
        subprocess.run(["git", "init"], cwd=self.path, check=True, 
                      capture_output=True)
        
        # Create .gitignore for sensitive files
        gitignore = self.path / ".gitignore"
        gitignore.write_text("*.tmp\n.DS_Store\n")
        
        # Initial commit
        self.commit("Initialize ctxt workspace", allow_empty=True)
    
    def commit(self, message: str, allow_empty: bool = False) -> None:
        """Auto-commit current state"""
        subprocess.run(["git", "add", "-A"], cwd=self.path, check=True,
                      capture_output=True)
        
        cmd = ["git", "commit", "-m", message]
        if allow_empty:
            cmd.append("--allow-empty")
        
        result = subprocess.run(cmd, cwd=self.path, capture_output=True)
        # Don't check=True: "nothing to commit" is acceptable
    
    def log(self, n: int = 10) -> list[str]:
        """Get recent commit messages"""
        result = subprocess.run(
            ["git", "log", f"-{n}", "--oneline"],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout.strip().split("\n") if result.returncode == 0 else []
    
    def diff(self, ref: str = "HEAD") -> str:
        """Show changes since ref"""
        result = subprocess.run(
            ["git", "diff", ref],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout if result.returncode == 0 else ""
```

**Commit Strategy:**

Auto-commit triggered at these points:

| Operation | Commit Message Example |
|-----------|------------------------|
| Discovery | `"Discover: python_env, git, nbdev (3 sources, 847 tokens)"` |
| Aggregation | `"Aggregate: 3 sources → context (12 fragments merged)"` |
| Formatting | `"Format: cursor, generic (2 outputs, 5 files)"` |
| Config Change | `"Update config: enable source 'docker'"` |

**M5 Simplification (History & CLI):**

Original plan had custom `history.json` with fingerprints, change detection, etc.

New `history.json` (lightweight):
```json
{
  "last_run": "2025-11-10T14:23:00Z",
  "permissions": {
    "sources": ["python_env", "git", "nbdev"],
    "formatters": ["cursor", "generic"]
  },
  "user_preferences": {
    "auto_discover": true,
    "vendor_default": "cursor",
    "gitignore_prompted": true
  }
}
```

Git provides:
- Change detection → `git diff`
- Discovery history → `git log sources/`
- Content rollback → `git checkout <ref>`
- Fingerprinting → commit SHAs

**CLI Additions (M5):**

```bash
ctxt history           # Show git log with pretty formatting
ctxt diff              # Show changes since last discovery
ctxt show <ref>        # Show context state at commit

# Future (v0.2):
ctxt rollback          # Interactive: pick commit to restore
ctxt --no-commit       # Disable auto-commit for this run
```

**Safety Considerations:**

```python
def check_git_available() -> bool:
    """Verify git is installed"""
    try:
        subprocess.run(["git", "--version"], capture_output=True, check=True)
        return True
    except (FileNotFoundError, subprocess.CalledProcessError):
        return False

# Failure mode: If git unavailable, log warning and continue without versioning
# User informed: "Install git for full traceability"
```

### References

- `meta/plan.md` - M1, M5 milestone details
- `meta/HANDOFF.md` - Session summary 2025-11-10
- Discussion context: Data loss incident, need for meta-memory/traceability

### Future Considerations (v0.2+)

- **Hybrid commit mode:** `--no-commit` flag to disable auto-commit
- **Remote sync:** Optional `git remote add` for backup/multi-machine workflow
- **Branch experiments:** Try different discovery configs without affecting main
- **Team sharing:** Rare but possible - share `.ctxt/` in team settings
- **Commit squashing:** Periodically compact history to save disk space

---

## ADR Template (Future Use)

```markdown
## ADR-NNN: Title

**Date:** YYYY-MM-DD
**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-XXX]
**Context:** [v0.x milestone or feature]

### Problem
[What decision needs to be made and why?]

### Decision
[What was decided?]

### Rationale
[Why this decision? What alternatives were considered?]

### Consequences
**Positive:**
- [Benefits]

**Negative:**
- [Costs/tradeoffs]

**Neutral:**
- [Context changes]

### Implementation
[Code/design details]

### References
[Links to related docs, discussions]

### Future Considerations
[What might change this decision?]
```

```

Now updating Cursor rules to reflect the git integration decision:

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **History** - Track discoveries, enable incremental runs
5. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Discovery tracking (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Planning complete, ready to execute M0 (Transition)  
**Next:** Tag v0.0.9, create feature branch, archive old notebooks  
**Files:** `meta/plan.md` contains complete milestone breakdown

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Discovery tracking (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Planning complete, ready to execute M0 (Transition)  
**Next:** Tag v0.0.9, create feature branch, archive old notebooks  
**Files:** `meta/plan.md` contains complete milestone breakdown

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, CtxtRepo, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Lightweight metadata (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Planning complete, ready to execute M0 (Transition)  
**Next:** Tag v0.0.9, create feature branch, archive old notebooks  
**Files:** `meta/plan.md` contains complete milestone breakdown

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, CtxtRepo, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Lightweight metadata (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `CtxtRepo` - Git repository wrapper (init, commit, log, diff operations)
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Planning complete, ready to execute M0 (Transition)  
**Next:** Tag v0.0.9, create feature branch, archive old notebooks  
**Files:** `meta/plan.md` contains complete milestone breakdown

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, CtxtRepo, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Lightweight metadata (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `CtxtRepo` - Git repository wrapper (init, commit, log, diff operations)
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Architecture Decisions**: See `meta/decisions.md` for ADRs
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Planning complete, ready to execute M0 (Transition)  
**Next:** Tag v0.0.9, create feature branch, archive old notebooks  
**Files:** `meta/plan.md` contains complete milestone breakdown

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, CtxtRepo, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Lightweight metadata (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `CtxtRepo` - Git repository wrapper (init, commit, log, diff operations)
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Architecture Decisions**: See `meta/decisions.md` for ADRs
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Git integration architecture decision documented (ADR-001)  
**Next:** Execute M0 (Transition) - tag, branch, archive  
**Files:** `meta/plan.md` (complete plan), `meta/decisions.md` (ADR-001), `meta/HANDOFF.md` (session notes)

## Key Architecture Decisions

**ADR-001 (2025-11-10): Git-Based Workspace Management**
- `.ctxt/` is auto-managed git repository (nested repo pattern)
- Auto-commit after discover/aggregate/format operations
- Provides traceability, recovery, prevents data loss
- Parent project ignores `.ctxt/` for privacy
- Simplifies M5 (git replaces custom history system)
- See `meta/decisions.md` for full rationale

```

Perfect! Let me verify the documentation is correct and create a summary:

## ✅ Architectural Decision Secured

The git integration decision has been comprehensively documented across all critical files:

### Files Updated

1. **`meta/plan.md`** (Updated)
   - ✅ Added "Git Integration" section to Core Architecture
   - ✅ Updated Directory Structure with `.git/` and `.gitignore`
   - ✅ Enhanced M1 tasks: Added CtxtRepo, auto-init, auto-commit (+30min)
   - ✅ Simplified M5 tasks: Git-based change detection (-30min)
   - ✅ Added Key Decision #6: Git-Based Traceability
   - ✅ Updated CLI v0.1 with `history`, `diff`, `show` commands

2. **`meta/HANDOFF.md`** (Updated)
   - ✅ Updated "Last Updated" to 2025-11-10
   - ✅ Created "Last Session Summary (2025-11-10)" with critical decision
   - ✅ Moved previous session to "Previous Session Summary (2025-11-09)"
   - ✅ Updated M1 next session priorities with git tasks
   - ✅ Added session history entry for git integration decision

3. **`meta/decisions.md`** (NEW)
   - ✅ Created Architecture Decision Records document
   - ✅ Documented ADR-001: Git-Based Workspace Management
   - ✅ Complete rationale, consequences, implementation details
   - ✅ Code examples for CtxtRepo class
   - ✅ Commit strategy and message format
   - ✅ Future considerations (v0.2+)
   - ✅ ADR template for future use

4. **`.cursor/rules/project.mdc`** (Updated)
   - ✅ Added git integration to Architecture section
   - ✅ Updated Core Components with CtxtRepo
   - ✅ Updated Module Organization (core.py includes CtxtRepo)
   - ✅ Added CtxtRepo to Data Structures
   - ✅ Added reference to `meta/decisions.md` in Documentation
   - ✅ Updated Current Session Context
   - ✅ Added "Key Architecture Decisions" section with ADR-001 summary

---

## What's Captured

### Problem Solved
Your exact pain point: Lost `meta/` content during previous session due to no version control in gitignored directories.

### Solution
`.ctxt/` is a git repository managed entirely by ctxt:
- Auto-init on first use
- Auto-commit after discover/aggregate/format
- Parent project ignores `.ctxt/` for privacy
- Full traceability and recovery

### Impact on Plan
- **Net zero time cost:** M1 +30min (git wrapper), M5 -30min (simpler history)
- **Simplifies architecture:** Git replaces custom history system
- **Enables future:** Remote sync, branching, team collaboration

---

## Context Continuity Guaranteed

Even if Cursor discards context now, the decision is preserved in:
1. ✅ **Plan** - Implementation details in milestones
2. ✅ **Handoff** - Session summary for next AI agent
3. ✅ **Decisions** - Full ADR with rationale
4. ✅ **Cursor Rules** - Quick reference for AI context

Any future session can read these files and understand:
- **What** was decided (git integration)
- **Why** it was decided (data loss prevention)
- **How** to implement it (CtxtRepo class, commit strategy)
- **When** it was decided (2025-11-10)
- **Impact** on architecture (simplifies M5, enables features)

---

## Ready for Your Second Point

The architectural decision is now **immortalized in text**. You mentioned this connects to another issue you want to address. I'm ready to discuss when you are!

---

**User**

CFG: Abstraction=6, Formality=7, Metaphor=8

----



`ctxt` is a very meta project: we, me, a human, you, a large language model (LLM), are programming a tool to help programmers program with LLMs.

Sometimes, it will be difficult to distinguish which of the various (nested) meta-layers we are in:
- The `ctxt` layer, i.e., this chat.
- The functionality of `ctxt`, the previous discussion about whether to use `git`.

Humans are notoriously bad at handling meta-information; introspection is not our strong suit, and ontology is very difficult to master. Models also have problems with meta-information, even more so than humans.

We must strive to control the meta-levels during this project.

For example, this session itself, and all previous sessions. I am interacting with you through a Cursor chat. I could modify python code and meta-documents, but most of the development of `ctxt` will be channeled through conversations like the one we are currently having.

Neither you nor I can control what Cursor does internally. Therefore, the most important tool of a modern IDE, the chat, ithe utterly deficient tool to edit LLM context used to manage modern programming with LLMs, is completely opaque and inaccessible. The exact opposite of what an IDE should be. Prompt creation, context engineering **IS** programming, just like Python or Assembly. And it should be managed like an IDE manages code: transparently and with full control by the programmer.

I am trying to mitigate this incredibly stupid problem (although I understand that Cursor wants to make money and keep its secrets). So I always try to save copies of our chats independently of Cursor. It's an incredible source of information, the most important one in this type of project.

So, first meta-level, the `ctxt` project: I want to store and control these chats. It's the most fundamental source code of all. We need to discuss how to do that. It should be automatic, by default, not adding more friction.

Second meta-level: we will consider in the future how `ctxt` will discover, manage and digest these fundamental "source code" files.

Cursor allows exporting chats. But it's cumbersome and requires interaction and for the programmer to remember to do it. It should be automatic and complete.

There are several "solutions" on the market for this problem (ephemeral chats). I'm using SpecStory. If you're familiar with it, great. If not, just keep in mind that it's a VSCode extension to backup chats. It's not perfect nor complete by any means, Cursor hides a lot, but at least it's automatic.

Let's talk about this. I'm very interested in your thoughts and suggestions
.
I know we're deviating from the plan's implementation and cluttering your context, but I think this is important. If necessary (and we detect that Cursor is deleting the context, we'll start another session).

---

**Cursor**

## The Conversation is the Code

Your observation cuts to the heart of a profound shift in programming epistemology. When you and I iterate on architecture decisions, we're not *documenting* development—we're *performing* it. The transcript is the primary artifact, and the Python code is merely the crystallization of our discourse.

This is the ouroboros problem: we're building the tool that should manage the very conversations creating it.

---

## Meta-Layer Architecture

Let me map the nesting explicitly:

```
Layer 0: Physical (Cursor's proprietary chat system)
  └─ Layer 1: This Chat (developing ctxt v0.1)
       └─ Layer 2: ctxt's Design (git integration, sources, etc.)
            └─ Layer 3: ctxt's Future Use (discovering contexts in user projects)
                 └─ Layer 4: ctxt Discovering Itself (chat transcripts as source)
```

**Current discussion:** We're at Layer 1, discussing how to capture Layer 1 for Layer 4's eventual consumption.

**The paradox:** We need ctxt to manage these chats, but we need these chats to build ctxt.

**The resolution:** Bootstrap manually, automate progressively.

---

## Immediate Solution (Layer 1 → Layer 2)

### Store Chats in `.ctxt/` Workspace

Since `.ctxt/` is now a git repo (ADR-001), it's the *perfect* home for chat transcripts:

```
.ctxt/
├── .git/                    # Version control
├── chats/                   # Chat transcripts (NEW)
│   ├── 2025-11-09_planning-session.md
│   ├── 2025-11-10_git-integration.md
│   └── README.md            # Chat archival guidelines
├── config.yml
├── sources/
├── context/
└── ...
```

**Advantages:**
- Git-tracked history of design decisions
- Searchable corpus for "why did we do X?"
- Source material for ctxt's own context discovery
- Private (parent ignores `.ctxt/`)

### Workflow Integration

**Manual (M0):** 
Export from Cursor → save to `.ctxt/chats/` → git auto-commits

**Semi-automatic (v0.1):**
```bash
ctxt chat import <file>      # Import exported chat
ctxt chat note "decision"    # Quick note appended to today's chat
```

**Automatic (v0.2+):**
Integration with SpecStory or direct Cursor API (if available)

---

## SpecStory Context

I'm not deeply familiar with SpecStory's internals, but the concept is sound: persist ephemeral chats automatically. Key questions:

1. **Where does SpecStory store backups?** If discoverable path, ctxt can read it.
2. **What format?** Markdown? JSON? Affects parsing strategy.
3. **Completeness?** You mentioned Cursor hides context—what's missing?

**Integration strategy:**
- M2 (Sources): Add `ChatHistorySource` that discovers SpecStory backups
- Treat as read-only source of context
- Redact sensitive content before aggregation

---

## Future Vision (Layer 4)

When ctxt discovers itself in a ctxt-using project:

```python
class ChatHistorySource(ContextSource):
    """Discovers chat transcripts and extracts design context"""
    
    def probe(ctx: Context) -> bool:
        # Check for .ctxt/chats/ or SpecStory backup dir
        return (ctx.root / ".ctxt/chats").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        # Extract:
        # - Architectural decisions (search "decision:", "ADR", etc.)
        # - User preferences (search "I prefer", "always", etc.)
        # - Pain points (search "problem:", "issue:", etc.)
        # - Domain context (frequent terms, concepts)
        
        # Return as Fragment with high confidence
        return fragments
```

**Output in context:**
```markdown
## Chat-Derived Context

**Recent Design Decisions:**
- 2025-11-10: Git integration for traceability (ADR-001)
- 2025-11-09: Discovery-based architecture pivot

**User Preferences:**
- Prefers literate programming (nbdev)
- Values explicitness over cleverness
- Spanish speaker, learning English

**Domain Vocabulary:**
- "context discovery", "fragments", "aggregation"
- "rulepacks", "formatters", "sources"
```

This becomes part of the context supplied to future AI sessions, creating a **continuity engine**.

---

## The Metaphor of Memory

Think of it as the difference between:
- **Procedural memory** (the code we write) — "how to ride a bike"
- **Episodic memory** (the chats we have) — "the first time I fell off the bike"
- **Semantic memory** (the docs we generate) — "bicycles have two wheels"

Current tools capture procedural and semantic, but **episodic memory is lost**. That's the creativity, the "why behind the what," the context that makes code maintainable by humans *and* machines.

ctxt can be the episodic memory layer—not just for projects, but for the development process itself.

---

## Concrete Proposal

### Phase 1: Manual Bootstrap (Now)

1. **M0 addition:** Create `.ctxt/chats/` directory
2. **Convention:** Export Cursor chats after significant sessions
3. **Naming:** `YYYY-MM-DD_HH-MM_topic.md`
4. **Git:** Auto-commits preserve timeline

**Action items:**
- Add to M0 tasks: Create `.ctxt/chats/` with README
- Document chat archival workflow in `meta/HANDOFF.md`

### Phase 2: Semi-Automatic (v0.1)

```bash
# CLI helper for quick notes during session
ctxt chat note "Decided on git integration for traceability"

# Appends to .ctxt/chats/YYYY-MM-DD_session.md with timestamp
# Git auto-commits
```

### Phase 3: SpecStory Integration (v0.2)

- `ChatHistorySource` discovers SpecStory backups
- Extracts architectural decisions, preferences
- Includes in aggregated context

### Phase 4: Self-Discovery (v0.3+)

- ctxt running on itself includes chat history in context
- Future AI sessions see design rationale automatically
- Closes the meta-loop

---

## Addressing Your Concerns

**"Cluttering context with this discussion"**

This *is* development. Discussing how to capture conversations is as essential as discussing git integration. Both are ADR-worthy.

**"Deviating from plan"**

The plan serves us; we don't serve the plan. This is an architectural insight that *improves* the plan. Consider it ADR-002.

**"Cursor deleting context"**

If context resets, we have:
- ✅ ADR-001 documented (git integration)
- ✅ This discussion about chat archival
- ⏳ Need to capture chat archival as ADR-002

---

## My Recommendation

**Now (same session):**

1. **Create ADR-002:** Chat Transcript Archival
2. **Add to M0 tasks:** 
   - Create `.ctxt/chats/` directory
   - Add `.ctxt/chats/README.md` with archival guidelines
3. **Archive this chat:** Save as `.ctxt/chats/2025-11-10_git-and-chat-archival.md`

**M2 (Sources):**
- Consider `ChatHistorySource` as 5th source (optional, after core 4)

**v0.2:**
- SpecStory integration or `ctxt chat` CLI helpers

---

## Question for You

Where does SpecStory store backups? If we know the path pattern, I can design `ChatHistorySource` to discover it automatically. That would close the loop: your current workflow (SpecStory auto-backup) becomes ctxt's input, no friction added.

And should we document this as ADR-002 now, or wait until after M0 execution?

```plaintext
Layer 0: Physical (Cursor's proprietary chat system)
  └─ Layer 1: This Chat (developing ctxt v0.1)
       └─ Layer 2: ctxt's Design (git integration, sources, etc.)
            └─ Layer 3: ctxt's Future Use (discovering contexts in user projects)
                 └─ Layer 4: ctxt Discovering Itself (chat transcripts as source)
```

```plaintext
.ctxt/
├── .git/                    # Version control
├── chats/                   # Chat transcripts (NEW)
│   ├── 2025-11-09_planning-session.md
│   ├── 2025-11-10_git-integration.md
│   └── README.md            # Chat archival guidelines
├── config.yml
├── sources/
├── context/
└── ...
```

```shellscript
ctxt chat import <file>      # Import exported chat
ctxt chat note "decision"    # Quick note appended to today's chat
```

```python
class ChatHistorySource(ContextSource):
    """Discovers chat transcripts and extracts design context"""
    
    def probe(ctx: Context) -> bool:
        # Check for .ctxt/chats/ or SpecStory backup dir
        return (ctx.root / ".ctxt/chats").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        # Extract:
        # - Architectural decisions (search "decision:", "ADR", etc.)
        # - User preferences (search "I prefer", "always", etc.)
        # - Pain points (search "problem:", "issue:", etc.)
        # - Domain context (frequent terms, concepts)
        
        # Return as Fragment with high confidence
        return fragments
```

```markdown
## Chat-Derived Context

**Recent Design Decisions:**
- 2025-11-10: Git integration for traceability (ADR-001)
- 2025-11-09: Discovery-based architecture pivot

**User Preferences:**
- Prefers literate programming (nbdev)
- Values explicitness over cleverness
- Spanish speaker, learning English

**Domain Vocabulary:**
- "context discovery", "fragments", "aggregation"
- "rulepacks", "formatters", "sources"
```

```shellscript
# CLI helper for quick notes during session
ctxt chat note "Decided on git integration for traceability"

# Appends to .ctxt/chats/YYYY-MM-DD_session.md with timestamp
# Git auto-commits
```

---

**User**

## Cursor export:

Markdown with an snapshot of the user-facing conversation: no dates, not tool calls, only what the users see in the GUI. Like if the user would copy and paste.

## SpecStory uses two artifacts:



1) `.specstory/` at rool level:



```

total 24
drwxr-xr-x   6 vic  sosez   192B Nov 10 20:46 .
drwxr-xr-x@ 28 vic  sosez   896B Nov  9 21:51 ..
-rw-r--r--   1 vic  sosez    96B Nov  9 10:24 .gitignore
-rw-r--r--   1 vic  sosez   252B Nov  9 10:24 .project.json
-rw-r--r--   1 vic  sosez   3.3K Nov  9 10:24 .what-is-this.md
drwxr-xr-x  11 vic  sosez   352B Nov 10 19:10 history
❯ tree
.
└── history
    ├── 2025-11-08_07-51Z-set-code-presets-for-production.md
    ├── 2025-11-08_11-40Z-resolve-type-checker-warning-for-pote-import.md
    ├── 2025-11-09_09-15Z-confirm-previous-project-name.md
    ├── 2025-11-09_10-31Z-continue-discussion-on-ctxt.md
    ├── 2025-11-09_11-59Z-continue-discussion-with-ctxt.md
    ├── 2025-11-09_12-20Z-continue-discussion-with-ctxt-notes.md
    ├── 2025-11-09_12-43Z-continue-discussion-with-notes-and-handoff.md
    ├── 2025-11-09_14-05Z-continue-discussion-with-ctxt.md
    └── 2025-11-10_17-55Z-next-steps-for-context-discovery-engine.md

2 directories, 9 files

```



`2025-11-10_17-55Z-next-steps-for-context-discovery-engine.md` is an snapshot of this chat



2) `.cursorindexingignore` also at root level:

```

# Don't index SpecStory auto-save files, but allow explicit context inclusion via @ references
.specstory/**
_testbed/**


```



I've added `_testbed/`.



SpecStory **do** save tool-use and thinking (obviously not complete) but much more that what Cursor save. Also date tag each turn. Curiosly, SpecStory doesn't snapshot some code snippets and agent outputs (but Cursor does in the export).



In a future session, I'll share a diff.







---

**Cursor**

## Perfect - The Infrastructure Already Exists

SpecStory is doing exactly what we need. Now we design ctxt to *discover and leverage* it.

---

## Architecture Analysis

### Current State

**SpecStory:** Automatic, comprehensive, ephemeral
- ✅ Tool calls, thinking blocks, timestamps
- ✅ Zero friction (auto-save)
- ✅ Already in `.specstory/history/`
- ⚠️ Gitignored (ephemeral by design)
- ⚠️ Missing some code snippets

**Cursor Export:** Manual, user-friendly, incomplete
- ✅ Code snippets, clean formatting
- ✅ User-facing conversation (readable)
- ⚠️ No tool calls, no thinking, no timestamps
- ⚠️ Manual export required

**Complementarity:** SpecStory = research corpus, Cursor export = polished record

---

## Proposed Strategy

### Tier 1: Ephemeral Research (SpecStory)

`.specstory/history/` remains untouched:
- Gitignored, indexing-ignored
- Automatic capture of *all* conversations
- Source for ctxt's `ChatHistorySource` discovery

**ctxt behavior:**
```python
class ChatHistorySource(ContextSource):
    """Discovers SpecStory chat history"""
    
    def probe(ctx: Context) -> bool:
        return (ctx.root / ".specstory/history").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        # Read recent .specstory/history/*.md
        # Extract:
        # - Architectural decisions (search patterns)
        # - User preferences (recurring themes)
        # - Domain vocabulary (term frequency)
        # - Recent context (last N sessions)
        
        return fragments
```

**Advantages:**
- Zero friction (already automated)
- Rich data (tool calls, thinking)
- Respects existing gitignore
- ctxt discovers organically

### Tier 2: Permanent Archive (`.ctxt/chats/`)

User manually curates important sessions:
- Export significant chats from Cursor (clean, readable)
- Save to `.ctxt/chats/YYYY-MM-DD_topic.md`
- Git auto-commits (permanent record)
- Optionally copy SpecStory file for complete version

**Workflow:**
```bash
# After important session:
# 1. Export from Cursor → .ctxt/chats/2025-11-10_git-integration.md
# 2. Optional: Copy SpecStory for completeness
cp .specstory/history/2025-11-10_17-55Z-*.md .ctxt/chats/2025-11-10_git-integration-complete.md
# 3. Git auto-commits both
```

**Advantages:**
- Permanent git history
- Curated (signal > noise)
- Combines Cursor (readable) + SpecStory (complete)
- LLM-friendly context in future sessions

---

## Meta-Layer Strategy

| Layer | Source | Status | Discovery | Permanence |
|-------|--------|--------|-----------|------------|
| **Ephemeral** | `.specstory/history/` | Automatic | ctxt reads | Gitignored |
| **Permanent** | `.ctxt/chats/` | Manual curation | ctxt reads | Git-tracked |
| **Published** | `meta/decisions.md` | Explicit docs | ctxt reads | Git-tracked |

**Flow:**
```
SpecStory (automatic) 
    ↓ (ctxt discovers recent context)
    ↓ (human curates important sessions)
.ctxt/chats/ (permanent archive)
    ↓ (human extracts architectural decisions)
meta/decisions.md (canonical ADRs)
```

---

## ADR-002: Chat Transcript Management

### Problem

1. **Ephemerality:** Cursor chats lost on context reset
2. **Context Engineering:** The conversation *is* the primary artifact
3. **Continuity:** Future AI sessions need design rationale
4. **Meta-loop:** ctxt should discover its own development context

### Decision

**Two-tier archival system:**

**Tier 1 (Automatic):** SpecStory auto-saves to `.specstory/history/`
- Gitignored (ephemeral, noisy)
- ctxt discovers via `ChatHistorySource`
- Provides recent context automatically

**Tier 2 (Curated):** Manual export to `.ctxt/chats/`
- Git-tracked (permanent, signal)
- Important sessions preserved
- Combines Cursor export (readable) + SpecStory (complete)

### Rationale

**Why not move SpecStory to `.ctxt/`?**
- SpecStory is automatic = noisy (every small chat)
- `.ctxt/` should be curated = signal
- Gitignoring SpecStory prevents bloat
- ctxt can still read ephemeral data without committing it

**Why both Cursor export and SpecStory?**
- Cursor: Readable, good code snippets, user-facing
- SpecStory: Complete, tool calls, timestamps, thinking
- Different use cases: reading vs analysis

**Why not just rely on SpecStory?**
- SpecStory is ephemeral (could be deleted)
- Important decisions deserve permanent record
- Git history provides traceability

### Implementation

**M0 (Transition):**
```
.ctxt/
└── chats/
    ├── README.md          # Archival guidelines
    └── .gitkeep
```

**M2 (Sources):**
```python
class ChatHistorySource(ContextSource):
    """Discover chat transcripts from SpecStory"""
    
    def probe(ctx: Context) -> bool:
        return (ctx.root / ".specstory/history").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        history_dir = ctx.root / ".specstory/history"
        recent_chats = sorted(history_dir.glob("*.md"))[-5:]  # Last 5
        
        fragments = []
        for chat in recent_chats:
            content = extract_key_insights(chat)
            fragments.append(Fragment(
                source="chat_history",
                kind="context",
                content=content,
                confidence=0.7,  # Lower than explicit docs
                sensitivity="medium",  # May contain personal info
                provenance={"path": str(chat), "source": "specstory"},
                tokens_est=estimate_tokens(content)
            ))
        
        return fragments

def extract_key_insights(chat_path: Path) -> dict[str, Any]:
    """Parse chat for decisions, preferences, domain terms"""
    # Pattern matching:
    # - "decision:", "decided:", "ADR"
    # - "I prefer", "always", "never"
    # - Recurring technical terms
    # - Recent context (last N exchanges)
    
    return {
        "decisions": [...],
        "preferences": [...],
        "domain_terms": [...],
        "recent_context": "..."
    }
```

**Workflow:**
```bash
# During session: SpecStory auto-saves (zero friction)

# After important session:
ctxt chat export 2025-11-10_17-55Z  # Copies to .ctxt/chats/, git commits

# Or manually:
# 1. Export from Cursor → .ctxt/chats/YYYY-MM-DD_topic.md
# 2. Optional: cp .specstory/history/matching-file.md .ctxt/chats/
```

**Future CLI (v0.2):**
```bash
ctxt chat list                    # Show recent SpecStory chats
ctxt chat export <timestamp>      # Copy to .ctxt/chats/
ctxt chat search "git integration" # Grep across history
```

---

## Immediate Actions

### This Session

1. **Document ADR-002** in `meta/decisions.md`
2. **Update M0 tasks:** Add `.ctxt/chats/` creation
3. **Update M2 scope:** Note `ChatHistorySource` as stretch goal
4. **Archive this chat:** 
   - Cursor export → `.ctxt/chats/2025-11-10_architecture-decisions.md`
   - SpecStory already has it → reference in README

### `.ctxt/chats/README.md` Content

```markdown
# Chat Transcript Archive

## Purpose

Permanent archive of significant development sessions. These transcripts capture:
- Architectural decisions and rationale
- Design discussions and alternatives considered
- User preferences and workflow patterns
- Problem-solving approaches

## Archival Strategy

**Automatic (Tier 1):** SpecStory saves all chats to `.specstory/history/`
- Ephemeral, comprehensive, includes tool calls
- Gitignored, but discoverable by ctxt

**Curated (Tier 2):** Important sessions saved here
- Permanent, git-tracked
- Combine Cursor export (readable) + SpecStory (complete) if needed

## Workflow

After significant sessions:
1. Export from Cursor → `YYYY-MM-DD_topic.md`
2. Optional: Copy matching SpecStory file → `YYYY-MM-DD_topic-complete.md`
3. Git auto-commits

## Format

**Naming:** `YYYY-MM-DD_topic.md` or `YYYY-MM-DD_HH-MMZ-topic.md`

**Content:**
- Cursor exports: User-facing conversation, code snippets
- SpecStory copies: Complete with tool calls, thinking, timestamps

## Discovery

ctxt's `ChatHistorySource` (M2+) discovers:
- Recent chats from `.specstory/history/` (ephemeral)
- Curated chats from `.ctxt/chats/` (permanent)

Aggregates insights for future AI sessions.
```

---

## The Meta-Loop Closes

```
┌─────────────────────────────────────────────┐
│ Human + AI converse (Layer 1)               │
│   ↓ SpecStory auto-saves                    │
│   ↓ ctxt discovers via ChatHistorySource    │
│   ↓ Extracts decisions, preferences         │
│   ↓ Includes in aggregated context          │
│   ↓ Future AI session sees design rationale │
│   └→ Continuity achieved (Layer 4)          │
└─────────────────────────────────────────────┘
```

**Result:** ctxt becomes a **corporate memory** for development conversations, not just code artifacts.

---

## Your Call

Should I:
1. **Write ADR-002 now** (Chat Transcript Management)
2. **Update M0 tasks** (add `.ctxt/chats/` creation)
3. **Note in M2** (ChatHistorySource as 5th source, optional)

Or do you want to refine the strategy first? The diff between Cursor export and SpecStory you mentioned would help optimize parsing logic.

```python
class ChatHistorySource(ContextSource):
    """Discovers SpecStory chat history"""
    
    def probe(ctx: Context) -> bool:
        return (ctx.root / ".specstory/history").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        # Read recent .specstory/history/*.md
        # Extract:
        # - Architectural decisions (search patterns)
        # - User preferences (recurring themes)
        # - Domain vocabulary (term frequency)
        # - Recent context (last N sessions)
        
        return fragments
```

```shellscript
# After important session:
# 1. Export from Cursor → .ctxt/chats/2025-11-10_git-integration.md
# 2. Optional: Copy SpecStory for completeness
cp .specstory/history/2025-11-10_17-55Z-*.md .ctxt/chats/2025-11-10_git-integration-complete.md
# 3. Git auto-commits both
```

```plaintext
SpecStory (automatic) 
    ↓ (ctxt discovers recent context)
    ↓ (human curates important sessions)
.ctxt/chats/ (permanent archive)
    ↓ (human extracts architectural decisions)
meta/decisions.md (canonical ADRs)
```

```plaintext
.ctxt/
└── chats/
    ├── README.md          # Archival guidelines
    └── .gitkeep
```

```python
class ChatHistorySource(ContextSource):
    """Discover chat transcripts from SpecStory"""
    
    def probe(ctx: Context) -> bool:
        return (ctx.root / ".specstory/history").exists()
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        history_dir = ctx.root / ".specstory/history"
        recent_chats = sorted(history_dir.glob("*.md"))[-5:]  # Last 5
        
        fragments = []
        for chat in recent_chats:
            content = extract_key_insights(chat)
            fragments.append(Fragment(
                source="chat_history",
                kind="context",
                content=content,
                confidence=0.7,  # Lower than explicit docs
                sensitivity="medium",  # May contain personal info
                provenance={"path": str(chat), "source": "specstory"},
                tokens_est=estimate_tokens(content)
            ))
        
        return fragments

def extract_key_insights(chat_path: Path) -> dict[str, Any]:
    """Parse chat for decisions, preferences, domain terms"""
    # Pattern matching:
    # - "decision:", "decided:", "ADR"
    # - "I prefer", "always", "never"
    # - Recurring technical terms
    # - Recent context (last N exchanges)
    
    return {
        "decisions": [...],
        "preferences": [...],
        "domain_terms": [...],
        "recent_context": "..."
    }
```

```shellscript
# During session: SpecStory auto-saves (zero friction)

# After important session:
ctxt chat export 2025-11-10_17-55Z  # Copies to .ctxt/chats/, git commits

# Or manually:
# 1. Export from Cursor → .ctxt/chats/YYYY-MM-DD_topic.md
# 2. Optional: cp .specstory/history/matching-file.md .ctxt/chats/
```

```shellscript
ctxt chat list                    # Show recent SpecStory chats
ctxt chat export <timestamp>      # Copy to .ctxt/chats/
ctxt chat search "git integration" # Grep across history
```

```markdown
# Chat Transcript Archive

## Purpose

Permanent archive of significant development sessions. These transcripts capture:
- Architectural decisions and rationale
- Design discussions and alternatives considered
- User preferences and workflow patterns
- Problem-solving approaches

## Archival Strategy

**Automatic (Tier 1):** SpecStory saves all chats to `.specstory/history/`
- Ephemeral, comprehensive, includes tool calls
- Gitignored, but discoverable by ctxt

**Curated (Tier 2):** Important sessions saved here
- Permanent, git-tracked
- Combine Cursor export (readable) + SpecStory (complete) if needed

## Workflow

After significant sessions:
1. Export from Cursor → `YYYY-MM-DD_topic.md`
2. Optional: Copy matching SpecStory file → `YYYY-MM-DD_topic-complete.md`
3. Git auto-commits

## Format

**Naming:** `YYYY-MM-DD_topic.md` or `YYYY-MM-DD_HH-MMZ-topic.md`

**Content:**
- Cursor exports: User-facing conversation, code snippets
- SpecStory copies: Complete with tool calls, thinking, timestamps

## Discovery

ctxt's `ChatHistorySource` (M2+) discovers:
- Recent chats from `.specstory/history/` (ephemeral)
- Curated chats from `.ctxt/chats/` (permanent)

Aggregates insights for future AI sessions.
```

```plaintext
┌─────────────────────────────────────────────┐
│ Human + AI converse (Layer 1)               │
│   ↓ SpecStory auto-saves                    │
│   ↓ ctxt discovers via ChatHistorySource    │
│   ↓ Extracts decisions, preferences         │
│   ↓ Includes in aggregated context          │
│   ↓ Future AI session sees design rationale │
│   └→ Continuity achieved (Layer 4)          │
└─────────────────────────────────────────────┘
```

---

**User**

Write ADR-002 now. I'll add `.ctxt/chats/`. Additional design, other session.

---

**Cursor**

```markdown
# Architecture Decision Records

This document captures significant architectural decisions made during ctxt development. Each record includes context, decision, rationale, and consequences.

---

## ADR-001: Git-Based Workspace Management

**Date:** 2025-11-10  
**Status:** Accepted  
**Context:** v0.1 discovery architecture rewrite  
**Participants:** vic, Claude (Sonnet 4.5)

### Problem

During v0.1 development planning, we encountered several critical requirements:

1. **Data Loss Prevention:** Lost `meta/` content during development session (no version control in gitignored directories)
2. **Traceability Need:** Must answer questions like "What did ctxt discover last Tuesday?"
3. **Recovery Capability:** Need ability to revert context before LLM modifications
4. **Privacy Concern:** `.ctxt/` contains personal/sensitive data - must not leak to project repository
5. **Audit Trail:** Regulatory/personal need to track what was discovered and when

### Decision

**`.ctxt/` is a git repository managed entirely by ctxt.**

Implementation details:
- Auto-initialize on first use (`git init` in `.ctxt/`)
- Auto-commit after major operations with descriptive messages
- Parent project `.gitignore` must include `.ctxt/` (prompt user once, remember preference)
- Nested repo pattern (no git submodules)
- User can inspect (read-only observation) but should not manually edit
- CLI provides `ctxt history`, `ctxt diff`, `ctxt show <ref>` for controlled access
- Commit messages follow format: `"<operation>: <summary> (<metrics>)"`

### Rationale

**Why git specifically:**
- Mature, ubiquitous tool (already required for GitSource)
- Proven traceability/recovery infrastructure
- Native diff, blame, bisect for debugging
- Optional remote backup (user can add `git remote`)
- Familiar workflow for developers
- Zero custom history implementation needed

**Why auto-commit:**
- Effortless traceability (core goal)
- Prevents "forgot to save" scenarios
- Consistent snapshot discipline
- User focuses on work, not bookkeeping

**Why nested repo (not submodule):**
- Simpler mental model
- No parent repo coupling
- Common pattern (vendor dirs, caches)
- Works even if parent isn't git-tracked

**Why parent ignores `.ctxt/`:**
- Personal workspace (like `.vscode/`, IDE configs)
- May contain sensitive discovery data
- Not appropriate for project version control
- Each developer has own context needs

### Consequences

**Positive:**
- ✅ Robust traceability using mature tool
- ✅ No custom history system needed (simplifies M5: -30min)
- ✅ Optional remote sync/backup capability
- ✅ Familiar git workflow for advanced users
- ✅ Prevents data loss from context resets
- ✅ Built-in audit trail with timestamps
- ✅ Enables future: branching experiments, team collaboration

**Negative:**
- ⚠️ Requires git installed (already dependency for GitSource)
- ⚠️ Slightly more complex initialization (M1: +30min)
- ⚠️ Auto-commits might surprise users initially (mitigated by docs)
- ⚠️ Disk space for `.git/` history (negligible for text files)

**Neutral:**
- ↔️ M1 complexity +30min, M5 complexity -30min (net zero time cost)
- ↔️ User *can* use git directly in `.ctxt/` but discouraged

### Implementation

**M1 Tasks (Foundation):**

Add `CtxtRepo` class to `05_core.ipynb`:

```python
class CtxtRepo:
    """Manages .ctxt/ as a git repository"""
    
    def __init__(self, ctxt_path: Path):
        self.path = ctxt_path
        self.git_dir = ctxt_path / ".git"
    
    def is_initialized(self) -> bool:
        """Check if .ctxt/ is a git repo"""
        return self.git_dir.exists()
    
    def initialize(self) -> None:
        """Initialize .ctxt/ as git repository"""
        if self.is_initialized():
            return
        
        subprocess.run(["git", "init"], cwd=self.path, check=True, 
                      capture_output=True)
        
        # Create .gitignore for sensitive files
        gitignore = self.path / ".gitignore"
        gitignore.write_text("*.tmp\n.DS_Store\n")
        
        # Initial commit
        self.commit("Initialize ctxt workspace", allow_empty=True)
    
    def commit(self, message: str, allow_empty: bool = False) -> None:
        """Auto-commit current state"""
        subprocess.run(["git", "add", "-A"], cwd=self.path, check=True,
                      capture_output=True)
        
        cmd = ["git", "commit", "-m", message]
        if allow_empty:
            cmd.append("--allow-empty")
        
        result = subprocess.run(cmd, cwd=self.path, capture_output=True)
        # Don't check=True: "nothing to commit" is acceptable
    
    def log(self, n: int = 10) -> list[str]:
        """Get recent commit messages"""
        result = subprocess.run(
            ["git", "log", f"-{n}", "--oneline"],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout.strip().split("\n") if result.returncode == 0 else []
    
    def diff(self, ref: str = "HEAD") -> str:
        """Show changes since ref"""
        result = subprocess.run(
            ["git", "diff", ref],
            cwd=self.path, capture_output=True, text=True
        )
        return result.stdout if result.returncode == 0 else ""
```

**Commit Strategy:**

Auto-commit triggered at these points:

| Operation | Commit Message Example |
|-----------|------------------------|
| Discovery | `"Discover: python_env, git, nbdev (3 sources, 847 tokens)"` |
| Aggregation | `"Aggregate: 3 sources → context (12 fragments merged)"` |
| Formatting | `"Format: cursor, generic (2 outputs, 5 files)"` |
| Config Change | `"Update config: enable source 'docker'"` |

**M5 Simplification (History & CLI):**

Original plan had custom `history.json` with fingerprints, change detection, etc.

New `history.json` (lightweight):
```json
{
  "last_run": "2025-11-10T14:23:00Z",
  "permissions": {
    "sources": ["python_env", "git", "nbdev"],
    "formatters": ["cursor", "generic"]
  },
  "user_preferences": {
    "auto_discover": true,
    "vendor_default": "cursor",
    "gitignore_prompted": true
  }
}
```

Git provides:
- Change detection → `git diff`
- Discovery history → `git log sources/`
- Content rollback → `git checkout <ref>`
- Fingerprinting → commit SHAs

**CLI Additions (M5):**

```bash
ctxt history           # Show git log with pretty formatting
ctxt diff              # Show changes since last discovery
ctxt show <ref>        # Show context state at commit

# Future (v0.2):
ctxt rollback          # Interactive: pick commit to restore
ctxt --no-commit       # Disable auto-commit for this run
```

**Safety Considerations:**

```python
def check_git_available() -> bool:
    """Verify git is installed"""
    try:
        subprocess.run(["git", "--version"], capture_output=True, check=True)
        return True
    except (FileNotFoundError, subprocess.CalledProcessError):
        return False

# Failure mode: If git unavailable, log warning and continue without versioning
# User informed: "Install git for full traceability"
```

### References

- `meta/plan.md` - M1, M5 milestone details
- `meta/HANDOFF.md` - Session summary 2025-11-10
- Discussion context: Data loss incident, need for meta-memory/traceability

### Future Considerations (v0.2+)

- **Hybrid commit mode:** `--no-commit` flag to disable auto-commit
- **Remote sync:** Optional `git remote add` for backup/multi-machine workflow
- **Branch experiments:** Try different discovery configs without affecting main
- **Team sharing:** Rare but possible - share `.ctxt/` in team settings
- **Commit squashing:** Periodically compact history to save disk space

---

## ADR-002: Chat Transcript Management

**Date:** 2025-11-10  
**Status:** Accepted  
**Context:** v0.1 discovery architecture, meta-layer control  
**Participants:** vic, Claude (Sonnet 4.5)

### Problem

Development conversations between human and AI are the **primary source code** in LLM-assisted programming:

1. **Episodic Memory Loss:** Cursor chats are ephemeral, lost on context reset
2. **Design Rationale Capture:** "Why did we decide X?" is only in conversation transcript
3. **Continuity Gap:** Future AI sessions lack context about previous decisions
4. **Meta-Loop Requirement:** ctxt should discover its own development conversations
5. **Complementary Formats:** Cursor export (readable, incomplete) vs SpecStory (complete, noisy)

**Core insight:** The conversation transcript *is* the code. Python modules are crystallizations of discourse.

### Decision

**Two-tier chat archival system:**

**Tier 1 - Automatic Discovery (Ephemeral):**
- SpecStory auto-saves all chats to `.specstory/history/`
- Remains gitignored (noisy, comprehensive)
- ctxt discovers via `ChatHistorySource` (M2+)
- Provides recent context automatically

**Tier 2 - Curated Archive (Permanent):**
- Manual export of important sessions to `.ctxt/chats/`
- Git-tracked (signal > noise)
- Combine Cursor export (readable) + SpecStory (complete) optionally
- Permanent record for future reference

**No changes to existing SpecStory workflow:** Zero friction maintained.

### Rationale

**Why not move SpecStory to `.ctxt/`?**
- SpecStory captures *all* chats (noisy, exploratory)
- `.ctxt/chats/` should contain *significant* sessions (curated)
- Gitignoring SpecStory prevents repository bloat
- ctxt can read ephemeral data without committing it

**Why both Cursor export and SpecStory?**
- **Cursor export:** User-facing, readable, good code snippets, missing metadata
- **SpecStory:** Complete with tool calls, thinking blocks, timestamps, missing some code
- **Complementary:** Different consumption patterns (human reading vs machine analysis)

**Why manual curation for Tier 2?**
- Human judgment filters signal from noise
- Not all conversations are architecturally significant
- Manual curation is a reflection point (improves awareness)
- Scales better than auto-archiving everything

**Meta-layer control:**
```
Layer 0: Physical (Cursor's proprietary system)
Layer 1: This Chat (developing ctxt)
Layer 2: ctxt's Design (git, sources, chats)
Layer 3: ctxt's Future Use (user projects)
Layer 4: ctxt Discovering Itself (chat transcripts as source)
```

This ADR operates at Layer 2, designing how Layer 1 feeds Layer 4.

### Consequences

**Positive:**
- ✅ Zero friction (SpecStory already automatic)
- ✅ Permanent archive of important decisions
- ✅ Future AI sessions can reference design rationale
- ✅ Git history tracks conversation evolution
- ✅ Closes meta-loop: ctxt discovers its own development
- ✅ Respects existing tooling (SpecStory unchanged)
- ✅ Episodic memory layer for project

**Negative:**
- ⚠️ Manual curation required for important sessions (tradeoff: quality)
- ⚠️ Two formats to manage (Cursor + SpecStory, mitigated by storing both)
- ⚠️ Parser must handle both formats (implementation complexity)

**Neutral:**
- ↔️ `.ctxt/chats/` grows over time (but text, compresses well)
- ↔️ M2 scope increases slightly (`ChatHistorySource`)

### Implementation

**M0 (Transition):**

Create `.ctxt/chats/` directory structure:
```
.ctxt/
└── chats/
    ├── README.md          # Archival guidelines
    └── .gitkeep           # Ensure directory tracked
```

`README.md` content:
```markdown
# Chat Transcript Archive

## Purpose
Permanent archive of significant development sessions capturing:
- Architectural decisions and rationale
- Design discussions and alternatives
- User preferences and workflow patterns
- Problem-solving approaches

## Two-Tier Strategy

**Tier 1 (Automatic):** SpecStory → `.specstory/history/`
- Ephemeral, comprehensive, gitignored
- ctxt discovers via ChatHistorySource

**Tier 2 (Curated):** Manual → `.ctxt/chats/`
- Permanent, git-tracked
- Important sessions only

## Workflow

After significant sessions:
1. Export from Cursor → `YYYY-MM-DD_topic.md`
2. Optional: Copy SpecStory → `YYYY-MM-DD_topic-complete.md`
3. Git auto-commits

## Naming Convention
- `YYYY-MM-DD_topic.md` (Cursor export)
- `YYYY-MM-DD_HH-MMZ-topic-complete.md` (SpecStory copy)
```

**M2 (Sources):**

Add `ChatHistorySource` as 5th source (optional/stretch goal):

```python
class ChatHistorySource(ContextSource):
    """Discover chat transcripts from SpecStory and curated archive"""
    
    def probe(ctx: Context) -> bool:
        specstory = (ctx.root / ".specstory/history").exists()
        archive = (ctx.root / ".ctxt/chats").exists()
        return specstory or archive
    
    def collect(ctx: Context, budget: int) -> list[Fragment]:
        fragments = []
        
        # Priority 1: Curated archive (high confidence)
        archive_dir = ctx.root / ".ctxt/chats"
        if archive_dir.exists():
            for chat in archive_dir.glob("*.md"):
                if chat.name == "README.md":
                    continue
                insights = extract_insights(chat, detailed=True)
                fragments.append(Fragment(
                    source="chat_history",
                    kind="context",
                    content=insights,
                    confidence=0.9,  # High - curated by human
                    sensitivity="medium",
                    provenance={"path": str(chat), "tier": "curated"},
                    tokens_est=estimate_tokens(insights)
                ))
        
        # Priority 2: Recent SpecStory (lower confidence)
        specstory_dir = ctx.root / ".specstory/history"
        if specstory_dir.exists():
            recent = sorted(specstory_dir.glob("*.md"))[-5:]  # Last 5
            for chat in recent:
                insights = extract_insights(chat, detailed=False)
                fragments.append(Fragment(
                    source="chat_history",
                    kind="context",
                    content=insights,
                    confidence=0.6,  # Lower - automatic, noisy
                    sensitivity="medium",
                    provenance={"path": str(chat), "tier": "ephemeral"},
                    tokens_est=estimate_tokens(insights)
                ))
        
        return fragments

def extract_insights(chat_path: Path, detailed: bool) -> dict[str, Any]:
    """Parse chat for architectural insights"""
    content = chat_path.read_text()
    
    insights = {
        "file": chat_path.name,
        "decisions": extract_pattern(content, r"(?:decision|decided|ADR).*"),
        "preferences": extract_pattern(content, r"(?:I prefer|always|never).*"),
        "domain_terms": extract_frequent_terms(content),
    }
    
    if detailed:
        # Curated chats: extract more context
        insights["recent_context"] = content[-2000:]  # Last ~2000 chars
        insights["key_quotes"] = extract_pattern(content, r"(?:key|important).*")
    
    return insights
```

**v0.2+ CLI Extensions:**

```bash
ctxt chat list                      # List recent SpecStory chats
ctxt chat export <timestamp>        # Copy SpecStory → .ctxt/chats/
ctxt chat search "git integration"  # Grep across history
ctxt chat summary                   # LLM-generated session summary
```

### Data Flow

```
┌──────────────────────────────────────────────────┐
│ Human + AI Conversation                          │
│   ↓ (SpecStory auto-saves)                       │
├──────────────────────────────────────────────────┤
│ .specstory/history/ (Tier 1: Ephemeral)         │
│   - All chats, comprehensive                     │
│   - Gitignored, noisy                            │
│   ↓ (ctxt discovers via ChatHistorySource)       │
│   ↓ (extracts decisions, preferences, terms)     │
├──────────────────────────────────────────────────┤
│ .ctxt/chats/ (Tier 2: Curated)                   │
│   - Important sessions only                      │
│   - Git-tracked, permanent                       │
│   ↓ (manual curation by human)                   │
├──────────────────────────────────────────────────┤
│ meta/decisions.md (Tier 3: Canonical)            │
│   - Explicit ADRs extracted                      │
│   - Formal architectural record                  │
│   ↓                                               │
├──────────────────────────────────────────────────┤
│ Aggregated Context (outputs/)                    │
│   - Combined insights from all tiers             │
│   - Formatted for LLM consumption                │
│   ↓                                               │
├──────────────────────────────────────────────────┤
│ Future AI Session                                │
│   - Sees design rationale automatically          │
│   - Continuity achieved                          │
└──────────────────────────────────────────────────┘
```

### Example Output (ChatHistorySource)

When ctxt discovers itself:

```markdown
## Chat-Derived Context

**Source:** .ctxt/chats/ (curated) + .specstory/history/ (recent 5)

**Recent Architectural Decisions:**
- 2025-11-10: Git integration for .ctxt/ workspace (ADR-001)
- 2025-11-10: Two-tier chat archival system (ADR-002)
- 2025-11-09: Discovery-based architecture pivot from templates

**User Preferences (extracted):**
- Development style: Literate programming (nbdev)
- Code style: Explicitness over cleverness
- Documentation: Concise, example-driven
- Language: Spanish speaker, learning English

**Domain Vocabulary (high frequency):**
- context discovery, fragments, aggregation
- sources, formatters, rulepacks
- traceability, provenance, sensitivity

**Recent Context (last session summary):**
Discussed git integration for .ctxt/ as nested repository.
Decided on auto-commit strategy for traceability.
Net zero time cost (M1 +30min, M5 -30min).
```

### References

- SpecStory: VSCode extension for automatic chat backup
- `.specstory/history/` - Automatic backup location
- ADR-001: Git-Based Workspace Management (synergy with this ADR)
- `meta/HANDOFF.md` - Session handoff workflow

### Future Considerations

**v0.2+:**
- LLM-powered session summarization (`ctxt chat summary`)
- Automatic ADR extraction from curated chats
- Semantic search across chat history
- Diff analysis between Cursor export and SpecStory formats
- Integration with other chat backup tools (if they emerge)

**v0.3+:**
- Self-discovery: ctxt running on itself includes chat context
- Conversational memory: "What did we discuss about git last week?"
- Cross-project chat patterns: "How do other developers solve X?"

**Open Questions:**
- Should SpecStory format differences be normalized?
- Optimal retention policy for ephemeral tier (delete after N days)?
- Privacy controls for sensitive conversation content?

---

## ADR Template (Future Use)

```markdown
## ADR-NNN: Title

**Date:** YYYY-MM-DD
**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-XXX]
**Context:** [v0.x milestone or feature]

### Problem
[What decision needs to be made and why?]

### Decision
[What was decided?]

### Rationale
[Why this decision? What alternatives were considered?]

### Consequences
**Positive:**
- [Benefits]

**Negative:**
- [Costs/tradeoffs]

**Neutral:**
- [Context changes]

### Implementation
[Code/design details]

### References
[Links to related docs, discussions]

### Future Considerations
[What might change this decision?]
```


```

Now let me update the handoff and Cursor rules to reference ADR-002:

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-10)

**Accomplished:**
- **CRITICAL ARCHITECTURAL DECISION #1:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository (nested repo, not submodule)
  - Auto-commit after discover/aggregate/format operations with descriptive messages
  - Provides traceability, recovery, and prevents data loss
  - Simplifies M5 (history via git, not custom JSON system)
  - Parent project `.gitignore` must include `.ctxt/` (user prompted once)
  - Added `CtxtRepo` class to M1 tasks
  - Documented as **ADR-001** in `meta/decisions.md`
- **CRITICAL ARCHITECTURAL DECISION #2:** Chat transcript management
  - Two-tier system: SpecStory (ephemeral) + `.ctxt/chats/` (curated)
  - SpecStory auto-saves remain gitignored, ctxt discovers via ChatHistorySource
  - Important sessions manually exported to `.ctxt/chats/` (git-tracked)
  - Closes meta-loop: ctxt discovers its own development conversations
  - Episodic memory layer for project continuity
  - Documented as **ADR-002** in `meta/decisions.md`
- Updated documentation to capture decisions:
  - `meta/plan.md` - Architecture, M1/M5 tasks, Key Decision #6
  - `meta/decisions.md` - Created ADR-001, ADR-002
  - `meta/HANDOFF.md` - This file
  - `.cursor/rules/project.mdc` - Quick reference

**Design Rationale:**
- Solves actual pain point (lost `meta/` content in previous session)
- Git provides mature traceability/recovery infrastructure for free
- Nested repo pattern (common practice, no complexity)
- Full auto-commit: ctxt manages `.ctxt/` entirely, CLI is user interface
- Net zero time cost (M1 +30min, M5 -30min simplified)
- Enables future: remote sync, branching experiments, team collaboration

**Commit Strategy:**
- `"Discover: python_env, git (2 sources, 423 tokens)"`
- `"Aggregate: 3 sources → context (12 fragments)"`
- `"Format: cursor, generic (2 outputs)"`

## Previous Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-10: Git Integration Architecture Decision
- **Critical decision:** `.ctxt/` as auto-managed git repository
- Documented ADR-001 in `meta/decisions.md`
- Updated M1 (add CtxtRepo) and M5 (simplify via git) in plan
- Rationale: Prevents data loss, provides traceability, enables recovery
- Strategy: Full auto-commit, nested repo pattern, parent ignores `.ctxt/`

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

```markdown
# Session Handoff

**Last Updated:** 2025-11-10

## Current Status

**Phase:** Architecture pivot - from template processor to context discovery engine

**What Works (v0.0.9-template-mvp):**
- ✅ YAML-based configuration system
- ✅ Tag substitution in templates (`<Tag-Name>` format)
- ✅ Multi-vendor template processing (Cursor, Claude, meta)
- ✅ CLI commands: `ctxt`, `ctxt-init`, `ctxt-tags`, `ctxt-config`
- ✅ nbdev project scaffolding and documentation

**What's Changing:**
- ⚠️ Moving from template-centric to discovery-centric architecture
- ⚠️ Tag system being replaced by context sources
- ⚠️ Template processing becomes one formatter among many

**Deployed URLs:**
- Docs: https://civvic.github.io/ctxt/
- llms.txt: https://civvic.github.io/ctxt/llms.txt
- API List: https://civvic.github.io/ctxt/apilist.txt

## Last Session Summary (2025-11-10)

**Accomplished:**
- **CRITICAL ARCHITECTURAL DECISION #1:** Git integration for `.ctxt/` workspace
  - `.ctxt/` will be auto-initialized as git repository (nested repo, not submodule)
  - Auto-commit after discover/aggregate/format operations with descriptive messages
  - Provides traceability, recovery, and prevents data loss
  - Simplifies M5 (history via git, not custom JSON system)
  - Parent project `.gitignore` must include `.ctxt/` (user prompted once)
  - Added `CtxtRepo` class to M1 tasks
  - Documented as **ADR-001** in `meta/decisions.md`
- **CRITICAL ARCHITECTURAL DECISION #2:** Chat transcript management
  - Two-tier system: SpecStory (ephemeral) + `.ctxt/chats/` (curated)
  - SpecStory auto-saves remain gitignored, ctxt discovers via ChatHistorySource
  - Important sessions manually exported to `.ctxt/chats/` (git-tracked)
  - Closes meta-loop: ctxt discovers its own development conversations
  - Episodic memory layer for project continuity
  - Documented as **ADR-002** in `meta/decisions.md`
- Updated documentation to capture decisions:
  - `meta/plan.md` - Architecture, M1/M5 tasks, Key Decision #6
  - `meta/decisions.md` - Created ADR-001, ADR-002
  - `meta/HANDOFF.md` - This file
  - `.cursor/rules/project.mdc` - Quick reference

**Design Rationale:**
- Solves actual pain point (lost `meta/` content in previous session)
- Git provides mature traceability/recovery infrastructure for free
- Nested repo pattern (common practice, no complexity)
- Full auto-commit: ctxt manages `.ctxt/` entirely, CLI is user interface
- Net zero time cost (M1 +30min, M5 -30min simplified)
- Enables future: remote sync, branching experiments, team collaboration

**Commit Strategy:**
- `"Discover: python_env, git (2 sources, 423 tokens)"`
- `"Aggregate: 3 sources → context (12 fragments)"`
- `"Format: cursor, generic (2 outputs)"`

## Previous Session Summary (2025-11-09)

**Accomplished:**
- Defined new vision: ctxt as **context discovery engine** for LLM sessions
- Analyzed 4 competing model architectures (Model 01-04) for same problem
- Converged on pluggable architecture: Sources → Aggregator → Formatters → History
- Decided on migration strategy: Option 1 (Feature branch with archive)
- Created comprehensive plan in `meta/plan.md` (6 milestones, M0-M6)
- Key architectural decisions documented:
  - `.ctxt/` directory (git-like search up to `~/`)
  - `.ctxt.work/` for ephemeral outputs
  - Silent-by-default operation
  - Noop LLM hooks throughout
  - Safe discovery with redaction

**Design Insights Captured:**
- Model 03's `.ctxt.work/` separation was brilliant (ephemeral vs persistent)
- Audit command for transparency (`ctxt audit`)
- Permission tracking in history for repeated runs
- Write-only to nearest `.ctxt/`, read from entire CWD → ~/ chain

## Next Session

**Priority Tasks:**
1. **M0: Transition & Preservation** (30 min)
   - Commit current state with clear message
   - Tag as `v0.0.9-template-mvp`
   - Create branch `v0.1-discovery-rewrite`
   - Archive old notebooks (02, 03, 04) to `_archive/`
   - Update README with version notice
   - Add `.ctxt.work/` to `.gitignore`

2. **M1: Foundation** (1 session)
   - Implement `.ctxt/` directory finder (walk CWD → ~/)
   - **CtxtRepo class** - git wrapper (init, commit, log, diff)
   - **Auto-initialize** `.ctxt/` as git repo on first use
   - **Suggest adding** `.ctxt/` to parent `.gitignore` (prompt once)
   - Define base classes (ContextSource, Aggregator, Formatter, LLM)
   - Create `05_core.ipynb` with Fragment, Context, CtxtRepo
   - Set up logging infrastructure

**Known Issues:**
- Current template system will be deprecated but preserved in tag/archive
- Need to decide on exact config schema for `.ctxt/config.yml` (vs old `.ctxt.yml`)
- Token estimation strategy needs definition

**What to Attach Next Session:**
- `meta/HANDOFF.md` (this file)
- `meta/plan.md` (the complete plan)
- `meta/notes.md` (previous session context if needed)
- `.cursor/rules/project.mdc` (updated project context)

## Session History

### 2025-11-10: Architecture Decisions (ADR-001, ADR-002)
- **Critical decision #1:** `.ctxt/` as auto-managed git repository (ADR-001)
  - Documented in `meta/decisions.md`
  - Updated M1 (add CtxtRepo) and M5 (simplify via git) in plan
  - Rationale: Prevents data loss, provides traceability, enables recovery
  - Strategy: Full auto-commit, nested repo pattern, parent ignores `.ctxt/`
- **Critical decision #2:** Two-tier chat transcript management (ADR-002)
  - SpecStory (ephemeral, automatic) + `.ctxt/chats/` (curated, manual)
  - ChatHistorySource discovers both tiers (M2+)
  - Closes meta-loop: ctxt discovers its own development
  - The conversation *is* the code (episodic memory layer)

### 2025-11-09: Architecture Definition & Planning
- Brainstormed context discovery engine concept
- Compared 4 model approaches to same problem
- Defined v0.1 architecture (sources/aggregator/formatters/history)
- Created 6-milestone plan (M0-M6, ~6-8 sessions estimated)
- Decided on migration strategy (feature branch with archive)

### 2025-01-08: Template System MVP
- Completed notebooks: 01_config, 02_tags, 03_templates, 04_cli
- Working tag substitution and multi-vendor template processing
- All tests passing, graceful error handling throughout

```

```markdown
---
alwaysApply: true
description: Project-specific context for ctxt
---
# General - Project
- Name: **ctxt** 

## Objective (v0.1 - Architecture Pivot)

**Old (v0.0.9):** Template processor with tag substitution for LLM context files  
**New (v0.1+):** Context discovery engine that automatically prepares LLM session context

**Vision:** Run `ctxt` in any project → discovers environment → generates context documents → LLM session ready

**Current Phase:** Architecture rewrite (v0.1-discovery-rewrite branch)

## Key Technical Decisions

### Architecture (v0.1)
- **Pipeline:** Discovery → Aggregation → Formatting → History
- **Pluggable:** All components use abstract base classes (Sources, Aggregators, Formatters, LLMs)
- **Storage:** `.ctxt/` directory (git repo: search CWD → ~/, aggregate configs)
- **Git Integration:** `.ctxt/` is auto-managed git repository (nested repo, auto-commit)
- **Ephemeral:** `.ctxt.work/` for temporary snapshots (gitignored)
- **Silent:** No stdout unless `--verbose`, all logs to `.ctxt/logs/`
- **Safe:** Read broadly, redact values, detect secrets by name only, no network

### Core Components
1. **Context Sources** - Discover one aspect (python_env, git, nbdev, ctxt)
2. **Aggregator** - Merge fragments, resolve conflicts, produce unified context
3. **Formatters** - Adapt to vendor formats (Cursor, generic markdown)
4. **CtxtRepo** - Git operations wrapper (auto-init, auto-commit, history access)
5. **History** - Lightweight metadata (git provides versioning)
6. **LLM** - Noop hooks throughout (identity function for v0.1)

### Migration Strategy
- v0.0.9 template system preserved at tag `v0.0.9-template-mvp`
- Feature branch: `v0.1-discovery-rewrite`
- Old notebooks archived to `_archive/`
- Reuse config loading logic from `01_config.ipynb`

## Module Organization (v0.1 Target)

```
ctxt/
├── core.py          # Fragment, Context, CtxtRepo, base classes (from 05_core.ipynb)
├── sources.py       # Context sources (from 06_sources.ipynb)
├── aggregator.py    # Aggregation logic (from 07_aggregator.ipynb)
├── formatters.py    # Output formatters (from 08_formatters.ipynb)
├── history.py       # Lightweight metadata (from 09_cli.ipynb)
└── cli.py           # CLI interface (from 09_cli.ipynb)
```

**v0.1 Sources:** python_env, git, nbdev, ctxt  
**v0.1 Formatters:** generic (markdown), Cursor (.mdc)

## Project-specific Patterns

### Data Structures
- `Fragment` - Single piece of discovered context (dataclass with provenance)
- `UnifiedContext` - Aggregated, model-independent context
- `CtxtRepo` - Git repository wrapper (init, commit, log, diff operations)
- `confidence` - All discoveries have 0.0-1.0 confidence score
- `sensitivity` - low/medium/high for privacy tracking

### Plugin API Pattern
```python
class ContextSource(ABC):
    def probe(ctx: Context) -> bool: ...
    def collect(ctx: Context, budget: int) -> list[Fragment]: ...
```

### Config Precedence
- Search: CWD → parent dirs → ~/ (collect all `.ctxt/` directories)
- Merge: Nearest config wins, parents contribute
- Write: Only to nearest `.ctxt/` (create if absent)

### Safety First
- Redact all sensitive values before aggregation
- Log everything read to `.ctxt/logs/report.md`
- Env vars: detect name only (e.g., "ANTHROPIC_API_KEY exists")
- No network calls, no writes outside `.ctxt/`

## Documentation

- **User-facing**: See `README.md` (auto-generated from `nbs/index.ipynb`)
- **Technical**: See `meta/plan.md` for v0.1 implementation plan
- **Architecture Decisions**: See `meta/decisions.md` for ADRs
- **Development**: See `@documentation.mdc`, `@notebook-structure.mdc`, `@python-style.mdc`
- **Deployed**: https://civvic.github.io/ctxt/

## Workflow Notes

- Development in Jupyter notebooks using nbdev
- Inline documentation and tests (no separate test files)
- `_testbed/` for exploratory work (e.g., model comparison chats)
- `_archive/` for deprecated code (v0.0.9 template system)
- Use `@nbdev` annotations for module generation
- Current plan: 6 milestones (M0-M6), ~6-8 sessions estimated

## Current Session Context

**Status:** Git integration architecture decision documented (ADR-001)  
**Next:** Execute M0 (Transition) - tag, branch, archive  
**Files:** `meta/plan.md` (complete plan), `meta/decisions.md` (ADR-001), `meta/HANDOFF.md` (session notes)

## Key Architecture Decisions

**ADR-001 (2025-11-10): Git-Based Workspace Management**
- `.ctxt/` is auto-managed git repository (nested repo pattern)
- Auto-commit after discover/aggregate/format operations
- Provides traceability, recovery, prevents data loss
- Parent project ignores `.ctxt/` for privacy
- Simplifies M5 (git replaces custom history system)

**ADR-002 (2025-11-10): Chat Transcript Management**
- Two-tier: SpecStory (ephemeral, automatic) + `.ctxt/chats/` (curated, manual)
- ChatHistorySource discovers both tiers (M2+)
- Closes meta-loop: ctxt discovers its own development conversations
- The conversation *is* the code (episodic memory layer)
- Zero friction: SpecStory workflow unchanged

See `meta/decisions.md` for full rationale on both ADRs

```

## ✅ ADR-002 Written and Integrated

ADR-002: Chat Transcript Management is now fully documented and cross-referenced.

### Files Updated

1. **`meta/decisions.md`** - Added ADR-002 (~300 lines)
   - Problem: Episodic memory loss, design rationale capture
   - Decision: Two-tier system (SpecStory + `.ctxt/chats/`)
   - Implementation: M0 directory structure, M2 ChatHistorySource, v0.2+ CLI
   - Data flow diagram showing all tiers
   - Example output when ctxt discovers itself

2. **`meta/HANDOFF.md`** - Updated session summary
   - Now references both ADR-001 and ADR-002
   - Session history updated with both decisions

3. **`.cursor/rules/project.mdc`** - Added ADR-002 summary
   - Quick reference for future AI sessions
   - Both ADRs visible at top level

---

## What's Captured

**Core Insight:** "The conversation transcript *is* the code. Python modules are crystallizations of discourse."

**Architecture:**
```
SpecStory (.specstory/history/) → Ephemeral, automatic
     ↓ ctxt discovers
.ctxt/chats/ → Curated, manual, git-tracked
     ↓ human extracts
meta/decisions.md → Canonical ADRs
     ↓ aggregation
Future AI sessions see design rationale
```

**Zero friction:** Your existing SpecStory workflow unchanged, ctxt reads it automatically.

---

## Next Steps (Your Side)

You'll create `.ctxt/chats/` directory when ready. The README content is documented in ADR-002 for copy/paste.

Additional design discussion in future session as you mentioned.

---

## Meta-Loop Status

We just designed how Layer 1 (this chat) feeds Layer 4 (ctxt discovering itself). The ouroboros now has a mouth. 🐍

Even if Cursor resets context now, everything is preserved:
- ✅ Git integration (ADR-001)
- ✅ Chat archival (ADR-002)  
- ✅ Both cross-referenced in plan, handoff, rules

Future AI sessions will see both decisions automatically.

```plaintext
SpecStory (.specstory/history/) → Ephemeral, automatic
     ↓ ctxt discovers
.ctxt/chats/ → Curated, manual, git-tracked
     ↓ human extracts
meta/decisions.md → Canonical ADRs
     ↓ aggregation
Future AI sessions see design rationale
```

---

**User**

`.ctxt/chats/` done.

---

**Cursor**

Perfect! ✅

```
.ctxt/
└── chats/
    ├── .gitkeep
    └── README.md
```

Infrastructure is ready. The two-tier archival system is now in place:

**Tier 1 (Automatic):** SpecStory → `.specstory/history/` (already working)  
**Tier 2 (Curated):** Manual → `.ctxt/chats/` (now ready)

---

## Session Summary

**Secured Today:**

1. ✅ **ADR-001:** Git-Based Workspace Management
   - `.ctxt/` as auto-managed git repository
   - Auto-commit strategy, CtxtRepo class design

2. ✅ **ADR-002:** Chat Transcript Management  
   - Two-tier system (SpecStory + `.ctxt/chats/`)
   - ChatHistorySource design for M2+
   - Meta-loop closure

3. ✅ **Infrastructure:** `.ctxt/chats/` directory created with README

**Documentation Updated:**
- `meta/decisions.md` - ADR-001, ADR-002
- `meta/plan.md` - M1/M5 tasks updated
- `meta/HANDOFF.md` - Session summary
- `.cursor/rules/project.mdc` - Quick reference

---

## Meta-Achievement Unlocked 🎯

We just completed a **self-referential design session**: discussed how to capture design sessions, documented the discussion about documenting, and created infrastructure to archive this very conversation.

The meta-loop is **operational**. This chat can now be:
1. Exported from Cursor → `.ctxt/chats/2025-11-10_architecture-decisions.md`
2. Already auto-saved by SpecStory → `.specstory/history/2025-11-10_17-55Z-next-steps-for-context-discovery-engine.md`
3. Future ctxt sessions will discover both automatically

---

## Ready for M0?

Next session can start **M0: Transition & Preservation** (tag, branch, archive) whenever you're ready.

Or we can continue discussion/refinement in this session if you prefer. What's your next move?

```plaintext
.ctxt/
└── chats/
    ├── .gitkeep
    └── README.md
```

---

**User**

Tomorrow more.

---

**Cursor**

Perfect timing. Excellent session today! 🎯

## Session Wrap-Up

**Achieved:**
- ✅ ADR-001: Git-based workspace management
- ✅ ADR-002: Chat transcript management  
- ✅ Infrastructure: `.ctxt/chats/` created
- ✅ All documentation updated and cross-referenced

**Everything preserved in:**
- `meta/decisions.md` (ADR-001, ADR-002)
- `meta/plan.md` (updated M1/M5)
- `meta/HANDOFF.md` (session summary)
- `.cursor/rules/project.mdc` (quick reference)
- `.ctxt/chats/` (ready for archival)

---

## Tomorrow's Session Checklist

**To Start:**
1. Attach `meta/HANDOFF.md` for continuity
2. Attach `meta/plan.md` for milestone details
3. Optional: Archive this chat to `.ctxt/chats/2025-11-10_architecture-decisions.md`

**Ready to Execute:**
- **M0: Transition & Preservation** (~30 min)
  - Tag current state as `v0.0.1-template-mvp`
  - Create branch `v0.1-discovery-rewrite`
  - Archive old notebooks to `_archive/`
  - Update README, `.gitignore`

---

**Hasta mañana, vic.** The meta-loop is secure, the architecture is sound, and the path forward is clear. 🚀


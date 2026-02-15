# 09 — Submitting to Archipelago

If you want your game to be included in the official Archipelago distribution, you will
need to submit a Pull Request to the main repository and go through the review process.

## Before You Submit

### Quality Checklist

- [ ] **All generic tests pass.** Run `pytest test/general/` — these run automatically
      against every world.
- [ ] **Your custom tests pass.** Run `pytest worlds/your_game/test/`.
- [ ] **The full test suite passes.** Run `pytest` with no filters.
- [ ] **No `eval()` usage.** Prohibited for security reasons.
- [ ] **No direct `yaml.load()`.** Use `Utils.parse_yaml()` instead.
- [ ] **No `multiworld.regions = ...`** or `multiworld.itempool = ...`. Always use
      `append`, `extend`, or `+=`.
- [ ] **Code follows the [style guide](/docs/style.md).** 120-char line limit,
      double-quoted strings, PEP 8, type annotations.
- [ ] **Game info doc exists** (at least `en_<Game Name>.md`).
- [ ] **Setup doc exists** (at least one tutorial/setup guide).
- [ ] **Items equal locations** (or filler balances them).
- [ ] **Completion condition is set.**
- [ ] **`create_item` is implemented** and works for any item name.

### File Structure Checklist

```
worlds/your_game/
├── __init__.py             ✓ World subclass with game name, IDs, hooks
├── items.py                ✓ Item definitions
├── locations.py            ✓ Location definitions
├── options.py              ✓ Options dataclass
├── rules.py                ✓ Access rules
├── archipelago.json        ✓ Metadata
├── docs/
│   ├── en_Your Game.md     ✓ Game info page
│   └── setup_en.md         ✓ Setup guide
└── test/
    ├── __init__.py          ✓ Empty init
    └── test_access.py       ✓ Logic tests
```

## GitHub Workflow

### Step 1 — Fork and Branch

```bash
# If you haven't already:
git clone https://github.com/<your-username>/Archipelago.git
cd Archipelago
git remote add upstream https://github.com/ArchipelagoMW/Archipelago.git

# Create your feature branch from the latest main:
git fetch upstream
git checkout -b world/your-game-name upstream/main
```

### Step 2 — Develop and Test

```bash
# Make your changes...
# Run tests frequently:
pytest worlds/your_game/test/ -v
pytest test/general/ -v

# Run the full suite before submitting:
pytest
```

### Step 3 — Commit and Push

```bash
git add worlds/your_game/
git commit -m "Add Your Game world integration"
git push origin world/your-game-name
```

### Step 4 — Open a Pull Request

1. Go to the upstream Archipelago repository on GitHub.
2. Click **"Compare & pull request"** for your branch.
3. Fill in the PR template:
   - **Title:** `New World: Your Game`
   - **Description:** Explain what the game is, what the integration does, and any
     notable design decisions.
   - **Testing:** Describe what you tested manually and what automated tests cover.
4. Submit the PR.

### Step 5 — Enable CI

Make sure GitHub Actions is enabled on your fork:
- Go to your fork → **Actions** tab → **Enable workflows**.
- CI will run automatically on your PR. Fix any failures.

### Step 6 — Respond to Review

Reviewers will check:

| Area | What they look for |
|------|--------------------|
| **Correctness** | Logic rules match the actual game. Fill succeeds. |
| **Style** | Follows the Archipelago style guide. |
| **Security** | No `eval`, no unsafe `yaml.load`, no file system abuse. |
| **Performance** | Generation doesn't take too long. Tests are fast. |
| **Documentation** | Game info and setup docs are clear and complete. |
| **Compatibility** | No breaking changes to core or other worlds. |

Common review feedback:
- "Use `Utils.parse_yaml()` instead of `yaml.load()`."
- "This item should be `progression`, not `useful`, because it's in a rule."
- "Please add a test for this edge case."
- "Line exceeds 120 characters."

Address all feedback, push updates, and re-request review.

## After Merge — Maintainer Responsibilities

When your world is merged, you become a **World Maintainer**. This means:

- **Monitor Discord** (`#ap-world-dev`) for bug reports and suggestions.
- **Review PRs** that affect your world.
- **Fix breakage** when core changes impact your code.
- **Test during RC phases** (release candidates) to ensure compatibility.
- **Communicate availability** — let core devs know if you'll be unavailable for
  extended periods.

If you cannot maintain the world long-term, consider:
- Nominating a co-maintainer.
- Releasing as an unofficial `.apworld` instead.

Your name will be added to the `CODEOWNERS` file for your world directory.

## Updating Your World After Merge

```bash
# Sync with upstream:
git fetch upstream
git checkout main
git merge upstream/main

# Create a new branch for updates:
git checkout -b world/your-game-fix-whatever

# Make changes, test, push, open PR...
```

## Next Step

Proceed to [10 - Troubleshooting](10_troubleshooting.md) for solutions to common problems.

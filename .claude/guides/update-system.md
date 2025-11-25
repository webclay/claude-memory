# Update System Guide

## For Vibe Coders: How to Update Claude Memory

### How to Update

Just type: `update`

Claude will ask you which way you prefer:

**Example:**
```
You: update

Me: "Checking for updates...

✅ New version available: v1.2.0

What's new:
- Better Next.js App Router support
- New Stripe webhook patterns
- Improved error handling

How would you like to update?

A. Automatic (I'll download it for you from GitHub)
B. Manual (You already downloaded it, I'll guide you through copying files)

Your choice?"

You: "A"

Me: "Great! Updating automatically...
✅ Created backup at .claude/.backup-v1.0.0/
✅ Downloaded 8 files from GitHub
✅ Updated system files
✅ Your projectbrief.md and tasks.md are safe

All done! Claude Memory is now v1.2.0"
```

**OR if you choose Manual:**

```
You: "B"

Me: "Perfect! I'll guide you step by step.

First, have you already downloaded the new Claude Memory ZIP from GitHub?"

You: "Yes"

Me: "Great! Unzip it if you haven't already, then I'll tell you exactly which files to copy..."
```
Me: "I'll walk you through this step by step!

From the NEW .claude folder you downloaded, copy this folder:
→ guides/

Paste it into YOUR current .claude/ folder. Replace when asked.

Done? Type 'yes'"

You: "yes"

Me: "Great! Now copy this folder:
→ stacks/

Paste it into YOUR .claude/ folder.

Done?"

You: "yes"

Me: "Perfect! Now copy these files one by one:
→ CLAUDE.md
→ commands.md
→ triggers.md
→ 00-start-here.md
→ standards.md

Done?"

You: "yes"

Me: "✅ Update complete! Your Claude Memory is now v1.2.0

Your projectbrief.md and tasks.md are safe - they weren't touched.

Type 'setup' and I'll verify everything works!"
```

---

### Rollback (If Something Goes Wrong)

If the update breaks something, type: `restore`

```
You: restore

Me: "I found a backup from before the update.

Restore to version 1.0.0? This will undo the update."

You: "yes"

Me: "Restoring...
✅ Restored all system files
✅ Your project data was never changed

Back to v1.0.0. Everything should work again!"
```

---

## For Developers: Update System Technical Details

### Implementation Notes

**Version Check Method:**
The system uses a **simple, no-API approach**:
1. Read local version from `.claude/VERSION`
2. Fetch GitHub README: `https://raw.githubusercontent.com/webclay/claude-memory/main/README.md`
3. Extract remote version using WebFetch tool (look for `**Version:** X.X.X` line)
4. Compare versions using semantic versioning

**No GitHub API required** - this avoids rate limits and authentication complexity.

### File Categories

**Never Update (User Data):**
- `.claude/projectbrief.md`
- `.claude/tasks.md`
- Any custom files user added

**Always Update (System Files):**
- `.claude/CLAUDE.md`
- `.claude/commands.md`
- `.claude/triggers.md`
- `.claude/00-start-here.md`
- `.claude/standards.md`
- `.claude/VERSION`
- `.claude/guides/**/*`
- `.claude/templates/**/*`

**Smart Update (Stack Modules):**
- `.claude/stacks/**/*`
- Check if user modified (compare checksums)
- If modified: show diff and ask user
- If not modified: update automatically

### Update Workflow

1. **Check Version**
   - Read local `.claude/VERSION`
   - Fetch README from GitHub: `https://raw.githubusercontent.com/webclay/claude-memory/main/README.md`
   - Extract version from the line: `**Version:** X.X.X`
   - Compare versions (simple semantic version comparison)

2. **Create Backup**
   ```
   .claude/.backup-v1.0.0/
   ├── CLAUDE.md
   ├── commands.md
   ├── triggers.md
   └── ... (all system files)
   ```

3. **Download Updates** (Automatic Mode Only)
   - Guide user to download from GitHub repository
   - Or fetch files directly using WebFetch from GitHub raw URLs
   - Example: `https://raw.githubusercontent.com/webclay/claude-memory/main/.claude/CLAUDE.md`

4. **Apply Updates**
   - Replace system files
   - Smart merge stack modules
   - Update VERSION file
   - Keep user data untouched

5. **Verify**
   - Check all files present
   - Validate format
   - Test basic functionality

### GitHub Release Format

Each release should:
- Update `README.md` with new version number on line `**Version:** X.X.X`
- Update `.claude/VERSION` file to match
- Create a git tag: `v1.2.0`
- Optionally include release notes on GitHub Releases page

### Version Comparison

```javascript
// Simple semantic versioning comparison
function compareVersions(current, latest) {
  const [maj1, min1, pat1] = current.split('.').map(Number);
  const [maj2, min2, pat2] = latest.split('.').map(Number);

  if (maj2 > maj1) return 'major';
  if (min2 > min1) return 'minor';
  if (pat2 > pat1) return 'patch';
  return 'same';
}
```

### Backup Strategy

- Keep last 3 backups
- Auto-delete older backups
- Each backup labeled with version
- Restore command picks latest backup

### Manual Update Process

Guide users through copying:
1. Folders first (guides/, stacks/, templates/)
2. Individual files second (CLAUDE.md, commands.md, etc.)
3. Verify at each step
4. Never mention projectbrief.md or tasks.md (they stay untouched)

---

## Publishing Updates to GitHub

### For Repository Maintainers

**Step 1: Prepare Release**
1. Update VERSION file to new version (e.g., `1.2.0`)
2. Update README.md version number
3. Write CHANGELOG.md entry
4. Test everything works

**Step 2: Create GitHub Release**
1. Go to GitHub repo → Releases → Create new release
2. Tag: `v1.2.0`
3. Title: `Version 1.2.0 - [Brief description]`
4. Description: User-friendly release notes (what's new, what's improved)
5. Attach: ZIP file of the `.claude` folder

**Step 3: Release Notes Format**
```markdown
## What's New in v1.2.0

### ✨ New Features
- Better support for Next.js App Router
- New Astro framework module added

### 🔧 Improvements
- Improved Stripe webhook handling
- Better error messages for vibe coders

### 🐛 Bug Fixes
- Fixed task tracking not updating
- Corrected projectbrief template formatting

### 📚 Documentation
- Added more examples to setup guide
- Simplified troubleshooting section
```

---

## Commands Reference

| Command | What It Does |
|---------|--------------|
| `update` | Check for updates and choose automatic or manual installation |
| `update check` | Just check if update available (don't install) |
| `restore` | Rollback to previous version from backup |

---

**Last Updated:** 2025-01-16
**Maintained By:** Claude Memory Team

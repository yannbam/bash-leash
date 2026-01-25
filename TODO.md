# TODO

## mv: Add destination-only path checking

**Priority:** Low
**Discovered:** Session 58a0ae94 (2026-01-25)

### Issue

`mv` is in `DANGEROUS_COMMANDS` but lacks the dest-only checking that `cp` has. This means `mv SOURCE DEST` validates BOTH paths, blocking legitimate "move from outside to inside" operations.

### Current behavior

```bash
mv /outside/file.txt /project/file.txt  # BLOCKED (should be allowed)
```

### Expected behavior

Same as `cp` — only validate the destination path, since the source is just being read/unlinked.

### Fix

Add the same dest-only logic as `cp` in `command-analyzer.ts`:

```typescript
// mv: only check destination (last path), source is just read/unlink
if (baseCmd === "mv" && paths.length > 0) {
  const dest = paths[paths.length - 1];
  if (!this.isPathAllowed(dest, true)) {
    return {
      blocked: true,
      reason: `Command "${baseCmd}" targets path outside allowed directories: ${dest}`,
    };
  }
  return { blocked: false };
}
```

### Notes

- Pre-existing issue, not caused by the extractPaths fix
- Low priority since moving files into project from outside is less common than copying

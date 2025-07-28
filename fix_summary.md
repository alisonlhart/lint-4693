# Fix Applied for Ansible-Lint Issue #4693

## Problem
The ansible-lint GitHub Action was installing development versions instead of stable releases when using tagged versions like `ansible/ansible-lint@v25.5.0`.

## Root Cause
The issue was in the `action.yml` file, specifically in the "Determine github action ref" step (lines 46-55). The expression:

```bash
action_ref="${{ github.action_ref || 'main' }}"
```

Was not working correctly. When `github.action_ref` was empty/undefined, the `|| 'main'` fallback within the GitHub expression wasn't functioning properly, causing the entire expression to evaluate to an empty string instead of "main".

## Fix Applied

### 1. Fixed the git reference logic in `action.yml`

**Before:**
```bash
action_ref="${{ inputs.gh_action_ref }}"
if [[ -z "${{ inputs.gh_action_ref }}" ]]; then
  action_ref="${{ github.action_ref || 'main' }}"
fi
```

**After:**
```bash
action_ref="${{ inputs.gh_action_ref }}"
if [[ -z "${{ inputs.gh_action_ref }}" ]]; then
  action_ref="${{ github.action_ref }}"
  # If github.action_ref is also empty, default to main
  if [[ -z "$action_ref" ]]; then
    action_ref="main"
  fi
fi
```

### 2. Fixed linter error - Changed boolean to string

**Before:**
```yaml
setup_python:
  description: If false, this action will not setup python and will instead rely on the already installed python.
  required: false
  default: true
```

**After:**
```yaml
setup_python:
  description: If false, this action will not setup python and will instead rely on the already installed python.
  required: false
  default: "true"
```

## How the Fix Works

1. **Input Check**: First checks if `gh_action_ref` input is provided (for manual override)
2. **Context Check**: If not, extracts `github.action_ref` into the bash variable
3. **Fallback Logic**: Uses bash-level logic to check if the extracted value is empty
4. **Default**: Only defaults to "main" if both the input and context are empty

This ensures that when someone uses `ansible/ansible-lint@v25.5.0`, the `github.action_ref` context (which should contain `v25.5.0`) is properly captured and used for pip installation.

## Expected Result

After this fix:
- `ansible/ansible-lint@v25.5.0` should install ansible-lint version 25.5.0
- `ansible/ansible-lint@main` should install the latest development version
- No more warnings about missing `[lock]` extras
- No more "pre-release version" warnings

## Files Modified
- `ansible-lint/action.yml` - Fixed git reference logic and linter error 
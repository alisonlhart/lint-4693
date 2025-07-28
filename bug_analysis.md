# Ansible-Lint Issue #4693 - Root Cause Analysis

## Issue Summary
When using the ansible-lint GitHub Action with a specific version tag (e.g., `ansible/ansible-lint@v25.5.0`), the action installs a development version of ansible-lint (`25.6.2.dev25`) instead of the expected stable version.

## Root Cause
Based on the GitHub Actions logs, the issue is in the action's internal logic for determining the git reference to use for installation.

### Evidence from Test Results
```bash
# Lines 113-115: The action checks for an empty variable and defaults to "main"
action_ref=""
if [[ -z "" ]]; then
  action_ref="main"
fi

# Lines 118-119: This results in using "main" instead of the tag
ACTION_REF: main
GH_ACTION_REF: main
```

### Expected vs Actual Behavior
- **Expected**: `pip install "ansible-lint[lock] @ git+https://github.com/ansible/ansible-lint@v25.5.0"`
- **Actual**: `pip install "ansible-lint[lock] @ git+https://github.com/ansible/ansible-lint@main"`

## Impact
1. **Wrong Version**: Installs `25.6.2.dev29` (development) instead of `25.5.0` (stable)
2. **Missing Extras**: Development version doesn't properly support the `[lock]` extra
3. **Pre-release Warning**: Users get warnings about using pre-release software

## Technical Details
- The action correctly downloads the `v25.5.0` tag (SHA: 4114ad63edbc25dcd9afc4f41b29dbcbebdf21ca)
- However, the action's shell script fails to pass the tag reference to the pip installation command
- The variable that should contain `v25.5.0` is empty, causing the fallback to `main`

## Potential Solutions

### 1. Fix the Action (Upstream)
The ansible-lint action needs to be fixed to properly capture and use the git reference from the action context. The issue is likely in the action.yml file where it sets the `GH_ACTION_REF` environment variable.

### 2. Workaround: Use Different Action Versions
- Use `ansible/ansible-lint@main` and specify the version explicitly
- Use a different linting approach that doesn't rely on the broken action logic

### 3. Workaround: Manual Installation
Install ansible-lint manually in the workflow instead of using the action:

```yaml
- name: Install specific ansible-lint version
  run: pip install ansible-lint==25.5.0

- name: Run ansible-lint
  run: ansible-lint
```

## Files Created for Reproduction
- `.github/workflows/ansible-lint.yml` - Reproduces the issue
- `test.yml` - Simple ansible playbook for testing
- `meta/requirements.yml` - Requirements file
- `test_results.txt` - Captured output showing the bug

## Next Steps
1. Report this analysis to the ansible-lint maintainers
2. The fix should be in the action.yml file to properly set GH_ACTION_REF variable
3. Until fixed, users should use manual installation or pin to working action versions 
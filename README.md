# Ansible-Lint Issue #4693 Reproduction

This repository reproduces the bug where the ansible-lint GitHub Action installs a development version instead of the stable release when using tagged versions.

## Issue Description

When using `ansible/ansible-lint@v25.5.0` or `ansible/ansible-lint@v25`, the action installs a development version of ansible-lint (`25.6.2.dev25`) instead of the expected stable version, resulting in warnings about missing extras.

## Expected Behavior

The action should install ansible-lint version 25.5.0 when using the `@v25.5.0` tag.

## Actual Behavior

The action installs development version `25.6.2.dev25` and displays warnings:
- `WARNING: ansible-lint 25.6.2.dev25 does not provide the extra 'lock'`
- `You are using a pre-release version of ansible-lint.`

## Root Cause Investigation

The issue appears to be that the v25.5.0 tag points to a commit where the version in the codebase is already `25.6.2.dev25`.

## Files

- `.github/workflows/ansible-lint.yml` - GitHub workflow that reproduces the issue
- `test.yml` - Simple ansible playbook for testing
- `meta/requirements.yml` - Requirements file for the action

## Investigation

To investigate this issue further, we need to:
1. Check the ansible-lint repository's action.yml at tag v25.5.0
2. Examine the version information in the codebase at that tag
3. Understand why the tag is pointing to a development version 
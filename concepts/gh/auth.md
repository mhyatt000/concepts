# GitHub Auth

## Two auth mechanisms

**`GH_TOKEN` (env var)**
- A raw Personal Access Token (PAT) generated at github.com/settings/tokens
- Read by `gh` CLI, the GitHub MCP server, and any tool that checks `GH_TOKEN`
- Set in shell profile: `export GH_TOKEN=ghp_...`

**`gh auth login` (keyring)**
- OAuth flow — `gh` stores the token in the macOS keychain
- Git uses it via the credential helper `gh auth git-credential`, which is wired into gitconfig automatically by `gh`

## Priority

When both exist, `GH_TOKEN` wins. An invalid or expired `GH_TOKEN` breaks git pushes and `gh` commands even if the keyring token is fine.

## Git credential helper

`gh auth login` injects this into `~/.gitconfig`:

```
[credential "https://github.com"]
    helper = !/usr/local/bin/gh auth git-credential
```

So `git push` calls `gh` to get a token. If `GH_TOKEN` is set, that's what `gh` hands back.

## Switching accounts

`gh` supports multiple logged-in accounts. Use `gh auth switch` to change which keyring account is active:

```sh
gh auth status               # see all accounts and which is active
gh auth switch --user <user> # make a specific account active
```

This is useful when `GH_TOKEN` is absent and you have multiple accounts in the keychain. The active account is what `gh auth git-credential` returns to git.

## Common failure

```
remote: Invalid username or token.
fatal: Authentication failed for 'https://github.com/...'
```

Cause: `GH_TOKEN` is set to a stale/invalid value.

Fix: unset it or replace it with a valid PAT.

```sh
unset GH_TOKEN  # temporary
# or remove from shell profile for permanent fix
```

## MCP server note

The GitHub MCP server also reads `GH_TOKEN`. If you want both `gh` CLI and the MCP server authenticated, set a valid PAT in `GH_TOKEN`.

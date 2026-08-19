# Repository Working Agreement

- Work directly on `main` for this documentation repository.
- Commit and push completed book and documentation changes without opening a pull request, unless the user explicitly asks for one.
- Preserve unrelated or unfinished local changes.
- On macOS, run authenticated `gh` commands outside the sandbox because GitHub CLI credentials are
  stored in Keychain. A sandboxed `gh auth status` can falsely report an invalid token even when the
  host session is valid; verify and execute credentialed `gh` operations with escalated host access
  instead of asking the user to authenticate again.

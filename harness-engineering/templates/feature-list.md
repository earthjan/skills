# Feature List

Every entry needs the full triple — behavior, verification command, state. An agent may move a feature to `blocked` or `active` itself; it may never set `passing` by assertion — only a green run of the verification command earns that state.

## <feature name>

- **Behavior**: one sentence, observable from the outside — what a caller/user can now do.
- **Verify**: `<exact command that must exit 0 to count this as done>`
- **State**: not_started | active | blocked | passing
- **Blocked on** (if `blocked`): ...

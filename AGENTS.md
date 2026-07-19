# Repository Working Agreement

These instructions apply to every change made in this repository.

## Repository ownership and remotes

- This is the `HeartlessAUS/juhradial-mx` fork.
- Fetch and push only to repositories under the `HeartlessAUS` GitHub account.
- Never push to the original JuhLabs repository or to any other original-author remote.
- Before every pull or push, inspect `git remote -v` and confirm the destination is under `HeartlessAUS/`.
- Do not contact, open a pull request against, or otherwise involve the original author unless the repository owner explicitly asks. We do not know the original author's preferences regarding AI-assisted work.
- Keep changes suitable for a possible future upstream pull request: avoid unnecessary rewrites, formatting churn, generated-file churn, or unrelated refactors.

## Branch and integration policy

- Never implement a feature or fix directly on `master` or `main`.
- Create a new, clearly named branch for each idea before editing, such as `feature/<topic>`, `fix/<topic>`, or `docs/<topic>`.
- Push feature branches to `HeartlessAUS/juhradial-mx`. Review and merge them into the fork's main branch there.
- Do not merge, force-push, rewrite published history, or delete remote branches unless explicitly requested.

## Decision making

- Do not invent missing requirements or silently choose among meaningful alternatives.
- Search the repository and its documentation first. If a question remains and affects architecture, behavior, UX, compatibility, safety, or scope, ask the repository owner before proceeding.
- State and document any small, unavoidable implementation fact inferred from existing code.
- Preserve user configuration and choice wherever practical; avoid hardcoded behavior when a documented setting is appropriate.

## Engineering priorities

- Target Arch Linux with KDE Plasma 6.7 on Wayland first.
- Preserve broader Linux, compositor, and distribution compatibility whenever possible.
- Prefer small, modular, maintainable changes with focused responsibilities and clear interfaces.
- Document non-obvious logic, data flow, edge cases, assumptions confirmed by the owner, setup changes, and troubleshooting information.
- Keep the core code close to the original fork. Add only the changes needed for the task.

## Completion requirements

- Test changes in proportion to their risk. Include automated tests for behavior where practical.
- Provide easy-to-read, copyable setup and test steps, including what each test verifies.
- Include manual checks for UI, Wayland, KDE, device, or integration behavior that automation cannot establish.
- If a repeated maintenance task is stable and genuinely useful, a documented workflow may be added. Do not add speculative automation.

See `docs/heartlessaus-development.md` for the repository overview, current boundaries, Arch/KDE development setup, and test commands.

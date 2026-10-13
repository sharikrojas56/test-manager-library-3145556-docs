# Commit Convention

This specification follows [Conventional Commits](https://www.conventionalcommits.org/). Each commit message uses the format:

```text
<type>(<optional scope>): <short imperative summary>
```

Use these types:

| Type | Use it for |
|---|---|
| `feat` | A new user-visible capability or specification section |
| `fix` | A correction to existing behavior or documentation |
| `docs` | Documentation-only changes |
| `refactor` | Internal restructuring with no behavior change |
| `test` | Adding or updating tests |
| `chore` | Tooling, maintenance, or configuration changes |
| `build` | Build system or dependency changes |
| `ci` | Continuous-integration configuration changes |

Examples:

```text
docs(spec-php): add English Laravel constitution
feat(api): add loan return endpoint
fix(api): reject duplicate books in a loan
chore(infra): add local environment template
```

Keep each commit focused on one completed, reviewable change. Use the present imperative mood, keep the summary lower-case, and do not end it with a period.

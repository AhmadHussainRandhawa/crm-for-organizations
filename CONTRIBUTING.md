# Contributing to CRM for Organizations

First off — thank you for considering contributing. This project exists to be a clear, well-engineered reference for multi-tenant SaaS architecture in Django, and every contribution that keeps it that way (clear, correct, well-tested) is genuinely valued.

This document exists so that contributing feels predictable: you'll know what a good branch looks like, what a good commit looks like, and what a good PR looks like, before you write a single line of code.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Ways to Contribute](#ways-to-contribute)
- [Before You Start](#before-you-start)
- [Development Setup](#development-setup)
- [The One Rule That Matters Most](#the-one-rule-that-matters-most-tenant-isolation)
- [Branching Strategy](#branching-strategy)
- [Commit Message Conventions](#commit-message-conventions)
- [Code Style](#code-style)
- [Testing Requirements](#testing-requirements)
- [Pull Request Process](#pull-request-process)
- [Reporting Bugs](#reporting-bugs)
- [Proposing Features](#proposing-features)
- [Documentation Changes](#documentation-changes)
- [Review Process & What to Expect](#review-process--what-to-expect)
- [Recognition](#recognition)

---

## Code of Conduct

Be respectful, be direct, assume good faith. Disagreements about code and architecture are welcome and expected — this project has opinions, and pushback on those opinions is how it gets better. Disrespect toward people is not welcome, ever.

---

## Ways to Contribute

You don't need to write a feature to contribute meaningfully:

- **Bug reports** — a clear, reproducible bug report is a real contribution
- **Bug fixes** — small, focused fixes are the easiest PRs to review and merge
- **Features from the [Roadmap](README.md#roadmap)** — the README roadmap is deliberately kept current; pick anything unclaimed
- **Tests** — the `leads/tests/` suite is not exhaustive; coverage gaps are good first contributions
- **Documentation** — README clarity, docstrings, and this file itself can always improve
- **Architecture review** — if you spot a place where the organization-scoping pattern isn't followed, that's a high-value issue even without a fix attached

---

## Before You Start

- **Check open issues first.** Someone may already be working on it, or there may be context you're missing.
- **For anything non-trivial, open an issue before opening a PR.** A quick issue describing your intended approach saves everyone time — including you, if the maintainer would have suggested a different direction.
- **Small fixes (typos, obvious bugs) can skip straight to a PR.**

---

## Development Setup

Full setup instructions live in the [README](README.md#getting-started). The short version:

```bash
git clone git@github.com:AhmadHussainRandhawa/crm-for-organizations.git
cd crm-for-organizations
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Before writing any code, run the existing test suite once to confirm your environment is clean:

```bash
python manage.py test
```

---

## The One Rule That Matters Most: Tenant Isolation

This project's entire value proposition is that data is correctly isolated per organization. That means there is exactly one rule in this codebase that overrides every other style preference:

> **Every queryset that touches `Lead`, `Category`, or any future tenant-owned model MUST be filtered by `organization`. No exceptions, no "I'll fix it later."**

```python
# Correct — every access path is organization-scoped
Lead.objects.filter(organization=request.user.userprofile)

# Wrong — this leaks every organization's leads to whoever hits this view
Lead.objects.all()
```

If your PR adds a new model that belongs to an organization, or a new view/queryset that touches `Lead` or `Category`, the reviewer will check this first, before readability, before style, before anything else. A PR that regresses tenant isolation will not be merged regardless of how good the rest of the change is — this is the one line I hold firm on.

If you're unsure whether something needs organization scoping, ask in the PR description rather than guessing.

---

## Branching Strategy

This repo uses a lightweight, purpose-driven branching model off `main`. No `develop` branch — the project is small enough that it adds process without adding safety.

| Branch prefix | Use for |
|---|---|
| `feature/<name>` | New functionality (e.g. `feature/lead-notes`) |
| `fix/<name>` | Bug fixes (e.g. `fix/agent-queryset-leak`) |
| `docs/<name>` | Documentation-only changes (e.g. `docs/contributing-guide`) |
| `refactor/<name>` | Internal restructuring with no behavior change |
| `test/<name>` | Test additions or improvements with no production code change |
| `hotfix/<name>` | Urgent fixes to `main`, typically security or data-isolation issues |

**Rules:**
- Branch names are lowercase, hyphen-separated, and describe the change — not the issue number alone (`fix/agent-queryset-leak`, not `fix/issue-42`)
- One branch = one concern. If your branch is doing two unrelated things, it should be two branches and two PRs
- Rebase onto `main` before opening a PR if `main` has moved significantly — keep history linear where reasonably possible

---

## Commit Message Conventions

This project follows **[Conventional Commits](https://www.conventionalcommits.org/)**. Every commit message has a type, an optional scope, and a clear description.

```
<type>(<scope>): <short description>

<optional longer body explaining why, not just what>
```

**Types used in this project:**

| Type | Use for |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `docs` | Documentation only |
| `test` | Adding or correcting tests |
| `style` | Formatting, whitespace — no logic change |
| `perf` | Performance improvement |
| `build` | Dependency or build-tooling changes |
| `ci` | CI/CD configuration changes |
| `chore` | Maintenance tasks that don't fit elsewhere |

**Good commit messages:**
```
feat(leads): add category-based lead filtering to agent dashboard
fix(agents): scope agent list queryset to request user's organization
refactor(leads): extract organization-scoping logic into a shared mixin
docs(readme): add multi-tenancy architecture diagram
test(leads): add coverage for cross-organization access attempts
```

**Not acceptable, ever:**
```
update
fix bug
changes
wip
final
```

**Commit hygiene:**
- **Atomic commits.** One commit = one logical change. If you can't describe a commit in one sentence, split it.
- **Commit early, squash before merge if history is messy.** It's fine to commit often while working; before opening (or right before merging) a PR, squash exploratory/fixup commits into clean, meaningful ones.
- **The commit body should explain *why*, not just repeat the diff**, whenever the reasoning isn't obvious from the code itself. This matters most for anything touching organization-scoping or permission logic.

---

## Code Style

- Follow **PEP 8** for Python; follow existing Django conventions in the codebase (class-based views, not function-based, unless there's a strong reason)
- Keep views thin — business logic belongs in model methods, managers, or forms, not sprawled across `views.py`
- Name things for what they are: `organization_queryset`, not `qs2`
- New forms that touch agent/lead assignment must filter their querysets by the logged-in user's organization, following the existing pattern in `leads/forms.py`
- Run a formatter (e.g. `black`) before committing if you have one configured — consistency matters more than any specific style choice

---

## Testing Requirements

- **New features require tests.** A PR adding a feature with zero test coverage will be asked to add tests before merge.
- **Bug fixes should include a regression test** that fails without the fix and passes with it.
- **Anything touching organization-scoping or permissions requires a test that verifies isolation** — e.g. that a user from Organization A cannot access Organization B's leads through the new code path. This is the highest-value test category in this repo.
- Run the full suite before opening a PR:

```bash
python manage.py test
```

---

## Pull Request Process

1. **Branch from `main`** using the naming convention above.
2. **Keep the PR scoped** to the branch's stated concern. If you find an unrelated issue while working, open a separate issue or PR for it rather than folding it in.
3. **Write a clear PR description** covering:
   - **What** changed and **why**
   - **Testing performed** (what you ran, what you verified manually)
   - **Breaking changes**, if any
   - **Screenshots**, for any UI change
4. **Ensure the test suite passes** locally before requesting review.
5. **Link the related issue**, if one exists (`Closes #12`).
6. **Expect review comments** — they're part of the process, not a rejection. See below.

### PR Title Format

PR titles follow the same Conventional Commits format as individual commits, since a squash-merge uses the title as the final commit message:

```
feat(leads): add lead activity history
fix(agents): prevent cross-organization agent visibility in assignment form
```

---

## Reporting Bugs

Open an issue with:

- **Steps to reproduce** — precise enough that a maintainer can hit the same bug
- **Expected behavior** vs **actual behavior**
- **Environment** (Python version, OS, browser if UI-related)
- **Whether it involves cross-organization data exposure** — flag this explicitly and prominently if so; it's treated as a priority issue, not a routine bug

---

## Proposing Features

Open an issue describing:

- **The problem** the feature solves, not just the feature itself
- **How it fits the existing architecture** — particularly, how it respects organization-scoping if it touches tenant data
- **Rough scope** — is this a small addition or a significant architectural change? Both are welcome, but the review bar and discussion depth will differ accordingly

Roadmap items in the [README](README.md#roadmap) are pre-approved in direction — you can go straight to a design discussion rather than justifying the feature itself.

---

## Documentation Changes

Docs-only changes (`docs/` branch prefix) still go through the PR process, but review is lighter-weight. If your change to code would make the README, architecture diagrams, or this file inaccurate, updating those docs in the same PR is expected, not optional — a PR that changes the isolation logic without updating the architecture diagram will be asked to include that update.

---

## Review Process & What to Expect

Every PR is reviewed for, roughly in this order:

1. **Tenant isolation correctness** — non-negotiable, checked first
2. **Test coverage** — especially for permission and scoping logic
3. **Scope discipline** — does the PR do one thing well
4. **Code clarity** — is it readable by someone new to the codebase
5. **Style consistency** — last, and usually the easiest to fix

Expect feedback, not silence. If a PR sits without review longer than a few days, a polite ping is completely fine.

---

## Recognition

Every merged PR gets credit in the commit history and, for significant contributions, a mention in release notes. This is a learning-focused project — contributors improving the codebase's clarity are doing exactly what it's meant for.

---

Thank you for reading this far — that alone puts you ahead of most contributors on most repos. Looking forward to your PR.

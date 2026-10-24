# Project template

Use **Use this template** on GitHub to start a project with the shared
[Workflow Kit](https://github.com/snagg-app/workflow-kit). The kit lives in a shared
checkout beside the project. CI pins its revision; improvements arrive as update PRs.
Add your own application and project-specific contracts without copying kit code.

The current kit supports Codex and Claude Code directly. Other agents can read
`AGENTS.md` and `.agents/skills` manually; this template provides no automatic
hook or discovery adapter for them. Obsolete generated host copies are removed.

For agent-guided setup, give your agent the [setup entry](https://github.com/snagg-app/workflow-kit/blob/main/SETUP.md).
It asks for repository, board and integration choices, then performs and verifies setup.

## Setup

Use Git, Node 26 and authenticated GitHub CLI:

```sh
git clone --recurse-submodules https://github.com/snagg-app/workflow-kit.git
git clone https://github.com/OWNER/REPO.git
cd REPO
node ../workflow-kit/scripts/init-project.mjs --cwd . --existing
node ../workflow-kit/scripts/setup-skills.mjs --cwd .
node .vendor/workflow-kit/scripts/setup-github.mjs OWNER/REPO
```

Reuse an existing shared kit checkout; set `WORKFLOW_KIT_DIR` if it lives elsewhere.
Setup creates the ignored `.vendor/workflow-kit` link. Shared harness commands run
from this project's root and act on its checkout.
This template contains no forwarding
scripts; keep only actual product-specific scripts when adding an application.

The final command copies the public board template, links this repository and
creates missing labels. Complete and commit `.github/workflow-project.json`; its
generic `skills: "all"` preserves this template's skill set without personal board IDs.
Configure native
statuses in order: Backlog → Ready → In progress → Automated review → Human review → Done.
Ready for Review starts Automated review; Human review requires completed automatic
reviews, dispositioned findings and passed checks (disclose confirmed unavailable
reviewers under the shared workflow exception). Further edits restart the review cycle.
Done still requires human acceptance and merge. Configure native
Project Auto-add for your repo and verify status automations in the GitHub UI;
cancelled work must not become Done. No private Vaultdex items are copied.

After reviewing the checkout, install hooks explicitly:

```sh
node .vendor/workflow-kit/scripts/provider.mjs ponytail install
node .vendor/workflow-kit/scripts/provider.mjs impeccable install
```

Review/enable hook definitions in each agent. Personal settings and plugin installs
are preserved. CodeRabbit requires GitHub App access; Codex requires a connected
repository and automatic reviews enabled in Codex settings. Organization-wide App
access can already cover the new repo. Configuration alone does not prove activation.
Read the kit's setup/security notes first; the known Ponytail PATH finding
vaultdex/workflow-kit#3 is not fixed by moving scripts into the kit.

## Working and updating

Follow [CONTRIBUTING](CONTRIBUTING.md) and [AGENTS.md](AGENTS.md). Fill in your
architecture, build/test commands and product acceptance. Product CI and Sonar
project keys are specific to your project; this template does not invent them.

Renovate proposes kit, Actions and any subsequently added supported manifest updates.
All update types, including major and security updates, may auto-merge after required
native checks pass; tests are never ignored. Dependabot handles recognized security
updates; its metadata-only queue requests native auto-merge without checking out PR code.
Version updates come from Renovate to avoid duplicate proposals.

Git-ref updates have no release timestamp and are exempt from the
three-day release-age rule. If a Kit update changes generated outputs, the
existing integration check fails until those outputs are committed to that
same update PR using the commands below. Once checks pass, auto-merge still
applies to every update type; a red integration check never grants a merge.

Before enabling repository auto-merge in a generated project, configure its required
checks and their GitHub Apps. Copying a GitHub template does not copy branch protection
or repository settings. Kit integration proves generated outputs, not future application
behavior; add appropriate product checks before relying on them.

The session-start hook updates the shared checkout; committed skill snapshots stay
unchanged until explicitly refreshed. CI independently checks the exact
`WORKFLOW_KIT_REF` in `.github/workflows/workflow-kit.yml`, updated by Renovate's
[git-refs manager](https://docs.renovatebot.com/modules/datasource/git-refs/).
For an update PR, inspect that revision and use a separate kit checkout to avoid
changing a shared checkout used by other projects:

```sh
git clone --recurse-submodules https://github.com/snagg-app/workflow-kit.git ../workflow-kit-review
git -C ../workflow-kit-review checkout <reviewed-kit-commit>
git -C ../workflow-kit-review submodule update --init --recursive
node ../workflow-kit-review/scripts/init-project.mjs --cwd . --existing
node ../workflow-kit-review/scripts/setup-skills.mjs --cwd . --refresh
```

Commit generated provider skills, companions and managed configuration in the same
PR. Setup moves replaced skill copies into ignored `.workflow-kit/replaced/`;
init preserves foreign hook handlers. Personal hook installation and trust remain
separate steps, including renewed review of changed hook definitions.
CI regenerates managed configuration and discovery, stages new paths with
`git add --intent-to-add --all`, then checks `git diff --exit-code HEAD` so missing
or stale committed output fails. Afterwards, run `init-project` and `setup-skills`
without `--refresh` from the regular shared checkout to restore its local link.

For upgrades from the submodule-based template, preserve any local kit edits first,
remove the old `.vendor/workflow-kit` submodule, then run setup from the shared
checkout. Update external workspace bootstrap and hook-install callers accordingly.

The included PR check verifies kit integration, not application correctness.
Budget: one standard Linux job, no schedule/cache/artifacts, maximum five minutes
per run. Public standard-runner usage is free; review private-repo and reviewer
budgets before enabling new projects. No paid plans are enabled by setup.

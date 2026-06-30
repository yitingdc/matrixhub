# Contributing to MatrixHub

Welcome, and thank you for your interest in contributing to MatrixHub! 🎉

MatrixHub is a community-led open source project. Contributions of all kinds are
welcome — code, tests, documentation, bug reports, design feedback, reviews, and
helping other users. This guide explains how to get involved.

By participating, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md)
(the CNCF Code of Conduct).

## Ways to contribute

You don't have to write code to make a difference:

- **Report bugs** and **request features** using the [issue templates](https://github.com/matrixhub-ai/matrixhub/issues/new/choose).
- **Improve documentation** — the `docs/`, the `website/`, and inline docs.
- **Review pull requests** and help triage issues.
- **Answer questions** and help others in [Slack](https://cloud-native.slack.com/archives/C0A8UKWR8HG) and [GitHub Discussions](https://github.com/matrixhub-ai/matrixhub/discussions).
- **Write code** — bug fixes and features.

New here? Look for issues labeled
[`good first issue`](https://github.com/matrixhub-ai/matrixhub/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
and
[`help wanted`](https://github.com/matrixhub-ai/matrixhub/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22).
Feel free to comment on an issue to let us know you're working on it.

## Reporting issues

- **Bugs & features:** open an issue via the [templates](https://github.com/matrixhub-ai/matrixhub/issues/new/choose).
  Before filing, please search existing [issues](https://github.com/matrixhub-ai/matrixhub/issues)
  and [pull requests](https://github.com/matrixhub-ai/matrixhub/pulls) to avoid duplicates.
- **Security vulnerabilities:** **do not** open a public issue. Follow the private
  reporting process in [SECURITY.md](SECURITY.md).

## Communication

- **GitHub Issues / Pull Requests** — primary development venue.
- **GitHub Discussions** — questions, ideas, and broader design conversations.
- **Slack** — [`#matrixhub`](https://cloud-native.slack.com/archives/C0A8UKWR8HG) in the CNCF Slack workspace.
- **Public roadmap** — [docs/roadmap.md](docs/roadmap.md).

For large or breaking changes, please open an issue or discussion **before**
investing significant implementation effort, so maintainers can align on the
approach early (see [GOVERNANCE.md](GOVERNANCE.md)).

## Local development

### Prerequisites

- [Go](https://go.dev/) (see the version in [`go.mod`](go.mod))
- [Node.js](https://nodejs.org/en) + [pnpm](https://pnpm.io/) (for the UI and website)
- [Docker](https://www.docker.com/) (for MySQL and building images)
- MySQL — required by the API server (see [docs/development.md](docs/development.md))

### Run the API server

The API server needs MySQL running first (see [docs/development.md](docs/development.md)
for the connection setup):

```bash
make local-run-api        # go run ./cmd/matrixhub apiserver
```

### Run the web frontend

The UI dev server (Vite) runs separately and proxies API calls to the backend —
no MySQL needed just to work on the frontend:

```bash
make local-run-web        # cd ui && pnpm i && pnpm dev
```

### Build a binary or container image

```bash
go build ./cmd/matrixhub  # build the matrixhub binary
make image-build          # build the container image
```

### Documentation website

```bash
make serve-website        # run the docs site locally
```

### Faster builds behind a proxy

If downloads are slow, use HTTP(S) proxies, an image mirror, and/or a Go module proxy:

```bash
export HTTP_PROXY=http://your-proxy:port
export HTTPS_PROXY=http://your-proxy:port
export BASE_IMAGE_PREFIX=m.daocloud.io/docker.io
export GOPROXY=https://goproxy.io,direct
make image-build
```

For a quick non-development run via Docker Compose, see the **Quick Start** in the
[README](README.md).

## Making a change

1. **Align on scope.** Open or pick up an issue (use the issue templates). For
   large or complex work, agree on the approach in the issue/discussion first; for
   complex backend changes, write a short design doc under `docs/design/` and get
   maintainer review **before coding**.
2. **Fork** the repository and create a topic branch.
3. **Make your change**, following the architecture rules in
   [AGENTS.md](AGENTS.md) (backend) and [ui/AGENTS.md](ui/AGENTS.md) (frontend).
4. **Add tests** (see the [testing policy](#testing-policy)) and run them locally.
5. **Run the checks** below and make sure they pass.
6. **Sign your commits** (see [Developer Certificate of Origin](#developer-certificate-of-origin-dco)).
7. **Open a pull request** that links the issue (e.g. `Closes #123`) and fills in
   the PR checklist.

## Coding standards

- **License headers:** every Go source file must start with the Apache 2.0 license
  header (enforced by the `goheader` linter).
- **Linting:** run and fix lints before pushing:

  ```bash
  make lint-fix             # golangci-lint v2.8.0 with --fix
  ```

- **Generated code is not hand-edited.** Regenerate it after changing the source of
  truth:

  ```bash
  make genproto             # after editing api/proto/v1alpha1/*.proto
  make gen_openapi_sdk      # after swagger changes (test HTTP SDK)
  make generate-mocks       # after changing a mocked interface
  ```

- Follow the conventions and dependency direction described in
  [AGENTS.md](AGENTS.md) and [CLAUDE.md](CLAUDE.md).

## Developer Certificate of Origin (DCO)

All commits must be **signed off** to certify that you wrote the patch or otherwise
have the right to submit it under the project's open source license, per the
[Developer Certificate of Origin](https://developercertificate.org/).

Add a `Signed-off-by` line by committing with `-s`:

```bash
git commit -s -m "Your commit message"
```

This adds a line like:

```
Signed-off-by: Your Name <your.email@example.com>
```

The name and email must match your Git author identity. If you forget, you can
amend the last commit with `git commit --amend -s`, or sign off a range of commits
with `git rebase --signoff`.

## Testing policy

MatrixHub relies on an automated test suite to stay stable. As a matter of policy:

- **New functionality and bug fixes MUST come with tests.** A pull request that
  adds or changes behavior is expected to add or update automated tests covering
  that behavior. Reviewers may request tests before approving.
- **Domain logic must have unit tests.** Code under `internal/domain/...` and
  `internal/jobserver/...` is pure, dependency-light logic and must be covered by
  unit tests. Adapters (`internal/repo/...`) and transport/handlers are primarily
  covered by end-to-end tests.
- **Coverage of changed code is checked in CI.** New/changed lines are measured by
  Codecov's patch report on each PR.

### Running the tests

```bash
go test ./...                                  # all unit tests
go test ./internal/jobserver/...               # jobserver logic (no DB needed)
make test.e2e level=smoke                      # e2e against a running instance
make test.e2e.kind                             # full e2e in a KIND cluster
```

See [docs/development.md](docs/development.md) for prerequisites.

## Pull request and review process

- Keep PRs focused and reasonably small; link the issue they address.
- Ensure **CI is green** (lint, unit tests, and other checks) and that your commits
  are **signed off**.
- Fill in the PR template `release-note` block for user-visible changes (see
  [Release notes in pull requests](#release-notes-in-pull-requests)).
- Reviews follow the [`OWNERS`](OWNERS) model: maintainers use `/lgtm` and `/approve`
  on PRs. A PR is merged once it has the required approvals and CI passes.
- Be responsive to review feedback; maintainers aim to review promptly but this is a
  community project, so please be patient.

See [GOVERNANCE.md](GOVERNANCE.md) for roles, decision making, and how to become a
maintainer.

## Release notes in pull requests

MatrixHub follows the [Kubernetes release notes model](https://github.com/kubernetes/community/blob/main/contributors/guide/release-notes.md):

- **Official releases only** (`vX.Y.Z`) publish release notes on the
  [GitHub Releases](https://github.com/matrixhub-ai/matrixhub/releases) page and in
  [`CHANGELOG/`](CHANGELOG/README.md). RC and dev tags do **not** publish release notes.
- **Every pull request** with a user-visible change must include a `release-note`
  block in the PR description (or `NONE` if not user-facing). The PR template already
  includes this section.
- Reviewers should check release note quality during review (clear, past tense,
  user-focused).

Example:

```release-note
Added permission-based filtering to the project list API. (#664, @contributor)
```

Maintainers aggregate these notes into `CHANGELOG/` when cutting an official release.
See [Release process](docs/release-process.md) for maintainer steps.

## License of contributions

MatrixHub is licensed under the [Apache License 2.0](LICENSE). By contributing, you
agree that your contributions will be licensed under the same license (inbound =
outbound), and you certify your right to do so via the DCO sign-off described above.

---

Thank you for helping make MatrixHub better! If anything here is unclear, open an
issue or ask in [Slack](https://cloud-native.slack.com/archives/C0A8UKWR8HG).

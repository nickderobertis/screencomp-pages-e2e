# AGENTS

Durable constraints for this fixture. Everything here exists to prove one thing
about [`screencomp`](https://github.com/nickderobertis/screencomp) that its own
repository cannot: that a gallery deploy really reaches GitHub Pages, and that a
build which fails really fails the run.

## Keep this repository

Public, undeleted, unarchived. It is the **only** live proof of the Pages deploy
paths, it is re-runnable by design, and the galleries it publishes must stay
anonymously fetchable (GitHub's image proxy, which renders PR-comment thumbnails,
only reads public Pages sites). Its `gh-pages` content is generated and
disposable; the four files on `main` are not.

## What cannot be tested upstream, and why

screencomp's offline Rust suite executes the shipped shell and stubs the GitHub
API, but it cannot execute a composite action, the artifact hand-off between
jobs, the `peaceiris/actions-gh-pages` push, or a real Pages build. This
workflow does all four.

It mirrors screencomp's reusable workflow's job graph rather than calling it: the
reusable workflow pins its composite actions to the floating `@v0` tag, and a
brand-new or just-changed action does not exist under `@v0` until it ships. So a
pre-release self-test has to reference the actions by the ref under test —
`screencomp-ref.txt`.

## Running a proof

Write one scenario into `scenario.txt`, the screencomp ref into
`screencomp-ref.txt`, and push to `main`. One push runs one scenario; runs
serialize on a static concurrency group because Pages builds are per-repository
and overlapping runs are the very race under test.

| `scenario.txt` | Proves |
|:---|:---|
| `multi` | Two project lanes → **one** `gh-pages` commit, both galleries published, Pages reaches `built`. |
| `single-direct` | One lane on the direct per-lane deploy (`pages-artifact` empty). Asserts in-lane that its own build settled before the action returned, and records the published tree. |
| `single-coalesced` | The same lane through the coalesced path. Its tree must match `single-direct` byte for byte — the single-project no-op proof. |
| `errored` | Forces a genuinely failing Pages build and asserts the gate fails the run loudly. |

Re-run `multi`, `single-direct`, and `single-coalesced` whenever the coalescing,
the artifact staging, or the build gate changes; re-run `errored` whenever the
gate's failure handling changes.

## Constraints that bite

- Timing: a healthy Pages build settles in ~20-30s, but GitHub holds a build it
  is going to FAIL in `building` for **13-27 minutes** (measured on this
  fixture). Size any wait against that, and never assume a failing build reports
  promptly. A superseded build is the exception — it errors in seconds.
- `permissions:` on the deploy/report jobs are deliberately what `screencomp
  init` scaffolds (`pages: read`, no `pages: write`), so the gate meets an
  errored build with its one rebuild denied and must still fail. Widening them
  would make the proof weaker than the real caller.
- The `before` job clears the lanes the run republishes before snapshotting.
  peaceiris commits nothing when the published bytes are unchanged, so without
  that a re-run adds 0 commits and the "exactly one commit" assertion proves
  nothing.
- The `errored` scenario must always restore a buildable site (`.nojekyll`, no
  `_config.yml`) even when it fails, or every later run starts broken.
- Nothing here is authored content. Do not hand-edit `gh-pages`.

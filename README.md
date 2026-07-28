# screencomp-pages-e2e

Disposable CI fixture for [`screencomp`](https://github.com/nickderobertis/screencomp).

It proves screencomp's **GitHub Pages deploy paths** end to end against a real
Pages site: the composite actions really run, galleries really move through the
artifact boundary, `gh-pages` is really pushed, and the real Pages build is
really polled. Nothing here is authored — every file under `gh-pages` is
generated, and `scenario.txt` selects which proof the next push runs.

Scenarios (write one into `scenario.txt` and push to `main`):

| `scenario.txt` | Proves |
|:---|:---|
| `multi` | Two project lanes → **one** `gh-pages` commit, both galleries published, Pages reaches `built`. |
| `single-direct` | One lane on the direct per-lane deploy (`pages-artifact` empty). Asserts in-lane that its own Pages build settled before the action returned, and records the published tree. |
| `single-coalesced` | The same lane through the coalesced path. Its tree must match `single-direct` byte for byte. |
| `errored` | Forces a genuinely failing Pages build and asserts the gate fails the run loudly. |

The `screencomp` ref under test is `screencomp-ref.txt`. See [AGENTS.md](AGENTS.md)
for the constraints this fixture is held to.

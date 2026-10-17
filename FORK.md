# Fork provenance and maintenance

The first fork release is `v1.1.2`, based on
[`makefu/jura-connect-hass` v1.1.1](https://github.com/makefu/jura-connect-hass/tree/v1.1.1),
commit `780758792628f4a4ac69567640bb9081ab18a18d`.
The original MIT license and copyright notice are preserved.

## Scope

There are two production fixes:

1. Poll health is distinct from retained last-known values. Only an actual
   successful backend fetch updates the UTC timestamp. Every caught fetch
   failure clears the success flag, including tolerated offline failures.
2. The existing coordinator lock covers a ten-second quiet period after the
   preceding backend operation ends. All four I/O paths use the gate. The
   first operation is immediate, elapsed quiet time is reused, and the gate
   never retries a command.

The protocol dependency is pinned to `jura_connect==0.14.0`. There is no SDK
fork, new brew-completion API, background post-brew refresh, single-session
recipe runner or recipe definition in Python. `jura.brew` still follows its
progress stream and awaits the stock post-brew refresh before returning.

## Validation and limits

The tracked regression tests use mocked I/O and a coordinator-local test
clock. The legacy suite disables real cooldown sleeps through a checked-in
fixture; the dedicated session tests restore the ten-second gap and verify
its timing without changing the process-wide clock or event loop.

The production coordinator, sensor and backend files match the previously
validated working deployment:

| File | SHA-256 |
| --- | --- |
| `coordinator.py` | `6a9fb3cc526f2ea6c3beca404ded2c53607abc6c812598e933b4d73f2e48882d` |
| `sensor.py` | `e472da8f3aaffa6c7f0929dee3959f4a2b365ce1ccf98846a6468c90fea05c92` |
| `backends/jura.py` | `f03daea9ba7e634a79c5d7b9269b60ab5c08533d0cb298e8955edcb3cbd4b030` |

This gate coordinates one active coordinator. Another process, J.O.E. or a
second independently configured client is outside its lock. Cancellation of
an awaited coroutine does not prove that its synchronous worker or a physical
drink has stopped. No cross-process or physical-abort guarantee is claimed.

These regression tests do not establish a reset's exact cause, a universal
minimum gap, measured delivered volume or temperature, or support for
protected-PIN authentication on every dongle. They exercise synthetic backend
responses and never contact a real appliance.

Future updates should first inspect upstream changes, rebase the two focused
fixes, keep the protocol version explicit, and rerun tests, formatting, lint,
typing and the HACS/hassfest checks before release. Recipe optimizations are
a separate change; they must not enter this initial fork release silently.

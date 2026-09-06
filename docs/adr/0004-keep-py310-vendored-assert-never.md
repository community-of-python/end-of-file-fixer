# Keep Python 3.10; vendor `_assert_never` until 3.10 EOL

**Decision:** Support Python 3.10 for now and enforce match exhaustiveness with a locally-defined
`_assert_never(value: NoReturn) -> NoReturn`, rather than `typing.assert_never` (3.11+) or a
`typing_extensions` dependency.

Typing the EOF-action seam wanted an exhaustive `match` pinned at the match site.
`typing.assert_never` is 3.11+ (verified absent on 3.10); the package floors at `>=3.10` and CI tests
3.10. `typing_extensions` is only a marker-limited transitive dependency (absent on `>=3.13`), so
using it would mean a new direct dependency.

A third option needed no helper at all: `fix_file` is typed `-> bool`, so an unhandled variant makes
the `match` fall through and the function implicitly return `None`, which ty already rejects
(verified on ty 0.0.42). That was the original plan for the seam and was dropped while implementing
it — the error surfaces on the *return type*, away from the `match` that actually lost a case, and
it disappears entirely for any future consumer of `_EofAction` that legitimately returns `None`. A
`case _: _assert_never(action)` arm pins the error at the match site regardless of the consumer.

`typing.assert_never`'s body is ~2 lines; vendoring it as `_assert_never` gives identical runtime
behavior and the same ty exhaustiveness enforcement on 3.10-3.14 with zero dependencies. Dropping
3.10 now, with its security-only EOL roughly four months out, would be a breaking, outward-facing
change to save a few lines — not worth it while 3.10 is still supported. The vendored helper is the
reversible choice; dropping the floor is not.

**Revisit trigger:** Python 3.10 reaches end-of-life (2026-10-31). At that point consider bumping
`requires-python` to `>=3.11`, dropping the 3.10 classifier and CI entry, and replacing
`_assert_never` with `from typing import assert_never`. That bump is a breaking change warranting a
minor/major release.

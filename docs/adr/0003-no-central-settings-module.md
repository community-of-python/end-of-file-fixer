# Constants stay local; no central settings module

**Decision:** Promote magic literals to named module-level constants in the module whose behavior
they parameterize. Do **not** introduce a central `settings.py` / `constants.py`.

While deepening the fix behavior into `eof_fixer/fixer.py`, the question arose of where defaults like
`.cache` / `.uv-cache`, the `1024`-byte binary sample, and the terminator bytes should live —
co-located, or gathered into one settings/constants module. `discovery.py` already kept
`GITIGNORE_NAME` and `ALWAYS_PRUNE` as local module-level constants, and the remaining literals
partition cleanly by responsibility: `.git` / `.gitignore` belong to file discovery, the
`.cache` / `.uv-cache` default skips to the fix capability, the binary-sample size and terminator
bytes to EOF normalization. None of them are user-tunable: there is no env var, config file, or
`[tool.eof-fixer]` section — they are fixed constants, not settings.

A central module would group these by *type* (they are all constants) rather than by
*responsibility*, which hurts locality: reading `_is_binary` would mean bouncing to `constants.py`
for `1024` and back. Keeping each constant beside the logic it governs names the value without
scattering the behavior, and mirrors the pattern `discovery.py` already uses. This is a deliberate
locality choice, not an oversight, so a future review should not re-suggest a settings module on the
grounds that "the constants are scattered."

**Revisit trigger:** configuration becomes **runtime-loaded** — a config file, environment variables,
or a `[tool.eof-fixer]` pyproject section. At that point a dedicated config/settings module that
parses and validates that input is the right home, and this decision no longer applies.

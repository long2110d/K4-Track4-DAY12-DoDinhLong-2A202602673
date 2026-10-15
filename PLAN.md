# PLAN — Lab 12 (replay & closed-loop testing)

## Project overview

- **Stack:** Python 3 (numpy-based), tests use `unittest` in `tests/test_lab12.py` (pytest can run them). Virtualenv already exists at `.venv/` (Windows: `.venv/Scripts/python.exe`). Always set `PYTHONUTF8=1` (Vietnamese text in output).
- **Layout:**
  - `lab12/` — library. Student stubs raise `NotImplementedError("TODO ...")`: `replay.py` (`compare_with_tol`), `scenarios.py` (`ttc`, `required_decel`, `unavoidable`, `mine_braking`), `models.py` (`l2_summary`), `stats.py` (`wilson`), `gate.py` (`rule_closed_loop_logs`, `rule_scenario_grid`), `augment.py` (`dagger_windows`). Each stub has a docstring + Vietnamese hints ("Gợi ý") and refers to a step in `GUIDE.md`.
  - `tests/test_lab12.py` — spec for all functions (classes `Metrics`, `Mining`, `OpenLoop`, `Gate`, `Replay`, `Dagger`, ...).
  - `run_all.py` — runs the whole pipeline, writes `evidence/`.
  - `scripts/check_submission.py` — submission checker (`--save-verify`).
  - `REPORT.md` — report template to fill in; `RUBRIC.md`, `GUIDE.md` — instructions.
- **Conventions:** Only replace the `raise NotImplementedError` bodies; keep signatures, docstrings, and existing style (numpy vectorized, must work on scalars and arrays where tests require). Do not edit tests, do not modify other library code. Read only the stub, its GUIDE.md step, and the matching tests. **Never run `git push`** (no git commands needed at all).

TEST_CMD: PYTHONUTF8=1 .venv/Scripts/python.exe -m pytest -q tests

## Task 1: Test configuration sanity + replay.compare_with_tol

- Ensure TEST_CMD runs (if `pytest` is missing in `.venv`, run `.venv/Scripts/python.exe -m pip install pytest`; do not add other files). Failures from other unimplemented stubs are expected at this stage.
- Implement `compare_with_tol` in `lab12/replay.py` per its docstring and GUIDE.md Bước 1.
- **Files:** `lab12/replay.py`.
- **Acceptance:** `pytest -q tests -k "Replay"` passes (incl. `test_same_code_twice_is_identical_refactor_needs_tolerance`).

## Task 2: Scenario metrics (ttc, required_decel, unavoidable, mine_braking)

- Implement the four functions in `lab12/scenarios.py` per docstrings and GUIDE.md Bước 2. `ttc`/`required_decel` must work on numpy arrays (inf/no-closing cases as documented).
- **Files:** `lab12/scenarios.py`.
- **Acceptance:** `pytest -q tests -k "ttc or unavoidable or slide_examples or Mining"` passes.

## Task 3: l2_summary and wilson

- Implement `l2_summary` in `lab12/models.py` (Bước 3) and `wilson` in `lab12/stats.py` (Bước 4), including edge cases (n=0, k=0, k=n) as tested.
- **Files:** `lab12/models.py`, `lab12/stats.py`.
- **Acceptance:** `pytest -q tests -k "OpenLoop or wilson"` passes.

## Task 4: Release gate rules

- Implement `rule_closed_loop_logs` and `rule_scenario_grid` in `lab12/gate.py` per docstrings and GUIDE.md Bước 6 (follow the shape of existing rules in the same file).
- **Files:** `lab12/gate.py`.
- **Acceptance:** `pytest -q tests -k Gate` passes.

## Task 5: DAgger windows

- Implement `dagger_windows` in `lab12/augment.py` per docstring and GUIDE.md Bước 7 (same window layout as training data, expert labels).
- **Files:** `lab12/augment.py`.
- **Acceptance:** full TEST_CMD passes with 0 failures and 0 errors.

## Task 6: Run the full pipeline

- Run: `PYTHONUTF8=1 .venv/Scripts/python.exe run_all.py --student-id 2A202602673` (may take several minutes; use a long timeout). Fix any library bug it exposes in the six implemented functions only.
- **Files:** generated `evidence/*` (and whatever `run_all.py` writes); no hand edits.
- **Acceptance:** command exits 0; `evidence/` contains fresh outputs for student id 2A202602673; TEST_CMD still passes.

## Task 7: Fill REPORT.md with real numbers

- Fill `REPORT.md` header: name **Đỗ Đình Long**, MSSV **2A202602673**, GitHub **long2110d**.
- Answer all 8 questions using only numbers read from files in `evidence/` (do not invent or round inconsistently). Each answer 4–8 lines and cites **at least 3 concrete numbers** from evidence. Keep the template's headings/structure; remove placeholder text. Check `RUBRIC.md` and `scripts/check_submission.py` for exact format requirements.
- **Files:** `REPORT.md`.
- **Acceptance:** every question has 4–8 lines with ≥3 numbers that appear in `evidence/`; no leftover placeholders; TEST_CMD passes.

## Task 8: Submission check

- Run `PYTHONUTF8=1 .venv/Scripts/python.exe scripts/check_submission.py --save-verify`. Fix any reported issue (in REPORT.md or rerun `run_all.py`) until 100% PASS.
- **Files:** `REPORT.md` / `evidence/*` only if the checker requires; whatever file `--save-verify` writes.
- **Acceptance:** checker reports all items PASS (100%); TEST_CMD passes. Do not push.

## RPICodes — Copilot instructions

Purpose
- Primary artifact is a single hardware control script: `joystick_move` (Python). It reads joystick events from `/dev/input/js0` and drives two DC motors via an Adafruit Crickit board.

Big picture
- Single script repo: joystick -> motor control. There are no services or complex dependencies beyond the hardware and the Adafruit CircuitPython libraries.
- Data flow: joystick event bytes -> unpacked with `Struct('I h B B')` -> `Event` namedtuple -> `handle_event` -> call motor throttle routines.

Key files
- `joystick_move` — the main executable Python script (single source of truth for behavior).
- `README.md` — short project description, add run/setup notes here as you update the repository.

Hardware & runtime expectations
- This runs on a Raspberry Pi connected to an Adafruit Crickit and a Linux joystick device (`/dev/input/js0`).
- Requires Python 3 and the following Python libraries: `adafruit-circuitpython-crickit` (usually requires `blinka` too). Install with pip3 if not preinstalled. Example:
  ```bash
  sudo apt update
  sudo apt install -y python3 python3-pip joystick
  pip3 install adafruit-blinka adafruit-circuitpython-crickit
  ```
- The script reads `/dev/input/js0`; it typically requires `sudo` (or adding the running user to the input group) to access device nodes.

Running the program
- Confirm joystick device present: `ls /dev/input/js*` and optionally inspect inputs with `jstest-gtk` or `sudo jstest /dev/input/js0`.
- Run with root for device access (default):
  ```bash
  sudo python3 joystick_move
  ```
- The script now accepts CLI flags for development and debugging:
  - `--device /dev/input/jsN` — specify a joystick device path (default: `/dev/input/js0`).
  - `--mock` — run without hardware using an internal mock Crickit implementation. Useful for dev and CI.
  - `--verbose` — print extra debug output.
  Examples:
  ```bash
  # Run using mock motors (no hardware needed)
  python3 joystick_move --mock

  # Use a different device path
  sudo python3 joystick_move --device /dev/input/js1
  ```
- Environment variables used by script:
  - `ROBO_DC_ADJUST_R` and `ROBO_DC_ADJUST_L` — decimal multipliers used to adjust throttle per motor, defaults to `1`.
  Example to compensate a weaker motor:
  ```bash
  ROBO_DC_ADJUST_R=0.9 ROBO_DC_ADJUST_L=1.0 sudo python3 joystick_move
  ```

- Event parsing uses `Struct('I h B B')` -> `Event(time, value, type, number)`; `value` is int16 for axis magnitude, `type == 2` signals an axis event.
- Axis mapping is defined by `AXIS = {0: 'left_x', 1: 'left_y', 3: 'right_x', 4: 'right_y'}` and the script responds only to `right_y` currently.
Project-specific conventions & noteworthy patterns
- Event parsing uses `Struct('I h B B')` -> `Event(time, value, type, number)`; `value` is int16 for axis magnitude, `type == 2` signals an axis event.
- Axis mapping is defined by `AXIS = {0: 'left_x', 1: 'left_y', 3: 'right_x', 4: 'right_y'}` and the script responds only to `right_y` currently.
- Xbox controller note: Axis/button mappings vary by kernel driver and controller model. For XBox 360/Xbox One controllers on Linux, typical mappings are similar but not universal — confirm with `jstest`/`evtest` and update the `AXIS` mapping or pass a custom mapping via a code edit.
  - To discover axis mapping on your machine, run: `sudo jstest --normal /dev/input/js0` (or use the GUI `jstest-gtk`). The output lists axes and which stick they map to.
  - If your controller maps `right_y` to a different axis number, edit the `AXIS` dict (or propose a small PR to add a `--axis-mapping` option). Do not assume a single consistent mapping.
- Event parsing uses `Struct('I h B B')` -> `Event(time, value, type, number)`; `value` is int16 for axis magnitude, `type == 2` signals an axis event.
- Axis mapping is defined by `AXIS = {0: 'left_x', 1: 'left_y', 3: 'right_x', 4: 'right_y'}` and the script responds only to `right_y` currently.
- Throttle levels are configured by `THROTTLE_SPEED = {0: 0, 1: 0.5, 2: 0.7, 3: 0.9}`; change these values to tune response.
- `MOTOR` is built from `crickit.dc_motor_1` and `crickit.dc_motor_2` and uses the throttle property directly: `MOTOR['L'].throttle = ...`.

Lint, test & repetitive tasks
- There are no tests or CI setup in this repo. Basic checks for contributors:
  - Run `python -m pyflakes joystick_move` or `pylint` (if lint config exists) to catch lint or syntax issues.
  - Confirm the shebang on `joystick_move` is `#!/usr/bin/env python3` (current script has `#!/usr/bin/env python 3`, which is invalid). Fix before distributing to make the script executable via `./joystick_move`.

Debugging & diagnosing runtime problems
- Common issues:
  - `ModuleNotFoundError: adafruit_crickit` → install `adafruit-circuitpython-crickit` / `adafruit-blinka`.
  - Permission errors reading `/dev/input/js0` → run with `sudo` or add your user to the `input` group: `sudo usermod -a -G input $USER` and re-login.
  - No joystick device: check `ls /dev/input` and/or ensure joystick is supported and connected.
- Add `print` statements or replace calls to `MOTOR[...]` with mocked objects for local testing.

Suggested small refactors for contributions (helpful for AI agents)
- `--mock` mode is already implemented; consider adding a `--axis-mapping` CLI argument to override `AXIS` from the command line.
- Add a JSON-config or `.env` support with `AXIS`, `THROTTLE_SPEED`, and adjustment multipliers so different controllers can be used without code edits.
- Add CI or unit tests that run in `--mock` mode; simulate joystick inputs and validate that motors' throttle values change accordingly.

What to avoid
- Avoid adding features that require extra hardware without providing a fallback simulation or unit tests (e.g., new motor controllers). Work in small increments and add a `mock` mode for tests.

If you're uncertain
- If any change could interfere with motor safety (e.g., throttle mapping or direction), add a small hardware-safe test case (simulate joystick inputs for short durations and low throttle) and/or require a test harness.

Where to find more info
- `README.md` — repository description and high-level notes; update it for setup and run instructions as you modify the repo.

Questions for maintainers
- Should we add a `requirements.txt` or `pyproject.toml` to pin Python dependencies? (Recommended.)
- Should we add a `mock` mode/test harness to reduce risk and enable standard CI checks?

When editing code
- Keep changes minimal and hardware-safe if you’re not familiar with the physical robot. Prefer to add unit tests and a mock mode instead of running new code on the real hardware.

— End of Copilot instructions —

# RPICodes

Small repo that reads an Xbox (or other) joystick device and drives two DC motors via an Adafruit Crickit breakout.

Requirements
- Raspberry Pi or Linux host with `/dev/input/js*` joystick devices
- Python 3
- The Adafruit libraries if you want to run on hardware:
	- `adafruit-blinka`
	- `adafruit-circuitpython-crickit`

See `requirements.txt` for a simple dependency list.

Running the script
- Quick local test (no hardware, mock motors):
	```bash
	python3 joystick_move --mock
	```
- Run on hardware (may require `sudo`):
	```bash
	sudo python3 joystick_move --device /dev/input/js0
	```

Xbox controller notes
- Xbox controller axis mapping can vary by kernel driver and device. Confirm mapping with `jstest` or `evtest` and update the `AXIS` dictionary in `joystick_move` if needed.

Contributions
- Keep changes small and hardware-safe. If you add features requiring hardware, provide a `--mock` mode or a CI-friendly simulation.


# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

MeshAlyzer (Lake Lab, WashU BME) is a bench device that simulates cough pressures on a porcine abdominal wall to test hernia mesh. A Raspberry Pi 5 runs the CustomTkinter GUI (`main.py`) and drives valves/motor over GPIO; a Raspberry Pi 3 reads the pressure transducers through an ADC and streams readings to the Pi 5 over TCP.

## Running

- Target hardware: Raspberry Pi 5 (GUI + control) and Raspberry Pi 3 (sensor reader). Most modules import `lgpio`, `spidev`, `board`/`busio`, or `RPi.GPIO` at import time and **will fail on Windows/macOS**. Don't try to run `main.py` or hardware scripts off-device; verify changes with `python -m py_compile <file>` instead.
- Pi 5: `pip install -r requirements.txt` (plus `sudo apt-get install python3-lgpio`), then `python main.py`.
- Pi 3: `PressureSensorReader_rasp3.py` runs as a systemd service (see README).
- No test suite or linter is configured.

## Architecture

- `main.py` (~2k lines) — `App(ctk.CTk)`: GUI, protocol runner, live graphing, data saving, trial viewer. `ProtocolViewer` lists files in `protocols/`.
- `PressureSensorReader.py` — `PressureReceiver` TCP server on `0.0.0.0:65432`. Expects newline-delimited JSON `{"sensors": {"channel_0": ..., "channel_3": ...}}`. Latest values are stored in a class attribute; read with `PressureReceiver.getpressures()`.
- `PressureSensorReader_rasp3.py` — Pi 3 client that reads the ADC and sends that JSON.
- `ADS1263.py`, `config.py` — Waveshare ADC driver and pin config (vendor code; avoid editing).
- `ValveController.py` — `lgpio` valve driver. **Active-LOW**: writing 1 is neutral/off, 0 activates. Valve 1 = supply 20 / vent 27; valve 2 = supply 12 / vent 24 (set in `App.__init__`).
- `clamp_motor.py` — `MotorController` for the tissue clamp; `Ardunio/` and `arduino_adc/` hold Arduino sketches.
- `calibrate_page.py`, `settings_page.py`, `valve_control_dropdown.py` — GUI pages/widgets.
- `calibrating_pressure_transducers/` — transducer calibration scripts and `.joblib` models.
- `inflation_time_model.pkl` / `deflation_time_model.pkl` — sklearn models used by `SmartInflateML` / `SmartDeflateML` to predict valve-open duration (built by `deflate_model.py`).
- `debug/` — one-off hardware test scripts.
- Settings: `default_settings.txt` / `settings.txt` (`key = value` lines, read by `read_settings()`).

## Protocol files (`protocols/*.txt`)

One command per line, parsed in `App.process_protocol`:

```
Inflate: time|pressure, <value>, valve1|valve2|both[, <metric>, <var_name> ...]
Deflate: time|pressure, <value>, valve1|valve2|both[, <metric>, <var_name> ...]
SmartInflateML: <target_psi>, valve1|valve2|both
SmartDeflateML: <target_psi>, valve1|valve2|both
Wait: <seconds>
Wait_for_user_input: ...
no_save
End:
```

When adding a command, add a branch in `process_protocol` and document it here.

## Data output

Trials are saved to `./data/{timestamp}_{provided_name}_{animal_id}_{trial:02d}/` (name segment optional). The CSV's last column is the protocol step number.

## Conventions and cautions

- Safety: code here moves real valves and pressurizes tissue. Always leave valves in `neutral()` on errors/stop, and don't change pin numbers, timing, or pressure limits without saying so explicitly.
- Keep GUI updates on the Tk main thread; sensor reading and protocol execution run in background `threading.Thread`s.
- Don't regenerate or overwrite the `.pkl`/`.joblib` models unless asked.
- Line endings: the repo is checked in with LF; avoid committing whole-file CRLF conversions.

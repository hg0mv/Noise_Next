# Noise Next

**HF noise floor logger for the Yaesu FT-891** – Linux and Windows. Optional Hamlib support for other radios (Linux).

Noise Next steps through a list of quiet frequencies (one per band), reads the radio's S-meter several times on each and records the noise floor. It runs for hours or days, writes everything to daily CSV files and shows when and on which band the noise goes up. This helps you find RFI sources: compare the graph with when the neighbour's solar inverter, a plasma TV or a switching power supply is on.

**The program only receives. It never transmits.**

![Timeline view](docs/screenshot-timeline.png)

## Features

- Scans up to 16 channels in a cycle (default: one quiet frequency on every band 160–6 m)
- Noise floor = a low percentile of several readings, so a station that happens to transmit does not spoil the value
- **Timeline** and **heatmap** views for 1 hour, 6 hours, 24 hours or 7 days
- **24-hour baseline** (median) per channel, and an **alarm** when the noise rises above it
- **Markers**: press M when you switch something off or on, and the moment appears on the graph
- Daily CSV files; the last 14 days are loaded back at start-up
- Fixed receiver settings during measuring (preamp, attenuator, AGC, noise blanker), restored afterwards
- Instrument-style display: dBm readout, analogue S-meter with the baseline mark
- English and Hungarian user interface

![Heatmap view](docs/screenshot-heatmap.png)

## Download and run

### Windows

1. Unzip `noisenext-0.1-beta-windows-x64.zip`.
2. Start `noisenext.exe` once and close it. This creates `%APPDATA%\noisenext\noisenext.conf`.
3. In the config set `port=` to the FT-891's **Enhanced** COM port (Device Manager → Ports) and `baud=` to the CAT RATE.
4. Start the program again and press **Start logging**.

### Linux

```
tar xzf noisenext-0.1-beta-linux-x86_64.tar.gz
cd noisenext-0.1-beta-linux-x86_64
./noisenext --demo      # try it without a radio
./noisenext
```

The config is `~/.config/noisenext/noisenext.conf`. Your user must be in the `dialout` group.

### Log files

| System | Folder |
|---|---|
| Linux | `~/noisenext-logs/` |
| Windows | `Documents\noisenext-logs\` |

- `noise_YYYYMMDD.csv` – one line per channel and cycle: `utc, epoch, channel, freq_hz, raw, db_rel_s9, s_units`
- `events_YYYYMMDD.csv` – markers and alarms

File names and times are in UTC; the program shows local time.

## Choosing the channels

Pick a frequency on every band where there is usually **no station**: the noise floor is what you want to measure. The defaults are only a starting point:

```
channel=160,1950000
channel=80,3630000
channel=40,7075000
channel=20,14105000
```

A third value of `0` disables a channel: `channel=6,50400000,0`. You can also switch the selected channel on or off with the **E** key while not logging.

**One channel only** = continuous monitoring of one frequency: useful while you walk around the house switching things off. Use a short cycle then, for example `interval_s=2`.

## Using it

- **Start logging** (Space) / **Stop** (Esc)
- Click a channel on the left to show its values on the right
- **Heatmap / Timeline** (V) switches the view, **Range** (R) cycles 1 h → 6 h → 24 h → 7 d
- **Mark** (M) places a numbered marker at the current time
- Move the mouse over the graph or heatmap to read a value

The amber dashed line on the timeline and the amber tick on the meter show the 24-hour baseline of the selected channel. The LED on a channel key is green (normal), yellow (6 dB or more above the baseline) or red (alarm).

## How a measurement works

1. Tune to the channel, set the mode and the receiver settings
2. Wait `settle_ms` (AGC settling)
3. Take `samples` readings, `sample_ms` apart
4. Store the `percentile`-th percentile (default: 20 %) as the noise floor

With 11 channels and the default settings one cycle takes about 17 seconds; a new cycle starts every `interval_s` seconds (default 60).

## Settings

| Key | Default | Meaning |
|---|---|---|
| `port` | `/dev/ttyUSB0` / `COM3` | CAT port |
| `baud` | 38400 | CAT speed |
| `lang` | en | `en` or `hu` |
| `backend` | native | `native` (FT-891 CAT) or `hamlib` |
| `rig_model` | 1036 | Hamlib model number (`rigctl -l`) |
| `mode` | USB | receive mode: USB, LSB, CW, AM |
| `interval_s` | 60 | a new cycle starts every N seconds |
| `settle_ms` | 400 | wait after tuning |
| `samples` | 10 | readings per channel |
| `sample_ms` | 100 | time between readings |
| `percentile` | 20 | noise floor percentile |
| `alarm_db` | 10 | alarm above baseline + N dB |
| `preamp` | off | `off` (IPO), `on`, `keep` |
| `att` | off | `off`, `on`, `keep` |
| `agc` | slow | `fast`, `mid`, `slow`, `keep` |
| `nb` | off | noise blanker and DSP NR: `off`, `on`, `keep` |
| `channel=` | | `name,frequency_Hz[,0]` |
| `scal=` | | S-meter calibration `raw,dB_rel_S9` |

Command line: `-p PORT`, `-b BAUD`, `-i SECONDS`, `--start` (start logging at once), `--demo`.

Keep the receiver settings the same for all your measurements, otherwise the values are not comparable. For long-term logging `preamp=off` and `agc=slow` are a good choice.

## S-meter accuracy

The FT-891 reports its S-meter as a raw value from 0 to 255. The conversion to S-units and dBm (S9 = −73 dBm) uses an **estimated** table. The *changes* in the noise level are what matter for finding RFI, and these are shown well even without calibration. For exact dBm values, measure with a signal generator and enter the points as `scal=raw,dB` lines.

## Hamlib (Linux)

```
sudo apt install libhamlib-dev
make clean && make HAMLIB=1
```

Then `backend=hamlib` and `rig_model=` in the config. Hamlib reports the S-meter already in dB relative to S9. In Hamlib mode the program sets the frequency and mode only; preamp, attenuator, AGC and noise blanker stay as they are on the radio.

## Building from source

```
sudo apt install build-essential libx11-dev       # Linux
make

sudo apt install mingw-w64 zip                    # Windows, cross-compiled on Ubuntu
make windows

make dist                                         # all release packages + SHA256SUMS.txt
```

## Notes

- While logging, the program controls the radio. Do not operate the radio in the meantime.
- At Stop, when the window closes or after an error, the original frequency, mode, preamp, attenuator, noise blanker, NR and AGC of every band that was touched are restored.
- In demo mode the last 24 hours are filled with simulated data and nothing is written to the log folder.

## License

Noise Next is free software, licensed under the **GNU General Public License v3.0 or later** (GPL-3.0-or-later). See [LICENSE](LICENSE). It comes with ABSOLUTELY NO WARRANTY.

# CS-1

A CS-80-style synthesizer firmware for the M-VAVE FM-1: a sound module for a dawless rig, played over
MIDI (TRS or USB) from a sequencer such as the Novation Circuit Tracks.

CS-1 is a Yamaha CS-80 in spirit: eight voices of two channels each (I and II), every channel a VCO
(sawtooth and / or pulse with PWM, noise), a resonant high-pass and low-pass filter in series, a sine added after
the filters, a filter envelope (IL, AL, A, D, R), a VCA envelope (A, D, S, R) and the touch response (velocity
and pressure to brilliance and level); a sub oscillator to pitch, filter and level; the ring modulator; the
chorus / tremolo (stereo here); portamento and glissando, channel II detune, the initial pitch bend, keyboard
control, sustain modes I and II, brilliance and resonance levers, and the SELECT encoder as the ribbon. For a
sequencer with no hands: an auto after touch curve each note plays by itself, MIDI learn, a CC map for the Circuit
Tracks' macros, and clock sync. Effects beyond the instrument: drive and two shelves, a flanger, a stereo delay with a tape character, a phaser,
and a Dattorro plate reverb with a shimmer at any interval. The 22 preset tones come from the owner's manual's
charts (`tools/gen_presets.py` says how they were read and mapped), and the voice was calibrated against a CS-80
emulation, one dry note at a time (`tools/compare_wav.py`). 52 sounds, 32 user presets, a documentation site with
the firmware running in the browser, and an installer in the browser.

## What it stands on

- [Dinghy](https://github.com/zvenson/dinghy) (zvenson): the minimal FM-1 firmware template, which is the
  [Felucca](https://github.com/hugelton/Felucca) / SLOOP platform (Leo Kuroshita, Hügelton Instruments; isod89).
- [FM-106](https://github.com/patttf/fm-106) (PatttF): the second core as an audio worker
  (`firmware/hal/fm1_multicore.h`, `firmware/src/audio_worker.h`) and MIDI IN on the TRS jack (`midi_uart.c`),
  ported onto the Dinghy tree here.

Everything is GPL-3.0-only; see `LICENSING.md`.

MIDI clock from the sequencer drives the delay's SYNC divisions (DELAY page) and the sub oscillator's (SUB OSC 2);
VINTAGE on PERFORM lets each voice drift a few cents on its own, as the voice cards did.

## The sounds

PRESETS (and program change) runs through these, then the 32 user slots; HOME + a key picks a tone for a channel,
ALGO picks a category on HOME (PAD, STRINGS, BRASS, KEYS, BELLS, BASS, LEAD, FX, USER) for PRESETS to browse, and turns a button's pages elsewhere. The classic ones are recipes after what is documented about the records (which presets,
the technique), not transcriptions; they pair a row I tone with a row II tone as the instrument does, and most
carry the chorus, a touch of detune, an auto after touch curve and the plate. The last group is converted from
CS-80 V factory presets on a computer with the plugin installed: `tools/converted.txt` lists them with our names,
`tools/arturia_convert.py` reads each preset's panel and maps it through the same calibration as the manual's tones
(`tools/converted.json` keeps the values, so the build does not need the plugin), and `tools/level_converted.py`
evens out their loudness by measurement.

| | |
| --- | --- |
| Vangelis | BLADE BRASS, BR BLUES, BR FLUTE, BR STRINGS, CHARIOTS, UNKNOWN MAN, BR BRASS 2 (Syntorial's recipe), BRASS SWELL (MusicTech's), BR BLUES 2, END TITLES, CHIMES |
| Toto, Thriller | AFRICA, AFRICA 2 (Syntorial's), ROSANNA, HUMAN NATURE (the chromatic glide), BILLIE JEAN, BILLIE KEYS (Syntorial's) |
| The eighties | BORN USA, XMAS BASS (the sub oscillator's inverted saw into the filter), WHO BASS, BABOOSHKA, DUCHESS, TURN IT ON, TIME STRINGS, VIENNA, OXYGENE, ENO WASH, WALKING, 4001 PLUCK, 4001 STRINGS, LONG BRASS, SHRIEK LEAD, SQUELCH BASS |
| The manual's own examples | TRUMPET, CLARINET, OBOE, TIMPANI |
| The plain pairs | STRINGS 1+2, BRASS 1+2, FLUTE+BASS, ORGANS, CLAVS, HARPSI, E.PIANO, FUNKY 1+2 |
| With the effects | TAPE PAD, SHIMMER CHOIR, PHASE STRINGS, ECHO KEYS, TAPE BRASS, DEEP DRIFT |
| After CS-80 V presets, our names | TYRELL TOWER, RAIN ON NEON, BRASS SECTION, RISING BRASS, SOFT BRASS, STRING MACHINE, STRINGS 81, CHAMBER, WARM SKY, FIELD FLUTE, ICE BELLS, BELL TOWER, GLASS ORGAN, MUSIC BOX, CLAV FUNK, SWIM BASS, FAR SHORE, RAINDROPS |

## Layout

| | |
| --- | --- |
| `firmware/src/cs1.c` | the app: the panel's pages, the screen, keys, MIDI, settings |
| `firmware/src/cs1_voice.h`, `cs1_dsp.h` | the voice (patch, envelopes, filters, the block render on both cores) and the fixed-point primitives |
| `firmware/gen/cs1_tables.h` | the voice's tables, from `tools/gen_tables.py` (committed; no FPU on the chip) |
| `firmware/gen/cs1_presets.h` | the CS-80's 22 preset tones, from `tools/gen_presets.py` (the manual's charts, with the mapping's assumptions) |
| `tools/wav_check.py` | pitch, level and harmonics of a host render: a change to the voice checked without ears |
| `firmware/src/platform.c` | the platform wiring (boot, USB, updater, audio ISR, settings, CPU1, TRS MIDI) |
| `firmware/hal/` | every hardware register access; nothing outside it touches a register (the build checks) |
| `tests/host.c`, `tests/run_host.sh` | the app on a computer: a scripted run that writes a WAV and screen PNGs |
| `web/emu/` | the browser emulator (Emscripten + AudioWorklet): play it, with Web MIDI in |
| `web/site/`, `web/site.css`, `tools/gen_docs.py` | the documentation site: overview and install, manual, sound design guide, a reference generated from the firmware's page table, with the emulator under `emu/` |
| `tools/build.py` | the device build (JieLi toolchain, in Docker on macOS) and the `.fwsc` package |
| `docs/` | `API.md` (what the platform offers), `RULES.md` (the chip's rules: no float, no 64-bit division) |

## Build and run on a computer

```sh
pip3 install Pillow
tests/run_host.sh          # -> build/host/cs1.wav, build/host/cs1-*.png
```

On macOS without the Xcode licence accepted, point the compiler at the Command Line Tools:
`DEVELOPER_DIR=/Library/Developer/CommandLineTools tests/run_host.sh`.

## Play it in the browser, and the site

The firmware runs in an AudioWorklet against a fake FM-1, with the panel on the page and Web MIDI in and out
(Chrome or Edge). The documentation site wraps it: the pages in `web/site/`, a reference that `tools/gen_docs.py`
generates from the firmware's own page and sound tables, and the emulator under `emu/`. Needs Emscripten
(`tools/get_emsdk.sh` installs it into `~/.emsdk`, no Xcode or Homebrew) and Node for the test:

```sh
web/site/build.sh                # -> build/site/ (index, manual, sound, reference, img/, emu/)
node web/emu/emu_test.mjs        # the wasm in Node: boots, sounds, LEDs, MIDI, settings, determinism, cost
python3 tools/serve_web.py       # http://localhost:8000/  (the emulator at /emu/; Web MIDI needs localhost or https)
```

Click **Switch on**, then play the keys (mouse, touch, or the <kbd>Q</kbd> and <kbd>A</kbd> rows), turn the
knobs (drag or wheel), and pick your MIDI interface under **MIDI in** to play it from a sequencer. CC 80 to 87
and MIDI learn (REC) map a controller's knobs, as on the FM-1. `emu/?sound=N` opens a sound, which the site's
"Try" links use. The settings sector and the user bank are kept in the browser's localStorage; **Export presets**
writes the bank to a `.cs1bank` file. What the emulator cannot tell you: CPU cost on the real chip, the
dual-core split, USB audio and the updater.

`tests/run_tests.sh` runs the host build, the site build and the emulator test together. `build/site/` is the
whole site, ready for any static host (GitHub Pages included).

## Build for the FM-1

Needs the JieLi toolchain (`tools/get_toolchain.sh`), the AC79 SDK checkout, and Docker on macOS:

```sh
./build.sh                 # -> build/cs1.fwsc  (DINGHY_DUAL_CORE=0 ./build.sh for single-core)
python3 tools/fm1_install.py build/cs1.fwsc
```

The package identity is `FM-1_95N` (N = minor version). Installing any custom firmware is at your own risk:
hold OCT- at power-on for the USB rescue; M-VAVE's updater or the Firmware Hub's "Back to official V15" takes
you back to stock, and any firmware with Felucca's or M-VAVE's update loader installs over CS-1 (from 0.3.2;
before, the official package was refused on a unit with an older flash head). tests/ota_test.c runs the update
entry on a computer against a package. `tools/arminator_read.py` and `tools/arturia_convert.py --all` read whole
preset libraries (Arminator 2's banks via tests/vstdump.c under Wine, the CS-80 V's) into a corpus; `tools/gen_random.py`
reduces the 910 sounds to the RANDOM page's model (a mean a category and twelve directions, firmware/gen/cs1_random.h);
nothing of the libraries themselves is shipped.

You can also try CS-1 in the browser, no flashing needed, in the free community [FM-1 Emulator](https://fm1-emulator.atst.tools/).

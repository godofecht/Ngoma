# Ngoma

A drone synthesizer instrument built in Max MSP as an academic project.

## What this is

Ngoma is a drone synth. A drone is one long, held note that slowly changes over time, like the hum of a didgeridoo or the low note of bagpipes. A synthesizer (synth) is an instrument that makes sound electronically from scratch.

You play and hold a note on a MIDI keyboard (a keyboard that sends note messages to a computer rather than making sound itself). Ngoma turns it into an evolving drone. You then shape the sound by moving a dot around an XY pad, a square where left-right and up-down each control settings. A small machine learning model links each point on the pad to ten sliders at once, so one mouse movement changes many parts of the sound together.

It is built in Max, a visual programming language by Cycling '74 where you connect boxes with lines (patch cords) instead of typing code. It runs as a Max for Live device: a Max program that loads inside the Ableton Live DAW (digital audio workstation, the software you record and produce music in).

## What you can learn from it

Music Technology:

- How harmonics (the quieter, higher notes inside every sound that give it its tone) build a rich sound, and why a sine wave on its own sounds plain.
- Additive synthesis (building a sound by stacking sine waves) and FM synthesis (frequency modulation: using one wave to wobble the pitch of another very fast, which creates new harmonics).
- Filters: effects that cut or boost some frequencies. A high-pass filter removes low frequencies. A band-pass filter keeps a narrow band and removes the rest.
- Formants: the resonant peaks that make "ee" and "oo" sound different. Ngoma uses them for a vowel-like "wah".
- How reverb, delay and a vocoder (an effect that makes one sound "speak" with the shape of another) are built from simple parts.
- Physical modelling: imitating a real instrument, here a didgeridoo, by simulating how it works.

Computer Science:

- Linear regression used as a mapping tool: you teach it a few examples and it fills in the space between them.

## Run it

### On your own computer

**You will need:**

- [Ableton Live](https://www.ableton.com/en/live/) with [Max for Live](https://www.ableton.com/en/live/max-for-live/). Max for Live comes with Live Suite and is a paid add-on for Live Standard. Ableton offers a [30-day free trial of Live 12 Suite](https://www.ableton.com/en/trial/), which includes Max for Live.
- The patch files were saved in Max 8.0.2. They should open in later versions of Max.
- A MIDI keyboard or controller.
- Windows or macOS. The device was built on Windows. It has not been tested on macOS.

You can also open the patches in [Max](https://cycling74.com/products/max) on its own to read how they work. Cycling '74 offers a 30-day free trial of Max with everything unlocked. The device was built to run inside Live, so use Live to hear it.

Steps:

1. Download this repo with **Code > Download ZIP** on GitHub and unzip it. Keep all the files in one folder.
2. Install the external objects from the `externals` folder. An external is an add-on object for Max written in C. The patch will not work without `ml.mulreg`. On Windows use `ml.mulreg.mxe64`. On macOS unzip `ml.mulreg.mxo.zip`. Put it in a folder that Max searches, such as the folder that holds `Drone.amxd`, or add the folder in Max under **Options > File Preferences**.
3. In Live, drag `Drone.amxd` onto a MIDI track.
4. Click **read** and select the `formants.json` file.
5. Press the toggle to start the patch.
6. When it opens, the patch loads its trained model, `regressionmodel1.data`, through Max's search path. Keep it in the same folder as `Drone.amxd`. If the Max window says it cannot find the file, add that folder under **Options > File Preferences** and click **read regressionmodel1.data**.
7. Play and hold a MIDI note. You should hear a drone. Move the dot on the XY pad with the mouse to change the sound.

## How it works

The sound passes through three stages.

1. Synthesis. The note you play sets a base frequency. Ngoma adds sine waves at whole-number multiples of it (the harmonics) up to the 64th, in separate groups for odd and even harmonics. That sum is then frequency-modulated by another sine wave.
2. Vowels and breath. White noise (a hiss with all frequencies at once) is filtered into vowel shapes using formant data. A vocoder then shapes this breath with the synth sound.
3. Effects. A tube model adds didgeridoo-like resonance. Filters, a delay and a reverb finish the sound.

The XY pad sits on top. A regression model maps each pad position to ten sliders. A randomizer can also move the pad on its own for wind-like, chaotic sounds.

| File or folder | What is in it |
|---|---|
| `Drone.amxd` | The main Max for Live device, and the file you load in Live. |
| `MonoSynth.maxpat` | One synth voice: the odd and even harmonic generators, FM and the envelope. `Drone.amxd` loads it with `poly~`. |
| `vocoderpfft.maxpat` | The FFT part of the vocoder, loaded with `pfft~`. |
| `RegressionMapper.maxpat` | The XY pad, the `ml.mulreg` regression model and the randomizer. |
| `formants.json` | Formant frequencies (F1 and F2) for each vowel preset. |
| `regressionmodel1.data` | A trained regression model, loaded by the device. |
| `externals/` | The `ml.mulreg` and `ml.dtw` machine learning objects for Windows and macOS. |

## Ideas to try

1. Add a new vowel. Look up the F1 and F2 formant frequencies for a vowel and add a slot to `formants.json`.
2. Train your own mapping. Follow the training steps under "Technical details" and save a new model with the **write** message in `RegressionMapper.maxpat`. It opens a dialog so you can choose where to save the file.
3. Change how fast the randomizer moves. In `RegressionMapper.maxpat`, find the `metro 100` object (it fires every 100 milliseconds) and try slower or faster values.
4. Change the envelope. `MonoSynth.maxpat` uses `adsr~` with attack, decay, sustain and release. Try a long attack so the drone fades in slowly.
5. Ambitious: change how many harmonics are used. `MonoSynth.maxpat` has message boxes listing the odd harmonics (1 to 63) and even harmonics (2 to 64). Remove the higher ones and listen to the tone get darker.

## Technical details

The design notes below are from the original README.

### Introduction

This document details the development of an evolving, monophonic Drone Instrument in Max4Live. The primary goal was to create a synthesizer capable of generating a wide range of monophonic, evolving soundscapes, with controls that allow for extensive sound design possibilities.

### Physical modelling

#### Oscillators

Drone sounds are characterized by rich harmonic interactions. Sine waves lack harmonics, so sawtooth and square waves were used to generate the complex harmonic content required for this project.

#### Lip simulation

Acoustic formant effects, produced by human lips in instruments like the didgeridoo, matter for authentic drone sounds. These effects were simulated using a resonant bandpass filter, designed to mimic the resonant frequency of human lips.

#### Wah-wah simulation

To replicate the multiple resonances found in human vocalization, such as in vowel sounds, several bandpass filters were used in parallel. These filters were tuned according to vowel formant data, allowing for a dynamic wah-wah effect.

#### Blowing pressure simulation

To simulate the control of airflow, synthesized air noise was multiplied with the existing input signal. This technique mimics the effect of varying blowing pressure, adding a layer of realism to the sound.

#### Delay and output filters

Delays and output filters were applied to further refine the realism and texture of the drone sound.

#### Reverb in the physical model

Reverb was added to enhance the realism, with careful control to avoid masking the drone's harmonic details.

### Synthesis

#### Harmonic generation

Harmonics were generated separately for odd and even multiples up to the 64th harmonic using additive synthesis. Even harmonics were created using sine waves at even multiples of the input frequency, while odd harmonics included the fundamental frequency and its odd multiples.

#### FM synthesis

The output from the additive synthesis served as the carrier in FM synthesis, with a sine wave functioning as the modulator. The input frequency was added to the signal, driving the additive synthesis to create complex modulated sounds.

#### Noise synthesis and vocoder

White noise was synthesized using the `noise~` object, then passed through a formant filter to create vowel-like effects. This filtered noise was then used as a carrier in a vocoder, with the output from the FM synthesis acting as the modulator. The vocoder split the signal into several bands, each modulated to match the harmonic content of the vowel sounds.

### Effects

#### High-pass filter

Due to the absence of a dedicated high-pass filter in Max4Live, the `lowres~` object was adapted. By applying a low-pass filter to the signal and subtracting the processed signal from the original, a high-pass filter effect was achieved.

#### Resonant bandpass filter

A resonant bandpass filter was implemented using the `biquad~` object. The filter's frequency was mapped to an LFO (low-frequency oscillator, a slow wave used to move a setting up and down), enabling dynamic frequency sweeping.

#### Tube simulation

This effect simulates the resonance found in instruments like the contrabassoon and didgeridoo. By modelling a tube with specific length and radius parameters, resonances are emphasized at particular length/radius ratios, adding depth to the sound.

#### Delay

Delay effects were created using the `tapin~` and `tapout~` objects. These objects work together to store and manipulate input signals based on specified delay times, enhancing the drone's spatial characteristics.

#### Reverb

Reverb was implemented using the Schroeder algorithm, which involves passing the input signal through eight parallel comb filters with varying delays. The output is then summed and processed through four all-pass filters, each with decreasing delays and variable gain.

#### Vocoder

The vocoder in this patch splits the modulator signal into multiple bands, each processed through bandpass filters that match the modulator's frequency content. The processed signal is then convolved using FFT, significantly reducing processing power while maintaining a similar effect to traditional vocoding methods.

### Machine learning

Machine learning was used to map positions on an XY pad to 10 sliders controlling various parameters. Linear regression allowed for interpolation between defined many-to-many mappings, enabling real-time updates to the sliders based on the XY pad's position.

#### Training procedure

1. Select **add** in the radio selector.
2. Move the XY selector to the desired position.
3. Adjust slider values to achieve the desired effect parameters.
4. Move the XY selector again to generate new noise data.
5. Switch to the first radio selector option.
6. Click **train** to finalize the model.

### Randomizer

Randomizers, driven by `metro` objects, were used to automate the movement of the XY selector. This feature is useful for generating chaotic soundscapes, such as wind noises.

### References

1. [The Didjeridu (Didgeridoo)](https://newt.phys.unsw.edu.au/jw/dij/dij.html). Newt.phys.unsw.edu.au. (2020).
2. [Physical Modelling of a Didgeridoo](https://homes.esat.kuleuven.be/~bdmdotbe/bdm2013/documents/doc_080327_14.10.pdf). Homes.esat.kuleuven.be. (2020).
3. [MATHEMATICS OF THE DISCRETE FOURIER TRANSFORM (DFT) WITH AUDIO APPLICATIONS, SECOND EDITION](https://ccrma.stanford.edu/~jos/mdft/). Ccrma.stanford.edu. (2020).
4. [Synthesizing the Dynamic Spectra of the Didgeridoo](https://quod.lib.umich.edu/cgi/p/pod/dod-idx/synthesizing-the-dynamic-spectra-of-the-didgeridoo.pdf?c=icmc;idno=bbp2372.2010.120). Quod.lib.umich.edu. (2020).
5. [Physical Modelling of a Didgeridoo](https://www.kvraudio.com/forum/viewtopic.php?t=218612). Kvraudio.com. (2020).
6. [Schroeder Allpass Sections](https://ccrma.stanford.edu/~jos/pasp/Schroeder_Allpass_Sections.html). Ccrma.stanford.edu. (2020).

## Project status

This was an academic project from 2020 and is no longer being developed. Known problems:

- The model used to load from a fixed path on the author's computer. In 2026 the patches were changed to find `regressionmodel1.data` through Max's search path (`absolutepath` and `conformpath`), and **write** now asks where to save. This change has not yet been tested in Max or Live.
- The `ml.mulreg` externals are compiled files from 2020. They may not load in current versions of Max or on Apple Silicon Macs.
- There is no screenshot or audio example yet.

## Licence

You can use, copy, change and share this project, as long as anything you share that is built from it is also released under GPL-3.0 with its source. See [LICENSE](LICENSE) (GNU General Public License v3.0).

The compiled files in `externals/` (`ml.mulreg` and `ml.dtw`) are from ml.lib, which is released under the GNU General Public License, version 2 or later. Its source code is at [irllabs/ml-lib](https://github.com/irllabs/ml-lib).

## Credits

- Built by Abhishek Shivakumar.
- The `ml.mulreg` and `ml.dtw` objects in `externals/` are from [ml.lib](https://github.com/irllabs/ml-lib) by Ali Momeni and Jamie Bullock (copyright Carnegie Mellon University, 2013), built on Nick Gillian's [Gesture Recognition Toolkit](https://github.com/nickgillian/grt). Licence: GPL-2.0-or-later.
- Built in [Max](https://cycling74.com/products/max) by Cycling '74 and [Ableton Live](https://www.ableton.com/en/live/).
- The references above, especially the didgeridoo research from UNSW and Julius O. Smith III's books at CCRMA, Stanford.

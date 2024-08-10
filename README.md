# Ngoma - A drone synth made in Max MSP

## Table of Contents

1. [Introduction](#introduction)
2. [Physical Modelling](#physical-modelling)
   - [Oscillators](#oscillators)
   - [Lip Simulation](#lip-simulation)
   - [Wah-Wah Simulation](#wah-wah-simulation)
   - [Blowing Pressure Simulation](#blowing-pressure-simulation)
   - [Delay and Output Filters](#delay-and-output-filters)
   - [Reverb](#reverb)
3. [Synthesis](#synthesis)
   - [Harmonic Generation](#harmonic-generation)
   - [FM Synthesis](#fm-synthesis)
   - [Noise Synthesis and Vocoder](#noise-synthesis-and-vocoder)
4. [Effects](#effects)
   - [Filters](#filters)
     - [High-Pass Filter](#high-pass-filter)
     - [Resonant Bandpass Filter](#resonant-bandpass-filter)
   - [Tube Simulation](#tube-simulation)
   - [Delay](#delay)
   - [Reverb](#reverb)
   - [Vocoder](#vocoder)
5. [Machine Learning](#machine-learning)
6. [Randomizer](#randomizer)
7. [Patch Operation](#patch-operation)
8. [References](#references)

## Introduction

This document details the development of an evolving, monophonic Drone Instrument in Max4Live. The primary goal was to create a synthesizer capable of generating a wide range of monophonic, evolving soundscapes, with controls that allow for extensive sound design possibilities.

## Physical Modelling

### Oscillators

Drone sounds are characterized by rich harmonic interactions. Given that sine waves lack harmonics, sawtooth and square waves were used to generate the complex harmonic content required for this project.

### Lip Simulation

Acoustic formant effects, produced by human lips in instruments like the Didgeridoo, are crucial for authentic drone sounds. These effects were simulated using a resonant bandpass filter, designed to mimic the resonant frequency of human lips.

### Wah-Wah Simulation

To replicate the multiple resonances found in human vocalization, such as in vowel sounds, several bandpass filters were used in parallel. These filters were tuned according to vowel formant data, allowing for a dynamic wah-wah effect.

### Blowing Pressure Simulation

To simulate the control of airflow, synthesized air noise was multiplied with the existing input signal. This technique mimics the effect of varying blowing pressure, adding a layer of realism to the sound.

### Delay and Output Filters

Delays and output filters were applied to further refine the realism and texture of the drone sound.

### Reverb

Reverb was added to enhance the realism, with careful control to avoid masking the drone's harmonic details.

## Synthesis

### Harmonic Generation

Harmonics were generated separately for odd and even multiples up to the 64th harmonic using additive synthesis. Even harmonics were created using sine waves at even multiples of the input frequency, while odd harmonics included the fundamental frequency and its odd multiples.

### FM Synthesis

The output from the additive synthesis served as the carrier in FM synthesis, with a sine wave functioning as the modulator. The input frequency was added to the signal, driving the additive synthesis to create complex modulated sounds.

### Noise Synthesis and Vocoder

White noise was synthesized using the `noise~` object, then passed through a formant filter to create vowel-like effects. This filtered noise was then used as a carrier in a vocoder, with the output from the FM synthesis acting as the modulator. The vocoder split the signal into several bands, each modulated to match the harmonic content of the vowel sounds.

## Effects

### Filters

#### High-Pass Filter

Due to the absence of a dedicated high-pass filter in Max4Live, the `lowres~` object was adapted. By applying a low-pass filter to the signal and subtracting the processed signal from the original, a high-pass filter effect was achieved.

#### Resonant Bandpass Filter

A resonant bandpass filter was implemented using the `biquad~` object. The filter's frequency was mapped to an LFO, enabling dynamic frequency sweeping.

### Tube Simulation

This effect simulates the resonance found in instruments like the contrabassoon and Didgeridoo. By modeling a tube with specific length and radius parameters, resonances are emphasized at particular length/radius ratios, adding depth to the sound.

### Delay

Delay effects were created using the `tapin~` and `tapout~` objects. These objects work together to store and manipulate input signals based on specified delay times, enhancing the drone's spatial characteristics.

### Reverb

Reverb was implemented using the Schroeder algorithm, which involves passing the input signal through eight parallel comb filters with varying delays. The output is then summed and processed through four all-pass filters, each with decreasing delays and variable gain, creating a rich reverb effect.

### Vocoder

The vocoder in this patch splits the modulator signal into multiple bands, each processed through bandpass filters that match the modulator's frequency content. The processed signal is then convolved using FFT, significantly reducing processing power while maintaining a similar effect to traditional vocoding methods.

## Machine Learning

Machine Learning was used to map positions on an XY pad to 10 sliders controlling various parameters. Linear Regression allowed for easy interpolation between defined many-to-many mappings, enabling real-time updates to the sliders based on the XY pad's position.

### Training Procedure

1. Select 'add' in the radio selector.
2. Move the XY selector to the desired position.
3. Adjust slider values to achieve the desired effect parameters.
4. Move the XY selector again to generate new noise data.
5. Switch to the first radio selector option.
6. Click "train" to finalize the model.

## Randomizer

Randomizers, driven by `metro` objects, were used to automate the movement of the XY selector. This feature is particularly useful for generating chaotic soundscapes, such as wind noises.

## Patch Operation

1. Copy external objects into the Max 8 externals folder, specifically the `ml.mulreg.mxe` object. The patch will not function without these.
2. Click "read" and select the `formants.json` file.
3. Press the toggle to start the patch.
4. Edit the destination name in the read message to point toward the locally saved regression model.

To synthesize a drone, play and hold a MIDI note on your controller. Use the mouse and the XY pad to modify drone parameters based on their mappings.

## References

1. [The Didjeridu (Didgeridoo)](https://newt.phys.unsw.edu.au/jw/dij/dij.html). Newt.phys.unsw.edu.au. (2020).
2. [Physical Modelling of a Didgeridoo](https://homes.esat.kuleuven.be/~bdmdotbe/bdm2013/documents/doc_080327_14.10.pdf). Homes.esat.kuleuven.be. (2020).
3. [MATHEMATICS OF THE DISCRETE FOURIER TRANSFORM (DFT) WITH AUDIO APPLICATIONS, SECOND EDITION](https://ccrma.stanford.edu/~jos/mdft/). Ccrma.stanford.edu. (2020).
4. [Synthesizing the Dynamic Spectra of the Didgeridoo](https://quod.lib.umich.edu/cgi/p/pod/dod-idx/synthesizing-the-dynamic-spectra-of-the-didgeridoo.pdf?c=icmc;idno=bbp2372.2010.120). Quod.lib.umich.edu. (2020).
5. [Physical Modelling of a Didgeridoo](https://www.kvraudio.com/forum/viewtopic.php?t=218612). Kvraudio.com. (2020).
6. [Schroeder Allpass Sections](https://ccrma.stanford.edu/~jos/pasp/Schroeder_Allpass_Sections.html). Ccrma.stanford.edu. (2020).

# Max4Live-Drone-Generator
Drone sound generator

The main patcher is "Drone.amxd"
Import into your Ableton Project or Max MSP setup


Table of Contents: 
1.	Introduction 
2.	Physical Modelling 
3.	Synthesis 
4.	Effects 
5.	Machine Learning 
6.	Randomizer 
7.	Patch Operation 
8.	References 
 
 
Introduction 
 
 
 
This document details the development of an evolving, monophonic Drone Instrument in Max4Live. The original aim of the project was to develop a synthesizer with control setups necessary to generate a wide range of monophonic, evolving soundscapes. 
 
Physical Model: 
 
Considering the didgeridoo, which is probably the most famous of the acoustic drone instruments, the aim was to draw up a physical model, and then to simulate those physics to a degree. The aim is to obtain an objective drone sound. 
 
1.	Oscillators 
Drones portray interesting harmonic qualities, due to the interactions between the harmonics produced by their oscillators. As Sine waves lack harmonics, sawtooth or square waves were considered instead. 
 
2.	Lip simulation 
Acoustic formant effects are produced by the human lips, in many drone instruments. This is evident in Tuvan throat singing, Russian Basso Profundo, Didgeridoo and this would consist of a resonant bandpass filter with the same resonant frequency as the human lips. 
 
3.	Wah-Wah Simulation 
The output from the nonlinearity gets filtered again. You might want to have 2 or 3 bandpass filters in parallel here, since the voice has multiple resonances. Here, you'll want to look at a document about vowel formants, it's all in there. 
 
4.	Multiply by blowing pressure 
To control the flow of air, it needs to be multiplied with something. By synthesizing air noise, and multiplying it with an existing input signal, it is possible to make the input signal sound like it is being controlled with actual breaths. 
 
5.	Delay and Output Filters 
 
6.	Reverb 
Adds a lot of realism to the sound that would be extremely hard to generate from scratch. However, it is necessary to limit the amount of reverb so as not to hide interesting harmonic content. 
 
Synthesis 
 
First, the odd and even harmonics were generated separately, up to the 64th harmonic. In order to do this, additive synthesis was used. For the even harmonics, a Sine wave was generated for each even multiple of the input frequency. For the odd harmonics, a Sine wave was generated for each odd multiple of the input frequency, including the first. 
 
FM Synthesis was then implemented. The output of the additive synth was used as a carrier, and a Sine wave was used as the modulator. This was done by adding the input frequency to the signal and using that to drive each additive synth. 
 
The noise~ object was used to synthesize white noise. This noise was passed through a formant filter to obtain vowel effects. In order to do this, the filtered noise was then passed through a vocoder, using the output of the FMSynth~ object as a carrier. 
 
 
 
 
Effects 
 
Filters: 
 
High-Pass: 
 
Due to lack of a dedicated high-pass filter in Max4Live, the lowres~ object needed to be re-adapted. This was done by applying a low-pass filter to the signal, and then subtracting the processed signal from the original signal. 
 
Resonant Bandpass: 
 
A resonant bandpass filter was implemented using the biquad~ object. After setting Q and gain values, the frequency was mapped to an LFO to allow for frequency sweeping. 
 
 
Tube simulation: 
 
Instruments like the contrabassoon and didgeridoo demonstrate resonance due to the length of their chambers. This algorithm demonstrates that effect by applying a physical model of a tube with a specific length and radius. At specific length/radius ratios, certain resonances are emphasized as a result. 
 
Delay 
 
Here, samples from the input signal are pushed forward by a certain timestep, multiplied by fractional gain, and then summed onto the original signal. The tapin~ object copies an input signal to the delay line. The tapout~ object allows the user to read from the delay line, based on an input delay time. Together, both objects work in tandem to store input on a delay line to be read based on time. 
 
 
 
Reverb 
 
Reverb is a collection of delay-implemented filters in parallel, summed and passed through a series of all pass filters. Here, the input signal is passed through 8 parallel comb filters with differing delays. This implementation is called the Schroder algorithm. The input passed through these filters is then summed and sent through 4 all pass filters with decreasing delays and variable gain. 
 
 
Vocoder 
 
A normal vocoder splits a modulator signal into several bands, each of which is sent through a separate bandpass filter, set to the same frequency as the modulator signal. The level of each bandpass filter is made to match its corresponding frequency in the modulator signal. Essentially, the carrier is filtered so that its harmonic content is similar to that of the modulator signal. 
 
Here, the harmonic content of vowel syllables was saved as a file and subsequently loaded into a formant filter with the noise as signal input. The noise coming out of the filter was then passed into a vocoder as a carrier, with the output of the FM synthesis being used as the modulator. 
 
Instead of splitting the modulator signal into several bands, both FFT inputs were converted from Cartesian to Polar coordinates and then convolved, i.e. The processed, convolved audio was then converted back from polar to cartesian coordinates, and then passed as FFT output. This provided a similar solution at the cost of much less processing power. 
 
The FFT processing was isolated to the pfft~ block, which performs windowing, conversion and allocation functions. 
 
 
Machine Learning 
 
Machine Learning was used to map positions on the X and Y axes to 10 sliders set to parameters. The technique used here is Linear Regression, which allows for easy interpolation between defined many-to-many mappings. 
 
Each slider in the multi-slider references a different parameter, which it updates in real-time. The slider also scales and sends these values into the ‘regression’ sub patch. The regression sub patch can train, load, save and clear machine learning models. 
 
The procedure for training is as follows: 
1.	Select ‘add’ in radio selector. 
2.	Move position of XY selector. 
3.	Update slider values to obtain desired effects parameters. 
4.	Move position of XY selector in order to generate noise. 
5.	Select first radio selector option. 
6.	Click “train.” 
 
 
Randomizer 
 
Randomizers were provided in tandem with ‘metro’ objects in order to drive the position of the pointer on the XY selector. This was found useful to synthesize chaotic soundscapes, such as wind noises. 
 
Patch Operation: 
 
1.	Copy external objects into Max 8 externals folder, specifically the ‘ml.mulreg.mxe’ object. Without these the patch will not function properly. 
2.	Click “read” and select formants.json file. 
3.	Press Toggle to start patch. 
4.	Edit destination name in read message to point towards locally saved regression mode. 
 
Play and hold midi note on midi controller to synthesize drone. Use the mouse and the XY pad to modify drone parameters based on their mappings. 
 
 
 
 
References: 
1.	Newt.phys.unsw.edu.au. (2020). The Didjeridu (Didgeridoo)	. [online] Available at: 
https://newt.phys.unsw.edu.au/jw/dij/dij.html [Accessed 18 Jan. 2020]. 
2.	Homes.esat.kuleuven.be. (2020). [online] Available at: 
https://homes.esat.kuleuven.be/~bdmdotbe/bdm2013/documents/doc_080327_14.10.pdf [Accessed 18 Jan. 2020]. 
3.	Ccrma.stanford.edu. (2020). MATHEMATICS OF THE DISCRETE FOURIER TRANSFORM	 (DFT)WITH AUDIO APPLICATIONSSECOND EDITION. [online] Available at: 
https://ccrma.stanford.edu/~jos/mdft/ 
4.	Quod.lib.umich.edu. (2020). [online] Available at: 
https://quod.lib.umich.edu/cgi/p/pod/dod-idx/synthesizing-the-dynamic-spectra-of-the-didgerid oo.pdf?c=icmc;idno=bbp2372.2010.120 
5.	Kvraudio.com. (2020). KVR Forum: physical modelling of a didgeridoo	. [online] Available at: 
https://www.kvraudio.com/forum/viewtopic.php?t=218612 Post by MadBrain Fri Jun 13, 2008 3:48 pm. 
6.	Ccrma.stanford.edu. (2020). Schroeder Allpass Sections	. [online] Available at: 
https://ccrma.stanford.edu/~jos/pasp/Schroeder_Allpass_Sections.html  
 
 
 





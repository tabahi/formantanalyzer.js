# formantanalyzer.js

![npm](https://img.shields.io/npm/v/formantanalyzer?style=plastic)
![GitHub branch checks state](https://img.shields.io/github/checks-status/tabahi/formantanalyzer.js/main)
![NPM](https://img.shields.io/npm/l/formantanalyzer?style=plastic)
[![Inline docs](http://inch-ci.org/github/tabahi/formantanalyzer.js.svg?branch=main)](http://inch-ci.org/github/tabahi/formantanalyzer.js)


A JS [Web API](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext) based spectrum analyzer for speech and music analysis. It can be used for labeling or feature extraction.

![Preview](./assets/preview.png "Preview")

## Demo

Bare minimum starter file: [tabahi.github.io/formantanalyzer.js](https://tabahi.github.io/formantanalyzer.js/)

### Related projects:

- [Web Speech Analyzer](https://tabahi.github.io/WebSpeechAnalyzer/?dev=1)
- [Speech Emotion Analyzer](https://tabahi.github.io/WebSpeechAnalyzer/?type=cats&label=emotion)


## Installation

### Using JS Script

Load the javascript module as
```html
<script src="https://unpkg.com/formantanalyzer@1.1.8/index.js"></script>
```
Then use the entry point `FormantAnalyzer` to use the imported javascript library.

### Using Webpack Library

Prerequisites: [Node.js](https://nodejs.org/en/download/), check versions >12 for `node -v` and >6 for `npm -v`.

- For importing the JS package in webpack project: Install [formantanalyzer](https://www.npmjs.com/package/formantanalyzer) using `npm`

```cmd
npm i formantanalyzer
:: or
npm install formantanalyzer --save-dev
```


```javascript
const FormantAnalyzer = require('formantanalyzer'); //after npm installation
FormantAnalyzer.configure(launch_config);
FormantAnalyzer.LaunchAudioNodes(2, webAudioElement, call_backed_function, ['file_label'], false, false);
```



If you are creating a webpack project from scratch by just using JS files from `./src` then first initialize a new webpack project by `npx webpack` and create a new `webconfig.js` in project root directory. See instructions [here](https://webpack.js.org/guides/getting-started/#using-a-configuration).

## Usage

First, configure the fomant analyzer. Pass the `#SpectrumCanvas` element if plot is enabled. Pass `null` if no need for plot. See 'index.html' for a simple example.

HTML:
```html
<div id="canvas_div">
    <canvas id="SpectrumCanvas" width="1200" height="300" ></canvas>
</div>
```
In javascript:

```javascript
/*Using <script src="https://unpkg.com/formantanalyzer@1.1.8/index.js"></script>
Can also import in webpack as:
const FormantAnalyzer = require('formantanalyzer');
*/

function Configure_FormantAnalyzer()
{
    const BOX_HEIGHT = 300;
    const BOX_WIDTH = window.screen.availWidth - 50;
    document.getElementById('SpectrumCanvas').width = BOX_WIDTH;    //reset the size of canvas element
    document.getElementById('SpectrumCanvas').height = BOX_HEIGHT;
    
    let launch_config = { plot_enable: true,
    spec_type: 1,       //see below
    output_level: 2, //see below
    plot_len: 200, f_min: 50, f_max: 4000,
    N_fft_bins: 256,
    N_mel_bins: 128,
    window_width: 25, window_step: 15,
    pause_length: 200, min_seg_length: 50,
    auto_noise_gate: true, voiced_min_dB: 10, voiced_max_dB: 100,
    plot_lag: 1, pre_norm_gain: 1000, high_f_emph: 0.0,
    plot_canvas: document.querySelector('#SpectrumCanvas').getContext('2d'),
    canvas_width: BOX_WIDTH,
    canvas_height: BOX_HEIGHT };

    FormantAnalyzer.configure(launch_config);
}
```


Initialize an Audio Element, or local audio file binary element, or `null` if using the mic stream. Then pass it to `LaunchAudioNodes` with suitable parameters. See 'index.html' for examples for local audio file and mic streaming.

```javascript

var webAudioElement = new Audio("./audio_file.mp3");
/*Parameters:*/
const context_source = 2;   //1: Local file binary, 2: play from a web Audio, 3: mic
const test_mode = true; //plots only, it does not return callback
const offline = false;  //play on speakers, set true to play silently
const file_labels =[];     //array of labels that will be passed to callback after feature extraction

/* Wait for audio file to load */
webAudioElement.addEventListener("canplaythrough", event => {
    /* Launch Audio Nodes */
    FormantAnalyzer.LaunchAudioNodes(context_source, webAudioElement,
                    callback, file_labels, offline, test_mode).then(function()
        {
            console.log("Done");
        }).catch((err)=>{
            console.log(err);
        });
});

callback(seg_index, file_labels, seg_time, features)
{
//callback function to which extracted features are passed
}
```

## `LaunchAudioNodes()` 

Returns: This function returns a promise as `resolve(true)` after playback is finished or `reject(err)` if there is an error.
If you want an abrupt stop, then call the `FormantAnalyzer.stop_playing("no reason")` function. Then this function will return `resolve("no reason")`. Different audio contexts are buffered/streamed differently, therefore each has a separate function in `AudioNodes.js`.

### Parameters:

`context_source` (int):
- 1 --- Play from a locally loaded file (pass an audio binary as source_obj).
- 2 --- Play from an Audio element (pass an `Audio` object as source_obj)
- 3 --- Stream from mic

`source_obj` (object):
Source audio object.
- If `context_source==1` (playing from a local file) then pass a binary of file. Get binary from `FileReader` as: `FileReader.onload (e)=>(binary = e.target.result)`. See [`index.html`](https://github.com/tabahi/formantanalyzer.js/blob/main/index.html#L236).
- If `context_source==2` (playing from a web address) then pass an `Audio` object.
- If `context_source==3` (playing from mic Pass `null`). See [`index.html`](https://github.com/tabahi/formantanalyzer.js/blob/main/index.html#L144).

`callback`:
It is the callback function to be called after each segment ends. It should accept 4 variables; `segment_index`, `segment_time_array`, `segment_labels_array`, `segment_features_array`. Callback is called asynchronously, so there might be a latency between audio play and it's respective callback, that's why it's important to send the labels to async segmentor function.

`file_labels` (Array):
It is an array of labels for currently playing file. It is returned as it is to the callback function.
It is used to avoid the label mismatch during slow async processing in case if a new file is playing, but the callback sends the output of the previous one.
Sometimes callback is called with a delay of 2 seconds, so it helps to keep track which file was playing 2 seconds ago.

`offline_mode` (boolean):
If true then the locally loaded files will be played silently in an offline buffer.

`test_play` (boolean)
set it true to avoid calling the callback. Plots and AudioNodes will still work as it is, but there will be no call backs. It can be enabled to test plotting or re listening.

`play_offset` and `play_duration` are in seconds to play a certain part of the file, otherwise pass null.

As an example, `callback` function in `WebSpeechAnalyzer` app looks like this:

```javascript

async function callback(seg_index, seg_label, seg_time, features)
{
    if(launch_config.output_level == 13) //Syllable 53x statistical features for each segment
    {
        if(settings.collect)
        for(let segment = 0; segment < features.length; segment++ ) 
        {
            storage_mod.StoreFeatures(launch_config.output_level, settings.DB_ID, (seg_index + (segment/100)), seg_label, seg_time[segment], features[segment]);
        }
        
        if(settings.plot_enable && settings.predict_en)
        {
            pred_mod.predict_by_multiple_syllables(settings.predict_type, settings.predict_label, seg_index, features, seg_time);
        }
    }
}

```

Different types of features are returned to the `callback(seg_index, seg_label, seg_time, features)` at different output levels. Variables for `callback`:
- `seg_index` the index of segment since the play start. Segments are separated by significant pauses, the first segment of each file starts has `seg_index = 0`.
- `seg_label` is the same as `file_labels` passed to `LaunchAudioNodes()`.
- `seg_time` is an array of two elements `[start_time, total_duration]` at segment level. At syllable level, it is a 2D array of shape (syllables, 2), giving `[start_time, total_duration]` for each syllable separately.
- `features` is the array of extracted features that is of different shape at different `output_level`. Detailed descriptions for each level are given below.

## `configure()`

Before playing any audio source using `LaunchAudioNodes()`, the audio nodes must be configured otherwise, default `launch_config` settings will be assumed.

```javascript
let launch_config = { plot_enable: true,
    spec_type: 1,
    output_level: 4,
    plot_len: 200,
    f_min: 50,
    f_max: 4000,
    N_fft_bins: 256,
    N_mel_bins: 128,
    window_width: 25,
    window_step: 15,
    pause_length: 200,
    min_seg_length: 50,
    auto_noise_gate: true,
    voiced_min_dB: 10,
    voiced_max_dB: 100,
    plot_lag: 1,
    pre_norm_gain: 1000,
    high_f_emph: 0.0,
    plot_canvas: document.querySelector('#SpectrumCanvas').getContext('2d'),
    canvas_width: 900,
    canvas_height: 300 };

    FormantAnalyzer.configure(launch_config);
```

Available `spec_type` options:
- 1 = Mel-spectrum
- 2 = Power Spectrum
- 3 = Discrete FFT

Available `output_level` options:

- 1 = Bars (no spectrum, only the last filter bank)
- 2 = Spectrum
- 3 = Segments
- 4 = Segment Formants / segment
- 5 = Segment Features 53x / segment [ML]
- 10 = Syllable Formants / syllable
- 11 = Distributions 264x / file [ML]
- 12 = Syllable Curves 23x / syllable [ML]
- 13 = Syllable Features 53x / syllable [ML]

### Output Level Descriptions:

Different `features` are returned to the `callback(seg_index, seg_label, seg_time, features)` at different output levels. The shape of `seg_time` also differs from `(2)` to `(syllable, 2)` for segment vs syllable. The shape of `features` array at each level is as follows:

- `Bar` and `Spectrum` levels return a 1D array of raw FFT or Mel bins (depending on the `spec_type`) at the interval of each window step (~25 ms). The only difference is that `Spectrum` level keeps a history of bins for plotting the spectrum. `Spectrum` level returns callback with a 2D array of size `plot_len` x `N_bins` after each `min_seg_length` milliseconds. `Bar` returns an array of `1` x `N_bins`of the latest step.

- `Segments` level returns a 2D array of shape `(step, bins)` of FFT or Mel-bins for each segment. The 1st axis is along the window steps, 2nd axis is along the FFT or Mel bins at each step. Each segment is separated by pauses in speech.
- `Segment Formants` returns an array of shape `(steps, 9)`. The 9 features include frequency, energy and bandwidth of 3 most prominent formants at that particular window step. Indices `[0,1,2]` are the frequency, energy and bandwidth of the lowest frequency formant.
- `Syllable Formants` returns the same array of shape `(steps, 9)` as `Segment Formants` but in this case the division and the length (total number of steps) is much shorter because syllables are separated by even minor pauses and other sudden shifts in formant frequency and energy.
- `Segment Features 53x` returns a 1D array of shape `(53)` which has 53 statistical formant based features.
- `Syllable Features 53x` returns a 2D array of shape `(syllables, 53)` which has 53 statistical formant based features for each syllable in the segment.
- `Syllable Curves 23x` returns a 2D array of shape `(syllables, 23)`. The 23 features are the polynomial constants extracted by curve fitting of sum of energies of all formants and curves for f0, f1, f2 frequencies. The fitted curve of energy is visible on the plot but the scale is not Hz.
- `Distributions 264x` returns a 1D array of shape `(264)` for normalized cumulative features for complete file since the play started. The sum of features resets only when a new file starts, but the latest normalized feature set is updated at each segment pause using the features of each segment.

Levels `5,11,12,13` have fixed output vector sizes (either per segment, per file, or per syllable) that's why they can be used as input for an ML classifier. At level `11,13`, the plot is the same as level `10`, but the different types of extracted features are returned to the callback function.

`auto_noise_gate: true` automatically sets the speech to silence thresholds to detect voiced segments. To use manual thresholds, set it to `false` and set manual values for `voiced_min_dB` and `voiced_max_dB`.
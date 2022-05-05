# KSON Format Specification (version: `0.2.0-beta21`)
- Encoding: UTF-8 (without BOM), LF
- If a default values is specified in this document, undefined values are overwritten by the default value.
- `null` value is not allowed in the entire kson file.
- Support for parameters/options marked "(OPTIONAL)" are optional, but must be ignored if not supported.
- `xxx` and `...` denote placeholders.
- The resolution of pulse value (`y`) is 240 per beat (i.e., 960 per measure).
- The behavior of reading an invalid value is undefined and kson clients do not necessarily have to stop loading the kson file as an error.

-----------------------------------------------------------------------------------

## Top-level object
```
dictionary kson {
    DOMString     version;     // kson version (Semantic Versioning like "x.y.z")
    MetaInfo      meta;        // meta data, e.g. title, artist, ...
    BeatInfo      beat;        // beat-related data, e.g. bpm, time signature, ...
    GaugeInfo?    gauge;       // gauge-related data
    NoteInfo?     note;        // notes on each lane
    AudioInfo?    audio;       // audio-related data
    CameraInfo?   camera;      // camera-related data
    BgInfo?       bg;          // background-related data
    ImplInfo?     impl;        // (OPTIONAL) implementation-dependent options for each client
}
```

-----------------------------------------------------------------------------------

## `meta`
```
dictionary MetaInfo {
    DOMString       title;                 // self-explanatory
    DOMString?      title_img_filename;    // (OPTIONAL) use an image instead of song title text
    DOMString       artist;                // self-explanatory
    DOMString?      artist_img_filename;   // (OPTIONAL) use an image instead of song artist text
    DOMString       chart_author;          // self-explanatory
    DifficultyInfo  difficulty;            // self-explanatory
    unsigned int    level;                 // self-explanatory, 1-20
    DOMString       disp_bpm = "";         // displayed bpm (allowed characters: 0-9, "-", ".")
    double          std_bpm = 0;           // (OPTIONAL) standard bpm for hi-speed values (should be between minimum bpm and maximum bpm in the chart); automatically set if zero
    DOMString?      jacket_filename;       // self-explanatory (can have a preset image "nowprinting1"/"nowprinting2"/"nowprinting3")
    DOMString?      jacket_author;         // self-explanatory
    DOMString?      information;           // (OPTIONAL) optional information shown in song selection
    DOMString?      ksh_version;           // (OPTIONAL) "ver" field of KSH file (specified only if converted from KSH)
}
```

### `meta.difficulty`
```
dictionary DifficultyInfo {
    unsigned char idx;  // 0-3 (0:light, 1:challenge, 2:extended, 3:infinite)
}
```

-----------------------------------------------------------------------------------

## `beat`
```
dictionary BeatInfo {
    ByPulse<double>[] bpm;                        // bpm changes
    ByMeasureIndex<TimeSig>[]? time_sig;          // time signature changes
                                                  // this is used for drawing bar lines and audio effects
    GraphPoint[]? scroll_speed;                   // scroll speed changes (e.g. stop events)
}
```

### `beat.time_sig`
```
dictionary TimeSig {
    unsigned long n;      // numerator
    unsigned long d;      // denominator
}
```

-----------------------------------------------------------------------------------

## `gauge`
```
dictionary GaugeInfo {
    unsigned long total = 0;  // total ascension of gauge percentage in the entire chart (0 or 100-)
                              // automatically set if zero
}
```

-----------------------------------------------------------------------------------

## `note`
```
dictionary NoteInfo {
    Interval[4][]? bt;               // BT notes (first index: lane) (l=0: chip note, l>0: long note)
    Interval[2][]? fx;               // FX notes (first index: lane) (l=0: chip note, l>0: long note)
    LaserSection[2][]? laser;        // laser notes (first index: lane (0: left knob, 1: right knob))
}
```

- For long BT/FX notes, if the end point of one note and the start point of another note are at the same time (i.e., `y` is the same as `y + l` of the previous note), they are joined as one long note when played. This is mainly used when several audio effects are switched in one long FX note.
- Two or more notes cannot be overlapped on a single lane.

### `note.laser[lane][idx]`
```
dictionary LaserSection : ByPulse<GraphSectionPoint[]> {
    unsigned long y;                // pulse number
    GraphSectionPoint[] v;          // laser points (0.0-1.0)
    unsigned char w = 1;            // x-axis scale (1-2), sets whether this laser section is 2x-widen or not
}
```

-----------------------------------------------------------------------------------

## `audio`
```
dictionary AudioInfo {
    BgmInfo? bgm;                   // bgm-related data
    KeySoundInfo? key_sound;        // key-sound-related data
    AudioEffectInfo? audio_effect;  // audio-effect-related data
}
```

### `audio.bgm`
```
dictionary BgmInfo {
    DOMString filename = "";  // self-explanatory
    double vol = 1.0;         // bgm volume
    long offset = 0;          // offset in milliseconds (starting point of the audio file)
    BgmPreviewInfo? preview;  // preview information
    LegacyBgmInfo? legacy;    // (OPTIONAL) legacy information
}
```

#### `audio.bgm.preview`
```
dictionary BgmPreviewInfo {
    unsigned long offset = 0;        // preview offset in milliseconds (starting point of the audio file)
    unsigned long duration = 15000;  // preview duration in milliseconds
}
```

### `audio.legacy` (OPTIONAL)
```
dictionary LegacyBgmInfo {
    DOMString[]? fp_filenames;  // filenames of prerendered BGM with audio effects from legacy KSH charts
                                // e.g. [ "xxx_f.ogg", "xxx_p.ogg", "xxx_fp.ogg" ]
}
```

### `audio.key_sound`
```
dictionary KeySoundInfo {
    KeySoundFXInfo? fx;        // key sound for FX notes
    KeySoundLaserInfo? laser;  // key sound for laser slams
}
```
- Note: `fx` and `laser` have different ways of specifying the volume of key sounds.

#### `audio.key_sound.fx`
```
dictionary KeySoundFXInfo {
    KeySoundInvokeListFX? chip_event;  // key sound for chip FX notes
}
```
- Note: `audio.key_sound.fx.chip_event.xxx[lane][].y` should be the same as `y` of an existing chip FX note on the corresponding lane, otherwise the event is ignored.

##### `audio.key_sound.fx.chip_event`
```
dictionary KeySoundInvokeListFX {
    ByPulse<KeySoundInvokeFX>[2][]? clap;         // (OPTIONAL)
    ByPulse<KeySoundInvokeFX>[2][]? clap_impact;  // (OPTIONAL)
    ByPulse<KeySoundInvokeFX>[2][]? clap_punchy;  // (OPTIONAL)
    ByPulse<KeySoundInvokeFX>[2][]? snare;        // (OPTIONAL)
    ByPulse<KeySoundInvokeFX>[2][]? snare_lo;     // (OPTIONAL)

    ByPulse<KeySoundInvokeFX>[2][]? ...;          // Custom key sounds can be inserted here by using the filename of a WAVE file (.wav) as a key
}
```

##### `audio.key_sound.fx.chip_event.xxx[lane][].v`
```
dictionary KeySoundInvokeFX {
    double vol = 1.0;  // key sound volume
}
```

#### `audio.key_sound.laser`
```
dictionary KeySoundLaserInfo {
    ByPulse<double>? vol;                  // laser slam volume (default: 0.5)
    KeySoundInvokeListLaser? slam_event;   // (OPTIONAL) key sound invocation by laser slam notes
    KeySoundLaserLegacyInfo? legacy;       // (OPTIONAL) legacy information
}
```
- Note: `audio.key_sound.laser.slam_event.xxx[].y` should be the same as `y` of an existing laser slam note, otherwise the event is ignored.
- Note: The `vol` value changes do not affect key sounds currently being played.

##### `audio.key_sound.laser.slam_event` (OPTIONAL)
```
dictionary KeySoundInvokeListLaser {
    ByPulse[]? slam_up;     // (OPTIONAL)
    ByPulse[]? slam_down;   // (OPTIONAL)
    ByPulse[]? slam_swing;  // (OPTIONAL)
    ByPulse[]? slam_mute;   // (OPTIONAL)

    // Note: Inserting custom key sounds here is not allowed
}
```

##### `audio.key_sound.laser.legacy`
```
dictionary KeySoundLaserLegacyInfo {
    bool auto_vol = false;  // "chokkakuautovol" in KSH format
}
```

### `audio.audio_effect`
```
dictionary AudioEffectInfo {
    AudioEffectFXInfo? fx;        // audio effects for FX notes
    AudioEffectLaserInfo? laser;  // audio effects for laser notes
}
```

#### `audio.audio_effect.fx`
```
dictionary AudioEffectFXInfo {
    DefList<AudioEffect>? def;                          // audio effect definitions
    InvokeList<ByPulse<AudioEffect>[]>? param_change;   // audio effect parameter changes by pulse
    InvokeList<ByPulse<AudioEffect>[2][]>? long_event;  // audio effect invocation (and parameter changes) by FX notes
}
```
- Note: `audio.audio_effect.fx.long_event.xxx[lane][].y` should be the same as `y` of an existing long FX note on the corresponding lane, otherwise the event is ignored.

#### `audio.audio_effect.laser`
```
dictionary AudioEffectLaserInfo {
    DefList<AudioEffect>? def;                         // audio effect definitions
    InvokeList<ByPulse<AudioEffect>[]>? param_change;  // audio effect parameter changes by pulse
    InvokeList<ByPulse[]>? pulse_event;                // audio effect invocation by pulse
}
```

##### `audio.audio_effect.fx.def`/`audio.audio_effect.laser.def`
```
dictionary Def<AudioEffect> {
    DOMString type;      // audio effect type (e.g. "flanger")
    AudioEffect? v {
        DOMString ...;   // audio effect parameter values
    };
    DOMString? filename; // This can be specified only if type="switch_audio". The filename of an audio file.
}
```
- Examples:
    1. "`type=Flanger;delay=80samples;depth=30samples>40samples-60samples`" in KSH
       ```
       {
           "type":"flanger",
           "v":{
                "delay": "80samples",
                "depth": "30samples>40samples-60samples"
           }
       }
       ```

    2. "`type=TapeStop;trigger=off>on;speed=20%`" in KSH
       ```
       {
           "type":"tapestop",
           "v":{
                "trigger": "off>on",
                "speed": "20%"
           }
       }
       ```

    3. "`type=Retrigger;waveLength=100ms`" in KSH
       ```
       {
           "type":"retrigger",
           "v":{
                "wave_length": "100ms"
           }
       }
       ```

    4. "`type=SwitchAudio;fileName=music.ogg`" in KSH
       ```
       {
           "type":"switch_audio",
           "filename":"music.ogg" // not in "v" because "filename" is not changeable in invocation
       }
       ```
- Audio effects in the "Audio effects & parameter list" are predefined with its default parameter values.


### Audio effect parameter types

All parameter values are given by string, but the values must follow one of the allowed formats of the specified type.

`[int]` is a placeholder that accepts integers (available characters:`0123456789-`).  
`[float]` is a placeholder that accepts integers and decimal numbers (available characters:`0123456789-.`).  
Leading plus signs (e.g., "`+1`") and scientific notation (e.g., "`1e-3`", "`1E+5`") are not allowed.

- length
    - Duration which can be tempo-synced
    - Allowed formats:
        - `1/[int]`: Specifies the percentage of one measure in fractional form. Tempo synced.
            - Requirement: int >= 1
            - Example: `1/4`
        - `[float]`: Specifies the percentage of one measure in decimal form. Tempo synced.
            - Requirement: float >= 0.0
            - Example: `2.0`, `0.25`, `0`
        - `[float]ms`: Specifies the duration in milliseconds. Not tempo synced.
            - Requirement: float > 0.0
            - Example: `100ms`, `10.5ms`
        - `[float]s`: Specifies the duration in seconds. Not tempo synced.
            - Requirement: float > 0.0
            - Example: `1s`, `0.1s`
- sample
    - Short duration
        - The value is the number of samples, where the sampling rate is 44100Hz
        - For example, `44100samples` means 1.0s
    - Allowed formats:
        - `[int]samples`
            - Requirement: 1 <= int <= 44100
            - Example: `40samples`
    - The trailing "s" cannot be omitted even if the value is 1 (i.e., "`1sample`" is illegal, use "`1samples`" instead).
- switch
    - Specifies a boolean value.
    - Allowed formats:
        - `on`
        - `off`
- rate
    - Ratio value
    - Allowed formats:
        - `1/[int]`
            - Requirement: int >= 1
            - Example: `1/2`
        - `[int]%`
            - Requirement: int >= 0
            - Example: `50%`
        - `[float]`
            - Requirement: float >= 0.0
            - Example: `0.5`
    - Note: Some parameters can have values higher than `1.0` (e.g. `flanger.vol`).
- freq
    - Frequency value (Hz)
    - Allowed formats:
        - `[int]Hz`
            - Requirement: 10 <= int <= 20000
        - `[float]kHz`
            - Requirement: 0.01 <= float <= 20.0
- dB
    - Decibel value
    - Allowed formats:
        - `[float]dB`
            - Requirement: float >= 0.0
            - Example: `3.0dB`
- pitch
    - Key in music (12 per octave)
    - Allowed formats:
        - `[float]`
            - Requirement: -48.0 <= float <= 48.0
            - Example: `12.0`, `-6`
    - (OPTIONAL) If the string value does not contain "." (i.e., the value is an integer), the transition value is quantized to an integer (e.g., `12.9`->`12`, `-1.1`->`-2`).
- int
    - Integer value
    - Allowed formats:
        - `[int]`
            - Example: `10`, `-5`
    - The transition value is quantized to an integer (e.g., `12.9`->`12`, `-1.1`->`-2`).
- float
    - Floating-point value
    - Allowed formats:
        - `[float]`
            - Example: `2.5`, `-10`


### Audio effect parameter value format

The parameter value consists of three values, Off/OnMin/OnMax, in the string format "Off>OnMin-OnMax".

While pressing the long FX note assigned to the corresponding audio effect, the value is set to OnMin, otherwise the value is set to Off. OnMax is ignored for long FX notes.

While the laser note is judged, the value transitions between OnMin and OnMax depending on the laser cursor position; otherwise the value is set to Off.

Parameter values are written in one of the following formats:
- `Off`
    - OnMin and OnMax inherit the Off value
    - Example: `50%` (equivalent to `50%>50%` or `50%-50%` or `50%>50%-50%`)
- `Off>OnMin`
    - OnMax inherits the OnMin value
    - Example: `0%>100%` (equivalent to `0%>100%-100%`)
- `OnMin-OnMax`
    - Off inherits the OnMin value
    - Example: `50%-100%` (equivalent to `50%>50%-100%`)
- `Off>OnMin-OnMax`
    - Example: `0%>50%-100%`


### Audio effects & parameter list

- `retrigger`: This effect repeats audio.
    - `update_period` (length, default:`1/2`)
        - Interval for automatic update of the repeat source
        - Additional requirement:
            - The formats `[float]ms` and `[float]s` are not allowed.
        - `0`: Automatic trigger update is disabled.
        - Note: `update_period` interval count is reset at the beginning of each measure if `update_period` has a non-zero value.
    - `wave_length` (length, default:`0`)
        - Length of repetition
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`. Also, parameter changes to `0` are ignored.)
        - Note: `wave_length` interval count is reset at the beginning of each measure if `update_period` has a non-zero value.
    - `rate` (rate, default:`70%`)
        - Length of the repeat audio
            - A value of 100% repeats the audio sample completely, and a smaller value gives a larger percentage of mute time in each period.
    - `update_trigger` (switch, default:`off`)
        - `on`: Updates the repeat source (the value is automatically set back to `off`)
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
- `gate`: This effect periodically switches the volume between 100% and 0%.
    - `wave_length` (length, default:`0`)
        - Interval
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`. Also, parameter changes to `0` are ignored.)
        - Note: `wave_length` interval count is reset at the beginning of each measure.
    - `rate` (rate, default:`60%`)
        - Length of the audio
            - A value of 100% has no effect, and a smaller value gives a larger percentage of mute time in each period.
    - `mix` (rate, default:`0%>90%`)
        - Blending ratio of the original audio and the effect audio
- `flanger`: This effect layers the delayed audio and the original audio. The delay time is oscillated by an LFO, which generates a sweeping comb filter.
    - `period` (length, default:`2.0`)
        - LFO period
    - `delay` (sample, default:`30samples`)
        - Minimum value of delay time
    - `depth` (sample, default:`45samples`)
        - LFO depth (magnitude of value change)
    - `feedback` (rate, default:`60%`)
        - Feedback rate
    - `stereo_width` (rate, default:`0%`)
        - LFO phase difference between the L/R channels
    - (OPTIONAL) `vol` (rate, default:`75%`)
        - Volume of the effect audio
    - `mix` (rate, default:`0%>80%`)
        - Blending ratio of the original audio and the effect audio
- `pitch_shift`: This effect changes the pitch (key) of the audio.
    - `pitch` (pitch, default:`0`)
        - Pitch (key)
    - (OPTIONAL) `chunk_size` (sample, default:`700samples`)
        - Size of the waveform section. The larger the value, the better the sound quality, but the sound will be delayed.
    - (OPTIONAL) `overlap` (rate, default:`40%`)
        - Crossfade time ratio between waveform sections
        - Note: This parameter was described as `overWrap` in KSH format, but it was a spelling mistake.
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
- `bitcrusher`: This effect reduces the quality of the audio wave. Also known as "Sample & Hold".
    - `reduction` (sample, default:`0samples`)
        - Number of samples to hold. A larger value results in lower sound quality.
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
- `phaser`: This effect applies multiple all-pass filters that shift the phase of the waveform and layers the effect audio and the original audio.
    - `period` (length, default:`1/2`)
        - LFO period
    - `stage` (int, default:`6`)
        - Number of all-pass filters. Usually an even number.
        - Additional requirement:
            - 0 <= int <= 12
    - `lo_freq` (freq, default:`1500Hz`)
        - Minimum frequency of LFO
    - `hi_freq` (freq, default:`20000Hz`)
        - Maximum frequency of LFO
    - `q` (float, default:`0.707`)
        - Q value of all-pass filters
        - Additional requirement:
            - 0.1 <= float <= 50.0
    - `feedback` (rate, default:`35%`)
        - Feedback rate
    - `stereo_width` (rate, default:`0%`)
        - LFO phase difference between the L/R channels
    - `mix` (rate, default:`0%>50%`)
        - Blending ratio of the original audio and the effect audio
        - Note: For phaser effects, the mix value is doubled when used. A typical phaser effect is usually most effective at a mix value of 50%, but this makes it most effective at a mix value of `100%`. Note that the default value `50%` is actually a mix value of 25%.
    - Note: `hiCutGain` parameter in KSH format has been removed in kson format because it is not a parameter of the phaser itself.
- `wobble`: This effect oscillates the cutoff frequency of the low-pass filter with an LFO.
    - `wave_length` (length, default:`0`)
        - LFO period
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`. Also, parameter changes to `0` are ignored.)
        - Note: `wave_length` interval count is reset at the beginning of each measure if `update_period` has a non-zero value.
    - `lo_freq` (freq, default:`500Hz`)
        - Minimum frequency of LFO
    - `hi_freq` (freq, default:`20000Hz`)
        - Maximum frequency of LFO
    - `q` (float, default:`1.414`)
        - Q value of the low-pass filter
    - `mix` (rate, default:`0%>50%`)
- `tapestop`: This effect slows down the playback speed of audio like a turntable.
    - `speed` (rate, default:`50%`)
        - Speed of slowdown
    - `trigger` (switch, default:`off>on`)
        - Whether the tapestop is activated
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
- `echo`: This effect is a retrigger effect with fadeout.
    - `update_period` (length, default:`0`)
        - Interval for automatic update of the repeat source
        - Additional requirement:
            - The formats `[float]ms` and `[float]s` are not allowed.
        - `0`: Automatic trigger update is disabled.
    - `wave_length` (length, default:`0`)
        - Length of repetition
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`. Also, parameter changes to `0` are ignored.)
        - Note: `wave_length` interval count is reset at the beginning of each measure if `update_period` has a non-zero value.
    - `update_trigger` (switch, default:`off>on`)
        - `on`: Updates the repeat source (the value is automatically set back to `off`)
    - `feedback_level` (rate, default:`100%`)
        - Ratio of the volume to the previous repetition
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
- `sidechain`: This effect simulates ducking by a side-chain compressor.
    - `period` (length, default:`1/4`)
        - Interval
        - Additional requirement:
            - The formats `[float]ms` and `[float]s` are not allowed.
    - `hold_time` (length, default:`50ms`)
        - Duration to simulate a condition where the audio level exceeds the compressor threshold
    - `attack_time` (length, default:`10ms`)
        - Attack time
    - `release_time` (length, default:`1/16`)
        - Release time
    - `ratio` (int, default:`1>5`)
        - Compression ratio
            - A value of `1` compresses the audio by a factor of 1/1 (i.e., the original audio), and a value of `5` compresses the audio by a factor of 1/5.
        - Additional requirement:
            - 1 <= int <= 100
- `switch_audio`: This effect switches the playback to another audio file.
    - (`filename` (string))
        - Note that this is not in `v` but at the root of `Def<AudioEffect>`
- `high_pass_filter`: Bi-quad high-pass filter.
    - `env` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Linear transition of the `env` value is translated into a log scale transition when used.
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 0.0
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 1.0
    - `q` (float, default:??? `/*FIXME*/`)
        - Q value of the filter
    - `mix` (rate, default:`0%>100%`)
    - Note: `lo_freq` value may exceed the `hi_freq` value.
- `low_pass_filter`: Bi-quad low-pass filter.
    - `env` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Linear transition of the `env` value is translated into a log scale transition when used.
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 0.0
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 1.0
    - `q` (float, default:??? `/*FIXME*/`)
        - Q value of the filter
    - `mix` (rate, default:`0%>100%`)
    - Note: `lo_freq` value may exceed the `hi_freq` value.
- `peaking_filter`: Bi-quad peaking filter.
    - `env` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Linear transition of the `env` value is translated into a log scale transition when used.
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 0.0
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
        - Cutoff frequency when `env` is 1.0
    - `gain` (dB, default:??? `/*FIXME*/`)
        - Gain value of the filter
    - `q` (float, default:??? `/*FIXME*/`)
        - Q value of the filter
    - (OPTIONAL) `delay` (length, default:`0ms`)
        - Delay time until the `env` value is applied
        - Additional requirement:
            - The formats `1/[int]` and `[float]` are not allowed.
            - 0ms <= delay <= 160ms
    - `mix` (rate, default:`0%>100%`)
    - Note: `lo_freq` value may exceed the `hi_freq` value.

-----------------------------------------------------------------------------------

## `camera`
```
dictionary CameraInfo {
    TiltInfo? tilt;        // tilt-related data
    CamInfo? cam;          // cam-related data
}
```

### `camera.tilt`
```
dictionary TiltInfo {
    ByPulse<double>[]? scale;                 // tilt scale (default: 1.0)
    ByPulse<GraphSectionPoint[]>[]? manual;   // manual tilt
                                              // Note: The left laser being on the right edge is equal to a manual value of 1.0, and the right laser being on the left edge is equal to a manual value of -1.0.
                                              // Note: Two or more graph sections cannot be overlapped.
                                              // Note: "camera.tilt.scale" does not affect the scale of manual tilt. Manual tilt is always evaluated with a scale of 1.0.
    ByPulse<bool>[]? keep;                    // whether tilt is kept or not
                                              // (while tilt is kept, the tilt amount value is updated only to a larger absolute value with the same sign)
}
```

### `camera.cam`
```
dictionary CamInfo {
    CamGraphs? body;                              // cam value changes
    CamPatternInfo? pattern;                      // cam pattern
}
```

#### `camera.cam.body`
```
dictionary CamGraphs {
    GraphPoint[]? zoom;                           // zoom_bottom
    GraphPoint[]? shift_x;                        // zoom_side
    GraphPoint[]? rotation_x;                     // zoom_top
    GraphPoint[]? rotation_z;                     // rotation degree  (affects both highway & jdgline relatively)
    GraphPoint[]? rotation_z.highway;             // (OPTIONAL) rotation degree  (highway only)
    GraphPoint[]? rotation_z.jdgline;             // (OPTIONAL) rotation degree  (judgment line only)
    GraphPoint[]? center_split;                   // center_split
}
```
- KSH: -300 - 300 => kson: -3.0 - 3.0
- Subparameters (`rotation_z.highway`/`rotation_z.jdgline`) affect the value relatively.
    - For example, the actual value of `rotation_z.highway` will be `rotation_z + rotation_z.highway`.
        - ```
          "rotation_z":[{"y":0, "v":1.0}],
          "rotation_z.highway":[{"y":0, "v":0.5}]
          ```
          is equivalent to 
          ```
          "rotation_z":[{"y":0, "v":0.0}],
          "rotation_z.highway":[{"y":0, "v":1.5}],
          "rotation_z.jdgline":[{"y":0, "v":1.0}]
          ```

#### `camera.cam.pattern`
```
dictionary CamPatternInfo {
    CamPatternLaserInfo? laser;         // cam pattern for laser slams
}
```

##### `camera.cam.pattern.laser`
```
dictionary CamPatternLaserInfo {
    CamPatternInvokeList? slam_event;   // cam pattern invocation by laser slam notes
}
```
- Note: This is a specification with the possibility of defining `camera.cam.pattern.laser.def` as a future extension.

##### `camera.cam.pattern.laser.slam_event`
```
dictionary CamPatternInvokeList {
    ByPulseWithDirection<CamPatternInvokeSpin>[]? spin;
    ByPulseWithDirection<CamPatternInvokeSpin>[]? half_spin;
    ByPulseWithDirection<CamPatternInvokeSwing>[]? swing;     // (OPTIONAL)
}
```
- Note: `camera.cam.pattern.laser.slam_event.xxx[].y` & `camera.cam.pattern.laser.slam_event.xxx[].d` should be the same as `y` & sign(`vf` - `v`) of an existing laser slam note, otherwise the event is ignored.

##### `camera.cam.pattern.laser.slam_event.spin[].v`/`camera.cam.pattern.laser.slam_event.half_spin[].v`
```
dictionary CamPatternInvokeSpin {
    unsigned long l = 960;          // duration
}
```

##### `camera.cam.pattern.laser.slam_event.swing[].v` (OPTIONAL)
```
dictionary CamPatternInvokeSwing {
    unsigned long l = 960;          // duration
    double scale = 1.0;             // scale
    unsigned long repeat = 1;       // number of repetitions
    int decay_order = 0;            // order of the decay that scales camera values (0-2)
                                    // (note that this decay is applied even if repeat=1)
                                    // - equation: `value * (1.0 - ((l - ry) / l))^decay_order`
                                    // - 0: no decay, 1: linear decay, 2: squared decay
}
```

-----------------------------------------------------------------------------------

## `bg`

Since the BG specification is still under discussion, the legacy KSH background image (e.g. "`desert`") and layer (e.g. "`snow;1100;1`") is converted to "`legacy`" as they are for now.

```
dictionary BgInfo {
    LegacyBgInfo? legacy;  // (OPTIONAL)
}
```

### `bg.legacy` (OPTIONAL)
```
dictionary LegacyBgInfo {
    KshBg[2]? bg;        // first element: when gauge < 70%, second element: when gauge >= 70%
    KshLayer? layer;
    KshMovie? movie;
}
```
- If `bg` has only a single element, that bg is always used, regardless of the percentage of the gauge.

#### `bg.legacy.bg[]` (OPTIONAL)
```
dictionary KshBg {
    DOMString filename = "desert";  // self-explanatory (can be KSM default BG image such as "`desert`")
}
```

#### `bg.legacy.layer` (OPTIONAL)
```
dictionary KshLayer {
    DOMString filename = "arrow";    // self-explanatory (can be KSM default animation layer such as "`arrow`")
    long duration = 0;               // one-loop duration in milliseconds
                                     //   If the value is negative, the animation is played backwards.
                                     //   If the value is zero, the play speed is tempo-synced and set to 1 frame per 0.035 measure (= 28.571... frames/measure).
    KshLayerRotationInfo? rotation;  // rotation conditions
}
```

##### `bg.legacy.layer[xxx].rotation` (OPTIONAL)
```
dictionary KshLayerRotationInfo {
    bool tilt = true;  // whether lane tilts affect rotation of BG/layer
    bool spin = true;  // whether lane spins affect rotation of BG/layer
}
```

#### `bg.legacy.movie` (OPTIONAL)
```
dictionary KshMovie {
    DOMString? filename;  // self-explanatory
    long offset = 0;      // movie offset in millisecond
}
```

-----------------------------------------------------------------------------------

### `impl` (OPTIONAL)
```
dictionary ImplInfo {
    // Not specified. This area is free for use by kson clients and kson editors.
    // To avoid conflicts with other clients, it is highly recommended to have a top object whose key is the client name.
}
```

-----------------------------------------------------------------------------------

## Common objects

### interval
```
dictionary Interval {
    unsigned long y;          // pulse number
    unsigned long l = 0;      // length
}
```

### event triggered by measure index
```
dictionary ByMeasureIndex<T> {
    unsigned long idx;        // measure index
    T? v;                     // body
}
```

### event triggered by pulse
```
dictionary ByPulse {
    unsigned long y;          // pulse number
}
```
```
dictionary ByPulse<T> {
    unsigned long y;          // pulse number
    T? v;                     // body
}
```

### event triggered by pulse (with laser slam direction)
```
dictionary ByPulseWithDirection<T> {
    unsigned long y;          // pulse number
    int d;                    // laser slam direction, -1 (left) or 1 (right)
    T? v;                     // body
}
```

### graph (whole chart)
```
dictionary GraphPoint {
    unsigned long y;          // absolute pulse number
    double? v;                // value
    double? vf;               // second value (for an immediate change)
    double a = 0.0;           // x-coordinate of the curve control point (0.0-1.0)
    double b = 0.0;           // y-coordinate of the curve control point (0.0-1.0)
}
```
- If `v` is undefined, it inherits the `vf` value of the previous element. Note that the first element must not have an undefined `v` value.
- If `vf` is undefined, it inherits the `v` value.

### graph point (for graph sections = `ByPulse<GraphSectionPoint[]>`)
```
dictionary GraphSectionPoint {
    unsigned long ry;        // relative pulse number
    double? v;               // value
    double? vf;              // second value (for an immediate change)
    double a = 0.0;          // x-coordinate of the curve control point (0.0-1.0)
    double b = 0.0;          // y-coordinate of the curve control point (0.0-1.0)
}
```
- If `v` is undefined, it inherits the `vf` value of the previous element. Note that the first element must not have an undefined `v` value.
- If `vf` is undefined, it inherits the `v` value.

### definition
```
dictionary Def<T> {
    T v;                     // changeable parameters (default values)

    ...                      // unchangeable parameters (e.g. filename, audio effect type)
}
```

### list of definitions
```
dictionary DefList<T> {
    Def<T>? ...(name is the key);
    ...
}
```

### list of invocations
```
dictionary InvokeList<T> {
    T? ...(name is the key);  // values which overwrites Def<T>.v
    ...
}
```

-----------------------------------------------------------------------------------

# Requirements

- All arrays that have `y` or `ry` (e.g. `ByPulse<T>[]`) must be ordered by `y` or `ry`.

- The first point of arrays that have `ry` (e.g. the first point of `GraphPoint[]`) must not have nonzero `ry`.

-----------------------------------------------------------------------------------

# Change Log

Change log before the version `0.1.0` can be referenced here: https://gist.github.com/m4saka/a89594a17dc9422d75e01998bcfd2722

- version `0.1.0` (06/14/2020)
    - No changes from the draft on 01/13/2020

-----------------------------------------------------------------------------------

# Acknowledgement
This specification is initially based on the BMSON format (https://bmson-spec.readthedocs.io/en/master/doc/index.html) and albshin's kshon format (https://gist.github.com/albshin/cf535afc3f94f7d7f7c7e3d1d9ff41cf).

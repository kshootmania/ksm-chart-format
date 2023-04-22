# KSON Format Specification (version: `0.6.2`)
## Basic Specifications
- **JSON format**: KSON files MUST use the JSON format.
- **File extension**: KSON files MUST use the `.kson` file extension.
- **Encoding**: KSON files MUST use UTF-8 (without BOM) with LF line endings.
- **Default values**: Undefined values are overwritten by specified defaults.
- **Null values not allowed**: `null` values MUST NOT be used in the entire KSON files.
- **Pulse number resolution**: `y` (pulse number) has a resolution of 240 per beat (960 per measure).

## Other Information and Guidelines
- **Placeholders**: `xxx` and `...` represent placeholders in this document.
- **Behavior for illegal values**: The behavior for illegal values is undefined, and KSON clients are not required to report them.
- **Optional support**: Parameters or options marked with "(OPTIONAL SUPPORT)" do not need to be supported by all KSON clients. However, unsupported parameters or options must be ignored.

-----------------------------------------------------------------------------------

## Top-level object
```
dictionary kson {
    version: string       // kson version (Semantic Versioning like "x.y.z")
    meta:    MetaInfo     // meta data, e.g. title, artist, ...
    beat:    BeatInfo     // beat-related data, e.g. bpm, time signature, ...
    gauge:   GaugeInfo?   // gauge-related data
    note:    NoteInfo?    // notes on each lane
    audio:   AudioInfo?   // audio-related data
    camera:  CameraInfo?  // camera-related data
    bg:      BGInfo?      // background-related data
    editor:  EditorInfo?  // (OPTIONAL SUPPORT) data used only in editors
    compat:  CompatInfo?  // (OPTIONAL SUPPORT) compatibility data with KSH format
    impl:    ImplInfo?    // (OPTIONAL SUPPORT) data that is sure to be for a specific client
}
```

-----------------------------------------------------------------------------------

## `meta`
```
dictionary MetaInfo {
    title:               string          // self-explanatory
    title_translit:      string?         // (OPTIONAL SUPPORT) transliterated title
    title_img_filename:  string?         // (OPTIONAL SUPPORT) use an image instead of song title text
    artist:              string          // self-explanatory
    artist_translit:     string?         // (OPTIONAL SUPPORT) transliterated artist name
    artist_img_filename: string?         // (OPTIONAL SUPPORT) use an image instead of song artist text
    chart_author:        string          // self-explanatory
    difficulty:          uint|string     // difficulty index 0-3 (0:light, 1:challenge, 2:extended, 3:infinite) or difficulty name (e.g. "gravity"/"maximum")
    level:               uint            // self-explanatory, 1-20
    disp_bpm:            string          // displayed bpm (allowed characters: 0-9, "-", ".")
    std_bpm:             double?         // (OPTIONAL SUPPORT) standard bpm for hi-speed values (should be between minimum bpm and maximum bpm in the chart); automatically set if undefined
    jacket_filename:     string?         // self-explanatory (preset images without file extensions are also acceptable; in KSM, either "nowprinting1"/"nowprinting2"/"nowprinting3")
    jacket_author:       string?         // self-explanatory
    icon_filename:       string?         // (OPTIONAL SUPPORT) icon image displayed on the music selection (preset images without file extensions are also acceptable; in KSM, files in "imgs/icon")
    information:         string?         // (OPTIONAL SUPPORT) optional information shown in song selection
}
```
- Note: If `meta.difficulty` is a string, the difficulty index is automatically set by the kson client. It is allowed for kson clients to ignore the difficulty name and fall back to the difficulty index `3`.
    - In KSM, the string value of `meta.difficulty` is always ignored (even if the value is set to "`light`"/"`challenge`"/"`extended`") and falls back to the difficulty index `3`.

-----------------------------------------------------------------------------------

## `beat`
```
dictionary BeatInfo {
    bpm:          ByPulse<double>[]                       // bpm changes
    time_sig:     ByMeasureIdx<TimeSig>[] = [[0, [4, 4]]] // time signature changes
    scroll_speed: GraphPoint[] = [[0, 1.0]]               // scroll speed changes
}
```

### `beat.time_sig[xxx]`
```
array TimeSig {
    [0]: uint  // numerator
    [1]: uint  // denominator
}
```

-----------------------------------------------------------------------------------

## `gauge`
```
dictionary GaugeInfo {
    total: uint = 0  // total ascension of gauge percentage in the entire chart (0 or 100-)
                     // automatically set if zero
}
```

-----------------------------------------------------------------------------------

## `note`
```
dictionary NoteInfo {
    bt: (uint|ButtonNote)[][4]?  // BT notes (first index: lane); uint represents y (pulse number) of chip note
    fx: (uint|ButtonNote)[][2]?  // FX notes (first index: lane); uint represents y (pulse number) of chip note
    laser: LaserSection[][2]?    // laser notes (first index: lane (0: left knob, 1: right knob))
}
```
- Two or more notes cannot be overlapped on a single lane.

### `note.bt[lane][idx]`/`note.fx[lane][idx]`
```
array ButtonNote {
    [0]: uint  // y: pulse number
    [1]: uint  // length (0: chip note, >0: long note)
}
```
- The array size of `ButtonNote` MUST be 2.

### `note.laser[lane][idx]`
```
array LaserSection {
    [0]: uint                 // y: pulse number
    [1]: GraphSectionPoint[]  // v: laser points (0.0-1.0)
    [2]: uint = 1             // w: x-axis scale (1-2), sets whether this laser section is 2x-widen or not
}
```
- The array size of `LaserSection` MUST be 2 or 3.

-----------------------------------------------------------------------------------

## `audio`
```
dictionary AudioInfo {
    bgm:          BGMInfo?          // bgm-related data
    key_sound:    KeySoundInfo?     // key-sound-related data
    audio_effect: AudioEffectInfo?  // audio-effect-related data
}
```

### `audio.bgm`
```
dictionary BGMInfo {
    filename:  string?           // self-explanatory
    vol:       double = 1.0      // bgm volume
    offset:    int = 0           // offset in milliseconds (starting point of the audio file)
    preview:   BGMPreviewInfo?   // preview information
    legacy:    LegacyBGMInfo?    // (OPTIONAL SUPPORT) legacy information
}
```

#### `audio.bgm.preview`
```
dictionary BGMPreviewInfo {
    offset:   uint = 0        // preview offset in milliseconds (starting point of the audio file)
    duration: uint = 15000    // preview duration in milliseconds
}
```

#### `audio.bgm.legacy` (OPTIONAL SUPPORT)
```
dictionary LegacyBGMInfo {
    fp_filenames: string[]      // filenames of prerendered BGM with audio effects from legacy KSH charts
                                // e.g. [ "xxx_f.ogg" ], [ "xxx_f.ogg", "xxx_p.ogg", "xxx_fp.ogg" ]
}
```

### `audio.key_sound`
```
dictionary KeySoundInfo {
    fx:     KeySoundFXInfo?     // key sound for FX notes
    laser:  KeySoundLaserInfo?  // key sound for laser slams
}
```
- Note: `fx` and `laser` have different ways of specifying the volume of key sounds.

#### `audio.key_sound.fx`
```
dictionary KeySoundFXInfo {
    chip_event: KeySoundInvokeListFX  // key sound for chip FX notes
}
```

##### `audio.key_sound.fx.chip_event`
```
dictionary KeySoundInvokeListFX {
    clap:        (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // (OPTIONAL SUPPORT) uint represents y (pulse number) of an invocation with default volume
    clap_impact: (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // (OPTIONAL SUPPORT)
    clap_punchy: (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // (OPTIONAL SUPPORT)
    snare:       (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // (OPTIONAL SUPPORT)
    snare_lo:    (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // (OPTIONAL SUPPORT)

    ...:         (uint|ByPulse<KeySoundInvokeFX>)[][2]?  // Custom key sounds can be inserted here by using the filename of a WAVE file (.wav) as a key
}
```
- Note: `y` (pulse number) should be the same as `y` of an existing chip FX note; otherwise, the event is ignored.

##### `audio.key_sound.fx.chip_event.xxx[lane][][1]`
```
dictionary KeySoundInvokeFX {
    vol: double = 1.0  // key sound volume
}
```

#### `audio.key_sound.laser`
```
dictionary KeySoundLaserInfo {
    vol:        ByPulse<double>[] = [[0, 0.5]] // laser slam volume
    slam_event: KeySoundInvokeListLaser?       // (OPTIONAL SUPPORT) key sound invocation by laser slam notes
    legacy:     KeySoundLaserLegacyInfo?       // (OPTIONAL SUPPORT) legacy information
}
```
- Note: The `vol` value changes do not affect key sounds currently being played.

##### `audio.key_sound.laser.slam_event` (OPTIONAL SUPPORT)
```
dictionary KeySoundInvokeListLaser {
    slam_up:    uint[]?  // (OPTIONAL SUPPORT) uint represents y (pulse number)
    slam_down:  uint[]?  // (OPTIONAL SUPPORT)
    slam_swing: uint[]?  // (OPTIONAL SUPPORT)
    slam_mute:  uint[]?  // (OPTIONAL SUPPORT)

    // Note: Inserting custom key sounds here is not allowed
}
```
- Note: `y` (pulse number) should be the same as `y` of an existing laser slam note; otherwise, the event is ignored.

##### `audio.key_sound.laser.legacy` (OPTIONAL SUPPORT)
```
dictionary KeySoundLaserLegacyInfo {
    vol_auto: bool = false  // Whether to automatically reduce the key sound volume
                            // based on the width of laser slams; "chokkakuautovol" in KSH format
}
```

### `audio.audio_effect`
```
dictionary AudioEffectInfo {
    fx:     AudioEffectFXInfo?     // audio effects for FX notes
    laser:  AudioEffectLaserInfo?  // audio effects for laser notes
}
```

#### `audio.audio_effect.fx`
```
dictionary AudioEffectFXInfo {
    def:          dictionary<AudioEffectDef>?                // audio effect definitions
    param_change: dictionary<dictionary<ByPulse<string>[]>>? // audio effect parameter changes by pulse
    long_event:   dictionary<(uint|ByPulse<dictionary<string>>)[][2]>? // audio effect invocation (and parameter changes) by long notes
}
```
- Note: `y` (pulse number) of `long_event` should be in the range `[y, y + length)` of an existing long FX note on the corresponding lane; otherwise, the event is ignored.
- Example for `audio.audio_effect.fx.param_change`/`audio.audio_effect.laser.param_change`:
    ```
    "param_change":{
        "retrigger":{
            "update_period":[
                [960, "0"],
                [1920, "1/2"]
            ],
            "update_trigger":[
                [1200, "on"]
            ]
        }
    }
    ```

#### `audio.audio_effect.laser`
```
dictionary AudioEffectLaserInfo {
    def: dictionary<AudioEffectDef>?                         // audio effect definitions
    param_change: dictionary<dictionary<ByPulse<string>[]>>? // audio effect parameter changes by pulse
    pulse_event: dictionary<uint[]>?                         // audio effect invocation by pulse
    peaking_filter_delay: uint = 0                           // (OPTIONAL SUPPORT) peaking filter delay time in milliseconds (0-160)
}
```
- Note: `audio.audio_effect.laser.pulse_event` cannot contain parameter changes. Use `audio.audio_effect.laser.param_change` instead.

##### `audio.audio_effect.fx.def.xxx`/`audio.audio_effect.laser.def.xxx`
```
dictionary AudioEffectDef {
    type: string               // audio effect type (e.g. "flanger")
    v:    dictionary<string>?  // audio effect parameter values
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
           "v":{
                "filename": "music.ogg"
           }
       }
       ```
- Audio effects in the "Audio effects & parameter list" are predefined with default parameter values. These predefined effects can be overridden by redefining them with the same name.
- An audio effect with a name of an empty string ("") is predefined as no effect. This is used to set a single long FX note to no audio effect from the middle of the note. Note that long FX notes with no effects assigned do not necessarily need to explicitly set `long_event` for this audio effect. The behavior of overriding an audio effect definition with an empty string ("") name is undefined.


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
            - Requirement: 0 <= int <= 100
            - Example: `50%`
        - `[float]`
            - Requirement: 0.0 <= float <= 1.0
            - Example: `0.5`
- freq
    - Frequency value (Hz)
    - Allowed formats:
        - `[int]Hz`
            - Requirement: 10 <= int <= 20000
        - `[float]kHz`
            - Requirement: 0.01 <= float <= 20.0
- pitch
    - Key in music (12 per octave)
    - Allowed formats:
        - `[float]`
            - Requirement: -48.0 <= float <= 48.0
            - Example: `12.0`, `-6`
    - (OPTIONAL SUPPORT) If the string value does not contain "." (i.e., the value is an integer), the transition value is quantized to an integer (e.g., `12.9`->`12`, `-1.1`->`-2`).
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
- filename
    - Filename string
    - Parameter values of this type can only be specified in `audio.audio_effect.xxx.def` and cannot be changed via `param_change`/`long_event`.


### Audio effect parameter value format

The parameter value consists of three values, Off/OnMin/OnMax, in the string format "Off>OnMin-OnMax".

While pressing the long FX note assigned to the corresponding audio effect, the value is set to OnMin; otherwise, the value is set to Off. OnMax is ignored for long FX notes.

While the laser note is judged, the value transitions between OnMin and OnMax depending on the laser cursor position; otherwise, the value is set to Off.

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
        - This parameter allows kson clients to use only the OnMin value and ignore the Off and OnMax values.
        - This parameter allows kson clients to ignore values specified in `audio.audio_effect.fx.long_event`.
        - Note: `update_period` interval count is reset at the beginning of each measure.
    - `wave_length` (length, default:`0`)
        - Length of repetition
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`.)
        - Note: `wave_length` interval count is reset at the beginning of each measure.
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
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`.)
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
    - (OPTIONAL SUPPORT) `vol` (rate, default:`75%`)
        - Volume of the effect audio
    - `mix` (rate, default:`0%>80%`)
        - Blending ratio of the original audio and the effect audio
- `pitch_shift`: This effect changes the pitch (key) of the audio.
    - `pitch` (pitch, default:`0`)
        - Pitch (key)
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
    - Note: `chunkSize` and `overWrap` (properly "overlap") parameters in KSH format has been removed in kson format because they assume that the audio waveform is processed in the time domain.
- `bitcrusher`: This effect reduces the quality of the audio wave. Also known as "Sample & Hold".
    - `reduction` (sample, default:`0samples-30samples`)
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
    - `freq_1` (freq, default:`1500Hz`)
        - First frequency of LFO
    - `freq_2` (freq, default:`20000Hz`)
        - Second frequency of LFO
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
    - Note: `freq_1` value may exceed the `freq_2` value.
    - Note: `hiCutGain` parameter in KSH format has been removed in kson format because it is not a parameter of the phaser itself.
- `wobble`: This effect oscillates the cutoff frequency of the low-pass filter with an LFO.
    - `wave_length` (length, default:`0`)
        - LFO period
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`.)
        - Note: `wave_length` interval count is reset at the beginning of each measure.
    - `freq_1` (freq, default:`500Hz`)
        - First frequency of LFO
    - `freq_2` (freq, default:`20000Hz`)
        - Second frequency of LFO
    - `q` (float, default:`1.414`)
        - Q value of the low-pass filter
        - Additional requirement:
            - 0.1 <= float <= 50.0
    - `mix` (rate, default:`0%>50%`)
        - Blending ratio of the original audio and the effect audio
    - Note: `freq_1` value may exceed the `freq_2` value.
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
        - This parameter allows kson clients to use only the OnMin value and just ignore the Off and OnMax values.
        - This parameter allows kson clients to ignore values specified in `audio.audio_effect.fx.long_event`.
        - Note: `update_period` interval count is reset at the beginning of each measure.
    - `wave_length` (length, default:`0`)
        - Length of repetition
        - `0`: Not specified. (In KSM, the effect is bypassed if the value is `0`.)
        - Note: `wave_length` interval count is reset at the beginning of each measure.
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
    - `ratio` (float, default:`1.0>5.0`)
        - Compression ratio
            - A value of `1.0` compresses the audio by a factor of 1/1 (i.e., the original audio), and a value of `5.0` compresses the audio by a factor of 1/5.
        - Additional requirement:
            - 1.0 <= float <= 100.0
- `switch_audio`: This effect switches the playback to another audio file.
    - `filename` (filename)
- `peaking_filter`: Peaking filter.
    - `v` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Note: This parameter is provided to make the frequency transition on a log (or log-like) scale rather than a linear scale.
    - (OPTIONAL SUPPORT) `freq` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 0.0
    - (OPTIONAL SUPPORT) `freq_max` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 1.0
    - (OPTIONAL SUPPORT) `bandwidth` (float, default:implementation-dependent)
        - Bandwidth around the cutoff frequency [oct]
    - `gain` (rate, default:`50%`)
        - Gain scale
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
    - Note: `freq` value may exceed the `freq_max` value.
- `high_pass_filter`: High-pass filter.
    - `v` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Note: This parameter is provided to make the frequency transition on a log (or log-like) scale rather than a linear scale.
    - (OPTIONAL SUPPORT) `freq` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 0.0
    - (OPTIONAL SUPPORT) `freq_max` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 1.0
    - (OPTIONAL SUPPORT) `q` (float, default:implementation-dependent)
        - Q value of the biquad filter
        - Additional requirement:
            - 0.1 <= float <= 50.0
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
    - Note: `freq` value may exceed the `freq_max` value.
- `low_pass_filter`: Low-pass filter.
    - `v` (rate, default:`0%-100%`)
        - Envelope value of the cutoff frequency
        - Note: This parameter is provided to make the frequency transition on a log (or log-like) scale rather than a linear scale.
    - (OPTIONAL SUPPORT) `freq` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 0.0
    - (OPTIONAL SUPPORT) `freq_max` (freq, default:implementation-dependent)
        - Cutoff frequency when `v` is 1.0
    - (OPTIONAL SUPPORT) `q` (float, default:implementation-dependent)
        - Q value of the biquad filter
        - Additional requirement:
            - 0.1 <= float <= 5.0
    - `mix` (rate, default:`0%>100%`)
        - Blending ratio of the original audio and the effect audio
    - Note: `freq` value may exceed the `freq_max` value.

-----------------------------------------------------------------------------------

## `camera`
```
dictionary CameraInfo {
    tilt: TiltInfo?        // tilt-related data
    cam:  CamInfo?         // cam-related data
}
```

### `camera.tilt`
```
dictionary TiltInfo {
    scale:  ByPulse<double>[] = [[0, 1.0]]    // tilt scale
    manual: ByPulse<GraphSectionPoint[]>[]?   // manual tilt
                                              // Note: The left laser being on the right edge is equal to a manual value of 1.0, and the right laser being on the left edge is equal to a manual value of -1.0.
                                              // Note: Two or more graph sections cannot be overlapped.
                                              // Note: "camera.tilt.scale" does not affect the scale of manual tilt. Manual tilt is always evaluated with a scale of 1.0.
    keep:   ByPulse<bool>[] = [[0, false]]    // whether tilt is kept or not
                                              // (while tilt is kept, the tilt amount value is updated only to a larger absolute value with the same sign)
}
```

### `camera.cam`
```
dictionary CamInfo {
    body:    CamGraphs?       // cam value changes
    pattern: CamPatternInfo?  // cam pattern
}
```

#### `camera.cam.body`
```
dictionary CamGraphs {
    zoom:               GraphPoint[]?  // zoom_bottom
    shift_x:            GraphPoint[]?  // zoom_side
    rotation_x:         GraphPoint[]?  // zoom_top
    rotation_z:         GraphPoint[]?  // rotation degree (affects both highway & jdgline relatively)
    rotation_z.highway: GraphPoint[]?  // (OPTIONAL SUPPORT) rotation degree (highway only)
    rotation_z.jdgline: GraphPoint[]?  // (OPTIONAL SUPPORT) rotation degree (judgment line only)
    center_split:       GraphPoint[]?  // center_split
}
```
- KSH: -300 - 300 => kson: -3.0 - 3.0
- Subparameters (`rotation_z.highway`/`rotation_z.jdgline`) affect the value relatively.
    - For example, the actual value of `rotation_z.highway` will be `rotation_z + rotation_z.highway`.
        - `rotation_z:1.0, rotation_z.highway:0.5` is equivalent to `rotation_z:0.0, rotation_z.highway:1.5, rotation_z.jdgline:1.0`

#### `camera.cam.pattern`
```
dictionary CamPatternInfo {
    laser: CamPatternLaserInfo?  // cam pattern for laser slams
}
```

##### `camera.cam.pattern.laser`
```
dictionary CamPatternLaserInfo {
    slam_event: CamPatternLaserInvokeList?  // cam pattern invocation by laser slam notes
}
```
- Note: This is a specification with the possibility of defining `camera.cam.pattern.laser.def` as a future extension.

##### `camera.cam.pattern.laser.slam_event`
```
dictionary CamPatternLaserInvokeList {
    spin:      CamPatternInvokeSpin[]?
    half_spin: CamPatternInvokeSpin[]?
    swing:     CamPatternInvokeSwing[]?  // (OPTIONAL SUPPORT)
}
```
- Note: `y` (pulse number) & `d` (laser slam direction) should be the same as `y` & sign(`vf` - `v`) of an existing laser slam note; otherwise, the event is ignored.

##### `camera.cam.pattern.laser.slam_event.spin[]`/`camera.cam.pattern.laser.slam_event.half_spin[]`
```
array CamPatternInvokeSpin {
    [0]: uint  // y: pulse number
    [1]: int   // direction: laser slam direction, -1 (left) or 1 (right)
    [2]: uint  // length: spin duration
}
```
- The array size of `CamPatternInvokeSpin` MUST be 3.

##### `camera.cam.pattern.laser.slam_event.swing[]` (OPTIONAL SUPPORT)
```
array CamPatternInvokeSwing {
    [0]: uint  // y: pulse number
    [1]: int   // direction: laser slam direction, -1 (left) or 1 (right)
    [2]: uint  // length: swing duration
    [3]: CamPatternInvokeSwingValue? // v: value
}
```
- The array size of `CamPatternInvokeSwing` MUST be 3 or 4.

##### `camera.cam.pattern.laser.slam_event.swing[][3]` (OPTIONAL SUPPORT)
```
dictionary CamPatternInvokeSwingValue {
    scale:  double = 1.0   // scale
    repeat: uint = 1       // number of repetitions
    decay_order: uint = 0  // order of the decay that scales camera values (0-2)
                           // (note that this decay is applied even if repeat=1)
                           // - equation: `value * (1.0 - ((l - ry) / l))^decay_order`
                           // - 0: no decay, 1: linear decay, 2: squared decay
}
```

-----------------------------------------------------------------------------------

## `bg`
```
dictionary BGInfo {
    filename: string?      // (OPTIONAL SUPPORT) filename of background graphics file
    offset:   int = 0      // (OPTIONAL SUPPORT) movie offset in millisecond
    legacy: LegacyBGInfo?  // (OPTIONAL SUPPORT)
}
```
- Note: The file format of `bg.filename` is not specified. If the format of the file specified in `bg.filename` is supported by the kson client, `bg.filename` is used; otherwise, it falls back to other built-in background graphics (which MAY be specified in `legacy`).
- Note: `bg.offset` is used only if supported by the BG file format and its player, the kson client.

### `bg.legacy` (OPTIONAL SUPPORT)
```
dictionary LegacyBGInfo {
    bg:    KSHBGInfo[2]?  // first element: when gauge < 70%, second element: when gauge >= 70%
    layer: KSHLayerInfo?
    movie: KSHMovieInfo?
}
```
- If `bg` has only a single element, that bg is always used, regardless of the percentage of the gauge.

#### `bg.legacy.bg[xxx]` (OPTIONAL SUPPORT)
```
dictionary KSHBGInfo {
    filename: string?  // self-explanatory (can be KSM default BG image such as "desert")
}
```

#### `bg.legacy.layer` (OPTIONAL SUPPORT)
```
dictionary KSHLayerInfo {
    filename: string?                // self-explanatory (can be KSM default animation layer such as "arrow")
    duration: int = 0                // one-loop duration in milliseconds
                                     //   If the value is negative, the animation is played backwards.
                                     //   If the value is zero, the play speed is tempo-synchronized and set to 1 frame per 0.035 measure (= 28.571... frames/measure).
    rotation: KSHLayerRotationInfo?  // rotation conditions
}
```

##### `bg.legacy.layer[xxx].rotation` (OPTIONAL SUPPORT)
```
dictionary KSHLayerRotationInfo {
    tilt: bool = true  // whether lane tilts affect rotation of BG/layer
    spin: bool = true  // whether lane spins affect rotation of BG/layer
}
```

#### `bg.legacy.movie` (OPTIONAL SUPPORT)
```
dictionary KSHMovieInfo {
    filename: string?  // self-explanatory
    offset:   int = 0  // movie offset in millisecond
}
```

-----------------------------------------------------------------------------------

### `editor` (OPTIONAL SUPPORT)
```
dictionary EditorInfo {
    comment: ByPulse<string>[]?  // (OPTIONAL SUPPORT) comments that can be written in the editor
}
```

-----------------------------------------------------------------------------------

### `compat` (OPTIONAL SUPPORT)
```
dictionary CompatInfo {
    ksh_version: string?          // (OPTIONAL SUPPORT) "ver" field of KSH file (specified only if converted from KSH)
    ksh_unknown: KSHUnknownInfo?  // (OPTIONAL SUPPORT) unrecognized data in ksh-to-kson conversion
}
```
- Note: If the "`ver`" field is not present in the KSH file, `ksh_version` is set to "`100`".

### `compat.ksh_unknown` (OPTIONAL SUPPORT)
```
dictionary KSHUnknownInfo {
    meta:   dictionary<string>?             // (OPTIONAL SUPPORT) unrecognized option lines before the first bar line
    option: dictionary<ByPulse<string>[]>?  // (OPTIONAL SUPPORT) unrecognized option lines after the first bar line
    line:   ByPulse<string>[]?              // (OPTIONAL SUPPORT) unrecognized non-option lines
}
```
- Note: In KSH format, a line with at least one "`=`" is recognized as an option line. The second or later "`=`" is recognized as part of the value.
    - Examples:
        - "`keyvalue`" => non-option line
        - "`key=value`" => option line (key:"`key`", value:"`value`")
        - "`key=value=value`" => option line (key:"`key`", value:"`value=value`")
- Note: Unrecognized non-option lines before the first bar line ("--") are stored in `compat.ksh_unknown.line` as `y` (pulse number) is `0`.
- Note: Since KSH format does not allow comment lines starting with "`;`", such lines are stored in `compat.ksh_unknown.option` or `compat.ksh_unknown.line` instead of `editor.comment`.
- Example:
    - KSH:
        ```
        title=...
        ;some-extension1
        extvalue=0
        --
        ;some-extension2
        extvalue=100
        0000|00|--
        extvalue=200
        0000|00|--
        --
        extvalue=300
        ;some-extension3
        ;some-extension4=100
        0000|00|--
        extvalue=400
        0000|00|--
        --
        ```
    - Converted `ksh_unknown` value in KSON:
        ```
        "ksh_unknown":{
            "meta":{
                "extvalue":"0"
            },
            "option":{
                "extvalue":[
                    [0, "100"],
                    [480, "200"],
                    [960, "300"],
                    [1440, "400"]
                ],
                ";some-extension4":[
                    [960, "100"]
                ]
            },
            "line":[
                [0, ";some-extension1"],
                [0, ";some-extension2"],
                [960, ";some-extension3"],
            ]
        }
        ```

-----------------------------------------------------------------------------------

### `impl` (OPTIONAL SUPPORT)
```
dictionary ImplInfo {
    // Not specified. This area is free for use by kson clients and kson editors.
    // To avoid conflicts with other clients, it is highly recommended to use a top-level object with a key that corresponds to the client name.
}
```

-----------------------------------------------------------------------------------

## Common objects

### event triggered by pulse
```
array ByPulse<T> {
    [0]: uint  // y: pulse number
    [1]: T     // v: value
}
```
- The array size of `ByPulse<T>` MUST be 2.

### event triggered by measure index
```
array ByMeasureIdx<T> {
    [0]: uint  // idx: measure index
    [1]: T     // v: value
}
```
- The array size of `ByMeasureIdx<T>` MUST be 2.

### graph value
```
array GraphValue {
    [0]: double  // v: value
    [1]: double  // vf: second value (for an immediate change)
}
```
- The array size of `GraphValue` MUST be 2.

### graph curve value (OPTIONAL SUPPORT)
```
array GraphCurveValue {
    [0]: double  // a: x-coordinate of the curve control point (0.0-1.0)
    [1]: double  // b: y-coordinate of the curve control point (0.0-1.0)
}
```
- The array size of `GraphCurveValue` MUST be 2.

### graph (whole chart)
```
array GraphPoint {
    [0]: uint                          // y: absolute pulse number
    [1]: double|GraphValue             // v: graph value; if double is used, v and vf are set to the same value
    [2]: GraphCurveValue = [0.0, 0.0]  // (OPTIONAL SUPPORT) curve: graph curve value
}
```
- The array size of `GraphPoint<T>` MUST be 2 or 3.

### graph point (for graph sections = `ByPulse<GraphSectionPoint[]>`)
```
array GraphSectionPoint {
    [0]: uint                          // ry: relative pulse number
    [1]: double|GraphValue             // v: graph value; if double is used, v and vf are set to the same value
    [2]: GraphCurveValue = [0.0, 0.0]  // (OPTIONAL SUPPORT) curve: graph curve value
}
```
- The array size of `GraphSectionPoint<T>` MUST be 2 or 3.

-----------------------------------------------------------------------------------

# Requirements

- Arrays of arrays that have `[0]` (a.k.a. `y`, `ry`, `idx`) must be ordered by `[0]`.

- The first point of arrays that have `ry` (e.g. the first point of `GraphSectionPoint[]`) must not have nonzero `ry`.

-----------------------------------------------------------------------------------

# Change Log

- `0.6.2` (04/22/2023)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/10/files
- [`0.6.1`](https://github.com/m4saka/ksm-chart-format-spec/blob/abca6466f1a5f429d024a82bf7cf96c5752702b5/kson_format.md) (03/12/2023)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/9/files
- [`0.6.0`](https://github.com/m4saka/ksm-chart-format-spec/blob/4df13dcb7114fefe05895556036dea6d20e617c1/kson_format.md) (12/10/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/8/files
- [`0.5.1`](https://github.com/m4saka/ksm-chart-format-spec/blob/47f4d942cedbf12bfc7fd5c26a61e081a98be24b/kson_format.md) (08/29/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/7/files
- [`0.5.0`](https://github.com/m4saka/ksm-chart-format-spec/blob/4a38a535d725f7a12d240cdbbad63de9421b9aa0/kson_format.md) (07/26/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/6/files
- [`0.4.0`](https://github.com/m4saka/ksm-chart-format-spec/blob/ece34cb84b55453fcda9e5b9b204194537344a12/kson_format.md) (07/02/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/5/files
- [`0.3.0`](https://github.com/m4saka/ksm-chart-format-spec/blob/c64b2e4613c8db8a458c53b7bb5dc7e1e4711fa2/kson_format.md) (05/21/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/4/files
- [`0.2.0`](https://github.com/m4saka/ksm-chart-format-spec/blob/bb88522d923a9051a71c1cd127f2db0d1fe20d0c/kson_format.md) (05/06/2022)
    - Changes: https://github.com/m4saka/ksm-chart-format-spec/pull/3/files
- [`0.1.0`](https://github.com/m4saka/ksh2kson/blob/8e9f39d5e93178dd306feaa890bcb43a6c4c9083/kson_format.md) (06/14/2020)
- [First draft](https://gist.github.com/m4saka/a89594a17dc9422d75e01998bcfd2722/e65ae6b6b1d424a14fa050c3825990fde494c688) (02/02/2019)

-----------------------------------------------------------------------------------

# Acknowledgements
This format is inspired by the BMSON format (https://bmson-spec.readthedocs.io/en/master/doc/index.html) and albshin's idea of kshon format (https://gist.github.com/albshin/cf535afc3f94f7d7f7c7e3d1d9ff41cf).

Thank you to all contributors and participants in the formatting discussions:  
@m4saka, @Drewol, @123jimin, @nashiora, @albshin, @TsFreddie

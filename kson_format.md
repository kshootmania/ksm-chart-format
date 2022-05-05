# KSON Format Specification (version: `0.2.0-beta16`)
- Encoding: UTF-8 (without BOM), LF
- If a default value is specified in this document, the undefined value is replaced by the default value. Note that `null` is not replaced by the default value.
- Supporting parameters/options marked "(OPTIONAL)" is optional in the kson client.

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
    ClientList?   impl;        // (OPTIONAL) implementation-dependent options for each client
}
```

-----------------------------------------------------------------------------------

## `meta`
```
dictionary MetaInfo {
    DOMString       title;                 // self-explanatory
    DOMString?      title_img_filename;    // (OPTIONAL) use an image instead of song title text
    DOMString?      title_translit;        // (OPTIONAL) transliterated title
    DOMString?      subtitle;              // (OPTIONAL) self-explanatory, can be a CD title or a music genre
    DOMString       artist;                // self-explanatory
    DOMString?      artist_img_filename;   // (OPTIONAL) use an image instead of song artist text
    DOMString?      artist_translit;       // (OPTIONAL) transliterated artist
    DOMString       chart_author;          // self-explanatory
    DifficultyInfo  difficulty;            // self-explanatory
    unsigned int    level;                 // self-explanatory, 1-20
    DOMString       disp_bpm = "";         // displayed bpm (allowed characters: 0-9, "-", ".")
    double          std_bpm = 0;           // (OPTIONAL) standard bpm for hi-speed values (should be between minimum bpm and maximum bpm in the chart); automatically set if zero
    DOMString?      jacket_filename;       // self-explanatory (can have a preset image "nowprinting1"/"nowprinting2"/"nowprinting3")
    DOMString?      jacket_author;         // self-explanatory
    DOMString?      information;           // (OPTIONAL) optional information shown in song selection
    DOMString?      lang;                  // (OPTIONAL) language (usually "ja" or "ko")
                                           // If the system language does not match this value, xxx_translit is used instead of xxx.
                                           // If not specified, the behavior depends on implementation (like xxx_translit is used only for English environment).
    DOMString?      ksh_version;           // (OPTIONAL) "ver" field of KSH file (specified only if converted from KSH)
}
```

### `meta.difficulty`
```
dictionary DifficultyInfo {
    DOMString?     name;        // (OPTIONAL) e.g. "Challenge", "Infinity", "Gravity", "Maximum", "FOUR DIMENSIONS"
    DOMString?     short_name;  // (OPTIONAL) e.g. "CH", "IN", "GRV", "MXM", "FD"
                                // (if not specified, it inherits the client defaults based on "idx". e.g. 1=>"CH")
    unsigned char  idx;         // 0-4  e.g. "CH"=>1, "IN"/"GRV"=>3, "MXM"/"FD"=>4
}
```

-----------------------------------------------------------------------------------

## `beat`
```
dictionary BeatInfo {
    ByPulse<double>[] bpm;                        // bpm changes
    ByMeasureIndex<TimeSig>[]? time_sig;          // time signature changes
                                                  // this is used for drawing bar lines and
                                                  // calculating duration of length parameters for audio effects
    ByPulse<GraphSectionPoint[]>[]? scroll_speed; // scroll speed changes (e.g. stop events)
    unsigned long resolution = 240;               // pulses per quarter note
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
    double? total;             // total ascension of gauge percentage in the entire chart (100-)
                               // set automatically if not specified
}
```

-----------------------------------------------------------------------------------

## `note`
```
dictionary NoteInfo {
    Interval[][4]? bt;               // BT notes (first index: lane) (l=0: chip note, l>0: long note)
    Interval[][2]? fx;               // FX notes (first index: lane) (l=0: chip note, l>0: long note)
    LaserSection[][2]? laser;        // laser notes (first index: lane (0: left knob, 1: right knob))
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
    BgmInfo? bgm;                               // bgm-related data
    KeySoundInfo? key_sound;                    // key-sound-related data
    AudioEffectInfo? audio_effect;              // audio-effect-related data
    LegacyAudioInfo? legacy;                    // (OPTIONAL) legacy data
}
```

### `audio.bgm`
```
dictionary BgmInfo {
    DOMString? filename;                        // self-explanatory
    double vol = 1.0;                           // bgm volume
    long offset = 0;                            // offset in milliseconds (starting point of the audio file)

    DOMString? preview_filename;                // specified only if another audio file is used for preview
    unsigned long preview_offset = 0;           // preview offset in milliseconds (starting point of the audio file)
    unsigned long preview_duration = 0;         // preview duration in milliseconds
}
```

### `audio.key_sound`
```
dictionary KeySoundInfo {
    KeySoundFXInfo fx;        // key sound for FX notes
    KeySoundLaserInfo laser;  // (OPTIONAL) key sound for laser slams
}
```
- Note: The key of `audio.key_sound.xxx.def` is the filename of a WAVE file (.wav).

### `audio.key_sound.fx`
```
dictionary KeySoundFXInfo {
    DefList<KeySound>? def;                           // key sound definitions
    InvokeList<ByPulse<KeySound>[][2]>? chip_event;  // key sound invocation by chip FX notes
}
```
- Note: `audio.key_sound.laser.chip_event` should be the same as `y` of an existing chip FX note on the corresponding lane, otherwise the event is ignored.

### `audio.key_sound.laser` (OPTIONAL)
```
dictionary KeySoundLaserInfo {
    DefList<KeySound>? def;                         // key sound definitions
    InvokeList<ByPulse<KeySound>[]>? slam_event;   // key sound invocation by laser slam notes
}
```
- Note: `audio.key_sound.laser.slam_event` should be the same as `y` of an existing laser slam, otherwise the event is ignored.
- Note: `slam` is predefined in the kson client and is invoked in default.
- (OPTIONAL) Note: `slam_up` & `slam_down` & `slam_swing` are predefined by kson clients.

#### `audio.key_sound.xxx.def`
```
dictionary Def<KeySound> {
    // parameters
    KeySound? v {
        double vol = 1.0;
    };
}
```

### `audio.audio_effect`
```
dictionary AudioEffectInfo {
    AudioEffectFXInfo? fx;
    AudioEffectLaserInfo? laser;
}
```

#### `audio.audio_effect.fx`
```
dictionary AudioEffectFXInfo {
    DefList<AudioEffect>? def;                           // audio effect definitions
    InvokeList<ByPulse<AudioEffect>[]>? param_change;    // audio effect parameter changes by pulse
    InvokeList<ByPulse<AudioEffect>[][2]>? long_event;  // audio effect invocation (and parameter changes) by FX notes
}
```
- Note: `audio.audio_effect.fx.long_event[].y` should be the same as `y` of an existing long FX note on the corresponding lane, otherwise the event is ignored.

#### `audio.audio_effect.laser`
```
dictionary AudioEffectLaserInfo {
    DefList<AudioEffect>? def;                         // audio effect definitions
    InvokeList<ByPulse<AudioEffect>[]>? param_change;  // audio effect parameter changes by pulse
    InvokeList<ByPulse<AudioEffect>[]>? pulse_event;  // audio effect invocation (and parameter changes) by pulse
}
```

##### `audio.audio_effect.xxx.def`
```
dictionary Def<AudioEffect> {
    DOMString type;      // audio effect type (e.g. "flanger")
    AudioEffect? v {
        // Custom fields for each audio effect type
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
                "delay": [
                    {
                        "y": 0,
                        "v": 80.0
                    }
                ],
                "depth": [
                    {
                        "y": 0,
                        "v": 30.0
                    }
                ],
                "depth.on": [
                    {
                        "y": 0,
                        "v": 40.0
                    }
                ],
                "depth.on.max": [
                    {
                        "y": 0,
                        "v": 60.0
                    }
                ]
           }
       }
       ```

    2. "`type=TapeStop;trigger=off>on;speed=20%`" in KSH
       ```
       {
           "type":"tapestop",
           "v":{
                "trigger": [
                    {
                        "y": 0,
                        "v": 0.0
                    }
                ],
                "trigger.on": [
                    {
                        "y": 0,
                        "v": 1.0
                    }
                ],
                "speed": [
                    {
                        "y": 0,
                        "v": 0.2
                    }
                ]
           }
       }
       ```

    3. "`type=Retrigger;waveLength=100ms`" in KSH
       ```
       {
           "type":"retrigger",
           "v":{
                "wave_length": [
                    {
                        "y": 0,
                        "v": 100.0
                    }
                ],
                "wave_length@tempo_sync": [
                    {
                        "y": 0,
                        "v": 0.0
                    }
                ]
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
- Audio effects in the "Audio effects & parameter list" are predefined with its default value.


#### Audio effect parameter types

Values of all these types are set by a floating-point value.

- length
    - Duration which can be tempo-synced
    - Fields:
        - `@tempo_sync` (bool)
            - `true`: the value is a measure value (e.g. `0.0625` means 1/16th)
            - `false`: the value is a millisecond value (e.g. `10.0` means 10ms)
- sample
    - Short duration
        - The value is the number of samples, where the sampling rate is 44100Hz
        - For example, `44100` means 1.0s
    - The value must be between `0` and `44100`
- bool
    - Regarded as `true` if the value > `0.0`
    - Regarded as `false` if the value <= `0.0`
        - Note that the value can be negative, and the negative value is regarded as `false`
- rate
    - Ratio value
        - For example, `0.1` means 10%
    - Usually the value is between `0.0` and `1.0`
        - The value can exceed `1.0` in some parameters (e.g. `flanger.vol`)
- freq
    - Frequency value (Hz)
    - The value must be between `10.0` and `20000.0`
- dB
    - Decibel value (dB)
- pitch
    - Key in music
        - `12.0` means one octave
    - The value must be between `-48.0` and `48.0`
    - Fields:
        - (OPTIONAL) `@quantize` (bool)
            - `true`: the value is ceiled (e.g. `12.9` -> `12.0`, `-1.1` -> `-2.0`)
            - `false`: the value is not ceiled
- int
    - Integer value
        - the value is ceiled (e.g. `12.9` -> `12.0`, `-1.1` -> `-2.0`)
- float
    - Floating-point value


#### Audio effects & parameter list

- `retrigger`
    - `update_period` (length, default:`0.5`)
    - `update_period@tempo_sync` (bool, default:`1.0`)
    - `wave_length` (length, default:`0.0`)
    - `wave_length@tempo_sync` (bool, default:`1.0`)
    - `rate` (rate, default:`0.7`)
    - `update_trigger` (bool, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `gate`
    - `wave_length` (length, default:`0.0`)
    - `wave_length@tempo_sync` (bool, default:`1.0`)
    - `rate` (rate, default:`0.6`)
    - `mix` (rate, default:`.off=0.0, .on=0.9`)
- `flanger`
    - `period` (length, default:`2.0`)
    - `period@tempo_sync` (bool, default:`1.0`)
    - `delay` (sample, default:`30.0`)
    - `depth` (sample, default:`45.0`)
    - `feedback` (rate, default:`0.6`)
    - `stereo_width` (rate, default:`0.0`)
    - (OPTIONAL) `vol` (rate, default:`0.75`)
    - `mix` (rate, default:`.off=0.0, .on=0.8`)
- `pitch_shift`
    - `pitch` (pitch, default:`0.0`)
    - (OPTIONAL) `pitch@quantize` (bool, default:`1.0`)
    - (OPTIONAL) `chunk_size` (sample, default:`700.0`)
    - (OPTIONAL) `overlap` (rate, default:`0.4`)
        - Note: This parameter is described as `overWrap` in KSH format, but it's a spelling mistake.
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `bitcrusher`
    - `reduction` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `phaser`
    - `period` (length, default:`0.5`)
    - `period@tempo_sync` (bool, default:`1.0`)
    - `stage` (int, default:`6.0`)
        - This parameter must be an even number.
    - `lo_freq` (freq, default:`1500.0`)
    - `hi_freq` (freq, default:`20000.0`)
    - `q` (float, default:`0.707`)
    - `feedback` (rate, default:`0.35`)
    - `stereo_width` (rate, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=0.5`)
        - `hiCutGain` parameter in KSH format has been deleted because it is not a parameter of "Phaser" itself.
- `wobble`
    - `wave_length` (length, default:`0.0`)
    - `wave_length@tempo_sync` (bool, default:`1.0`)
    - `lo_freq` (freq, default:`500.0`)
    - `hi_freq` (freq, default:`20000.0`)
    - `q` (float, default:`1.414`)
    - `mix` (rate, default:`.off=0.0, .on=0.5`)
- `tapestop`
    - `speed` (rate, default:`0.5`)
    - `trigger` (bool, default:`.off=0.0, .on=1.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `echo`
    - `update_period` (length, default:`0.0`)
    - `update_period@tempo_sync` (bool, default:`1.0`)
    - `wave_length` (length, default:`0.0`)
    - `wave_length@tempo_sync` (bool, default:`1.0`)
    - `update_trigger` (bool, default:`.off=0.0, .on=1.0`)
    - `feedback_level` (rate, default:`1.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `sidechain`
    - `period` (length, default:`0.25`)
    - `period@tempo_sync` (bool, default:`1.0`)
    - `hold_time` (length, default:`50.0`)
    - `hold_time@tempo_sync` (bool, default:`0.0`)
    - `attack_time` (length, default:`10.0`)
    - `attack_time@tempo_sync` (bool, default:`0.0`)
    - `release_time` (length, default:`0.0625`)
    - `release_time@tempo_sync` (bool, default:`1.0`)
    - `ratio` (float, default:`.off=1.0, .on=5.0`)
        - The type of this parameter has been changed from "int" to "float" (in KSH format).
- `switch_audio`
    - (`filename` (string))
        - Note that this is not in `v` but at the root of `Def<AudioEffect>`
    - (OPTIONAL) (`fallback` (string))
        - Note that this is not in `v` but at the root of `Def<AudioEffect>`
- `high_pass_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=1.0`)
        - This parameter determines an envelope value of the cutoff frequency.
            - The cutoff frequency = `lo_freq` + `env` * (`hi_freq`-`lo_freq`)
            - This parameter is set to `0.0-1.0` in default.
        - We need this parameter because the transition of cutoff frequency should be in a log scale instead of a linear scale.
            - If you simply set `freq.on.min=100, freq.on.max=20000`, the cutoff frequency moves in a linear scale, which causes an insufficient low-frequency transition.
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - (OPTIONAL) `delay` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `low_pass_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=1.0`)
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - (OPTIONAL) `delay` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `peaking_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=1.0`)
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `gain` (dB, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - (OPTIONAL) `delay` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)

### Audio effect subparameters

- Each audio effect parameter has `xxx.on` subparameter that is used when the audio effect is activated by notes.
    - `xxx.on` has `xxx.on.max` subparameter that is used when left laser cursor is at the rightmost (or right laser cursor is the leftmost).
        - For long BT/FX notes, `xxx.on.max` is just ignored.
    - Note that specifying parent parameter affects all the subparameters.
        - If a subparameter value is specified, it overrides the parent parameter value.
            - For example, if you specify `xxx` and `xxx.on.max`, first all values of `xxx`, `xxx.on` and `xxx.on.max` will be filled with the specified `xxx` value, then the value of `xxx.on.max` will be replaced with the specified `xxx.on.max` value.
        - Examples:
            - ```
              "xxx":[{"y":0, "v":100}]
              ```
              is identical to 
              ```
              "xxx":[{"y":0, "v":100}],
              "xxx.on":[{"y":0, "v":100}],
              "xxx.on.max":[{"y":0, "v":100}]
              ```
            - ```
              "xxx":[{"y":0, "v":100}],
              "xxx.on":[{"y":0, "v":200}]
              ```
              is identical to 
              ```
              "xxx":{"y":0, "v":100},
              "xxx.on":{"y":0, "v":200},
              "xxx.on.max":{"y":0, "v":200}
              ```
            - ```
              "xxx":[{"y":0, "v":100}],
              "xxx.on.max":[{"y":0, "v":200}]
              ```
              is identical to 
              ```
              "xxx":{"y":0, "v":100},
              "xxx.on":{"y":0, "v":100},
              "xxx.on.max":{"y":0, "v":200}
              ```

### `audio.legacy` (OPTIONAL)
```
dictionary LegacyAudioInfo {
    LegacyAudioBgmInfo? bgm;
}
```

### `audio.legacy.bgm` (OPTIONAL)
```
dictionary LegacyAudioBGMInfo {
    DOMString[]? fp_filenames;  // filenames of prerendered BGM with audio effects from legacy KSH charts
                                // e.g. [ "xxx_f.ogg", "xxx_p.ogg", "xxx_fp.ogg" ]
}
```

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
    ByPulse<double>[]? scale;                 // tilt scale (default: 1.0); i.e., normal/bigger/biggest tilt in KSH format
    ByPulse<GraphSectionPoint[]>[]? manual;   // manual tilt
                                              // Note: The left laser being on the right edge is equal to 1.0, and the right laser being on the left edge is equal to -1.0.
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
    GraphPoint[]? rotation_z;                     // rotation degree  (affects both lane & jdgline relatively)
    GraphPoint[]? rotation_z.lane;                // (OPTIONAL) rotation degree  (lane only)
    GraphPoint[]? rotation_z.jdgline;             // (OPTIONAL) rotation degree  (judgment line only)
    GraphPoint[]? center_split;                   // center_split
}
```
- KSH: -300 - 300 => kson: -3.0 - 3.0

#### `camera.cam.pattern`
```
dictionary CamPatternInfo {
    CamPatternLaserInfo laser;                    // cam pattern for laser slams
}
```

##### `camera.cam.pattern.laser`
```
dictionary CamPatternLaserInfo {
    DefList<CamPattern>? def;                        // cam pattern definitions
    InvokeList<ByPulse<CamPattern>[]>? slam_event;  // cam pattern invocation by laser slam notes
}
```

#### `camera.cam.pattern.laser.def`
```
dictionary Def<CamPattern> {
    // body (unchangeable in invocation)
    CamPatternGraphs body {
        GraphSectionRelPoint[]? zoom;                // zoom_bottom
        GraphSectionRelPoint[]? shift_x;             // zoom_side
        GraphSectionRelPoint[]? rotation_x;          // zoom_top
        GraphSectionRelPoint[]? rotation_z;          // rotation degree  (affects both lane & jdgline relatively)
        GraphSectionRelPoint[]? rotation_z.lane;     // rotation degree  (lane only)
        GraphSectionRelPoint[]? rotation_z.jdgline;  // rotation degree  (judgment line only)
        GraphSectionRelPoint[]? center_split;        // center_split
    };
    
    // standard duration
    //   if a different value is set in invocation.v.l, Def<CamPattern>.body.ry will be scaled
    //       ry = Def<CamPattern>.body.ry * invocation.v.l / Def<CamPattern>.l
    unsigned long l;

    // parameters (changeable in invocation, the default values are only used in Def)
    CamPattern v {
        unsigned long? l;                         // duration
        double scale = 1.0;                       // scale
        unsigned long repeat = 1;                 // number of repeat
        double repeat_scale = 1.0;                // rate of the current "scale" to that of the previous repeat
                                                  // (set "-1.0" to generate a loop like a sine wave)
        double decay_order = 0;                   // order of the decay that scales camera values
                                                  // (note that this decay is applied even if repeat=1)
                                                  // - equation: `value * (1.0 - ((l - y) / l))^decay_order`
                                                  // - examples
                                                  //     0: no decay, 1: linear decay, 2: squared decay, negative: increasing
    };
}
```
- "laser_slam" & "spin" & "half_spin" & "swing" are predefined by kson clients.
    - Predefined pattern example:
      ```
      {
          "name":"swing",
          "body":{
              "shift_x":[
                  {"ry":0, "rv":0, "a":1.0, "b":0.59}
                  {"ry":480, "rv":1.0, "a":0.0, "b":0.59}
                  {"ry":960, "rv":0}
              ]
          },
          "v":{
              "l":960,
              "repeat_scale":-1.0
          }
      }
      ```
- Unlike `audio.audio_effect`, subparameters (like `rotation_z.lane`/`rotation_z.jdgline`) affect the value relatively.
    - For example, the actual value of `rotation_z.lane` will be `rotation_z + rotation_z.lane`.
        - ```
          "rotation_z":[{"y":0, "v":1.0}],
          "rotation_z.lane":[{"y":0, "v":0.5}]
          ```
          is identical to 
          ```
          "rotation_z.lane":[{"y":0, "v":1.5}],
          "rotation_z.jdgline":[{"y":0, "v":1.0}]
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
    KshBg[]? bg;        // first index: when gauge < 70%, second index: when gauge >= 70%
    KshLayer[]? layer;  // first index: when gauge < 70%, second index: when gauge >= 70%
    KshMovie? movie;
}
```
- If the array of bg/layer has only one item, that bg/layer is always used regardless of the gauge percentage.
- If the array of layer has only one item and the layer image has two rows in one image, the first row (upper-half) is used when gauge < 70%, and the second row (lower-half) is used when gauge > 70%.

#### `bg.legacy.bg[xxx]` (OPTIONAL)
```
dictionary KshBg {
    DOMString filename = "desert";  // self-explanatory (can be KSM default BG image such as "`desert`")
    KshRotationInfo rotation = {"tilt":true, "spin":false};  // rotation conditions
}
```

#### `bg.legacy.layer[xxx]` (OPTIONAL)
```
dictionary KshLayer {
    DOMString filename = "arrow";  // self-explanatory (can be KSM default animation layer such as "`arrow`")
    long duration = 0;             // one-loop duration in milliseconds
                                   //   If the value is negative, the animation is played backwards.
                                   //   If the value is zero, the play speed is tempo-synced and set to 1f/0.035measure (=28.571f/measure).
    KshRotationInfo rotation = {"tilt":true, "spin":true};  // rotation conditions
}
```

#### `bg.legacy.bg[xxx].rotation` / `bg.legacy.layer[xxx].rotation` (OPTIONAL)
```
dictionary KshRotationInfo {
    bool? tilt;  // whether lane tilts affect rotation of BG/layer
    bool? spin;  // whether lane spins affect rotation of BG/layer
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

## `impl` (OPTIONAL)
```
// Client list using implementation-dependent options
dictionary ClientList {
    ImplInfo? ksm2; // Just an example
    ...
}
```

### `impl.xxx` (OPTIONAL)
```
dictionary ImplInfo {
    // Not specified. This area is free for use by kson clients and kson editors.
    // To avoid conflicts with other clients, it is recommended to have a top object whose key is the client name.
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
dictionary ByPulse<T> {
    unsigned long y;          // pulse number
    T? v;                     // body
}
```

### graph (whole chart)
```
dictionary GraphPoint {
    unsigned long y;          // absolute pulse number
    double v;                 // value
    double? vf;               // second value (for an immediate change)
    double a = 0.0;           // x-coordinate of the curve control point (0.0-1.0)
    double b = 0.0;           // y-coordinate of the curve control point (0.0-1.0)
}
```

### graph point (for graph sections = `ByPulse<GraphSectionPoint[]>`)
```
dictionary GraphSectionPoint {
    unsigned long ry;        // relative pulse number
    double v;                // value
    double? vf;              // second value (for an immediate change)
    double a = 0.0;          // x-coordinate of the curve control point (0.0-1.0)
    double b = 0.0;          // y-coordinate of the curve control point (0.0-1.0)
}
```

```
dictionary GraphSectionRelPoint {
    unsigned long ry;        // relative pulse number
    double rv;               // relative value
    double? rvf;             // second relative value (for an immediate change)
    double a = 0.0;          // x-coordinate of the curve control point (0.0-1.0)
    double b = 0.0;          // y-coordinate of the curve control point (0.0-1.0)
}
```

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
This specification is initially based on the BMSON format (https://bmson-spec.readthedocs.io/en/master/doc/index.html).

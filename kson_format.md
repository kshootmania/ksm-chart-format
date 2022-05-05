# KSON Format Specification (version: `0.1.0`)
- Encoding: UTF-8 (without BOM), LF
- If the default value is specified, omitted value or `null` will be replaced with it.

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
    ClientList?   impl;        // implementation-dependent options for each client
}
```

-----------------------------------------------------------------------------------

## `meta`
```
dictionary MetaInfo {
    DOMString       title;                 // self-explanatory
    DOMString?      title_translit;        // transliterated title
    DOMString?      subtitle;              // self-explanatory, can be a CD title or a music genre
    DOMString       artist;                // self-explanatory
    DOMString?      artist_translit;       // transliterated artist
    DOMString       chart_author;          // self-explanatory
    DifficultyInfo  difficulty;            // self-explanatory
    unsigned int    level;                 // self-explanatory, 1-20
    DOMString?      disp_bpm;              // displayed bpm (allowed characters: 0-9, "-", ".")
                                           // If not specified, the first value of BeatInfo.bpm will be used.
    double?         std_bpm;               // standard bpm for hi-speed values (should be between minimum bpm and maximum bpm in the chart)
    DOMString?      jacket_filename;       // self-explanatory (can have a preset image "nowprinting1"/"nowprinting2"/"nowprinting3")
    DOMString?      jacket_author;         // self-explanatory
    DOMString?      information;           // optional information shown in song selection
    DOMString?      lang;                  // language (usually "ja" or "ko")
                                           // If the system language does not match this value, xxx_translit is used instead of xxx.
                                           // If not specified, the behavior depends on implementation (like xxx_translit is used only for English environment).
    LegacyMetaInfo? legacy;
    DOMString?      ksh_version;           // "ver" field of KSH file (specified only if converted from KSH)
}
```

### `meta.difficulty`
```
dictionary DifficultyInfo {
    DOMString?     name;        // e.g. "Challenge", "Infinity", "Gravity", "Maximum", "FOUR DIMENSIONS"
    DOMString?     short_name;  // e.g. "CH", "IN", "GRV", "MXM", "FD"
                                // (if not specified, it inherits the client defaults based on "index". e.g. 1=>"CH")
    unsigned char  idx;         // 0-4  e.g. "CH"=>1, "IN"/"GRV"=>3, "MXM"/"FD"=>4
}
```

### `meta.legacy`
```
dictionary LegacyMetaInfo {
    DOMString? title_img_filename;   // use an image instead of song title text
    DOMString? artist_img_filename;  // use an image instead of song artist text
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
    Interval[][]? bt;               // BT notes (first index: lane) (l=0: chip note, l>0: long note)
    Interval[][]? fx;               // FX notes (first index: lane) (l=0: chip note, l>0: long note)
    LaserSection[][]? laser;        // laser notes (first index: lane (0: left knob, 1: right knob))
}
```

### `note.laser[lane][idx]`
```
dictionary LaserSection : ByPulse<GraphSectionPoint[]> {
    unsigned long y;                // pulse number
    GraphSectionPoint[] v;          // laser points (0.0-1.0)
    unsigned char wide = 1;         // 1-2, sets whether this laser section is 2x-widen or not
}
```

-----------------------------------------------------------------------------------

## `audio`
```
dictionary AudioInfo {
    BgmInfo? bgm;                               // bgm-related data
    KeySoundInfo? key_sound;                    // key-sound-related data
    AudioEffectInfo? audio_effect;              // audio-effect-related data
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
    DefList<KeySound>? def;                         // key sound definitions
    InvokeList<ByPulse<KeySound>[]>? pulse_event;   // key sound invocation by pulse
    InvokeList<ByNotes<KeySound>>? note_event;      // key sound invocation by notes
}
```

#### `audio.key_sound.def`
```
dictionary Def<KeySound> {
    // key sound filename (unchangeable in invocation)
    DOMString filename;

    // parameters (changeable in invocation, the default values are only used in Def)
    KeySound? v {
        double vol = 1.0;  // standard volume, this value is multiplied if specified in invocation
                           //     vol = Def<KeySound>.v.vol * invocation.v.vol
    };
}
```

### `audio.audio_effect`
```
dictionary AudioEffectInfo {
    DefList<AudioEffect>? def;                          // audio effect definitions
    InvokeList<ByPulse<AudioEffect>[]>? pulse_event;    // audio effect invocation (parameter changes) by pulse
    InvokeList<ByNotes<AudioEffect>>? note_event;       // audio effect invocation (parameter changes) by notes
}
// Note: In AudioEffectInfo.note_event.laser, the laser points inherit the audio effect type of the previous point.
//       But this is not applied beyond a section.
//       Note that the default audio effect (peaking filter) is set to the first point of every section as default.
```

#### `audio.audio_effect.def`
```
dictionary Def<AudioEffect> {
    DOMString type;      // name of the audio effect to inherit (e.g. "flanger")
    AudioEffect? v {
        // Custom fields for each audio effect type
    };
    DOMString? filename; // can be specified only if type="audio_swap"
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
                "depth.off": [
                    {
                        "y": 0,
                        "v": 30.0
                    }
                ],
                "depth.on.min": [
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
                "trigger.off": [
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
           "type":"audio_swap",
           "filename":"music.ogg" // not in "v" because "filename" is not changeable in invocation
       }
       ```
- No audio effects are predefined in default.
    - You need to define all audio effects used in kson explicitly.


### Audio effect parameter types

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
        - `@quantize` (bool)
            - `true`: the value is ceiled (e.g. `12.9` -> `12.0`, `-1.1` -> `-2.0`)
            - `false`: the value is not ceiled
- int
    - Integer value
        - the value is ceiled (e.g. `12.9` -> `12.0`, `-1.1` -> `-2.0`)
- float
    - Floating-point value


### Audio effects & parameter list

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
    - `vol` (rate, default:`0.75`)
    - `mix` (rate, default:`.off=0.0, .on=0.8`)
- `pitch_shift`
    - `pitch` (pitch, default:`0.0`)
    - `pitch@quantize` (bool, default:`1.0`)
    - `chunk_size` (sample, default:`700.0`)
    - `overlap` (rate, default:`0.4`)
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
        - The type of this parameter has been changed to "float" from "int" (in KSH format).
- `audio_swap`
    - (`filename` (filename))
        - Note that this is not in `v` but at the root of `Def<AudioEffect>`
- `high_pass_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=0.0`)
        - This parameter determines an envelope value of the cutoff frequency.
            - The cutoff frequency = `lo_freq` + `env` * (`hi_freq`-`lo_freq`)
            - This parameter is set to `0.0-1.0` in default.
        - We need this parameter because the transition of cutoff frequency should be in a log scale instead of a linear scale.
            - If you simply set `freq.on.min=100, freq.on.max=20000`, the cutoff frequency moves in a linear scale, which causes an insufficient low-frequency transition.
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - `delay` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `low_pass_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=0.0`)
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - `delay` (sample, default:`0.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)
- `peaking_filter`
    - `env` (rate, default:`.off=0.0, .on.min=0.0, .on.max=0.0`)
    - `lo_freq` (freq, default:??? `/*FIXME*/`)
    - `hi_freq` (freq, default:??? `/*FIXME*/`)
    - `gain` (dB, default:??? `/*FIXME*/`)
    - `q` (float, default:??? `/*FIXME*/`)
    - `delay` (sample, default:`50.0`)
    - `mix` (rate, default:`.off=0.0, .on=1.0`)

### Audio effect subparameters

- Each audio effect parameter has `xxx.on`/`xxx.off` subparameters used when the audio effect is active/inactive.
    - `xxx.on` has `xxx.on.min`/`xxx.on.max` subparameters used when left laser cursor is at left/right (or right laser cursor is right/left).
        - For long BT/FX notes, `xxx.on.min` is used.
    - Note that specifying parent parameter affects all the subparameters.
        - If a lower level of subparameter is specified, it has a higher priority of values.
            - If `xxx` and `xxx.on.max` is specified, firstly the values of all subparameters are filled with the specified value of `xxx`, then the value of `xxx.on.max` is replaced with the specified value.
        - Examples:
            - ```
              "xxx.on":[{"y":0, "v":100}]
              ```
              is identical to 
              ```
              "xxx.on.min":[{"y":0, "v":100}],
              "xxx.on.max":[{"y":0, "v":100}]
              ```
            - ```
              "xxx":[{"y":0, "v":100}],
              "xxx.on.max":[{"y":0, "v":200}]
              ```
              is identical to 
              ```
              "xxx.on.min":{"y":0, "v":100},
              "xxx.on.max":{"y":0, "v":200},
              "xxx.off":{"y":0, "v":100}
              ```

### `audio.legacy`
```
dictionary LegacyAudioInfo {
    bool laser_slam_auto_vol = false;  // whether to scale the sound volume considering the width of the laser slam
    LegacyAudioBgmInfo? bgm;
}
```

### `audio.legacy.bgm`
```
dictionary LegacyAudioBgmInfo {
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
    ByPulse<GraphSectionPoint[]>[]? manual;   // manual tilt (-1.0 - 1.0)
    Interval[]? keep;                         // tilt keep
                                              // (while a tilt keep event, the value is only updated to larger absolute values with the same sign)
}
```

### `camera.cam`
```
dictionary CamInfo {
    CamGraphs body;                               // cam value changes
    CamGraphs? tilt_assign;                       // tilt=>cam value translation scales (for NORMAL/BIGGER/BIGGEST tilt)
    CamPatternInfo pattern;                       // cam pattern
}
```
- The default value of "tilt_assign" is `{"rotation_z":[{"y":0, "v":14.0}]}`

#### `camera.cam.pattern`
```
dictionary CamPatternInfo {
    DefList<CamPattern>? def;                        // cam pattern definitions
    InvokeList<ByPulse<CamPattern>[]>? pulse_event;  // cam pattern invocation by pulse
    InvokeList<ByNotes<CamPattern>>? note_event;     // cam pattern invocation by notes
}
```

#### `camera.cam.body` / `camera.cam.tilt_assign`
```
dictionary CamGraphs {
    GraphPoint[]? zoom;                           // zoom_bottom
    GraphPoint[]? shift_x;                        // zoom_side
    GraphPoint[]? rotation_x;                     // zoom_top
    GraphPoint[]? rotation_z;                     // rotation degree  (affects both lane & jdgline relatively)
    GraphPoint[]? rotation_z.lane;                // rotation degree  (lane only)
    GraphPoint[]? rotation_z.jdgline;             // rotation degree  (judgment line only)
}
```
- KSM: -300 - 300 => kson: -3.0 - 3.0

#### `camera.cam.pattern.def`
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
    };

    // parameters (changeable in invocation, the default values are only used in Def)
    CamPattern v {
        unsigned long l;                          // standard duration
                                                  // if a differrent value is set in invocation.v.l, Def<CamPattern>.body.ry will be scaled
                                                  //     ry = Def<CamPattern>.body.ry * invocation.v.l / Def<CamPattern>.v.l
        double scale = 1.0;                       // standard scale (normally only specified in invocation)
                                                  //     scale = invocation.v.scale / Def<CamPattern>.v.scale
                                                  // (set this value to -1.0 for right-to-left laser slams)
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
    LegacyBgInfo? legacy;
}
```

<details>
<summary>
Previously-proposed BG specification here
</summary>

## `bg`
```
dictionary BgInfo {
    LayerInfo? layer;  // layer-related data
    MovieInfo? movie;  // movie-related data
}
```

### `bg.layer`
```
dictionary LayerInfo {
    DefList<Layer>? def;                        // layer definitions
    InvokeList<ByPulse<Layer>[]>? pulse_event;  // layer invocation (parameter changes) by pulse
    InvokeList<ByNotes<Layer>>? note_event;     // layer invocation (parameter changes) by notes
}
```

#### `bg.layer.def`
```
dictionary Def<Layer> {
    // image files
    //     one file: use one image regardless of the gauge
    //     two files: use second image when gauge >= 70%
    ImageFile image[] {
        // image filename
        DOMString filename;

        // size of one frame in pixels
        // (if not specified, the entire image is regarded as one frame)
        unsigned long? w;
        unsigned long? h;

        unsigned long frame;   // number of frames

        // offset position in pixels
        unsigned long offset_x = 0;
        unsigned long offset_y = 0;

        // relative pixels of the next frame
        unsigned long dx = w;
        unsigned long dy = 0;
    }

    // parameters (changeable in invocation, the default values are only used in Def)
    Layer? v {
        bool visible = true;         // self-explanatory
        long l = 0;                  // pulses for one loop of the animation
                                     // (if the value is set to 0, the animation stops at the current frame)
        double frame = 0.0;          // frame index (this parameter means "frame offset" in Def and "seeking to a frame" in invocation)
                                     // Note: the value is rounded down to an integer when it is used (e.g. 5.7 => 5),
                                     //       but the real value is considered in transition
                                     // Note: modulo value is used if the frame index is out of range ("frame" is the value already rounded down)
                                     //     (frame index) = frame % (number of frames)                        (if frame >= 0)
                                     //     (frame index) = frame % (number of frames) + (number of frames)   (if frame < 0)
        double x = 0.0;              // x-axis position (0 = center, positive = right)
        double y = 0.0;              // y-axis position (0 = center, positive = up)
        double z = 0.0;              // order of drawing layers (smaller = higher priority)
                                     //     negative values: in front of the lane
                                     //     positive values (or 0.0): behind the lane
        double opacity = 1.0;        // self-explanatory (0-1)
        double rotation = 0.0;       // rotation in degrees
        DOMString mode = "normal";   // blend mode ("normal"/"additive")
    };

    // layer pattern
    LayerPatternInfo pattern;
}
```

#### `bg.layer.def.pattern`
```
dictionary LayerPatternInfo {
    DefList<LayerPattern> def;                  // layer pattern definitions
    InvokeList<ByPulse<Layer>[]>? pulse_event;  // layer pattern invocation by pulse
    InvokeList<ByNotes<Layer>>? note_event;     // layer pattern invocation by notes
}
```

#### `bg.layer.def.pattern.def`
```
dictionary Def<LayerPattern> {
    // body (unchangeable in invocation)
    LayerPatternGraphs body {
        GraphSectionRelPoint[]? l;
        GraphSectionRelPoint[]? frame;
        GraphSectionRelPoint[]? x;
        GraphSectionRelPoint[]? y;
        GraphSectionRelPoint[]? z;
        GraphSectionRelPoint[]? opacity;
        GraphSectionRelPoint[]? rotation;
    };

    // parameters (changeable in invocation, the default values are only used in Def)
    LayerPattern v {
        unsigned long l;                          // standard duration
                                                  // if a differrent value is set in invocation.v.l, Def<LayerPattern>.body.ry will be scaled
                                                  //     ry = Def<LayerPattern>.body.ry * invocation.v.l / Def<LayerPattern>.v.l
        double scale = 1.0;                       // standard scale (normally only specified in invocation)
                                                  //     scale = invocation.v.scale / Def<LayerPattern>.v.scale
        unsigned long repeat = 1;                 // number of repeat
        double repeat_scale = 1.0;                // rate of the current "scale" to that of the previous repeat
                                                  // (set "-1.0" to generate a loop like a sine wave)
        double decay_order = 0;                   // order of the decay that scales layer values
                                                  // (note that this decay is applied even if repeat=1)
                                                  // - equation: `value * (1.0 - ((l - y) / l))^decay_order`
                                                  // - examples
                                                  //     0: no decay, 1: linear decay, 2: squared decay, negative: increasing
    };
}
```

### `bg.movie`
```
dictionary MovieInfo {
    DOMString filename;              // self-explanatory
    long offset = 0;                 // offset in milliseconds (starting point of the video file)
}
```

</details>

### `bg.legacy`
```
dictionary LegacyBgInfo {
    KshBg[] bg;        // first index: when gauge < 70%, second index: when gauge >= 70%
    KshLayer[] layer;  // first index: when gauge < 70%, second index: when gauge >= 70%
    KshMovie movie;
}
```
- If the array of bg/layer has only one item, that bg/layer is always used regardless of the gauge percentage.
- If the array of layer has only one item and the layer image has two rows in one image, the first row (upper-half) is used when gauge < 70%, and the second row (lower-half) is used when gauge > 70%.

#### `bg.legacy.bg[xxx]`
```
dictionary KshBg {
    DOMString filename;         // self-explanatory (can be KSM default BG image such as "`desert`")
    KshRotationInfo? rotation;  // rotation conditions
}
```

#### `bg.legacy.layer[xxx]`
```
dictionary KshLayer {
    DOMString filename;         // self-explanatory (can be KSM default animation layer such as "`arrow`")
    long duration;              // one-loop duration in milliseconds
    KshRotationInfo? rotation;  // rotation conditions
}
```

#### `bg.legacy.bg[xxx].rotation` / `bg.legacy.layer[xxx].rotation`
```
dictionary KshRotationInfo {
    bool tilt;  // whether lane tilts affect rotation of BG/layer
    bool spin;  // whether lane spins affect rotation of BG/layer
}
```

#### `bg.legacy.movie`
```
dictionary KshMovie {
    DOMString filename;  // self-explanatory
    long offset;         // movie offset in millisecond
}
```

-----------------------------------------------------------------------------------

## `impl`
```
// Client list using implementation-dependent options
dictionary ClientList {
    ImplInfo? ksm2; // Just an example
    ...
}
```

### `impl.xxx`
```
dictionary ImplInfo {
    any?     meta;    // meta data
    any?     beat;    // beat-related data
    any?     gauge;   // gauge-related data
    any?     note;    // note-related data
    any?     audio;   // audio-related data
    any?     camera;  // camera-related data
    any?     bg;      // background-related data
    any?     other;   // others
}
```
- This allows you to use undefined options in kson spec..
- If known parameters in kson spec. is set, these parameters are forked by clients.
    - The replacement affects each parameter
        - e.g. even if "note":{} is set, the notes will not be deleted
    - But an array is regarded as a parameter
        - e.g. if "note":{"bt":[]} is set, the BT notes will be deleted,
          and this will not affect the other lanes

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

### events triggered by BT/FX/laser notes
```
dictionary ByNotes<T> {
    ByBtnNote<T>[]? bt;       // events triggered by BT notes
    ByBtnNote<T>[]? fx;       // events triggered by FX notes
    ByLaserNote<T>[]? laser;  // events triggered by laser notes
}
```

### event triggered by BT/FX note
```
dictionary ByBtnNote<T> {
    unsigned long lane;     // lane index (corresponds to the 1st index of NoteInfo.bt/fx)
    unsigned long idx;      // note index (corresponds to the 2nd index of NoteInfo.bt/fx)
    T? v;                   // body
    bool dom = true;        // domination flag (only in audio.audio_effect.note_event)
                            // (true = allow only one audio effect whose note is pressed later, like SDVX
                            //  false = allow multiple audio effects to be activated simultaneously, like KSH)
}
```

### event triggered by laser point
```
dictionary ByLaserNote<T> {
    unsigned long lane;     // lane index (corresponds to the 1st index of NoteInfo.laser)
    unsigned long sec;      // section index (corresponds to the 2nd index of NoteInfo.laser)
    unsigned long idx;      // point index (corresponds to the index of LaserSection.point)
    T? v;                   // body
    bool dom = true;        // domination flag (only in audio.audio_effect.note_event)
                            // (true = allow only one audio effect whose knob has larger value is activated, like SDVX & KSH
                            //  false = allow multiple audio effects to be activated simultaneously)
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
                              // (exception: v.l & v.scale & v.vol are multiplied by values in an invocation)
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

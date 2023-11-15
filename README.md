# AdVent Database

This is a database for [AdVent](https://github.com/denis-stepanov/advent), the TV ads arrestor. For a description of how AdVent works, refer to its repository. This repository contains a database of TV jingle hashes. These are used by AdVent in order to decide when the ads sound has to be cut.

## Database v20231115

```
(advent-pyenv) $ db-djv-pg dbinfo
Dejavu database info:
  Fingerprinted / total tracks = 283 / 283
  Peak groups                  = 86855 (avg. ~= 307 per track)
  Fingerprints                 = 2353797 (avg. ~= 8317 per track)
  Total fingerprinted time    ~= 1341 s (avg. ~= 4.7 s per track)
  Database size               ~= 319 MB (avg. ~= 1.13 MB per track)
  Fingerprinting frequency    ~= 1755 Hz (~= 3.98% of sampling frequency 44100 Hz)
  Hash size                    = 10 B
  Hash collisions             ~= 35.9%
  First update                ~= 2023-05-06 00:40:23
  Last update                 ~= 2023-11-15 23:41:12
  Last vacuum                 ~= 2023-11-15 23:41:20

AdVent database info:
  Countries                    = 1
  TV channels                  = 12 (avg. ~= 12 per country)
  Jingles                      = 283 (avg. ~= 24 per TV channel)
  Pure entry / entry jingles   = 73 / 195
  Pure exit / exit jingles     = 88 / 210
  No action jingles            = 0
  Time coverage from           = 2022-02-05
  Time coverage till           = 2023-11-09
(advent-pyenv) $
```

## Database Population or Update for Regular Users

It is assumed that you have [installed AdVent](https://github.com/denis-stepanov/advent#installation). Make sure to switch to its virtual environment:

```
$ source advent-pyenv/bin/activate
```

where `advent-pyenv` is the folder created during AdVent installation.

To pull the latest updates into your database:

1. Download the latest snapshot (clone, pull or get a zip):

```
(advent-pyenv) $ git clone https://github.com/denis-stepanov/advent-db.git
(advent-pyenv) $ cd advent-db/DB
```

2. (recommended) Delete countries or TV channels you are not expected to watch. This will decrease CPU load and reduce the number of false positives while running AdVent:

```
(advent-pyenv) $ rm -r VA # ....
```

3. Import the update:

```
(advent-pyenv) $ db-djv-pg import -s .
```

Import might take a few minutes (depending on how much your database is out of sync).

## If You Want to Create Your Own Hashes, Read Further

You will need:

1. [installed](https://github.com/denis-stepanov/advent#installation) and [configured](https://github.com/denis-stepanov/advent#audio-inputs) AdVent (only sound capturing part; TV control module is not required);
2. a basic audio editor capable of track trimming (plus some familiarity with it).

 Make sure to switch to AdVent virtual environment:

```
$ source advent-pyenv/bin/activate
```

where `advent-pyenv` is the folder created during AdVent installation.

Example below is given for Linux Fedora, AdVent configured to listen to PulseAudio output (see ["Capturing a TV Webcast"](https://github.com/denis-stepanov/advent#capturing-a-tv-web-cast)) and [Audacity](https://www.audacityteam.org/) as sound editor.

### Step 1: Record TV Audio Containing Ads

As far as possible, stick to the same audio source that you will be using when running AdVent. Usually the simplest way not requiring messing up with cables is to record a portion of a TV broadcast from Internet. If you have got a choice between analog and digital recording, always prefer digital. Stereo sources are OK and even preferred (Dejavu will treat them as two-in-one).

Preferred audio format to produce is PCM (WAV), 16 bit low endian signed, 44.1 kHz, 2 channels (stereo). Many audio tools will produce this by default. Other formats or parameters have not been tested and may or may not work.

If you configured AdVent for PulseAudio input, you should be OK to record with default settings:

```
(advent-pyenv) $ parecord -v 6ter_20220725.wav
Opening a recording stream with sample specification 's16le 2ch 44100Hz' and channel map 'front-left,front-right'.
Connection established.
Stream successfully created.
Buffer metrics: maxlength=4194304, fragsize=352800
Using sample spec 's16le 2ch 44100Hz', channel map 'front-left,front-right'.
Connected to device alsa_output.pci-0000_00_1b.0.analog-stereo.monitor (index: 45, suspended: no).
Time: 4.608 sec; Latency: 608164 usec.
...
(stop with Ctrl-C)
(advent-pyenv) $
```

You can also use recorder of your choice, including Audacity. The file name is not important at this point; here it gives a hint of a channel and of a date of recording.

### Step 2: Single Out a Jingle of Interest

Load the recording into Audacity: `File` > `Open...`. Seek through the track to locate the ad jingle; use zoom if needed. Select the desired interval by clicking in the track area on the jingle start and dragging mouse to the jingle end. Whenever possible, keep your jingle record at least two seconds long; anything below one second would not work and shall be avoided. Take a note of the jingle position (before or after the ads).

![Selecting a jingle in Audacity](https://user-images.githubusercontent.com/22733222/184440958-9c4f2b8a-0f34-4633-8c2a-840dbe57ff9f.png)

At this point is it recommended to test the jingle with AdVent to make sure that you are not looking at the piece already known. Run AdVent in console, then press `Play` in Audacity.

```
(advent-pyenv) $ advent -t nil
AdVent v1.5.0
TV control is nil with action 'mute' for 600 s max
Recognition interval is 2 s with confidence of 10%
Started 2 listening thread(s)
....:::oooo:o.::ooooo:.......
(Ctrl-C twice or Ctrl-\)
(advent-pyenv) $ 
```

If AdVent does not detect your jingle during playback (no "hit" printed), you are good to continue. Trim the track: `Edit` > `Remove Special` > `Trim Audio`; then shift the result to the beginning: `Tracks` > `Align Tracks` > `Start to Zero`. Export the track: `File` > `Export` > `Export as WAV`. Give it a name as per naming convention [described below](#jingle-naming-convention) (in this case it would be something like `FR_6TER_220725_ELEMENTARY1_1.wav`); leave the encoding `Signed 16-bit PCM` (default). No need to fill any metadata; just click `OK`.

### Step 3: Generate a Hash

Fingerprint the jingle using standard Dejavu approach:

```
(advent-pyenv) $ dejavu -f . wav
Fingerprinting all .wav files in the . directory
Fingerprinting channel 1/2 for ./FR_6TER_220725_ELEMENTARY1_1.wav
Finished channel 1/2 for ./FR_6TER_220725_ELEMENTARY1_1.wav
Fingerprinting channel 2/2 for ./FR_6TER_220725_ELEMENTARY1_1.wav
Finished channel 2/2 for ./FR_6TER_220725_ELEMENTARY1_1.wav
(advent-pyenv) $ 
```

It is recommended (but not required) to keep already processed jingle WAV files around in the case fingerprinting needs to be reexecuted. You can re-submit the entire folders many times; Dejavu will recognize and skip records which have already been submitted.

(optional) At this point you can undo your changes in Audacity (press Ctrl-Z twice), run AdVent again and press `Play` in Audacity; AdVent should now recognize the track:


```
(advent-pyenv) $ advent -t nil
AdVent v1.5.0
TV control is nil with action 'mute' for 600 s max
Recognition interval is 2 s with confidence of 10%
Started 2 listening thread(s)
....:::oO
Hit: FR_6TER_220725_ELEMENTARY1_1
TV muted
OOOo...
(Ctrl-C twice or Ctrl-\)
(advent-pyenv) $ 
```

Export the hash from the database for future use:

```
(advent-pyenv) $ db-djv-pg export FR_6TER_220725_ELEMENTARY1_1
FR_6TER_220725_ELEMENTARY1_1
(advent-pyenv) $ 
```

(optional) Check the number of fingerprints generated for the jingle:

```
(advent-pyenv) $ wc -l FR_6TER_220725_ELEMENTARY1_1.djv
13351 FR_6TER_220725_ELEMENTARY1_1.djv
(advent-pyenv) $ 
```

A good jingle might have 5000 or more fingerprints. Unfortunately, you do not have much freedom here, as the number of fingerprints would depend on the length of the jingle and of its sound "richness". Jingles with fewer than 500 fingerprints should not be let into a shared database, as they would often result in false positives. Jingles with 500 - 1000 fingerprints are in the risk zone.

If you consider that your hashes could be of use for others, please submit a pull request. Make sure you follow the folder structure defined in this repository and the file naming conventions.

## Jingle Naming Convention

```
11_2..2_333333_4..4_5..5.ext
FR_TF1_220214_EVENING1_3.djv
```

1. [ISO 3166](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) two letter country code
2. TV channel name
3. Jingle capture date YYMMDD (approximate if unsure)
4. Jingle name (free format, alphanumeric)
5. Binary flags in decimal
   1. 0x1 - Jingle starts the ads (1 if unsure)
   2. 0x2 - Jingle ends the ads (1 if unsure)
   3. ...
6. File extension indicates recognizer provider
   1. djv - [Dejavu](https://github.com/denis-stepanov/dejavu)
   
If a jingle can be seen both at entry and exit, the flags would be `0x1 & 0x2 = 0x3` = decimal 3 (i.e., just 3 in the name).

Unless you have a specific reason, it is recommended to use capital letters only and to avoid any special characters (spaces, apostrophs, punctuation...).

## Jingle Hash File Format (.djv)

Variable CSV (comma-separated) format is used.

The first line describes the format:
```
<format>,<format_version>
```
The only supported format is `djv`. Current format version is `1`. Backward compatibility (reading files of older formats) is supported; forward compatibility (reading files of newer formats) is not supported.

The second line describes the jingle:
```
<name>,<fingerprinted>,<file_hash>,<num_fingerprints>
```
`name` is the full name of the jingle, as [defined above](#jingle-naming-convention) (omitting file extension). It is expected (but not required) that the name as stored in the file corresponds to the file name. `fingerprinted` is a flag `0`/`1` from the Dejavu database; it should read `1` in all regular AdVent usage scenarios. `file_hash` is a SHA1 hash of the audio file once submitted to Dejavu. `num_fingerprints` is a number of fingerprints generated for the jingle.

The remaining lines are individual fingerprints. Their number should correspond to the number of fingerprints defined.
```
<offset>,<hash>
```
`offset` is a fingerprint offset inside the jingle. `hash` is a fingerprint itself. It is normal to have several fingerprints for the same offset. The order of the fingerprint lines in the file is not important; for reproducibility of export they are ordered on save.

Example of a file `FR_6TER_220725_ELEMENTARY1_1.djv`:
```
djv,1
FR_6TER_220725_ELEMENTARY1_1,1,72e621563a21a344ead619ef6bfe14fa6a2d219c,13349
0,1d65a451ac6be35206b8
0,2b4157cae1e958cfe6e7
(.. 13347 more lines ..)
```

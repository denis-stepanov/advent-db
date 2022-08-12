# AdVent Database

This is a database for [AdVent](https://github.com/denis-stepanov/advent), the TV ads arrestor. For a description of how AdVent works, refer to its repository. This repository contains a database of TV jingle hashes. These are used by AdVent in order to decide when the ads sound has to be cut.

## Database Population or Update for Regular Users

It is assumed that you have [installed AdVent](https://github.com/denis-stepanov/advent#installation). To pull the latest updates into your database:

1. Download the latest snapshot (clone, pull or get a zip):

```
$ git clone https://github.com/denis-stepanov/advent-db.git
$ cd advent-db/DB
```

2. (recommended) delete countries or TV channels you are not expected to use. This will decrease CPU load and reduce the number of false positives while running AdVent:

```
$ rm -r VA # ....
```

3. Update or import your database:

```
(advent-pyenv) $ find . -name "*.djv" | xargs db-djv-pg import
```

If your want to overwrite existing definitions (slower), add `-o` to the end of the import command.

## If You Want to Create Your Own Hashes, Read Further

You will need:
1. Means of recording TV audio (note that depending on your country and TV provider this might be considered illegal)
2. A basic audio editor capable of track trimming (plus some familiarity with it)

Process:

### Step 1: Record TV Audio
As far as possible, stick to the same audio source that you will be using when running AdVent. If you have got a choice between analog and digital recording, always prefer digital. Stereo sources are OK (DejaVu will treat them as two-in-one).

Usually the simplest way not requiring messing up with cables is to record a portion of a TV broadcast from Internet. On Linux, one can [create a PulseAudio monitor device](https://wiki.archlinux.org/title/PulseAudio/Examples#Monitor_specific_output). Once this is done, you can use any recorder to capture sound (like a very basic KDE Recorder).

### Step 2: Single Out a Jingle of Interest
Load the recording into an audio editor (like Audacity), locate your ads jingle, trim it and export. Once located, you can run AdVent in background to confirm that it does not recognize your new jingle. Make sure your jingle record is between 2 and 5 seconds long (below 2 it might not be detected reliably; above 5 it would be detected with certainty, so no need to waste computer resources). If a jingle is surrounded with silent periods, you can include short parts of those before and after in order to improve detection. Take a note of jingle position (before or after the ads). Export the result into a WAV file and give it a name as per naming convention described below.

### Step 3: Generate a Hash
Fingerprint the jingle using standard DejaVu approach:
```
$ dejavu.py -f <path-to-jingle-folder> wav
```
It is recommended to keep already processed jingles around in the case fingerprinting needs to be reexecuted. DejaVu will recognize and skip records which have already been submitted.

Export the hash for future use:
```
$ db-djv-pg.py export <jingle-name>
```
where `jingle-name` is a name defined in step 2.

If you consider that your hashes could be of use for others, please submit a pull request. Make sure you follow the naming conventions.

## Jingle Naming Convention

```
11_2..2_333333_4..4_5..5.ext
FR_TF1_220214_EVENING1_0.djv
```

1. [ISO 3166](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) two letter country code
2. TV channel name
3. Jingle capture date YYMMDD (approximate if unsure)
4. Jingle name (free format, alphanumeric)
5. Binary flags in hex
   1. 0x1 - Jingle starts the ads (0 if unknown)
   2. 0x2 - Jingle ends the ads (0 if unknown)
   3. ...
6. File extension indicates recognizer provider
   1. djv - [Dejavu](https://github.com/denis-stepanov/dejavu)

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

Example of a file `FR_TF1_220205_EVENING1_2.djv`:
```
djv,1
FR_TF1_220205_EVENING1_2,1,8cf22931d2f8cbfb9439fc0eb6a123bb679b6b16,227
6,138160992563d4b07bb7
6,541d5e0da358554a10da
(.. 225 more lines ..)
```

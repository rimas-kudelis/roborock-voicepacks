# Audio Generator for Xiaomi Vacuum Generation 1 & Generation 2

# Author: Dennis Giese [dennis@dontvacuum.me]
# Copyright 2017 by Dennis Giese
# Rooted gen3 support copyright 2026 by Rimas Kudelis

# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.

# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

It is possible to create new language files which can be integrated into the rooted firmware. To generate the language files you can use the `generate_audio.py` script in this folder.

It will read the content of audio_xx.csv (whereby xx is a country language code like de_de or en_US) and use one of the supported engines to generate mp3 files for inclusion in the firmware. The language codes must be in [ISO-639-1](https://en.wikipedia.org/wiki/ISO_639-1) format, so that the engine can use a voice with suitable pronounciation.

You'll notice that each engine has its problems with mixed langauges (e.g. german with english words in it). A little testing is necessary to find a suitable pronunciation.


# Requirements
1. python3
1. [pipenv](https://github.com/pypa/pipenv) Install using: `pip install pipenv`
1. ffmpeg (to convert generated files into wav)

The `generate_audio.py` script can also generate files for newer Roborock robots, like S7, which encrypt their voice packs differently. If you want to use this functionality, there are two additional requirements:
1. OggEnc (to convert audio files to OGG format)
1. mksquashfs (to generate SquashFS filesystem image)

See also notes below.

# Installation
You can either install the python3 requirements manually or you install and use them with pipenv:

* install ffmpeg
* change to root folder of repository (where is placed Pipfile) and run:
* `pipenv install`
* `pipenv shell`
* `cd devices/xiaomi.vacuum/audio_generator`
* start script with `./generate_audio.py` program ask for your language selection

# Supported speech engines
## gtts (Google Text To Speech)
* https://github.com/pndurette/gTTS
## espeak (eSpeak NG Text-to-Speech)
* https://github.com/espeak-ng/espeak-ng
## macos (Mac OS X integrated Text To Speech)
* https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/say.1.html
* You may specify a voice directly after the say command in the documentation. You can get a complete list from the command `say -v ?`. If you do not specify a different voice with `-v VoiceName` Mac OS will use your current system language
## aws (Amazon Polly)
* https://docs.aws.amazon.com/polly/latest/dg/what-is.html

# Gen3 robot (e.g. Roborock S7) support

The following information is based on some experimentation on a Roborock S7.

The list of voice messages on this vacuum is slightly different from the ones listed under `language/audio_en.csv` (there are several new messages, and some old ones have been removed or revised), so a separate directory `language_s7/` for these messages has been created. This is not necessarily final though.

The `generate_audio.py` script currently doesn't know how to generate voice packs that would be accepted by the robot firmware and installable in the usual way. This newer voice pack file format hasn't been cracked yet, so, instead of building an installable voice pack, this script builds a SquashFS image which can be written directly onto the block device which is mounted as `/mnt/resources/audio_custom` in the robot (which is `/dev/nandj` on **my** S7).

To install it, you have to:
1. First and foremost: make sure there is enough space for the image the block device you plan to write it to. One way to do that is to `dd` the contents of said device to a file under `/tmp`, check the size of that file, and compare its size to that of the SquashFS image. The size of the image must be smaller or same.
1. Transfer the image file to the robot (you can do that by starting a web server on your box and using `wget` to download it onto the robot).
1. Unmount the filesystem which is mounted as `/mnt/resources/audio_custom` in the robot.
1. Write the contents of the image file to the block device you just unmounted using `dd`.
1. Mount the newly written filesystem and see if all of this worked.

Note: Valetudo will report the language of these language packs as "??". It looks like the language tag is inferred from the number specified in `/mnt/resources/audio_custom/sounds/sound.info`, but probably not by Valetudo itself. I haven't tested whether these unsupported voice packs work on a rooted robot without Valetudo. 

# speech_to_phonemes

# Description
This repository contains example data to use with the [speech_to_phonemes image](https://hub.docker.com/r/crimca/speech_to_phonemes).
The speech_to_phonemes service consists in a phonetic transcriber, which can train a model that can transcribe
an audio speech recording to a time-aligned list of IPA phonemes. The model can be trained for any language, as long as
the user provides the required data.

# Installation
The service is run directly from a Docker image.

You can pull the [speech_to_phonemes](https://hub.docker.com/r/crimca/speech_to_phonemes) Docker image from DockerHub.

# Usage
The service can be executed in two different modes : the training or the decoding mode.

### Training mode :
Trains a phonetic transcriber model, which can be reused in the decoding mode.
The data required for training must be formatted in .eaf files ([ELAN](https://archive.mpi.nl/tla/elan) Annotation Format).
Optional text (.txt) files can also be used to improve the model.
The user must also provide a g2p table (grapheme-to-phoneme table) according to the language being processed.

usage : 
```
docker run speech_to_phonemes:0.1.0 train 
                --train_dir TRAIN_DIR --dev_dir DEV_DIR
                [--wav_dir WAV_DIR] [--model_output_dir MODEL_OUTPUT_DIR]
                --g2p G2P [--skip_tokens SKIP_TOKENS [SKIP_TOKENS ...]]
                [--number_jobs NUMBER_JOBS]
```

#### Required arguments :
`train_dir` : Directory containing the training data, which should consist of .eaf files and optional text (.txt) files.
Make sure to only put files used as training data in this directory. 

`dev_dir` : Directory containing the testing data, which should consist of .eaf files and optional text (.txt) files.
Make sure to only put files used as testing data in this directory.

`g2p` : Conversion table that translates a grapheme to its corresponding IPA phoneme(s). The file should be formatted
with one definition per line, where a grapheme is followed by a list of space-separated IPA phonemes.

Example : `dʑ d ʑ` where `dʑ` is the grapheme and `d ʑ` is the list of corresponding phonemes.

For more example on the data format, refer to the data provided on this repository.

#### Optional arguments :
`wav_dir` : Directory containing all the audio data, which should consist of the wav files associated with the
input eaf files. This directory should contain both the train and dev audio files. The association between
an eaf file and its audio file will be deduced from the MEDIA_DESCRIPTOR info found in the eaf file. The
wav path will either be found from the relative path found in the descriptor combined with the wav_dir
argument path, or, if no relative path is found, the wav filename, found in the absolute path, will be used
to look for the file directly in the wav_dir path. 

If this argument is not included, the absolute path found in the eaf file's descriptor will be used to find the wav file.

`model_output_dir` : Directory where the output of the model training will be sent. This output directory will contain
the resulting model, which is necessary for the decoding mode. The default directory used is './kaldi_scripts', which is a
path relative to the main executed source code. This directory should be mounted to a volume when executing the Docker
image in order to be able to reuse the output for the decoding mode.

`skip_tokens` : A list of space separated string tokens of one or more characters that will be removed from the text data
before processing it for training.

`number_jobs` : If multiple cores are available, the data can be split in the number_jobs defined here to do processing in
parallel with Kaldi. (Default = 1, which represents no data splitting)


### Decoding mode :
Uses a model obtained from the training mode to transcribe an audio file to an IPA phonetic transcription.
The output will be saved into an eaf file.

usage : 
```
docker run speech_to_phonemes:0.1.0 decode 
                [-s SEGMENTS_FILENAME] [--model_dir MODEL_DIR]
                [--output_dir OUTPUT_DIR] [--number_jobs NUMBER_JOBS]
                wav_file
```

#### Required argument :
`wav_file` : Path to the wav file that will be transcribed to IPA phonemes.

#### Optional arguments :
`s` (segments) : Path to a file containing a list of segments. The file must use the Kaldi syntax for segments
files : 
`<speaker_id> <wav_filename> <segment_start_time> <segment_end_time>`
Providing a segments file will skip the Voice Activity Detection (VAD) step, which would create a list of segments where 
speech is detected.

`model_dir` : Path where the model used for decoding the audio file is found. This path is equivalent to the
path of the argument 'model_output_dir' found in the training mode, which is the path where the training model is created.
The default directory used is './kaldi_scripts', which is a path relative to the main executed source code.

`output_dir` : Output path where the resulting eaf file will be saved. If not provided will default to the directory
of the input wav file.

`number_jobs` : If multiple cores are available, the data can be split in the number_jobs defined here to do processing in
parallel with Kaldi.(Default = 1, which represents no data splitting)

# Data
The data found on this repository comes from the [sltu_corpora repository](https://github.com/gw17/sltu_corpora). 
The transcription files have been converted to .eaf files ([ELAN](https://archive.mpi.nl/tla/elan) Annotation Format)
and divided in train/dev sets. The original data comes from the [Pangloss collection](https://pangloss.cnrs.fr/).

# Credits
Researcher : Gilles Bouliane

Developer : Charles-William Cummings

# License
Copyright 2021 Centre de recherche informatique de Montréal (CRIM)

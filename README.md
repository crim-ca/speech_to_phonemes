# speech_to_phonemes

# Description
This repository contains example data to use with the [speech_to_phonemes docker image](https://hub.docker.com/r/crimca/speech_to_phonemes).
The speech_to_phonemes service consists in a phonetic transcriber that can be trained to transcribe
an audio speech recording to a time-aligned list of IPA phonemes. Its goal is to allow someone to automatically
do phonetic transcription of large amounts of audio recordings from a single
speaker, starting with only a small amount of already transcribed recordings
from the same speaker. The model can be trained for any language, as long as
the user provides the required data.
More details can be found in the paper: [Phoneme transcription of endangered languages: an evaluation of recent
ASR architectures in the single speaker scenario](https://aclanthology.org/2022.findings-acl.180).

# Installation
The service is run directly from a Docker image.

You can pull the [speech_to_phonemes](https://hub.docker.com/r/crimca/speech_to_phonemes) Docker image from DockerHub.

# Usage
The service can be executed in two different modes : the training or the decoding mode.
In training mode, the model is trained and then evaluated on a separate
development set, and optionnally saved for later use.
In decoding mode, a saved model is applied to transcribe audio recordings.

### Training mode :
Trains a phonetic transcriber model, which can be reused in the decoding mode.
The data required for training must be formatted in .eaf files ([ELAN](https://archive.mpi.nl/tla/elan) Annotation Format).
Optional text (.txt) files can also be used to improve the model.
The user must also provide a g2p table (grapheme-to-phoneme table) according to the language being processed.

usage :
```
docker run crimca/speech_to_phonemes:0.1.0 train
                --train_dir TRAIN_DIR --dev_dir DEV_DIR
                [--wav_dir WAV_DIR] [--model_output_dir MODEL_OUTPUT_DIR]
                --g2p G2P [--skip_tokens SKIP_TOKENS [SKIP_TOKENS ...]]
                [--number_jobs NUMBER_JOBS]
```

Note that `train_dir`, `dev_dir`, etc. are directories internal to the running
docker container. Use the docker --volume option to mount host directories so
they are visible inside the container (see step-by-step example below).

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
docker run crimca/speech_to_phonemes:0.1.0 decode
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

## Step-by-step example

1. Clone this repo and go to its root.

```
cd speech_to_phonemes
```

2. Select a language by its id.

```
lid=mlv
```

3. Run training and evaluation.

```
docker run -v `pwd`/data:/mnt/wdir crimca/speech_to_phonemes:0.1.0 train \
                                --train_dir /mnt/wdir/$lid/train_$lid \
                                --dev_dir /mnt/wdir/$lid/dev_$lid \
                                --wav_dir /mnt/wdir/$lid \
                                --g2p /mnt/wdir/$lid/g2p_$lid/roman_to_ipa.g2p \
                                --model_output_dir /mnt/wdir/model_$lid
```

The `--model_output_dir` option is used to save the model after training, but it is optional.
In this example, the model, along with detailed logs and intermediate results, will be saved in `data/model_$lid`.
If the option is omitted, the resulting files will be saved only on the container, 
in the default path `/stp/speech_to_phonemes/kaldi_scripts`.

Processing logs will be sent to standard output. In the last few lines, `%WER` is the phoneme error rate computed on the development set
(using Kaldi's compute-wer).

4. To apply the saved model to some audio recordings, use the decode mode:

```
docker run -v `pwd`/data:/mnt/wdir crimca/speech_to_phonemes:0.1.0 decode \
                                --model_dir /mnt/wdir/model_$lid \
                                /mnt/wdir/$lid/dev_$lid/147721_record_22km.101.wav
```

The file will be segmented into speaking turns and the phoneme transcription will be saved as an ELAN .eaf file in the same directory as the wave file.


# Data
The data for seven languages found in this repository comes from the [sltu_corpora repository](https://github.com/gw17/sltu_corpora), which originally comes from the [Pangloss collection](https://pangloss.cnrs.fr/). The data for the remaining languages has been downloaded directly from the [Pangloss collection](https://pangloss.cnrs.fr/) 

All transcription files have been converted to .eaf files ([ELAN](https://archive.mpi.nl/tla/elan) Annotation Format) and divided in train/dev sets.  

**List of provided data, with their code and their corresponding language :**

**From the [sltu_corpora repository](https://github.com/gw17/sltu_corpora) :**

* ers : Duoxo
* lif : Limbu
* mkd : Nashta
* mlv : Mwotlap
* nep : Dotyal
* nru / nru33 : Yongnin Na
* tvk : Vatlongo

**Directly from the [Pangloss collection](https://pangloss.cnrs.fr/):**

* abz : Abzakh 
* bhj : Bahing
* ers / ers2 : Ersu
* hiw : Hiw
* kkt : Koyi Rai 
* lrz : Lemerig 
* msn : Mwesen
* nee : Nêlêmwa-nixumwak
* nmk : Namakura
* olr : Olrat
* slr : Salar
* swb : Maore
* tdh : Thulung Rai
* tkw : Teanu
* wls : East Uvean

For each language, training and development sets of .eaf files and corresponding audio files are provided. A simple grapheme-to-phoneme (g2p) table is provided to convert text annotations found in the .eaf files to phonemes. 

# Results: 
The following section presents the phoneme error rate (%PER) for languages in both datasets, ordered by decreasing amount of audio used in training. The average gives equal weight to every language.

### Results for the dataset from the [sltu corpora repository](https://github.com/gw17/sltu_corpora) :

For each language of this dataset, training was done with 10 different random train/test partitions and the Student's t 95% uncertainty intervals were computed and presented in the %PER section.  


| Language<br>code | IPA    | Audio<br>(minutes) | %PER<br>HMM-GMM |
| ---------------- | -----  | ------------------ | --------------- |
| nru33            | True   | 151                | 19.3 ± 1.1      |
| lif              | True   | 99                 | 30.2 ± 0.9      |
| nep              | False  | 95                 | 62.0 ± 1.7      |
| ers              | True   | 29                 | 45.8 ± 1.7      |
| mkd              | True   | 23                 | 53.1 ± 3.0      |
| mlv              | False  | 20                 | 28.8 ± 2.6      |
| tvk              | False  | 13                 | 57.2 ± 3.6      |
| **Average**      |        | **61.4**           | **42.1 ± 3.7**  |


### Results for the dataset extracted directy from the [Pangloss collection](https://pangloss.cnrs.fr/) :

For each language of this dataset, training was done with a single train/set partition.

| Language<br>code | IPA   | Audio<br>(minutes) | %PER<br>HMM-GMM |
| ---------------- | ----- | ------------------ | --------------- |      
| kkt              | True  | 75.6               | 28.8            |
| hiw              | False | 74.7               | 29.3            |
| wls              | False | 63.0               | 13.2            |
| nee              | False | 54.9               | 33.2            |
| tdh              | True  | 35.2               | 29.7            |
| tkw              | False | 34.5               | 26.1            |
| lrz              | False | 30.5               | 39.0            |
| nmk              | False | 28.4               | 44.2            |
| ers              | False | 26.6               | 43.7            |
| bhj              | True  | 25.7               | 60.9            |
| abz              | True  | 24.8               | 45.3            |             
| swb              | False | 21.9               | 45.1            |
| msn              | False | 20.5               | 23.7            |
| slr              | True  | 20.1               | 46.4            |
| olr              | False | 17.7               | 42.4            |
| **Average**      |       | **36.1**           | **36.7**        |



# Credits
Researcher : Gilles Boulianne and Vishwa Gupta

Developer : Charles-William Cummings

# License
Copyright 2021 Centre de recherche informatique de Montréal (CRIM)

If you find this software useful please cite our paper: [Phoneme transcription of endangered languages: an evaluation of recent
ASR architectures in the single speaker scenario](https://aclanthology.org/2022.findings-acl.180).

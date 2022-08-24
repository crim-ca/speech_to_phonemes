# Grapheme to phoneme - Abzakh (abz)

The g2p tables list the phonemes used in the IPA transcription of Abzakh recordings provided in the [Pangloss collection](https://pangloss.cnrs.fr/corpus/Abzakh?lang=fr&mode=pro).
They were obtained by extracting all the unique IPA symbols used in the transcription files.

The two tables differ by how diacritics are handled:

* **roman_to_ipa :** diacritics are not preserved, but considered as fully realized phonemes.
  * For example, there is no distinction between the labialization diacritic [ʷ] and the phoneme [w]: both are mapped to [w].

* **roman_to_ipa_diacritics :** diacritics are preserved.
  * For example, there is a distinction between the labialization diacritic [ʷ] (mapped to [ʷ]) and the phoneme [w] (mapped to [w]).

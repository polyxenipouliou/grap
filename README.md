# GRAP: A Greek Rap Dataset
Syllable- & word-level annotations for 165 rap verses in Greek language.

## Motivation 
Rap music remains underrepresented in musicological research. To the best of our knowledge, the only public 
rap-focused dataset is [MCFlow](https://github.com/Computational-Cognitive-Musicology-Lab/MCFlow/tree/main) 
(Condit-Schultz, 2016), containing 124 English-language Billboard chart songs in Humdrum format. 

To address this gap of availability of non-English rap datasets, we present GRAP, a Greek Rap Dataset, aiming to 
initiate and support future research in MIR and computational ethnomusicology generally, and Greek and Balkan
rap and popular music genres specifically.

The dataset contains syllable- and word-level timestamp annotations of lyrics aligned to Greek rap music.

## Dataset Description
For the initial track collection, we wanted to have a wide range of tracks. Since IFPI local charts 
only show the most popular tracks of the week, we selected random tracks from artists based on a few existing online 
articles. The tracks collected were released between 2009-2026.

In total **70 tracks** were collected, by **36 artists**, spanning from *old-school hip-hop*, *underground*, *boom bap*, to *trap* 
and *drill* subgenres. We identified **165 verses** in total across these tracks. 

Our dataset is a work in progress, meaning that we currently have manually annotated 
**31 out of 70 tracks**, and **74 out of 165 identified verses**. 
**40 out of 70 tracks** have automatically annotated word-level timestamps (as of Sept 2026).

*(You can follow our progress below)* 

## Methodology
We assess the feasibility of whether this process can be automated by addressing the following research question: <br>

*Can we automate the alignment of Greek (non-English) rap audio to its lyrics at a syllable/word-level 
granularity using existing general-purpose forced-alignment models?*

**1. Tracks & Lyrics Collection**
Audio, lyrics and subgenre tags were sourced from YouTube, Genius, and Chosic, respectively.
<br>**2. Audio Trimming & Separation**
For the audio preprocessing stage, since most tracks had long pauses in the
beginning, we trimmed initial segments that were below -40dB sound level
*(We provide the durations of the trimmed segments for all tracks in the `metadata.csv`)*.
Next, we used [BS-RoFormer](https://ieeexplore.ieee.org/abstract/document/10446843) 
(Lu et al., 2024) to separate the tracks into vocal and instrumental stems. For the alignment process, 
we used the vocal stems. 
<br>**3. Lyrics Correction & Romanization**
For the lyrics preprocessing stage, we fixed typos and misheard lyrics. When the audio was 
inaudible even to us, we labeled it with the placeholder *"[inaudible]"*. Even though the songs are originally in 
Greek language, **833 unique instances of non-Greek words** were identified in the lyrics after 
removing artists' names, which rappers often include to pay tribute to one another, or announce that their rap is about to begin. These 833 instances were in English,
Spanish, Italian, French, and Arabic; only Arabic was in non-Latin characters, 
so we romanized it (*السَّلاَمُ عَلَيْكُمْ* → *salam alaykum*).
<br>**4. Audio Segmentation**
We segmented structural sections both manually and using [SongFormer](https://github.com/ASLP-lab/SongFormer/) (Hao et al., 2026). 
<br>**5. Manual Ground-Truth Annotation**
We manually annotated 30 tracks at the syllable-level, used to evaluate Forced Aligners' performance
on the vocal stem.
<br>**6. Audio Alignment** 
For the audio alignment step, we used [MMS-FA](https://huggingface.co/MahmoudAshraf/mms-300m-1130-forced-aligner)
(Pratap et al., 2024; Ashraf, 2024) and [Bournemouth FA](https://github.com/tabahi/bournemouth-forced-aligner)
(Rehman et al., 2025) on the vocal stem as input. MMS-FA produces character-level timestamps, which we then group into words, while BFA produces phoneme-, IPA-, and syllable-level timestamps.

## Results
Our findings answer the research question:

*Can we automate the alignment of Greek (non-English) rap audio to its lyrics at syllable/word-level 
granularity using existing general-purpose forced-alignment models?*  **YES,** ***but to a limited extent***. Word-level timestamps can be automated, but syllable-level timestamps require pre- and post-processing, such as syllabification of the lyrics and fusion of different aligners.

**Mean Absolute Error (Word-Level)**
|Anchoring Approach|MMS-FA|Bournemouth FA|
|---|---|---|
|w/o Section Anchors|**0.041s** median (0.044s avg ± 0.010s)|11.307s median (12.276s avg ± 4.777s)|
|Manually Annotated Anchors|0.412s median (0.698s avg ± 1.432s)|**2.344s** median (3.385s avg ± 2.802s)|
|SongFormer Section Anchors|0.191s median (1.962s avg ± 4.179s)|7.412s median (11.123s avg ± 16.867s)|
|Lyrics-driven SongFormer Section Anchors|0.042s median (0.091s avg ± 0.235s)|2.507s median (3.501s avg ± 2.557s)|  

**Comparison of the two aligners**
MMS-FA achieves the lowest error between the predicted 
timestamps and ground truth annotations on the word-level, when given the full, unanchored track. (**median MAE: 41ms**, approximate duration of a syllable). Its transformer architecture benefits from the global context of the rest of the audio and lyrics.
On the other hand, BFA improves substantially on a word-level (**median MAE: 2,344ms**), when given short, anchored sections, since its architecture uses frame-by-frame analysis instead.

MMS-FA is able to produce character-level timestamps and BFA produces phonemes, IPA, and syllables. 
Syllable-level timestamps could be extracted by combining both MMS-FA and BFA, but with a 44% coverage.


## Data Format
We currently release only the manually annotated tracks in the `ground_truth` folder. Each track file 
contains a JSON with the timestamps (e.g. `GR0001_alignment_canonical`), and a .txt file with the structural
annotations (timestamps and labels) (e.g. `GR0001_sections_canonical`).

Each JSON file looks like this:
```json
{
      "level": "syllable",
      "label": "τζι",
      "word": "τζιτζίκια",
      "start": 26.207241,
      "end": 26.386466,
      "section": "verse_1"
    },
    {
      "level": "syllable",
      "label": "τζί",
      "word": "τζιτζίκια",
      "start": 26.386466,
      "end": 26.592062,
      "section": "verse_1"
    },
    {
      "level": "syllable",
      "label": "κια",
      "word": "τζιτζίκια",
      "start": 26.592062,
      "end": 26.837376,
      "section": "verse_1"
    }
```
`level` represents the granularity level of the annotation, and `label` the syllable's text as pronounced. `word` shows the
full word that this syllable belongs to, `start` is the onset timestamp in seconds of the audio file, and 
`end` the offset timestamp in seconds. `section` shows the structural segment the part of the song it falls into, and matches the labels used in the `.txt` file.

Each .txt file looks like this: 
```txt
0.0 intro
25.877074 verse_1
92.672344 chorus
105.785998 verse_2
172.086748 chorus
198.539210 silence
205.195818 outro
```
The sections' duration is defined by the timestamp of the next line, which are in seconds. 

We additionally share a `metadata.csv` file with one row per track, and 14 columns: `ID`, `Song Title`, `Tempo` 
in BPM, `Duration`, `Year`, `Album`, `Artist`, `Writers`, `Producer`, `Url`, `Start/End` only when some tracks
share the same url because of an Album upload, `Trim Offset (s)` which is the duration of the silent part 
(below -40dB) we trimmed from the start, and `Tags` for subgenres each track falls into, separated by *;* ).  

## Progress
>In the `ground_truth` folder, some subfolders are named as *ID+s*; which denotes that only the labeled segments exist for that track.

|ID|Annotations|Segments|
|----|----|----|
|GR0001|&check;|&check;|
|GR0002|&check;|&check;|
|GR0003|&check;|&check;|
|GR0004|&check;|&check;|
|GR0005|&check;|&check;|
|GR0006|&check;|&check;|
|GR0007|&check;|&check;|
|GR0008|pending|&check;|
|GR0009|pending|&check;|
|GR0010|&check;|&check;|
|GR0011|&check;|&check;|
|GR0012|&check;|&check;|
|GR0013|&check;|&check;|
|GR0014|pending|&check;|
|GR0015|pending|&check;|
|GR0016|&check;|&check;|
|GR0017|&check;|&check;|
|GR0021|&check;|&check;|
|GR0022|&check;|&check;|
|GR0023|&check;|&check;|
|GR0024|&check;|&check;|
|GR0025|&check;|&check;|
|GR0026|&check;|&check;|
|GR0027|&check;|&check;|
|GR0028|&check;|&check;|
|GR0029|&check;|&check;|
|GR0030|&check;|&check;|
|GR0031|&check;|&check;|
|GR0032|&check;|&check;|
|GR0033|&check;|&check;|
|GR0034|&check;|&check;|
|GR0035|&check;|&check;|
|GR0036|pending|&check;|
|GR0037|pending|&check;|
|GR0038|pending|&check;|
|GR0039|&check;|&check;|
|GR0040|pending|&check;|
|GR0041|pending|&check;|
|GR0042|pending|pending|
|GR0043|pending|pending|
|GR0044|pending|pending|
|GR0046|pending|pending|
|GR0047|pending|pending|
|GR0048|pending|pending|
|GR0049|pending|pending|
|GR0050|pending|pending|
|GR0051|pending|pending|
|GR0052|pending|pending|
|GR0053|pending|pending|
|GR0054|pending|pending|
|GR0055|pending|pending|
|GR0057|&check;|&check;|
|GR0058|pending|pending|
|GR0062|pending|pending|
|GR0063|pending|pending|
|GR0064|pending|pending|
|GR0065|pending|pending|
|GR0066|pending|pending|
|GR0067|pending|pending|
|GR0068|pending|pending|
|GR0069|pending|pending|
|GR0070|pending|pending|
|GR0071|pending|pending|
|GR0073|&check;|&check;|
|GR0075|pending|pending|
|GR0076|pending|pending|
|GR0077|pending|pending|
|GR0078|pending|pending|
|GR0079|pending|pending|
|GR0080|pending|pending|

## Acknowledgments 
This work was presented at the 4th International Conference on Computational and Cognitive Musicology 
(ICCCM 2026) at Julius-Maximilians-Universität in Würzburg, Germany. 

We would like to thank Dr. Claire Arthur for her valuable guidance with this project and Foundation for 
Education and European Culture (IPEP) for supporting our studies at Universitat Pompeu Fabra.

## Citation
If you use GRAP in your research, please cite us: 
<br> **GRAP: A Greek Rap Dataset. Towards Benchmarking Cross-Cultural Rap Analysis (Pouliou & Saxena, 2026).**

## License
The annotations, timestamps, and metadata created for this project are released under CC BY 4.0. If you use or share them, please cite us (see Citation). 
The underlying audio remains the property of their original rights holders and is not distributed as part of this dataset. We provide source URLs alongside the timestamp annotations, so users can retrieve the original media themselves. 
By using this dataset, you agree to respect the copyright of the original audio.

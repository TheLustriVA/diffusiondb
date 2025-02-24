---
layout: default
title: Home
nav_order: 1
has_children: false

annotations_creators:
- no-annotation
language:
- en
language_creators:
- found
license:
- cc0-1.0
multilinguality:
- multilingual
pretty_name: DiffusionDB
size_categories:
- n>1T
source_datasets:
- original
tags:
- stable diffusion
- prompt engineering
- prompts
- research paper
task_categories:
- text-to-image
- image-to-text
task_ids:
- image-captioning
---

# DiffusionDB

<img width="100%" src="https://user-images.githubusercontent.com/15007159/198505835-bcc3a34f-a782-4064-989b-135e32b577a7.gif">

## Table of Contents

- [DiffusionDB](#diffusiondb)
  - [Table of Contents](#table-of-contents)
  - [Dataset Description](#dataset-description)
    - [Dataset Summary](#dataset-summary)
    - [Supported Tasks and Leaderboards](#supported-tasks-and-leaderboards)
    - [Languages](#languages)
  - [Dataset Structure](#dataset-structure)
    - [Data Instances](#data-instances)
    - [Data Fields](#data-fields)
    - [Data Splits](#data-splits)
    - [Loading Data Subsets](#loading-data-subsets)
      - [Method 1: Using Hugging Face Datasets Loader](#method-1-using-hugging-face-datasets-loader)
      - [Method 2. Use the PoloClub Downloader](#method-2-use-the-poloclub-downloader)
        - [Usage/Examples](#usageexamples)
          - [Downloading a single file](#downloading-a-single-file)
          - [Downloading a range of files](#downloading-a-range-of-files)
          - [Downloading to a specific directory](#downloading-to-a-specific-directory)
          - [Setting the files to unzip once they've been downloaded](#setting-the-files-to-unzip-once-theyve-been-downloaded)
      - [Method 3. Use `metadata.parquet` (Text Only)](#method-3-use-metadataparquet-text-only)
  - [Dataset Creation](#dataset-creation)
    - [Curation Rationale](#curation-rationale)
    - [Source Data](#source-data)
      - [Initial Data Collection and Normalization](#initial-data-collection-and-normalization)
      - [Who are the source language producers?](#who-are-the-source-language-producers)
    - [Annotations](#annotations)
      - [Annotation process](#annotation-process)
      - [Who are the annotators?](#who-are-the-annotators)
    - [Personal and Sensitive Information](#personal-and-sensitive-information)
  - [Considerations for Using the Data](#considerations-for-using-the-data)
    - [Social Impact of Dataset](#social-impact-of-dataset)
    - [Discussion of Biases](#discussion-of-biases)
    - [Other Known Limitations](#other-known-limitations)
  - [Additional Information](#additional-information)
    - [Dataset Curators](#dataset-curators)
    - [Licensing Information](#licensing-information)
    - [Citation Information](#citation-information)
    - [Contributions](#contributions)

## Dataset Description

- **Homepage:** [DiffusionDB homepage](https://poloclub.github.io/diffusiondb)
- **Repository:** [DiffusionDB repository](https://github.com/poloclub/diffusiondb)
- **Distribution:** [DiffusionDB Hugging Face Dataset](https://huggingface.co/datasets/poloclub/diffusiondb)
- **Paper:** [DiffusionDB: A Large-scale Prompt Gallery Dataset for Text-to-Image Generative Models](https://arxiv.org/abs/2210.14896)
- **Point of Contact:** [Jay Wang](mailto:jayw@gatech.edu)

### Dataset Summary

DiffusionDB is the first  large-scale text-to-image prompt dataset. It contains 2 million images generated by Stable Diffusion using prompts and hyperparameters specified by real users.

DiffusionDB is publicly available at [🤗 Hugging Face Dataset](https://huggingface.co/datasets/poloclub/diffusiondb).

### Supported Tasks and Leaderboards

The unprecedented scale and diversity of this human-actuated dataset provide exciting research opportunities in understanding the interplay between prompts and generative models, detecting deepfakes, and designing human-AI interaction tools to help users more easily use these models.

### Languages

The text in the dataset is mostly English. It also contains other languages such as Spanish, Chinese, and Russian.

## Dataset Structure

We use a modularized file structure to distribute DiffusionDB. The 2 million images in DiffusionDB are split into 2,000 folders, where each folder contains 1,000 images and a JSON file that links these 1,000 images to their prompts and hyperparameters.

```bash
./
├── images
│   ├── part-000001
│   │   ├── 3bfcd9cf-26ea-4303-bbe1-b095853f5360.png
│   │   ├── 5f47c66c-51d4-4f2c-a872-a68518f44adb.png
│   │   ├── 66b428b9-55dc-4907-b116-55aaa887de30.png
│   │   ├── 99c36256-2c20-40ac-8e83-8369e9a28f32.png
│   │   ├── f3501e05-aef7-4225-a9e9-f516527408ac.png
│   │   ├── [...]
│   │   └── part-000001.json
│   ├── part-000002
│   ├── part-000003
│   ├── part-000004
│   ├── [...]
│   └── part-002000
└── metadata.parquet
```

These sub-folders have names `part-00xxxx`, and each image has a unique name generated by [UUID Version 4](https://en.wikipedia.org/wiki/Universally_unique_identifier). The JSON file in a sub-folder has the same name as the sub-folder. Each image is a PNG file. The JSON file contains key-value pairs mapping image filenames to their prompts and hyperparameters.

### Data Instances

For example, below is the image of `f3501e05-aef7-4225-a9e9-f516527408ac.png` and its key-value pair in `part-000001.json`.

<img width="300" src="https://i.imgur.com/gqWcRs2.png">

```json
{
  "f3501e05-aef7-4225-a9e9-f516527408ac.png": {
    "p": "geodesic landscape, john chamberlain, christopher balaskas, tadao ando, 4 k, ",
    "se": 38753269,
    "c": 12.0,
    "st": 50,
    "sa": "k_lms"
  },
}
```

### Data Fields

- key: Unique image name
- `p`: Prompt
- `se`: Random seed
- `c`: CFG Scale (guidance scale)
- `st`: Steps
- `sa`: Sampler

At the top level folder of DiffusionDB, we include a metadata table in Parquet format `metadata.parquet`.
This table has seven columns: `image_name`, `prompt`, `part_id`, `seed`, `step`, `cfg`, and `sampler`, and it has 2 million rows where each row represents an image. `seed`, `step`, and `cfg` are We choose Parquet because it is column-based: researchers can efficiently query individual columns (e.g., prompts) without reading the entire table. Below are the five random rows from the table.

| image_name                               | prompt                                                                                                                                                                                                  | part_id | seed       | step | cfg | sampler |
|------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|------------|------|-----|---------|
| 49f1e478-ade6-49a8-a672-6e06c78d45fc.png | ryan gosling in fallout 4 kneels near a nuclear bomb                                                                                                                                                    | 1643    | 2220670173 | 50   | 7.0 | 8       |
| b7d928b6-d065-4e81-bc0c-9d244fd65d0b.png | A beautiful robotic woman dreaming, cinematic lighting, soft bokeh, sci-fi, modern, colourful, highly detailed, digital painting, artstation, concept art, sharp focus, illustration, by greg rutkowski | 87      | 51324658   | 130  | 6.0 | 8       |
| 19b1b2f1-440e-4588-ba96-1ac19888c4ba.png | bestiary of creatures from the depths of the unconscious psyche, in the style of a macro photograph with shallow dof                                                                                    | 754     | 3953796708 | 50   | 7.0 | 8       |
| d34afa9d-cf06-470f-9fce-2efa0e564a13.png | close up portrait of one calico cat by vermeer. black background, three - point lighting, enchanting, realistic features, realistic proportions.                                                        | 1685    | 2007372353 | 50   | 7.0 | 8       |
| c3a21f1f-8651-4a58-a4d4-7500d97651dc.png | a bottle of jack daniels with the word medicare replacing the word jack daniels                                                                                                                         | 243     | 1617291079 | 50   | 7.0 | 8       |

To save space, we use an integer to encode the `sampler` in the table above.

|Sampler|Integer Value|
|:--|--:|
|ddim|1|
|plms|2|
|k_euler|3|
|k_euler_ancestral|4|
|ddik_heunm|5|
|k_dpm_2|6|
|k_dpm_2_ancestral|7|
|k_lms|8|
|others|9|

### Data Splits

We split 2 million images into 2,000 folders where each folder contains 1,000 images and a JSON file.

### Loading Data Subsets

DiffusionDB is large (1.6TB)! However, with our modularized file structure, you can easily load a desirable number of images and their prompts and hyperparameters. In the [`example-loading.ipynb`](https://github.com/poloclub/diffusiondb/blob/main/notebooks/example-loading.ipynb) notebook, we demonstrate three methods to load a subset of DiffusionDB. Below is a short summary.

#### Method 1: Using Hugging Face Datasets Loader

You can use the Hugging Face [`Datasets`](https://huggingface.co/docs/datasets/quickstart) library to easily load prompts and images from DiffusionDB. We pre-defined 16 DiffusionDB subsets (configurations) based on the number of instances. You can see all subsets in the [Dataset Preview](https://huggingface.co/datasets/poloclub/diffusiondb/viewer/all/train).

```python
import numpy as np
from datasets import load_dataset

# Load the dataset with the `random_1k` subset
dataset = load_dataset('poloclub/diffusiondb', 'random_1k')
```

#### Method 2. Use the PoloClub Downloader

The PoloClub Downloader is a Python package that allows you to download and load DiffusionDB. It comes as part of the repository and is activated from the command line. Below is an example of loading a subset of DiffusionDB.

##### Usage/Examples

The script is run using command-line arguments as follows:

`-i` `--index` - File to download or lower bound of a range of files if `-r` is also set.

`-r` `--range` - Upper bound of range of files to download if `-i` is set.

`-o` `--output` - Name of custom output directory. Defaults to the current directory if not set.

`-z` `--unzip` - Unzip the file/files after downloading

###### Downloading a single file

The specific file to download is supplied as the number at the end of the file on HuggingFace. The script will automatically pad the number out and generate the URL.

```bash
python download.py -i 23
```

###### Downloading a range of files

The upper and lower bounds of the set of files to download are set by the `-i` and `-r` flags respectively.

```bash
python download.py -i 1 -r 2000
```

Note that this range will download the entire dataset. The script will ask you to confirm that you have 1.7Tb free at the download destination.

###### Downloading to a specific directory

The script will default to the location of the dataset's `part` .zip files at `images/`. If you wish to move the download location, you should move these files as well or use a symbolic link.

```bash
python download.py -i 1 -r 2000 -o /home/$USER/datahoarding/etc
```

Again, the script will automatically add the `/` between the directory and the file when it downloads.

###### Setting the files to unzip once they've been downloaded

The script is set to unzip the files _after_ all files have downloaded as both can be lengthy processes in certain circumstances.

```bash
python download.py -i 1 -r 2000 -z
```

#### Method 3. Use `metadata.parquet` (Text Only)

If your task does not require images, then you can easily access all 2 million prompts and hyperparameters in the `metadata.parquet` table.

```python
from urllib.request import urlretrieve
import pandas as pd

# Download the parquet table
table_url = f'https://huggingface.co/datasets/poloclub/diffusiondb/resolve/main/metadata.parquet'
urlretrieve(table_url, 'metadata.parquet')

# Read the table using Pandas
metadata_df = pd.read_parquet('metadata.parquet')
```

## Dataset Creation

### Curation Rationale

Recent diffusion models have gained immense popularity by enabling high-quality and controllable image generation based on text prompts written in natural language. Since the release of these models, people from different domains have quickly applied them to create awardwinning artworks, synthetic radiology images, and even hyper-realistic videos.

However, generating images with desired details is difficult, as it requires users to write proper prompts specifying the exact expected results. Developing such prompts requires trial and error, and can often feel random and unprincipled. Simon Willison analogizes writing prompts to wizards learning “magical spells”: users do not understand why some prompts work, but they will add these prompts to their “spell book.” For example, to generate highly-detailed images, it has become a common practice to add special keywords such as “trending on artstation” and “unreal engine” in the prompt.

Prompt engineering has become a field of study in the context of text-to-text generation, where researchers systematically investigate how to construct prompts to effectively solve different down-stream tasks. As large text-to-image models are relatively new, there is a pressing need to understand how these models react to prompts, how to write effective prompts, and how to design tools to help users generate images.
To help researchers tackle these critical challenges, we create DiffusionDB, the first large-scale prompt dataset with 2 million real prompt-image pairs.

### Source Data

#### Initial Data Collection and Normalization

We construct DiffusionDB by scraping user-generated images on the official Stable Diffusion Discord server. We choose Stable Diffusion because it is currently the only open-source large text-to-image generative model, and all generated images have a CC0 1.0 Universal Public Domain Dedication license that waives all copyright and allows uses for any purpose. We choose the official [Stable Diffusion Discord server](https://discord.gg/stablediffusion) because it is public, and it has strict rules against generating and sharing illegal, hateful, or NSFW (not suitable for work, such as sexual and violent content) images. The server also disallows users to write or share prompts with personal information.

#### Who are the source language producers?

The language producers are users of the official [Stable Diffusion Discord server](https://discord.gg/stablediffusion).

### Annotations

The dataset does not contain any additional annotations.

#### Annotation process

[N/A]

#### Who are the annotators?

[N/A]

### Personal and Sensitive Information

The authors removed the discord usernames from the dataset.
We decide to anonymize the dataset because some prompts might include sensitive information: explicitly linking them to their creators can cause harm to creators.

## Considerations for Using the Data

### Social Impact of Dataset

The purpose of this dataset is to help develop better understanding of large text-to-image generative models.
The unprecedented scale and diversity of this human-actuated dataset provide exciting research opportunities in understanding the interplay between prompts and generative models, detecting deepfakes, and designing human-AI interaction tools to help users more easily use these models.

It should note that we collect images and their prompts from the Stable Diffusion Discord server. The Discord server has rules against users generating or sharing harmful or NSFW (not suitable for work, such as sexual and violent content) images. The Stable Diffusion model used in the server also has an NSFW filter that blurs the generated images if it detects NSFW content. However, it is still possible that some users had generated harmful images that were not detected by the NSFW filter or removed by the server moderators. Therefore, DiffusionDB can potentially contain these images. To mitigate the potential harm, we provide a [Google Form](https://forms.gle/GbYaSpRNYqxCafMZ9) on the [DiffusionDB website](https://poloclub.github.io/diffusiondb/) where users can report harmful or inappropriate images and prompts. We will closely monitor this form and remove reported images and prompts from DiffusionDB.

### Discussion of Biases

The 2 million images in DiffusionDB have diverse styles and categories. However, Discord can be a biased data source. Our images come from channels where early users could use a bot to use Stable Diffusion before release. As these users had started using Stable Diffusion before the model was public, we hypothesize that they are AI art enthusiasts and are likely to have experience with other text-to-image generative models. Therefore, the prompting style in DiffusionDB might not represent novice users. Similarly, the prompts in DiffusionDB might not generalize to domains that require specific knowledge, such as medical images.

### Other Known Limitations

**Generalizability.** Previous research has shown a prompt that works well on one generative model might not give the optimal result when used in other models.
Therefore, different models can need users to write different prompts. For example, many Stable Diffusion prompts use commas to separate keywords, while this pattern is less seen in prompts for DALL-E 2 or Midjourney. Thus, we caution researchers that some research findings from DiffusionDB might not be generalizable to other text-to-image generative models.

## Additional Information

### Dataset Curators

DiffusionDB is created by [Jay Wang](https://zijie/wang), [Evan Montoya](https://www.linkedin.com/in/evan-montoya-b252391b4/), [David Munechika](https://www.linkedin.com/in/dmunechika/), [Alex Yang](https://alexanderyang.me), [Ben Hoover](https://www.bhoov.com), [Polo Chau](https://faculty.cc.gatech.edu/~dchau/).


### Licensing Information

The DiffusionDB dataset is available under the [CC0 1.0 License](https://creativecommons.org/publicdomain/zero/1.0/).
The Python code in this repository is available under the [MIT License](https://github.com/poloclub/diffusiondb/blob/main/LICENSE).

### Citation Information

```bibtex
@article{wangDiffusionDBLargescalePrompt2022,
  title = {{{DiffusionDB}}: {{A}} Large-Scale Prompt Gallery Dataset for Text-to-Image Generative Models},
  author = {Wang, Zijie J. and Montoya, Evan and Munechika, David and Yang, Haoyang and Hoover, Benjamin and Chau, Duen Horng},
  year = {2022},
  journal = {arXiv:2210.14896 [cs]},
  url = {https://arxiv.org/abs/2210.14896}
}
```

### Contributions

If you have any questions, feel free to [open an issue](https://github.com/poloclub/diffusiondb/issues/new) or contact [Jay Wang](https://zijie.wang).

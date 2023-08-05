<div align="center">

<img alt="LOGO" src="https://cdn.jsdelivr.net/gh/fishaudio/fish-diffusion@main/images/logo_512x512.png" width="256" height="256" />

# Fish Diffusion

<div>
<a target="_blank" href="https://github.com/fishaudio/fish-diffusion/actions/workflows/ci.yml">
<img alt="Build Status" src="https://img.shields.io/github/actions/workflow/status/fishaudio/fish-diffusion/ci.yml?style=flat-square&logo=GitHub"/>
</a>
<a target="_blank" href="https://hub.docker.com/r/lengyue233/fish-diffusion">
<img alt="Docker Hub" src="https://img.shields.io/docker/cloud/build/lengyue233/fish-diffusion?style=flat-square&logo=Docker&logoColor=white"/>
</a>
<a target="_blank" href="https://huggingface.co/spaces/fishaudio/fish-diffusion">
<img alt="Hugging Face" src="https://img.shields.io/badge/🤗%20Spaces-HiFiSinger-blue.svg?style=flat-square"/>
</a>
</div>

<div>
<a target="_blank" href="https://discord.gg/wbYSRBrW2E">
<img alt="Discord" src="https://img.shields.io/discord/1044927142900809739?color=%23738ADB&label=Discord&logo=discord&logoColor=white&style=flat-square"/>
</a>
<a target="_blank" href="https://colab.research.google.com/github/fishaudio/fish-diffusion/blob/main/notebooks/train.ipynb">
<img alt="Open In Colab" src="https://img.shields.io/static/v1?label=Colab&message=Notebook&color=F9AB00&logo=googlecolab&style=flat-square"/>
</a>
</div>

</div>

------

An easy to understand TTS / SVS / SVC training framework.

> Check our [Wiki](https://fishaudio.github.io/fish-diffusion/) to get started! 

[中文文档](README.zh.md)

## Terms of Use for Fish Diffusion
1. Obtaining Authorization and Intellectual Property Infringement: The user is solely accountable for acquiring the necessary authorization for any datasets utilized in their training process and assumes full responsibility for any infringement issues arising from the utilization of the input source. Fish Diffusion and its developers disclaim all responsibility for any complications that may emerge due to the utilization of unauthorized datasets.

2. Proper Attribution: Any derivative works based on Fish Diffusion must explicitly acknowledge the project and its license. In the event of distributing Fish Diffusion's code or disseminating results generated by this project, the user is obliged to cite the original author and source code (Fish Diffusion).

3. Audiovisual Content and AI-generated Disclosure: All derivative works created using Fish Diffusion, including audio or video materials, must explicitly acknowledge the utilization of the Fish Diffusion project and declare that the content is AI-generated. If incorporating videos or audio published by third parties, the original links must be furnished.

4. Agreement to Terms: By persisting in the use of Fish Diffusion, the user unequivocally consents to the terms and conditions delineated in this document. Neither Fish Diffusion nor its developers shall be held liable for any subsequent difficulties that may transpire.


## Summary
Using Diffusion Model to solve different voice generating tasks. Compared with the original diffsvc repository, the advantages and disadvantages of this repository are as follows:
+ Support multi-speaker
+ The code structure of this repository is simpler and easier to understand, and all modules are decoupled
+ Support [44.1khz Diff Singer community vocoder](https://openvpi.github.io/vocoders/)
+ Support multi-machine multi-devices training, support half-precision training, save your training speed and memory

## Preparing the environment
The following commands need to be executed in the conda environment of python 3.10

```bash
# Install PyTorch related core dependencies, skip if installed
# Reference: https://pytorch.org/get-started/locally/
conda install "pytorch>=2.0.0" "torchvision>=0.15.0" "torchaudio>=2.0.0" pytorch-cuda=11.8 -c pytorch -c nvidia

# Install PDM dependency management tool, skip if installed
# Reference: https://pdm.fming.dev/latest/
curl -sSL https://raw.githubusercontent.com/pdm-project/pdm/main/install-pdm.py | python3 -

# Install the project dependencies
pdm sync
```

## Vocoder preparation
Fish Diffusion requires the [FishAudio NSF-HiFiGAN](https://github.com/fishaudio/fish-diffusion/releases/tag/v2.0.0) vocoder to generate audio.

### Automatic download
```bash
python tools/download_nsf_hifigan.py
```

If you are using the script to download the model, you can use the `--agree-license` parameter to agree to the [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) license.

```bash
python tools/download_nsf_hifigan.py --agree-license
```

### Manual download
Download and unzip `nsf_hifigan-stable-v1.zip` from [Fish Diffusion Release](https://github.com/fishaudio/fish-diffusion/releases/tag/v2.0.0)  
Copy the `nsf_hifigan` folder to the `checkpoints` directory (create if not exist)

If you want to download [ContentVec](https://github.com/auspicious3000/contentvec) manually, you can download it from [here](https://github.com/fishaudio/fish-diffusion/releases/download/v1.12/content-vec-best-legacy-500.pt) and put it in the `checkpoints` directory.

## Dataset preparation
You only need to put the dataset into the `dataset` directory in the following file structure

```shell
dataset
├───train
│   ├───xxx1-xxx1.wav
│   ├───...
│   ├───Lxx-0xx8.wav
│   └───speaker0 (Subdirectory is also supported)
│       └───xxx1-xxx1.wav
└───valid
    ├───xx2-0xxx2.wav
    ├───...
    └───xxx7-xxx007.wav
```

```bash
# Extract all data features, such as pitch, text features, mel features, etc.
python tools/preprocessing/extract_features.py --config configs/svc_hubert_soft.py --path dataset --clean
```

## Baseline training
> The project is under active development, please backup your config file  
> The project is under active development, please backup your config file  
> The project is under active development, please backup your config file  

```bash
# Single machine single card / multi-card training
python tools/diffusion/train.py --config configs/svc_hubert_soft.py
# Multi-node training
python tools/diffusion/train.py --config configs/svc_content_vec_multi_node.py
# Environment variables need to be defined on each node,please see https://pytorch-lightning.readthedocs.io/en/1.6.5/clouds/cluster.html  for more information.

# Resume training
python tools/diffusion/train.py --config configs/svc_hubert_soft.py --resume [checkpoint file]

# Fine-tune the pre-trained model
# Note: You should adjust the learning rate scheduler in the config file to warmup_cosine_finetune
python tools/diffusion/train.py --config configs/svc_cn_hubert_soft_finetune.py --pretrained [checkpoint file]
```

## Inference
```bash
# Inference using shell, you can use --help to view more parameters
python tools/diffusion/inference.py --config [config] \
    --checkpoint [checkpoint file] \
    --input [input audio] \
    --output [output audio]


# Gradio Web Inference, other parameters will be used as gradio default parameters
python tools/diffusion/inference.py --config [config] \
    --checkpoint [checkpoint file] \
    --gradio
```

## Convert a DiffSVC model to Fish Diffusion
```bash
python tools/diffusion/diff_svc_converter.py --config configs/svc_hubert_soft_diff_svc.py \
    --input-path [DiffSVC ckpt] \
    --output-path [Fish Diffusion ckpt]
```

## Contributing
If you have any questions, please submit an issue or pull request.  
You should run `pdm run lint` before submitting a pull request.

Real-time documentation can be generated by
```bash
pdm run docs
```

## Credits
+ [diff-svc original](https://github.com/prophesier/diff-svc)
+ [diff-svc optimized](https://github.com/innnky/diff-svc/)
+ [DiffSinger](https://github.com/openvpi/DiffSinger/) [Paper](https://arxiv.org/abs/2105.02446)
+ [so-vits-svc](https://github.com/innnky/so-vits-svc)
+ [iSTFTNet](https://github.com/rishikksh20/iSTFTNet-pytorch) [Paper](https://arxiv.org/pdf/2203.02395.pdf)
+ [CookieTTS](https://github.com/CookiePPP/cookietts/tree/master/CookieTTS/_4_mtw/hifigan)
+ [HiFi-GAN](https://github.com/jik876/hifi-gan) [Paper](https://arxiv.org/abs/2010.05646)
+ [Retrieval-based-Voice-Conversion](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI)

## Thanks to all contributors for their efforts

<a href="https://github.com/fishaudio/fish-diffusion/graphs/contributors" target="_blank">
  <img src="https://contrib.rocks/image?repo=fishaudio/fish-diffusion" />
</a>

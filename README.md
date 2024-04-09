# LiteMedSAM for Scribble prompts


## Installation

The codebase is tested with: `Ubuntu 20.04` | Python `3.10` | `CUDA 11.8` | `Pytorch 2.1.2`

1. Create a virtual environment `conda create -n medsam python=3.10 -y` and activate it `conda activate medsam`
2. Install [Pytorch 2.0](https://pytorch.org/get-started/locally/)
3. `git clone -b LiteMedSAMScribble https://github.com/bowang-lab/MedSAM/`
4. Enter the MedSAM folder `cd MedSAM` and run `pip install -e .`


## Model Training

The data preprocessing is the same as the settings in [LiteMedSAM](https://github.com/bowang-lab/MedSAM/tree/LiteMedSAM#data-preprocessing). 


```bash
python train_one_gpu_scribble.py \
    -data_root data/MedSAM_train \
    -pretrained_checkpoint lite_medsam.pth \
    -work_dir work_dir \
    -num_workers 4 \
    -batch_size 4 \
    -num_epochs 1000
```

For additional command line arguments, see `python train_one_gpu_scribble.py -h`.

## Automatic Generation of Scribbles

Please note that for training, `sampler.py` and `scribble.py` in `visual_sampler` are used for faster automatic scribble generation whereas here we use `sampler.py` and `scribble.py` to generate refined scribbles.

### Installing Dependencies

In addition to the packages installed for MedSAM, please install `cc3d` and `largestinteriorrectangle`.

1. pip install connected-components-3d
2. pip install largestinteriorrectangle




## Quick tutorial on making submissions to CVPR 2024 MedSAM on Laptop Challenge

### Sanity test
- Download the LiteMedSAM checkpoint here and put it in `work_dir/LiteMedSAM`.
- Download the demo data [here](https://drive.google.com/drive/folders/1QOpXVpx-E05mviafi_wMMkkLs6VAMkCR)
- Run the following command for a sanity test

```bash
python CVPR24_LiteMedSAM_infer_scribble.py -i demo_scribble/imgs -o demo_scribble/segs
```

### Build Docker
```bash
docker build -f Dockerfile -t litemedsam .
```
> Note: don't forget the `.` in the end
Run the docker on the testing demo images
```bash
docker container run -m 8G --name litemedsam --rm -v $PWD/test_demo/imgs/:/workspace/inputs/ -v $PWD/test_demo/litemedsam-seg/:/workspace/outputs/ litemedsam:latest /bin/bash -c "sh predict.sh"
```
> Note: please run `chmod -R 777 ./*` if you run into `Permission denied` error.

### Save docker 

```bash
docker save litemedsam | gzip -c > litemedsam.tar.gz
```

### Compute Metrics

```bash
python evaluation/compute_metrics.py -s test_demo/litemedsam-seg -g test_demo/gts -csv_dir ./metrics.csv
```

### Prepare Demo Train Dataset
- We prepared a small subset of the train data in `scribble-train-demo`. Please download it.

### Generating Scribbles

- To run `generate_scribbles_demo.py`, please set the paths to the root and the destination directories.

```bash
python generate_scribbles_train.py -root ./scribble-train-demo -save_path ./train_scribbles
```

### Demo Inference script 
The interactive scribble prompt inference can be run using the following command. Please drag an image to the input, draw a scribble on it, and submit it to get the output mask using the Gradio API.

```bash
python demo_scribble_LiteMedSAM.py -medsam_lite_checkpoint_path ./checkpoints/medsam_lite_scribble.pth
```

## Acknowledgements
We thank the authors of [MobileSAM](https://github.com/ChaoningZhang/MobileSAM), [TinyViT](https://github.com/microsoft/Cream/tree/main/TinyViT), and [SEEM](https://github.com/UX-Decoder/Segment-Everything-Everywhere-All-At-Once) for making their source code publicly available.


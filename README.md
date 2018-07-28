## DDRNet: Depth Map Denoising and Refinement for Consumer Depth Cameras Using Cascaded CNNs
The major contributers of this repository include [Shi Yan](https://github.com/neycyanshi) and [Lizhen Wang](https://github.com/LizhenWangT).

### Introduction
**DDRNet** is a cascaded Depth Denoising and Refinement Network, which achieves superior performance over the state-of-the-art techniques.

<img src="dataset/pipe.png">

DDRNet is described in [ECCV 2018 paper](https://null). It is worth noticing that:
- DDRNet leverages the multi-frame fused geometry and the accompanying high quality color image through a joint training strategy.
- Each sub-network focuses on lifting the quality in different frequency domains.
- It combines supervised and unsupervised learning to solve the issue of lacking ground truth training data.

### Requirements: Software
- Python > 2.7
- TensorFlow >= 1.3.0
- numpy, scipy, scikit-image

### Requirements: Hardware
Any NVIDIA GPUs with 5GB memory suffices. For training, we recommend 4xGPU with 12G memory.

### Before Start
Clone the repository  
    ```shell
    git clone https://github.com/neycyanshi/DDRNet.git
    ```

### Preparation
1. Download dataset or prepare your own data. In the dataset folder:
    - `color_map` and `depth_map` are captured and aligned raw data.
    - `high_quality_depth` (depth_ref, *D_ref*) is generated using fusion method, paired with `depth_map`.
    - `depth_filled`, `mask` and `albedo` are generated by preprocess(optional).
        ```
        dataset
        ├── 20170907
        │   ├── group1
        │   │   ├── depth_map
        │   │   ├── high_quality_depth (depth_ref)
        │   │   ├── depth_filled
        │   │   ├── color_map
        │   │   ├── mask
        │   │   └── albedo
        │   ├── group2
        │   └── ... 
        ├── 20170910
        └── ...
        ```

2. Please prepare index files for training or testing. `train.csv` has at least 3 columns, pre-computed `mask` and `albedo` is optional, and can be added to the 4,5th column. `test.csv` is similar, and `depth_ref` path is not necessary.

3. (Optional) Preproccess
    - `depth_filled` and `mask`: fill small holes in depth map and get salient mask using the following command.
    ```shell
    python data_utils/dilate_erode.py dataset/20170907
    ```
    - `albedo`: get offline albedo using methods like [Retinex algorithm](https://github.com/lmurmann/retinex) or [Reflectance Filtering](https://github.com/tnestmeyer/reflectance-filtering).

### Testing
1. Please download our pre-trained model from [Google Drive](https://drive.google.com/open?id=10sAnwirBx4P98LpwI3Ku0v013VSQ_qEc) or [Baidu Pan](https://pan.baidu.com/s/1YRiFy3s1-vZAlt9sx9aiVw), and put them under folder `log/cscd/noBN_L1_sd100_B16/`, and we call this checkpoint directory as `$CKPT_DIR`.
    Make sure it looks like this:
    ```
    log/cscd/noBN_L1_sd100_B16/checkpoint
    log/cscd/noBN_L1_sd100_B16/graph.pbtxt
    log/cscd/noBN_L1_sd100_B16/noBN_L1_sd100_B16-6000.meta
    log/cscd/noBN_L1_sd100_B16/noBN_L1_sd100_B16-6000.index
    log/cscd/noBN_L1_sd100_B16/noBN_L1_sd100_B16-6000.data-00000-of-00001
    ```
2. Edit `eval.sh` by modifing `checkpoint_dir` and `csv_path` to yours.
3. Run test script and save results to `$RESULT_DIR` directory. Denoised depth map (*D_dn*) `dn_frame_%6d.png` and refined depth map (*D_dt*) `dt_frame_%6d.png` will be saved.
    ```shell
    sh eval.sh $RESULT_DIR/  # RESULT_DIR=sample
    ```

### Training
1. `train.sh` is an example of how to train this network, please edit `index_file` and `dataset_dir` to yours.
2. Run train script and save checkpoints to specified directory. Here `$LOG_DIR` is the parent directory of `$EXPM_DIR`. Log, network graph and model parameters (checkpoints) are saved in `$EXPM_DIR`. `$GPU_ID` is the selected gpu id. If `GPU_ID=0`, training is done on the first GPU.
    ```shell
    sh train.sh $LOG_DIR/$EXPM_DIR/ $GPU_ID  # LOG_DIR=cscd EXPM_DIR=noBN_L1_sd100_B16
    tensorboard --logdir=log/$LOG_DIR/$EXPM_DIR --port=6006  # visualizing training.
    ```

### Citing DDRNet
If you find DDRNet useful in your research, please consider citing:
```
@inproceedings{yan2018DDRNet,  
  author = {Shi Yan, Chenglei Wu, Lizhen Wang, Feng Xu, Liang An, Kaiwen Guo, and Yebin Liu},  
  title = {DDRNet: Depth Map Denoising and Refinement for Consumer Depth Cameras Using Cascaded CNNs},  
  booktitle = {ECCV},  
  year = {2018}  
}
```

### Misc.
Code has been tested under:
- Ubuntu 16.04 with 3 Maxwell Titan X GPU and Intel i7-6900K CPU @ 3.20GHz
- Ubuntu 14.04 with a GTX 1070 GPU and Intel Xeon CPU E3-1231 v3 @ 3.40GHz

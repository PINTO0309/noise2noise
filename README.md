# Noise2Noise
AI can now fix your grainy photos by only looking at grainy photos.<br>
**This is the Fork repository of https://github.com/yu4u/noise2noise.git.**<br><br>
https://news.developer.nvidia.com/ai-can-now-fix-your-grainy-photos-by-only-looking-at-grainy-photos/<br>
![01](https://github.com/PINTO0309/noise2noise/blob/PINTO0309work/media/01.png)<br><br>

This is an unofficial and partial Keras implementation of "Noise2Noise: Learning Image Restoration without Clean Data" [1].

There are several things different from the original paper
(but not a fatal problem to confirm the noise2noise training framework):
- Training dataset (orignal: ImageNet, this repository: [2])
- Model (original: RED30 [3], this repository: SRResNet [4])

## Dependencies
- Keras, TensorFlow, NumPy, OpenCV

## Train Noise2Noise

### Download Dataset

```bash
mkdir dataset
cd dataset
wget https://cv.snu.ac.kr/research/VDSR/train_data.zip
wget https://cv.snu.ac.kr/research/VDSR/test_data.zip
unzip train_data.zip
unzip test_data.zip
cd ..
```

### Train Model

#### Train with Gaussian noise
```bash
# train model using (noise, noise) pairs (noise2noise)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --output_path gaussian

# train model using (noise, clean) paris (standard training)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --target_noise_model clean --output_path clean
```


#### Train with text insertion

```bash
# train model using (noise, noise) pairs (noise2noise)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model text,0,50 --target_noise_model text,0,50 --val_noise_model text,25,25 --loss mae --output_path text_noise

# train model using (noise, clean) paris (standard training)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model text,0,50 --target_noise_model clean --val_noise_model text,25,25 --loss mae --output_path text_clean
```

Please see `python3 train.py -h` for optional arguments.

### Noise Models
Using `source_noise_model`, `target_noise_model`, and `val_noise_model` arguments,
arbitrary noise models can be set for source images, target images, and validatoin images respectively.
Default values are taken from the experiment in [1].

- Gaussian noise
  - gaussian,min_stddev,max_stddev (e.g. gaussian,0,50)
- Clean target
  - clean
- Text insertion
  - text,min_occupancy,max_occupancy (e.g. text,0,50)

You can see how these noise models work by:

```bash
python3 noise_model.py --noise_model text,0,50
```

### Results
#### Plot training history

```bash
python3 plot_history.py --input1 gaussian --input2 clean
```

##### Gaussian noise
<img src="result/val_loss.png" width="480px">


<img src="result/val_PSNR.png" width="480px">

From the above result, I confirm that we can train denoising model using noisy targets
but it is not comparable to the model trained using clean targets.

##### Text insertion
<img src="result/val_loss_text.png" width="480px">


<img src="result/val_PSNR_text.png" width="480px">

#### Check denoising result

```bash
python3 test_model.py --weight_file [trained_model_path] --image_dir dataset/Set14
```

##### Gaussian noise
Denoising result by clean target model (left to right: original, degraded image, denoised image):

<img src="result/baby_GT_clean.png" width="800px">

Denoising result by noise target model:

<img src="result/baby_GT_gaussian.png" width="800px">

##### Text insertion
Denoising result by clean target model

<img src="result/baby_GT_text_clean.png" width="800px">

Denoising result by noise target model:

<img src="result/baby_GT_text_noise.png" width="800px">


### TODOs

- [x] Compare (noise, clean) training and (noise, noise) training
- [x] Add different noise models
- [x] Write readme

## References

[1] J. Lehtinen, J. Munkberg, J. Hasselgren, S. Laine, T. Karras, M. Aittala, 
T. Aila, "Noise2Noise: Learning Image Restoration without Clean Data," in Proc. of ICML, 2018.

[2] J. Kim, J. K. Lee, and K. M. Lee, "Accurate Image Super-Resolution Using Very Deep Convolutional Networks," in Proc. of CVPR, 2016.

[3] X.-J. Mao, C. Shen, and Y.-B. Yang, "Image
Restoration Using Convolutional Auto-Encoders with
Symmetric Skip Connections," in Proc. of NIPS, 2016.

[4] C. Ledig, et al., "Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network," in Proc. of CVPR, 2017.

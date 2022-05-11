# Document Denoising with Black Perlin Noise

## Introduction

"Image denoising is a classical yet still active topic in low level vision since it is an indispensable step in many practical applications. The goal of image denoising is to recover a clean image `x` from a noisy observation `y` which follows an image degradation model `y = x + v`. One common assumption is that `v` is additive white Gaussian noise (AWGN) with standard deviation `σ`"¹. Another possible assumption is that `v` is black Perlin noise. This noise model creates a Perlin noise image with the dimensions of a clean image `x`, sets all pixels in the Perlin noise image to black if they are are less than some noise level and transparent otherwise, then overlays the Perlin noise image on top of the clean image `x` to create a noisy observation `y`. Such noise may approximate the noise in black-and-white documents.

In the paper "Beyond a Gaussian Denoiser: Residual Learning of Deep CNN for Image Denoising," Kai Zhang, Wangmeng Zuo, Yunjin Chen, Deyu Meng, and Lei Zhang propose the denoising convolutional neural network (DnCNN). "Rather than directly outputing the denoised image `xˆ`, the...DnCNN is designed to predict the residual image `vˆ`, i.e., the difference between the noisy observation and the latent clean image"¹. A DnCNN may be trained to remove noise from black-and-white documents by removing the predicted residual image `vˆ` which is assumed to be black Perlin noise.

For images with black Perlin noise, a DnCNN may be designed to predict if a black pixel is black Perlin noise or not for all pixels in the image. Designing a DnCNN to predict if other pixels are black Perlin noise or not is not necessary because only black pixels may be black Perlin noise. To design a DnCNN as such, the pepper noise error can be adopted as the loss function to learn the trainable parameters in the DnCNN. The pepper noise loss can be described as...

```
l = w ([y = b] * (v × log(vˆ) + (1 - v) × log(1 - vˆ)))
```

...where `l` is the pepper noise loss, `w` is a manual rescaling weight given to the loss of each batch element, `y` is the noisy observation, `b` is the value for black pixels, `v` is the residual image, `vˆ` is the predicted residual image, and `[y = b]` is the [Iverson bracket](https://en.wikipedia.org/wiki/Iverson_bracket) of the statement `y = b`. The mean of the pepper noise loss would be `sum(l) / sum([y = b])`. The sum of the pepper noise loss would be `sum(l)`.

## Experimental Results

For black Perlin noise denoising with unknown grid cell dimensions and unknown noise level, 3261 images of various sizes were used for training and 14 images of size 1700 × 2200 were used for validation. The images used for training are from 196 papers from [Denoising | Papers with Code](https://paperswithcode.com/task/denoising). The images used for validation are from the paper "No-Reference Image Quality Assessment in the Spatial Domain" by Anish Mittal, Anush Krishna Moorthy, and Alan Conrad Bovik. The data sets may be downloaded at [datasets.zip - Google Drive](https://drive.google.com/file/d/10-ZuuEosZlnq6aIlbR__qoogUOJfznDF/view?usp=sharing).

To train a single DnCNN model for black Perlin noise denoising with unknown grid cell dimensions and unknown noise level, the unknown grid cell dimensions `(x, y)` were set to factors of the dimensions of a clean image (patch) and the range of the noise level was set to [-1, 1]. The depth of the DnCNN was set to 30, giving the DnCNN a receptive field of 61 × 61. The weights of the DnCNN were initialized by the method in "Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification" by Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. The optimizer was set to RMSprop with the learning rate 1×10^-2, the momentum factor 0, the smoothing constant 0.99, the numerical stability term 1×10^-8, and the weight decay 0. The batch size was set to 128. The epochs were set to 2000. The models may be downloaded at [models.zip - Google Drive]().

For any sample in any minibatch, that sample was a noisy-residual image patch pair `(y, v)`. All noisy-residual image patch pairs `(y, v)` were created by selecting a clean image `x`, creating a clean image patch `z` by cropping the clean image `x` at a random location to the size 70 × 70, creating a residual image patch `v` of size 70 × 70, then creating a noisy observation patch `y` by "adding" the residual image patch `v` to the clean image patch `z`.

A noisy observation `y` was created by "adding" black Perlin noise noise `v` with grid cell dimensions `(x, y) = (425, 440)` and noise level -0.5 to an image from validation data set. After 1868 epochs, the DnCNN denoised the noisy observation `y` as such:

| Noisy Observation | Denoised Image    |
| ------------------| ----------------- |
| ![Noisy Observation](https://drive.google.com/uc?export=view&id=1zahHOTf3Xn78nh8aqlfP-q5i6HR0OUNh "Noisy Observation") | ![Denoised Image](https://drive.google.com/uc?export=view&id=18vr0n4aUnEsQ5h5g1cvMJ5tEC6xu-AKd "Denoised Image") |

After 513 epochs, the DnCNN denoised the image from [vU4m6.png (2544×3279)](https://i.stack.imgur.com/vU4m6.png) as such:

| Noisy Observation | Denoised Image    |
| ------------------| ----------------- |
| ![Noisy Observation](https://drive.google.com/uc?export=view&id=1-mh8G8GYEiFlXwP3ecuOzmX46_vIFT60 "Noisy Observation") | ![Denoised Image](https://drive.google.com/uc?export=view&id=1i26YWRFcBfg-Y-NnhMxPhLDPWq5OKWsH "Denoised Image") |

## References

1. Zhang, Kai, Wangmeng Zuo, Yunjin Chen, Deyu Meng, and Lei Zhang. "Beyond a Gaussian Denoiser: Residual Learning of Deep CNN for Image Denoising." August 2016. https://arxiv.org/pdf/1608.03981v1.pdf.

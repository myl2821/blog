---
title: 使用OpenCV对图片做卡通化处理
layout: post
guid: urn:uuid:54332b9b-b6c4-4875-a9cb-ccb2ac88281f
tags:
  - CV 
---

首先请出这次的模特，享誉全球的Lena小姐：
![Ladder](/media/files/2014/07/28/Lena.jpg)

对一张真实的图片做卡通处理，主要分为三个部分:

* 对原始图片的平坦部分做平滑处理
* 检测出原始图片的轮廓,并抽出线条
* 对以上两个图层做叠加

------------------------------------

## 轮廓检测

为了检测出轮廓，首先需要保持轮廓完整的同时用平滑滤波器对其余部分做平滑处理，紧接着再用一个轮廓滤波器就可以过滤出轮廓。

平滑的目的是为了减少轮廓检测时产生的噪声，采用的平滑方法是中值滤波，中值滤波不仅可以去除孤点噪声，而且可以保持图像的边缘特性，不会使图像产生显著的模糊，比较适合于实验中的人脸图像。

检测轮廓使用拉普拉斯算子过滤器，由于拉普拉斯算子过滤器使用灰度进行标度，我们还需要将BGR图片转为灰度图。拉普拉斯过滤器进行轮廓检测的原理其实也很简单，当其检测到某个像素的灰度与周围点发生较大变化时，就将其标记为轮廓。这个阈值可使用预设值也可以手动设定。

了解了流程，处理的部分也就很简单了:

    cvtColor(src, gray, CV_BGR2GRAY);
    const int MEDIAN_BLUR_FILTER_SIZE = 7; 
    medianBlur(gray, gray, MEDIAN_BLUR_FILTER_SIZE);
    Mat edges;
    const int LAPLACIAN_FILTER_SIZE = 5;
    Laplacian(gray, edges, CV_8U, LAPLACIAN_FILTER_SIZE);
    Mat mask;
    const int EDGES_THRESHOLD = 80;
    threshold(edges, mask, EDGES_THRESHOLD, 255, THRESH_BINARY_INV);

来检测一下处理的结果:
![Ladder](/media/files/2014/07/28/Lena_mask.jpg)


## 双边滤波

双边滤波器可以在保持轮廓的前提下平滑其余部分并去除噪声，非常适合此次任务。

但双边滤波的缺点是速度太慢，所以我们先收缩图片的大小，在缩小后的图片上进行滤波，再将其放大至原来的大小。这与直接对原图进行滤波所产生的结果是相似的(反正都会变模糊:P)。

    Mat tmp = Mat(smallSize, CV_8UC3);
    int repetitions = 7;  // Repetitions for strong cartoon effect.
    for (int i = 0; i<repetitions; i++) {
        int ksize = 9; // Filter size.  
        sigmaColor = 9; // Filter color strength.
        double sigmaSpace = 7; // Spatial strength. 
        bilateralFilter(smallImg, tmp, ksize, sigmaColor, sigmaSpace);         
        bilateralFilter(tmp, smallImg, ksize, sigmaColor, sigmaSpace);
￼￼}


![Ladder](/media/files/2014/07/28/Lena_filter.jpg)

## 叠加

用copyTo将两个矩阵叠加到一个新的矩阵即可

    bigImg.copyTo(dst, mask);

![Ladder](/media/files/2014/07/28/Lena_cartoon.jpg)

确实还有那么点感觉，卡通画的程度和边缘线条的粗重都是可以再调整的哦:P

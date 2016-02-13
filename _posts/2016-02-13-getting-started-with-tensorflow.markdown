---
layout: post
title: "Getting Started with TensorFlow"
date: 2016-02-13
---

It has been a while since I've posted anything. Wow, has it really been five years... that's a very long while. When I last posted I was mostly focused on languages and compilers. I'm still really interested in these things. I have some ideas that I'd love to come back to some day, but for the moment my attention has shifted to machine learning and in particular deep learning which has seen some really impressive advances in the last five years or so. As a part of this I took the popular [Coursera Machine Learning course](https://www.coursera.org/learn/machine-learning). It's popular for good reason. Professor Ng is a leader in the field but more importantly he has a knack for explaining complicated concepts in a way that emphasizes building intuition. It's also clear that he is sincerely invested in the success of his students, so it's a course I'd definitely recommend it for anyone who wants to either learn or review the essentials of Machine Learning. While deep learning is not covered in this course it does prepare you to tackle deep learning material on your own (see bottom of post for some references). One warning though: if you're fairly advanced in mathematics and hoping for proofs and a detailed technical treatment of the underlying mathematical mechanisms you're not going to find that in this course.

## So... TensorFlow

I haven't gotten very far with TensorFlow yet and so I just want to write down some of my experience with the install in case it's useful to anyone out there. The system requirements for TensorFlow are really limited (it definitely shows that this was an internally deployed product with very specific hardware requirements). Only mac and a specific version of Ubuntu are supported platforms. If you want GPU acceleration Ubuntu is your only choice. On a positive note the documentation is really useful and fairly complete from what I've seen so far. For reasons that aren't important I was limited to installing on OpenSUSE and 13.2 is the most recent version that NVidia supports the required version of CUDA on. Checking out and building TensorFlow on OpenSUSE *almost* works as described in the TensorFlow docs even though it isn't a supported configuration. The one exception I ran into is that you have to update the bazel CROSSTOOL files as explained here: http://stackoverflow.com/q/35256110/160015

I built TensorFlow with, and without GPU support in order to get an idea for what kind of performance difference the GPU could make. I'm developing on a Lenovo IdeaCentre K450 which has an Intel Core i7 4770 3.4 GHz processor and an NVIDIA GeForce GTX 650 graphics card. This card has a [CUDA Compute Capability](https://developer.nvidia.com/cuda-gpus) of 3.0 which is not officially supported by TensorFlow, but fortunately they provide a non-standard build feature that allows you to use a 3.0 card. As documented on the TensorFlow site you need to build using:

{% highlight shell %}
TF_UNOFFICIAL_SETTING=1 ./configure
# answer 3.0 for CUDA Compute Capability version when prompted
bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
{% endhighlight %}

In order to check the performance increase (I won't call this a benchmark since this check is in no way rigorous to the level a benchmark should be) that you can get from enabling GPU support I ran:

{% highlight shell %}
(tf-nogpu)kss@linux-9c32:~/tmp/no-gpu-bench> time python -m tensorflow.models.image.mnist.convolutional
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
Initialized!
Step 0 (epoch 0.00), 1.7 ms
Minibatch loss: 12.054, learning rate: 0.010000
Minibatch error: 90.6%
Validation error: 84.6%
Step 100 (epoch 0.12), 164.2 ms
Minibatch loss: 3.306, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 7.0%

...

Step 8500 (epoch 9.89), 169.1 ms
Minibatch loss: 1.604, learning rate: 0.006302
Minibatch error: 0.0%
Validation error: 0.9%
Test error: 0.8%

real    24m21.986s
user    127m42.442s
sys     11m46.360s
{% endhighlight %}

then with GPU acceleration enabled:

{% highlight shell %}
(tf-gpu)kss@linux-9c32:~/tmp/gpu-bench> time python -m tensorflow.models.image.mnist.convolutional
I tensorflow/stream_executor/dso_loader.cc:105] successfully opened CUDA library libcublas.so.7.0 locally
I tensorflow/stream_executor/dso_loader.cc:105] successfully opened CUDA library libcudnn.so.6.5 locally
I tensorflow/stream_executor/dso_loader.cc:105] successfully opened CUDA library libcufft.so.7.0 locally
I tensorflow/stream_executor/dso_loader.cc:105] successfully opened CUDA library libcuda.so locally
I tensorflow/stream_executor/dso_loader.cc:105] successfully opened CUDA library libcurand.so.7.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:900] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
I tensorflow/core/common_runtime/gpu/gpu_init.cc:102] Found device 0 with properties:
name: GeForce GTX 650
major: 3 minor: 0 memoryClockRate (GHz) 1.0585
pciBusID 0000:01:00.0
Total memory: 1.99GiB
Free memory: 1.61GiB
I tensorflow/core/common_runtime/gpu/gpu_init.cc:126] DMA: 0
I tensorflow/core/common_runtime/gpu/gpu_init.cc:136] 0:   Y
I tensorflow/core/common_runtime/gpu/gpu_device.cc:713] Creating TensorFlow device (/gpu:0) -> (device: 0, name: GeForce GTX 650, pci bus id: 0000:01:00.0)
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 1.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 2.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 4.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 8.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 16.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 32.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 64.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 128.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 256.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 512.0KiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 1.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 2.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 4.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 8.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 16.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 32.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 64.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 128.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 256.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 512.00MiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 1.00GiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:51] Creating bin of max chunk size 2.00GiB
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:73] Allocating 1.42GiB bytes.
I tensorflow/core/common_runtime/gpu/gpu_bfc_allocator.cc:83] GPU 0 memory begins at 0x600b80000 extends to 0x65b733000
Initialized!
Step 0 (epoch 0.00), 21.2 ms
Minibatch loss: 12.054, learning rate: 0.010000
Minibatch error: 90.6%
Validation error: 84.6%
Step 100 (epoch 0.12), 55.1 ms
Minibatch loss: 3.293, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 7.3%

...

Step 8500 (epoch 9.89), 55.8 ms
Minibatch loss: 1.620, learning rate: 0.006302
Minibatch error: 1.6%
Validation error: 0.8%
Test error: 0.8%

real    8m22.237s
user    2m49.802s
sys     1m28.485s
{% endhighlight %}

I am really happy with the GPU speed up. TensorFlow took 24m21.986s to fit the model just using the CPU (with all 8 virtual cores holding steady at about 75% utilization throughout) and 8m22.237s using the GPU for an almost 3x speedup. OK, so that's where I'm at for now. I plan on spending more time getting familiar with how to solve real problems using TensorFlow and expect to be posting on that experience pretty soon. Hopefully that will be more interesting than this post.

## Some Machine Learning/Deep Learning Resources

* [Coursera Machine Learning course](https://www.coursera.org/learn/machine-learning)
* [Deep Learning](http://www.deeplearningbook.org/): this book is a great resource. I haven't worked my way through the whole thing but I've already gotten a lot out of the chapters that I have read. Draft versions of the book are freely accessible for the moment but I'll probably buy a final version when it's published, in part to support the authors' efforts.

The following links look promising but I haven't yet had a chance to dig into these:

* [Stanford CS231n: Convolutional Neural Networks for Visual Recognition](http://networkflow.net/forums/forum/19-stanford-cs231n-convolutional-neural-networks-for-visual-recognition/): the [github page for the course](http://cs231n.github.io/) seems worth looking into even if you don't plan on watching the lectures
* [Stanford Deep Learning Tutorial](http://ufldl.stanford.edu/tutorial/)
* [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/): free online book
* [Understanding Neural Networks Through Deep Visualization](http://yosinski.com/deepvis)
* [Neural Network Data Normalization and Encoding](https://visualstudiomagazine.com/articles/2013/07/01/neural-network-data-normalization-and-encoding.aspx)

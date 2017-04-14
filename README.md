This is a caffe version implementation of a hash network(DNNH/NINH) for similarity-based visual research.

The hash network is based on this paper:
[Hanjiang Lai, Yan Pan, Ye Liu, and Shuicheng Yan. Simultaneous feature learning and hash coding with deep neural networks, CVPR 2015](https://arxiv.org/abs/1504.03410).

# My work
* **Deploy:** Given the definition of loss layer, deploy the deep hashing pipeline on linux.
* **Train:** Write prototxt to define dnnh and bash files to execute for training on preprocessed triplet CIFAR-10 dataset.
* **Test/Evaluate:** Write prototxt to encode images and bash files to execute for image retrieval. Implement the metric of mean average precision (mAP) for evaluation.
* **Analysis:** Draw performances for 12-bit, 24-bit and 48-bit hash code and make some analysis.

# How to run
## Deploy
You may directly download my caffe-dnnh zip and deploy (may need to fix errors dut to different environment and version). Or you can follow the instructions to add files/contents to the [newest caffe release](https://github.com/BVLC/caffe). Here `CAFFE-ROOT` refers to your root caffe directory and `caffe-dnnh` to mine.
1. Add file `caffe-dnnh/src/caffe/layers/triplet_ranking_hinge_loss_layer.cpp` to path `CAFFE-ROOT/src/caffe/layers` and file `caffe-dnnh/include/caffe/layers/triplet_ranking_hinge_loss_layer.hpp` to path `CAFFE-ROOT/include/caffe/layers`.
2. Modify file `CAFFE-ROOT/src/caffe/proto/caffe.proto`:
   * Add the following code directly.
``` cpp
// Message that stores parameters used by TripletRankingHingeLossLayer
message TripletRankingHingeLossParameter{
   //Dimension for computing
   optional int32 dim = 1 [default = 10];
   //Margin
   optional float margin = 2 [default = 1];
}
```
   * Find `message LayerParameter`, add `optional TripletRankingHingeLossParameter triplet_ranking_hinge_loss_param = 151;` in it.
   * Find `message V1LayerParameter`, add `optional TripletRankingHingeLossParameter triplet_ranking_hinge_loss_param = 43;` in it.
   * Find `enum LayerType` in `message V1LayerParameter`, add `TRIPLET_RANKING_HINGE_LOSS=40;` in it.
Attention: the number above like 151, 43 are ID and should not be conflict with others. Search `next available` in `caffe.proto` you will find comment like `// SolverParameter next available ID: 42 (last added: layer_wise_reduce)` and `// LayerParameter next available layer-specific ID: 147 (last added: recurrent_param)`. Use next available ID and update the comment.
3. Add folder `caffe-dnnh/runtime` to path `CAFFE-ROOT/`.
4. Modify file `CAFFE-ROOT/tools/caffe.cpp` refer to `caffe-dnnh/tools/caffe.cpp`: Search `++++++++++` in `caffe-dnnh/tools/caffe.cpp` and you will find what I add.

Attention: For CPU/GPU mode switch
1. check `CPU_ONLY := 1` in `CAFFE-ROOT/Makefile.config`
2. In folder `CAFFE-ROOT/runtime/`: check `solver_mode: GPU` in all `solver.prototxt` files (e.g. `CAFFE-ROOT/runtime/12bit/train12_solver.prototxt`), check `-gpu=0` in all `run_test.sh` files (e.g. `CAFFE-ROOT/runtime/12bit/run_test.sh`)

Then follow the official [Installation instructions](http://caffe.berkeleyvision.org/installation.html) to compile. Good luck!

## Train
``` bash
cd caffe-dnnh/runtime/12bit # or: 24bit, 48bit
sh ./run_train.sh # or: sh ./resume_train.sh
```

`run_train.sh` train deep hash neural network defined in prototxt and result models are stored in path `caffe-dnnh/runtime/model`. Read corresponding files for more details. You can modify parameters/paths or anything else.

## Test
``` bash
cd caffe-dnnh/runtime/12bit # or: 24bit, 48bit
sh ./run_test.sh
```

`run_test.sh`: uses forward pass of dnnh defined in `test12_query.prototxt` and `test12_pool.prototxt` to encode query images and pool set images. Then compile and run `CAFFE-ROOT/runtime/evaluate_map.cpp` for image retrieval evaluation. Read corresponding files for more details. You can modify parameters/paths or anything else.




# Credit
1. [Dr.Tao Mei](https://www.microsoft.com/en-us/research/people/tmei/) draw an outline of this research for me.
2. The triplet ranking hinge loss layer is implemented by @FuchenUSTC in [his caffe repository](https://github.com/FuchenUSTC/caffe).
3. Preprocessed triplet CIFAR-10 dataset in `caffe-dnnh/runtime/cifar_hash_dataset` is shared by @FuchenUSTC. Read [my post#dataset]() for more details about its structure so as to understand the structure of DNNH defined in prototxt.
4. Networks structure and parameters are refered to [codes_triplet_hashing1.zip](http://www.scholat.com/portaldownloadFile.html?fileId=4909) provide by first author [Hanjiang Lai](http://www.scholat.com/laihanj).


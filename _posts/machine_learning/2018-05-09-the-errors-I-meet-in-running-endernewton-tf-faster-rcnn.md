---
layout: post
title: 我在运行tf-faster-rcnn(endernewton)时遇到的错误
---
这次就不用蹩脚的英文了。讲讲我在运行[endernewton大佬的代码](https://github.com/endernewton/tf-faster-rcnn)时候遇到的错误，尽管很多都在他的issue里可以找到解决方法，但是没人整理啊，也没有中文呀——强行让自己写的东西有意义。

## 提示
该程序训练**后**会**自动执行**测试验证脚本。重复执行训练脚本无妨，因为有存档点，每5000次存一次。total 70000 iters。所以，训练完后执行训练脚本等于直接测试。
## 1. bash变量赋值和time指令在同一行导致
```bash
./experiments/scripts/train_faster_rcnn.sh: line 73: time: command not found
```
#### 解决方法
修改```./experiments/scripts/train_faster_rcnn.sh```的62行和72行（在```time```前添加```&&```），改变为:
```bash
CUDA_VISIBLE_DEVICES=${GPU_ID} && time python ./tools/trainval_net.py \
```
P.S.同样的问题在```test_faster_rcnn.sh```中的58行和67行也存在，解决方法相同 。否则，你会在测试的时候遇到一样的问题。

## 2. 忘了安装Python COCO API
```bash
ImportError: No module named 'pycocotools'
```
总之，不是不跑coco数据就不必安装，记得安装就好
#### 解决方法
```bash
cd data
git clone https://github.com/pdollar/coco.git
cd coco/PythonAPI
make
cd ../../..
```

## 3. 显存资源耗尽
```bash
ResourceExhaustedError (see above for traceback): OOM when allocating tensor with shape[512,512,3,3]
```
这是发生在测试的时候。一般说来，你在修改安装脚本的时候选了对应合适的架构(我用的是GTX 1080 (Ti)，对应sm_61)，理应不会发生这种情况的。有个用1070的老哥训练时出现这个情况，跑去找作者提[issue#286](https://github.com/endernewton/tf-faster-rcnn/issues/286)，作者直接建议换个好卡😜。因为gpu的占用会随不同尺寸的图像而改变。

至于我在测试的时候遇到这个问题，有两个可能的原因。一、我是租用的3块一小时的共享GPU云，显存被别人占用了？我觉得这个原因比较小，因为云提供商说的是，后面的任务会顶掉前面的任务。二、有趣的来了。这个小姐姐？的[issue#7](https://github.com/endernewton/tf-faster-rcnn/issues/7)也是在测试的时候遇到内存不够，分配错误的问题，但她说她没有用GPU干其他别的任何事情。有趣的是，小姐姐在同一天关闭了issue并tensorflow官方git下的[issue#5816](https://github.com/tensorflow/tensorflow/issues/5816)下面引用了这个issue。小姐姐自己认为是Jupyter notebook开着的原因导致的。事实上不是，我一直开着的，最后也成功了。感谢小姐姐带我到issue#5816这里。妈的，官方这么回复的：
> It's possible to have non-deterministic "out of memory" because you are unlucky. TensorFlow has non-deterministic order of execution, so depending on timing, you may have things scheduling in different order and needing different amounts of memory.

什么意思呢？TensorFlow的执行顺序具有不确定性。这是个bug，导致——如果你遇到"out of memory"就是你太霉了，该去去晦气了——找两条锦鲤来转发吧。当然，我们在自己写的时候可以用```tensorflow.contrib.graph_editor```来避免这种不确定性。

#### 解决方法
多运行几遍，总有一次可以成功的。我第一次测试就遇到两次😓，之后因为其他错误重新运行还遇到过一次。

## 4. 在记录每个类目测试数据的时候找不到文件
```bash
FileNotFoundError: [Errno 2] No such file or directory: '/root/tf-faster-rcnn/data/VOCdevkit2007/results/VOC2007/Main/
```
找不到目录就建立一个呗
#### 解决方法
参照[issue#246](https://github.com/endernewton/tf-faster-rcnn/issues/246)
```bash
mkdir -p /root/tf-faster-rcnn/data/VOCdevkit2007/results/VOC2007/Main/
```
多一嘴，记得/root对应你放代码的位置，/tf-faster-rcnn但愿你没改名字。

## 5. 代码中的类型错误
```bash
TypeError: write() argument must be str, not bytes
```

#### 解决方法
参照[issue#261](https://github.com/endernewton/tf-faster-rcnn/issues/261)
修改```tf-faster-rcnn/lib/datasets/voc_eval.py```文件的121行，把'b'改为'wb'：
```python
with open(cachefile,'wb') as f
```
正如issue261中所提的，还有地方需要修改，不然你会遇到[issue#171](https://github.com/endernewton/tf-faster-rcnn/issues/171)的问题。不知道你听不听劝，我听了，按如下修改：

修改同一个文件的第105行，改为：
```python
cachefile = os.path.join(cachedir, '%s_annots.pkl' % imagesetfile.split("/")[-1].split(".")[0])
```

### 至此，我目前遇到的所有问题解决了。
# FastSpeech2
对国际音标(IPA Chart)下自训练中文数据集的支持

本README重点在于表述如何使用自己的数据集训练一个中文的语音模型，如果想要阅读原有的README，请移步README_original.md  
是在单说话人的基础上进行的修改，如果需要多个说话人，也许需要额外的修改

## 数据准备1 获取音频
下载想要提取说话人声音的视频，然后使用[demucs](https://github.com/facebookresearch/demucs)等工具去除背景音

## 数据准备2 切分音频
获取字幕：我使用的是剪映，对就是那个剪视频的软件，我觉得准确率和速度都可以接受  
然后使用[SoundLabel](https://github.com/kslz/SoundLabel)导入音频和字幕进行切割  
同时修改字幕，将数字转换为汉字写法，英文的话，可以不转，也可以转换成<unk>  
执行到这一步你应该是拥有一个字幕文本文件和一个音频文件夹

## 搭建conda环境
cd到该程序的根路径，搭建一个conda的虚拟环境，下载python（我用的是3.8），激活环境，然后执行
```
pip3 install -r requirements.txt
```

## 数据准备3 获取与wav一对一的字幕文件
在项目根路径创建一个放语料库的文件夹，格式可以corpus_example，然后执行
```
python3 prepare_align.py config/AISHELL3/preprocess.yaml
```
这时你将得到一个raw_data文件夹，里面有你的wav以及每一个与之相对应的lab文字标注

## 数据准备4 mfa对齐
退出当前conda环境，执行
```
 conda create -n aligner -c conda-forge montreal-forced-aligner
```
创建mfa使用的虚拟环境同时安装依赖，然后``conda activate aligner``
激活mfa环境，然后cd到你上一步得到的声音与lab文本所在文件夹的上一级，执行
```
mfa model download acoustic mandarin_mfa
mfa model download dictionary mandarin_mfa
mfa align ~/mfa_data/my_corpus mandarin_mfa mandarin_mfa ~/mfa_data/my_corpus_aligned
```
获取切割好的TextGrid文件，将这些文件放到``preprocessed_data/AISHELL3/TextGrid/0/``当中

## 预处理数据
cd到项目根路径，退出mfa环境并激活fastspeech环境，然后执行
```
python3 preprocess.py config/AISHELL3/preprocess.yaml
```
这时你会在``preprocessed_data/AISHELL3``下得到四个文件夹

## 训练
执行
```
python3 train.py -p config/AISHELL3/preprocess.yaml -m config/AISHELL3/model.yaml -t config/AISHELL3/train.yaml
```
RTX3090上大概不到一个小时可以走20000步

## 验证
执行
```
python3 synthesize.py --text "大家好" --speaker_id 0 --restore_step 600000 --mode single -p config/AISHELL3/preprocess.yaml -m config/AISHELL3/model.yaml -t config/AISHELL3/train.yaml
```
这里的speaker_id 与``preprocessed_data/AISHELL3/speakers.json``的设定有关
restore_step表示使用``output/ckpt/AISHELL3/``里的哪个训练数据

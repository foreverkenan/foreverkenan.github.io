---
layout: post
title:  "数据驱动的住宅建筑室内规划生成"
comments: true
categories: Paper
---

笔记是做成功一遍后，再做第二遍。其中第二遍，边做边记录
# 零、版本
Python 3.9.10
torch 2.0.0+cu118
Qt 5.14.2
PyCharm 2023.1.1
Visual Studio 2019.16
Windows 10 21H2
# 一、准备
0. [Data-driven Interior Plan Generation for Residential Buildings](http://staff.ustc.edu.cn/~fuxm/projects/DeepLayout/index.html)
下载[Code](http://rec.ustc.edu.cn/share/185bdf80-d93e-11e9-820d-33b81cb7a670)，解压，下载[Data set: RPLAN](https://docs.google.com/forms/d/e/1FAIpQLSfwteilXzURRKDI5QopWCyOGkeb_CFFbRwtQ0SOPhEg0KGSfw/viewform?usp=sf_link)，解压
1. `deep_layout`文件夹放在某个位置
2. `deep_layout`文件夹下创建`dataset`文件夹，`dataset`文件夹下创建`train`、`val`文件夹
3. `deep_layout`文件夹下创建`pickle`文件夹，`pickle`文件夹下创建`train`、`val`文件夹
4. `deep_layout/dataset/train`、`deep_layout/dataset/val`文件夹下放入图片数据集，4000train+1000val，约5-10小时
5. PyCharm Open `deep_layout`文件夹，由于我的Python库都提前装在本地，**Settings(Ctrl+Alt+S)** - **...** - **Python Interpreter** - **Add Interpreter** - **Add Local Interpreter**，勾选**Inherit global site-packages**
6. Run `write_pickle.py`，其中`deep_layout/pickle/train`、`deep_layout/pickle/val`文件夹应该生成和数据集名字、数量对应的 **.pkl** 文件
7. 依次打开`deep_layout/train/xxx`文件夹（其实先后无关），除了`pretrained_model`文件夹，共训练4个
8. `config_continue.py`中
```python
train_data_root = '../pickle/train_dataset'
val_data_root = '../pickle/val_dataset'
// =>
train_data_root = '../../pickle/train'  
val_data_root = '../../pickle/val'
```
8. `train_continue.py`中
```python
score_connect = connect(score_model)  
loss = criterion(score_connect, target)
// =>
loss = criterion(score_connect, target)
target = target.type(t.int64)
loss = criterion(score_connect, target)  
```
9. Run `train_continue.py`，确保无报错，`PyCharm Terminal`中
```
cd train/Continue
python train_continue.py train
```
并等待
 10. `config_living.py`同8 `config_continue.py`
 11. Run `train_living.py`，确保无报错，`PyCharm Terminal`中
```
cd ../Living
python train_living.py train
```
并等待
12. `config_location.py`同8 `config_continue.py`
13. Run `train_location.py`，确保无报错，`PyCharm Terminal`中
```
cd ../Location
python train_location.py train
```
并等待
14. `config_wall.py`同8 `config_continue.py`
15. Run `train_wall.py`，确保无报错，`PyCharm Terminal`中
```
cd ../Wall
python train_wall.py train
```
并等待
16. 全部计算完成后，`deep_layout/train/xxx/checkpoints/yyy.pth`中，八个 **.pth** 复制到`deep_layout/synth/trained_model`
```
living_fc1_300.pth, living_resnet34_300.pth continue_fc2_300.pth, continue_resnet34_300.pth location_up1_100.pth, location_resnet34_100.pth wall_up1_100.pth, wall_resnet34_100.pth
```
17. `synth.py`中
```python
start_time = time.clock() //line 264
// =>
start_time = time.perf_counter()
```
```python
end_time = time.clock() //line 284
// =>
end_time = time.perf_counter()
```
```python
end_time = time.clock() //line 288
// =>
end_time = time.perf_counter()
```
18. `floorplan_synth.py`中
```python
self.boundary = t.from_numpy(floorplan[:,:,0])
self.inside = t.from_numpy(floorplan[:,:,3])
// =>
self.boundary = floorplan[:, :, 0]
self.inside = floorplan[:, :, 3]
```
19. Run `synth.py`，其中`deep_layout/synth/synth_output`文件夹应该生成与`deep_layout/synth/synth_input`文件夹对应图片，输出为绿色线条，[2022年（第十届）中国科学技术大学《计算机图形学》暑期课程](https://www.bilibili.com/video/BV1oZ4y1Y7bM?p=12&vd_source=45823cc7f3ac505694c035dd264cf594) P12 1:15:24，没有绿线说明训练数据集太少或其他未知问题
20. `vectorization.py`中
```python
start_time = time.clock() //line 755
// =>
start_time = time.perf_counter() //line 755
```
```python
end_time = time.clock() //line 778
// =>
end_time = time.perf_counter()
```
```python
end_time = time.clock() //line 782
// =>
end_time = time.perf_counter()
```
21. Run `vectorization.py`，`deep_layout/synth_normalization`生成矢量化图片
22. `deep_layout/visualization/Deeplayout/Deeplayout.vcxproj.user`中
```
<QTDIR>F:\Qt5.5.1\5.5\msvc2013_64</QTDIR> //line 5、9、13、17
// =>
<QTDIR>C:\Qt\Qt5.14.2\5.14.2\msvc2017_64</QTDIR> //example
```
23. Visual Studio Open `deep_layout/visualization/Deeplayout.sln`，如果上一步已经打开Visual Studio则重启Visual Studio
24. Qt加入**环境变量** - **系统变量** - **Path**
```
C:\Qt\Qt5.14.2\5.14.2\msvc2017_64\bin //example
```
25. 运行
26. Open 矢量化图片 Render
# 二、Houdini
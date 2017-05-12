图像采集及标注
------------------------------------
安装libfreenect2
------------------------------------
[https://github.com/OpenKinect/libfreenect2](https://github.com/OpenKinect/libfreenect2 "GitHub-libfreenect2")

若安装成功，即运行Demo程序`./bin/Protonect`，则可出现对应图像

安装对应python库pylibfreenct2
-------------------------------------

[https://github.com/r9y9/pylibfreenect2](https://github.com/r9y9/pylibfreenect2 "pylibfreenect2")

检查是否安装成功，可在terminal内运行python环境，即输入`python`回车，`import pylibfreenect2`没有报错则说明安装成功

若出现找不到编译好的文件，可以`export LD_LIBRARY_PATH=$HOME/freenect2/lib:$LD_LIBRARY_PATH`

采集图像
-------------------------------

- 下载公共文件夹\tmp下的bbox-label-tools
- 切换路径至该目录下，`python collect_a_frame.py`执行collect_a_frame的python文件，该文件调用libfreenect2的python接口显示当前图像，通过按键控制可保存图像，按键盘`f`键可保存RGB图像、ir图像、depth图像以及点云信息至默认文件夹，按键`q`可退出程序
- 可通过传递参数设定图像保存目录，如`python collect_a_frame.py --dir "home/iiie/pic/crayon" --num 100`可将图像保存至/home/iiie/pic/crayon目录下，起始的彩色图像序号为000100.jpg
##裁剪RGB图像##
- 在相应对象文件夹如crayon下创建crop_rgb文件，用于存放裁剪后的RGB图像
- 切换至bbox-label-tools目录下
- 运行cropto300文件，即`python cropto300.py`得到crop_rgb目录下裁剪后的图像
- 若裁剪移位可通过更改程序中的参数[120:420][300:600]来调整裁剪的范围，但需保持裁剪后的图像尺寸为300*300。
##人工标注RGB图像##
- 将crayon下的crop_rgb软链接到bbox-label-tools/Images下，并以三位数字命名，软连接命令为`ln -s 原地址 目的地址`
- 切换至bbox-label-tools目录下
- 运行create_bbox文件，即`python creat_bbox.py`，若出现缺少ImageTk插件提示，则安装该插件，然后在该界面下标注目标对象。先选择对象类别，再标注，标注生成的信息会保存在label文件夹下的对应目录中。

- 将标注生成的.txt文件转换成.xml文件：将需要参与转换的对应图像放到JPEGImages目录下，切换至bbox-label-tools目录，运行create_xml.py即可，即`python create_xml.py`，会在Annotations目录下生成JPEGImages中所有图像对应的.xml文件

- 生成ImageSet中Main下的.txt文件：执行generate_dataset.py，根据需求带需要的参数
 - `python generate_dataset.py --start-index 1 --end-index 1400 --test-pro 0.04 --val-pro 0.48`，表示编号1-1400的图像（单类别），其中，test的图像为56张，trainval的图像为1344，train的图像为672张，val的图像为672张。
 - `python generate_dataset.py --start-index 1401 --end-index 1600 --test-pro 0.72 --val-pro 0.14`，表示编号为1401-1600的图像（多类别），其中，test的图像为144张，trainval的图像为56张，train的图像为28张，val的图像为28张。
 - `python generate_dataset.py --start-index 1601 --end-index 3200 --test-pro 0.04 --val-pro 0.48`，表示编号为1601-3200的图像（单类别），其中，test的图像为64张，trainval的图像为1536张，train的图像为768张，val的图像为768张。
 - `python generate_dataset.py --start-index 3201 --end-index 3400 --test-pro 0.72 --val-pro 0.14`，表示编号为3201-3400的图像（多类别），其中，test的图像为144张，trainval的图像为56张，train的图像为28张，val的图像为28张。
 - 这四条命令按顺序执行则会在最终Main中得到随机抽取的按顺序排列的相应图像序列组。

整理成Pascal VOC2007
---------------------------------

参照共享文件夹中dataset目录下的VOC2007，根据我们现有的数据制作对应的15个类别的VOC2007（为了不冲突，可参照其余文件夹命名方式，将原有的VOC2007重命名），制作好的VOC2007上传到共享文件夹下的dataset目录下，就此完成数据集制作的过程。后面拓展更多的类，步骤也是相似。

SSD网络参数训练
-------------------------------------

- ssh以iiie的用户名远程登录工作站
- 切换目录至our-mxnet-ssd/data/devkit下，删除该目录下的软连接VOC2007，重新建立新的软连接，连接刚制作的/home/share/dataset/VOC2007数据集

网络训练
-------------------------------------

- 切换至our-mxnet-ssd目录，运行训练程序`python train.py --year 2007 --end-epoch 300 --log "./log/LogFor15Cls/train.log"`，表示采用VOC2007的数据，迭代300个训练周期，训练的日志trian.log保存在/our-mxnet-ssd/log/LogFor15Cls目录下，迭代300个周期可能需要6小时左右，也可适当减少迭代次数如100次，看看效果。训练得到的相关参数文件自动保存在model目录下。

网络评估
-------------------------------------

- 进入cache/voc_2007_test/目录，删除annotations.pkl文件
- 切换至our-mxnet-ssd目录，运行评估程序`python evaluate.py --epoch 300`，表示加载迭代300次后的参数文件对test中的图像进行检测评估网络预测的准确程度。terminal上会显示出评估的结果

预测结果可视化
--------------------------------------

- 进入result目录，新建文件夹ResultFor15Cls，进入该目录，新建文件夹（用于保存此次训练的预测结果）
- 打开脚本`/ssd_mxnet/detect/detector.py`
- 改148行`fig.savefig('results/ResultFor15Cls/'+ path.split('/')[-1])`中的`ResultFor15Cls`为保存该次预测结果的文件夹名称，并保存
- 切换目录至`/ssd_mxnet`，带参运行demo程序，`python demo.py --epoch 300`，即可在result目录下的对应文件夹中生成各图预测的可视化结果

需要保存的参数
--------------------------------------
- `/ssd_mxnet/model/ssd_300-0300.params`
- `/ssd_mxnet/model/ssd_300-symbol.json`
- `/ssd_mxnet/log/XX.log`
- (optional)切换目录至`/ssd_mxnet`，运行`python deploy.py --epoch 300`，然后保存生成的两个文件
  - `/ssd_mxnet/model/deploy_ssd_300-0300.aprams`
  - `/ssd_mxnet/model/deploy_ssd_300-symbol.json`

- （optioanl）检测结果
 - `/ssd_mxnet/result/XXX`
 - `/ssd_mxnet/data/VOCdevkit/results/VOC2007/Main`
 


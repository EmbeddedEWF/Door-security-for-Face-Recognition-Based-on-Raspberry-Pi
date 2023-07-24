# 树莓派人脸识别智能门锁

## 1、人脸识别API的注册与人脸库管理

打开百度智能云https://cloud.baidu.com/来进行录入人脸信息

1

选择人脸识别，进去之后选择立即使用

点击去创建

2

3

4

点击立即创建，下载SDK，在树莓派上所以选择python

5

再在上一级菜单中选择可视化人脸库

点击进入刚刚创建的应用，马上创建用户组，填写好信息

6

确认后选择用户组

7

点击01，再点击马上创建

用户ID为wuyanzu，选在图片，点击确认

8

在应用列表中将信息记录到记事本中

9

10

这里的这些数据就相当于将这个图片构造了一个key的信息



## 2、树莓派本地环境搭建

将SDK放到树莓派中，这也就是相当于一个IDE一样，大的文件包来检测图片的数据

解压缩包

```c
unzip aip-python-sdk-4.16.11.zip
```

进入aip

使用命令安装百度AI模块

```c
sudo pip install baidu-aip
```

接着安装下载的SDK

```c
sudo python3 setup.py install
```

到此树莓派人脸识别本地环境配置好了



## 3、实现检测人脸控制灯亮

在当前文件下创建一个文件test.py

将自己生成的key.txt文件中记录的百度人脸识别API账号信息APP_ID、API_KEY、SECRET_KEY更换为自己的图片信息，用户组GROUP = '01'和name == 'wuyanzu'更新为自己的信息

```python
from aip import AipFace
from picamera import PiCamera
import urllib.request
import RPi.GPIO as GPIO
import base64
import time


#百度人脸识别API账号信息
APP_ID = '36629406'
API_KEY = 'AGQltv7KRBzp8PansjdbC80L'
SECRET_KEY ='8NbWh18Z4hHQLfCjGSjV71mGGBlAku6a'
client = AipFace(APP_ID, API_KEY, SECRET_KEY)#创建一个客户端用以访问百度云
#图像编码方式
IMAGE_TYPE='BASE64'
camera = PiCamera()#定义一个摄像头对象
#用户组
GROUP = '01'
 
#照相函数
def getimage():
    camera.resolution = (1024,768)#摄像界面为1024*768
    camera.start_preview()#开始摄像
    time.sleep(2)
    camera.capture('faceimage.jpg')#拍照并保存
    time.sleep(2)
#对图片的格式进行转换
def transimage():
    f = open('faceimage.jpg','rb')
    img = base64.b64encode(f.read())
    return img
#上传到百度api进行人脸检测
def go_api(image):
    result = client.search(str(image, 'utf-8'), IMAGE_TYPE, GROUP);#在百度云人脸库中寻找有没有匹配的人脸
    if result['error_msg'] == 'SUCCESS':#如果成功了
        name = result['result']['user_list'][0]['user_id']#获取名字
        score = result['result']['user_list'][0]['score']#获取相似度
        if score > 80:#如果相似度大于80
            if name == 'wuyanzu':
                print("欢迎%s !" % name)
                time.sleep(1)
            if name == 'xiaohua':
                print("欢迎%s !" % name)
                time.sleep(3)
            if name == "xiaohong":
                print("欢迎%s !" % name)
                time.sleep(3)
            if name == "xiaoyu":
                print("欢迎%s !" % name)
                
        else:
            print("对不起,我不认识你!")
            name = 'Unknow'
            return 0
        curren_time = time.asctime(time.localtime(time.time()))#获取当前时间
 
        #将人员出入的记录保存到Log.txt中
        f = open('Log.txt','a')
        f.write("Person: " + name + "     " + "Time:" + str(curren_time)+'\n')
        f.close()
        return 1
    if result['error_msg'] == 'pic not has face':
        print('检测不到人脸')
        time.sleep(3)
        return -1
    else:
        print(result['error_code']+' ' + result['error_code'])
        return 0
#主函数
if __name__ == '__main__':
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(17, GPIO.OUT)
    while True:
        
        print('准备开始，请面向摄像头 ^_^')

        if True:
            getimage()#拍照
            img = transimage()  #转换照片格式
            res = go_api(img)   #将转换了格式的图片上传到百度云
            if(res == 1):       #是人脸库中的人
                GPIO.output(17, GPIO.HIGH)
                print("欢迎回家,门已打开")
                time.sleep(3)
                GPIO.output(17, GPIO.LOW)
            elif(res == -1):
                print("没有看见你,我要关门了")
                time.sleep(3) 
            else:
                print("关门")
            time.sleep(3)
            print('本次响应结束')
            time.sleep(5)
```

执行 python3 test.py 即可成功



树莓派默认使用的GPIO口编号模式是BCM（Broadcom）模式。BCM模式是树莓派官方推荐的GPIO引脚编号方式。在BCM模式下，每个GPIO口都有一个唯一的BCM编号，这些编号与引脚上的BCM物理引脚标识相对应。

当你使用树莓派的RPi.GPIO库进行GPIO控制时，默认情况下会使用BCM编号来标识GPIO口。

要注意的是，根据树莓派型号和版本，BCM编号与物理引脚之间的对应关系可能会有所变化。因此，在进行GPIO控制时，最好参考正确的GPIO引脚图以确定正确的BCM编号。

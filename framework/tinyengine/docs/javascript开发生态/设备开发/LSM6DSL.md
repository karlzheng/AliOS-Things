## #设备功能

lsm6dsl是一款具有数字加速度计和数字陀螺仪功能的低功耗传感器，通过I2C协议进行数据交互，这里我们会实现定时读取传感器的加速度值以及陀螺仪的旋转数据，并上报到LD云端；

## #硬件资源

DevelopKit开发板上自带有lsm6dsl传感器(图1红色圆圈处)，并连接到stm32的I2C2端口；



![image | left | 273x330](https://cdn.yuque.com/lark/0/2018/jpeg/66207/1526519192734-89fbbd82-d546-40a1-aee9-6186092f01a1.jpeg "")

图1
#### 
## #软件设计
根据lsm6dsl的数据手册，加速度计和陀螺仪的数据主要存放在寄存器0x28-0x2C、0x22-0x26中，初始化配置完成后，我们可以从这些寄存器中读取传感器的实时数据值：

```javascript
/*DevelopKit开发板配置文件*/
{
  "I2C": [
    {
      "id":"lsm6dsl",
      "port":1,
      "address_width":7,
      "freq":400000,
      "mode":1,
      "dev_addr":214
    }
  ]
}


/*lsm6dsl.js*/
......
/*读取加速度计数据*/
lsm6dsl.getAcc = function(){

    var acc = [0,0,0];
    if(0 == this.isInited){
        this.init_config();
        this.isInited = 1;
    }
    acc[0] = this.read_two(this.xlowAccReg);
    acc[0] = acc[0] * 61 / 1000;
    acc[1] = this.read_two(this.ylowAccReg);
    acc[1] = acc[0] * 61 / 1000;
    acc[2] = this.read_two(this.zlowAccReg);
    acc[2] = acc[0] * 61 / 1000;
    return acc;
};

/*读取陀螺仪数据*/
lsm6dsl.getGyro = function(){
    var gyro = [0,0,0];
    if(0 == this.isInited){
        this.init_config();
        this.isInited = 1;
    }
    gyro[0] = this.read_two(this.xlowGyroReg); 
    gyro[0] = gyro[0] * 70; 
    gyro[1] = this.read_two(this.ylowGyroReg);  
    gyro[1] = gyro[1] * 70; 
    gyro[2] = this.read_two(this.zlowGyroReg); 
    gyro[2] = gyro[2] * 70; 
    return gyro;
};

......


/*index.js*/
......
/*发送数据到云端*/
var postEvent = function(val) {
    var obj={};

    var id;
    var attrs = device.properties;
    obj['Xacc'] = val[0];
    obj['Yacc'] = val[1];
    obj['Zacc'] = val[2];
    obj['Xgyro'] = val[3];
    obj['Ygyro'] = val[4];
    obj['Zgyro'] = val[5];
    var event = device.events[0];
    device.update(event, obj);
};
......
```


## #运行验证
<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">传感器数据在串口的打印：</span></span>


![image.png | left | 367x193](https://cdn.yuque.com/lark/0/2018/png/66207/1526897430005-4446bfda-5079-4750-b60c-d1690b5ab277.png "")


设备运行并接入LD一站式开发平台后，设备上传的数据将显示在界面上：


![1.png | center | 747x362](https://cdn.yuque.com/lark/0/2018/png/66207/1526892529900-a72b3ca9-f3b6-4a5e-9727-b35c91b8c5c2.png "")




## #代码仓库
```c
http://gitlab.alibaba-inc.com/Gravity/gravity_lite/tree/master/devices/lsm6dsl
```

 
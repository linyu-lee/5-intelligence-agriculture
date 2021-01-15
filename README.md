# 5-intelligence-agriculture
## 项目名称：二氧化碳浓度监测仪
作者 | github ID
---------- | -------------
颜海月 | haiyue-yan
刘倚赫 |	YiHe-Liu
马姣姣 |	Gioia-Ma
杨弘毅 |	Hongyi-Yangz
李芋霖 |	linyu-lee
朱语 |	520zy520

## 一、问题定义
   
## 二、概念设计
   
## 三、详细设计

## 四、性能指标

## 五、代码
// 节点代码如下：

#include <LoRaNow.h>  //LoRa需要的头文件

#include <Wire.h>     //OLED需要的头文件

#include "SSD1306.h"   //OLED需要的头文件

#include "Adafruit_CCS811.h"//ccs811需要的头文件

Adafruit_CCS811 ccs;

SSD1306 display (0x3c, 21, 22);

#define MISO 19

#define MOSI 23

#define SCK 18

#define SS 5

#define DIO0 4

#define tmpADPin 35

int tmp;

void setup() {

  Serial.begin(9600);
  
  Serial.println("LoRaNow Simple Node");
  
  pinMode(tmpADPin,INPUT);
  
  display.init();
  
  display.setFont(ArialMT_Plain_24);
  
  display.drawString(0, 0, "CO2Monitor"); //显示二氧化碳检测器字样
  
  display.display();
  
   LoRaNow.setFrequencyCN(); 
   
   LoRaNow.setPinsSPI(SCK, MISO, MOSI, SS, DIO0); // Only works with ESP32
   
  if (!LoRaNow.begin()) {
  
   Serial.println("LoRa init failed. Check your connections.");
    
   while (true);
    
  }


  LoRaNow.onMessage(onMessage);
  
  LoRaNow.onSleep(onSleep);
  
  LoRaNow.showStatus(Serial);
  
}


void loop() {

  bool alert;
  
  tmp = CO2();
  
  String myc="  ";
  
  myc.concat(tmp);  				//将数字转化为字符串
  
  myc+="ppm";
  
  display.drawString(0, 0, myc);  	/*显示CO2浓度*/
  
  LoRaNow.loop();
  
  if(tmp>1600)          /*CO2超标警报*/
  
   alert=1;			//上限设置为1600ppm，是否科学有待考证
    
  else
  
   alert=0;
    
}


int CO2() //二氧化碳浓度函数,若二氧化碳浓度不准确，可在函数中进行修正

{
  int nongdu=ccs.geteCO2();
  
  return nongdu;
  
}


void onMessage(uint8_t *buffer, size_t size)

{

  Serial.print("Receive Message: ");
  
  Serial.write(buffer, size);
  
  Serial.println();
  
  Serial.println();
  
}

void onSleep()

{

  Serial.println("Sleep");
  
  Serial.println(tmp);
  
  delay(500); // "kind of a sleep"
  
  Serial.println("Send Message");
  
  LoRaNow.print("LoRaNow Node Message ");
  
  //Serial.println("LoRaNow Message sended");
  
  LoRaNow.print(millis());
  
  LoRaNow.print("-");
  
  LoRaNow.print(tmp);
  
  LoRaNow.send();
  
}


网关代码仍沿用期中的代码



## 六、代码如何运行

### 一、开发者开发环境：

   软件:Arduino,硬件esp32, LoRa, ccs811, oled等。

1.  Arduino中开发板管理器中下载esp32并在开发板中选择ESP32 Dev Module。 
  
2. 在库管理器中安装：WIFI, SSD1306, Adafruit CCS811 Library, LoRaNow, PubSubClient等库。即代码中各种头文件。
  
3. 将设备连入电脑。

4. 将代码复制到Arduino中，上传即可。

5. 在串口监视器9600波特率下能够看见输出的二氧化碳浓度。同时OLED屏幕上也能够显示出对应的数值。

6. 对网关进行同样的操作。

7. 完成上述步骤即完成了二氧化碳监测器的搭建。

8. 当网关和节点同时工作时即可完成二氧化碳浓度的远程监测。

### 二、对于产品使用用户：

1. 本产品已经将代码写入esp32，用户只需要将检测仪器放到需要检测的位置，无需其他任何操作。即可在节点的OLED获取相应信息，也可在网关处获得信息。当二氧化碳浓度超过一定限度时，网关和节点的蜂鸣器会同时报警。发出声音和产生亮光。

2. 在正式的产品内部，含有一个充电宝。当网关处无法得到信息时，考虑是否节点处电量耗尽。可直接从外部接线向充电宝充电。电量充满以后节点可继续使用。


# box
monitoring environment
#include <Servo.h> 

    #include <math.h>

    Servo myservo;

    #define TEMP_PIN A6  //定义温度传感器接口

    int a,val;   //定义变量

    

#include <Adafruit_NeoPixel.h> /*引用”Adafruit_NeoPixel.h”文件。引用的意思有点象“复制-粘贴”。include文件提供了一种很方便的方式共享了很多程序共有的信息。*/

#define PIN 6                         /*定义了控制LED的引脚，6表示Microduino的D6引脚，可通过Hub转接出来，用户可以更改 */

Adafruit_NeoPixel strip = Adafruit_NeoPixel(1, PIN, NEO_GRB + NEO_KHZ800);   //该函数第一个参数控制串联灯的个数，第二个是控制用哪个pin脚输出，第三个显示颜色和变化闪烁频率

#define Light_PIN A0  //光照传感器接AO引脚

#include "U8glib.h"  //oled显示相关

U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE);//定义OLED连接方式

float temperature;     //定义浮点型变量，用于存放转换后的温度

int B=3975;              //热敏电阻的基础参考值B

float resistance;

#define buzzer 2    //定义蜂鸣器接口D2

int sensorValue;

#define Light_value 500    

#define INTERVAL_Time 3000     //

#define sound_PIN A2         //声音传感器接口

#define sound_value1  100   //3个声音区间

#define sound_value2  250

#define sound_value3  400

int soundValue;

unsigned long Time_millis = millis();



void setup()                                //创建无返回值函数

 {

  Serial.begin(115200);               //初始化串口通信，并将波特率设置为115200

  strip.begin();                             //准备对灯珠进行数据发送

  strip.show();                              //初始化所有的灯珠为关的状态

  myservo.attach(4);  //定义舵机驱动端口

    pinMode(buzzer,OUTPUT);//设置数字IO脚模式，OUTPUT为输出

}



void draw(void) {

  u8g.setFont(u8g_font_7x13);//字体2

  u8g.setPrintPos(0, 16);

  u8g.print("guangshengjiankong");

  u8g.setPrintPos(0, 32);//换行显示须再定义

  u8g.print("light: ");

  u8g.print(sensorValue);

  u8g.print(" lux  ");

  u8g.setPrintPos(0,48 );

 u8g.print("volumn: ");

 u8g.print(soundValue);

 u8g.print("db");

}  //oled显示函数



void loop()                                  //无返回值loop函数

 {    a=analogRead(0); //读取温度传感器的模拟值

      resistance=(float)(1023-a)*10000/a; //计算出传感器的电阻值

      temperature=1/(log(resistance/10000)/B+1/298.15)-273.15;//将电阻值转换成温度值

      delay(500); //延时500毫秒

      val=map(temperature,0,50,0,180); //将转换的温度值映射到舵机的角度值*/

      myservo.write(val); //舵机转到相应的角度

   u8g.firstPage();  

  do {getsenserdata();

  draw();

  } while( u8g.nextPage() );

  lightjudge();

  if( temperature>60)

  {digitalWrite(buzzer,HIGH);//发声音

      delay(1000);//延时1ms

}}



void getsenserdata()

{

   sensorValue = analogRead(Light_PIN);             //光检测

  Serial.println(sensorValue);                                //彩色led灯根据光强调节颜色和亮度

  soundValue = analogRead(sound_PIN);             //声检测

  Serial.println(soundValue) ; 

}   //获取数据

void lightjudge(){

   if(sensorValue<Light_value&&soundValue>sound_value1)     //对光照和声音进行要求，满足条件发光，不满足熄灭

  {

    Time_millis = millis();

    if (soundValue< sound_value2)  //若光强小于400

{colorWipe(strip.Color(0, map(soundValue, 10, 400, 0, 255), 0));

delay(100);}

/*“map(val,x,y,m,n)”函数为映射函数，可将某个区间的值（x-y）变幻成（m-n），val则是你需要用来映射的数据，这里是将10到400的光对应用0到25的绿光标示*/

  else  if (soundValue >= sound_value2 && soundValue < sound_value3) 

  //若光强大于400小于800

    {colorWipe(strip.Color(0, 0, map(soundValue, 400, 800, 0, 255)));

    delay(200);}

    //将400到800的光分别用0到255的蓝光表示

 else if (soundValue >sound_value3)

{colorWipe(strip.Color(map(soundValue, 800, 960, 0, 255), 0, 0));

delay(200);}

//将800到960的光用0到255的红光表示

}

else

 {unsigned long time=millis();

 if(time-Time_millis>INTERVAL_Time)   

 colorWipe(strip.Color(0, 0, 0));

 }

}

void colorWipe(uint32_t c) {

  for (uint16_t i = 0; i < strip.numPixels(); i++)  //i从0自增到LED灯个数减1

 {

    strip.setPixelColor(i, c); //将第i个灯点亮

    strip.show(); //led灯显示

  }

}

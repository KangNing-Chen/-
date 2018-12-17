int Echo = A0;  // Echo回声脚(P2.0)
int Trig =A1;  //  Trig 触发脚(P2.1)

int Front_Distance = 0;
int Left_Distance = 0;
int Right_Distance = 0;

int Left_motor_back=5;     //左电机后退(IN1)
int Left_motor_go=6;     //左电机前进(IN2)

int Right_motor_go=9;    // 右电机前进(IN3)
int Right_motor_back=10;    // 右电机后退(IN4)

int servopin=12;//设置舵机驱动脚到数字口12
int myangle;//定义角度变量
int pulsewidth;//定义脉宽变量
int val;

void setup()
{
  Serial.begin(9600);     // 初始化串口
  //初始化电机驱动IO为输出方式
  pinMode(Left_motor_go,OUTPUT); // PIN 5 (PWM)
  pinMode(Left_motor_back,OUTPUT); // PIN 6 (PWM)
  pinMode(Right_motor_go,OUTPUT);// PIN 9 (PWM) 
  pinMode(Right_motor_back,OUTPUT);// PIN 10 (PWM)

  //初始化超声波引脚
  pinMode(Echo, INPUT);    // 定义超声波输入脚
  pinMode(Trig, OUTPUT);   // 定义超声波输出脚
//  lcd.begin(16,2);      //初始化1602液晶工作                       模式
  //定义1602液晶显示范围为2行16列字符  
  pinMode(servopin,OUTPUT);//设定舵机接口为输出接口
}
//=======================智能小车的基本动作=========================
 void run()     // 前进
{
  analogWrite(Right_motor_go,200);//右电机前进，PWM比例0~255调速，左右轮差异略增减
  analogWrite(Right_motor_back,0);
  analogWrite(Left_motor_go,200);// 左电机前进，PWM比例0~255调速，左右轮差异略增减
  analogWrite(Left_motor_back,0);
}

void brake()         //刹车，停车
{
  digitalWrite(Right_motor_go,LOW);
  digitalWrite(Right_motor_back,LOW);
  digitalWrite(Left_motor_go,LOW);
  digitalWrite(Left_motor_back,LOW);
}

void left()         //左转(左轮不动，右轮前进)
{
  analogWrite(Right_motor_go,200); //右电机前进，PWM比例0~255调速
  analogWrite(Right_motor_back,0);
  digitalWrite(Left_motor_go,LOW);   //左轮不动
  digitalWrite(Left_motor_back,LOW);
}

void spin_left()         //左转(左轮后退，右轮前进)
{
  analogWrite(Right_motor_go,200); //右电机前进，PWM比例0~255调速
  analogWrite(Right_motor_back,0);

  analogWrite(Left_motor_go,0); 
  analogWrite(Left_motor_back,200);//左轮后退PWM比例0~255调速
}

void right()        //右转(右轮不动，左轮前进)
{
  digitalWrite(Right_motor_go,LOW);   //右电机不动
  digitalWrite(Right_motor_back,LOW);

  analogWrite(Left_motor_go,200); 
  analogWrite(Left_motor_back,0);//左电机前进，PWM比例0~255调速
  } 


void spin_right()        //右转(右轮后退，左轮前进)
{
  analogWrite(Right_motor_go,0); 
  analogWrite(Right_motor_back,200);//右电机后退，PWM比例0~255调速

  analogWrite(Left_motor_go,200); //左电机前进，PWM比例0~255调速
  analogWrite(Left_motor_back,0);
}

void back()          //后退
{
  analogWrite(Right_motor_go,0);
  analogWrite(Right_motor_back,150);//右轮后退，PWM比例0~255调速

  analogWrite(Left_motor_go,0);
  analogWrite(Left_motor_back,150);//左轮后退，PWM比例0~255调速
}
//==========================================================

float Distance_test()   // 量出前方距离 
{
  digitalWrite(Trig, LOW);   // 给触发脚低电平2μs
  delayMicroseconds(2);
  digitalWrite(Trig, HIGH);  // 给触发脚高电平10μs，这里至少是10μs
  delayMicroseconds(10);
  digitalWrite(Trig, LOW);    // 持续给触发脚低电
  float Fdistance = pulseIn(Echo, HIGH);  // 读取高电平时间(单位：微秒)
  Fdistance= Fdistance/58;       //为什么除以58等于厘米，  Y米=（X秒*344）/2
  // X秒=（ 2*Y米）/344 ==》X秒=0.0058*Y米 ==》厘米=微秒/58
  //Serial.print("Distance:");      //输出距离（单位：厘米）
  //Serial.println(Fdistance);         //显示距离
  //Distance = Fdistance;
  return Fdistance;
}  

void Distance_display(int Distance)//显示距离
{
  if((2<Distance)&(Distance<400))
  {
//    lcd.home();        //把光标移回左上角，即从头开始输出   
//    lcd.print("    Distance: ");       //显示
//    lcd.setCursor(6,2);   //把光标定位在第2行，第6列
//    lcd.print(Distance);       //显示距离
//    lcd.print("cm");          //显示
  }
  else
  {
//    lcd.home();        //把光标移回左上角，即从头开始输出  
//    lcd.print("!!! Out of range");       //显示
  }
  delay(250);
//  lcd.clear();
}

void servopulse(int servopin,int myangle)/*定义一个脉冲函数，用来模拟方式产生PWM值*/
{
  pulsewidth=(myangle*11)+500;//将角度转化为500-2480 的脉宽值
  digitalWrite(servopin,HIGH);//将舵机接口电平置高
  delayMicroseconds(pulsewidth);//延时脉宽值的微秒数
  digitalWrite(servopin,LOW);//将舵机接口电平置低
  delay(20-pulsewidth/1000);//延时周期内剩余时间
}

void front_detection()
{
  //此处循环次数减少，为了增加小车遇到障碍物的反应速度
  for(int i=0;i<=5;i++) //产生PWM个数，等效延时以保证能转到响应角度
  {
    servopulse(servopin,90);//模拟产生PWM
  }
  Front_Distance = Distance_test();
  //Serial.print("Front_Distance:");      //输出距离（单位：厘米）
 // Serial.println(Front_Distance);         //显示距离
 //Distance_display(Front_Distance);
}

void left_detection()
{
  for(int i=0;i<=15;i++) //产生PWM个数，等效延时以保证能转到响应角度
  {
    servopulse(servopin,175);//模拟产生PWM
  }
  Left_Distance = Distance_test();
  //Serial.print("Left_Distance:");      //输出距离（单位：厘米）
  //Serial.println(Left_Distance);         //显示距离
}

void right_detection()
{
  for(int i=0;i<=15;i++) //产生PWM个数，等效延时以保证能转到响应角度
  {
    servopulse(servopin,5);//模拟产生PWM
  }
  Right_Distance = Distance_test();
  //Serial.print("Right_Distance:");      //输出距离（单位：厘米）
  //Serial.println(Right_Distance);         //显示距离
}
//===========================================================
void loop()
{
  while(1)
  {
    front_detection();//测量前方距离
    if(Front_Distance < 32)//当遇到障碍物时
    {
      back();//后退减速
      delay(200);
      brake();//停下来做测距
      left_detection();//测量左边距障碍物距离
      Distance_display(Left_Distance);//液晶屏显示距离
      right_detection();//测量右边距障碍物距离
      Distance_display(Right_Distance);//液晶屏显示距离
      if((Left_Distance < 35 ) &&( Right_Distance < 35 )){ //当左右两侧均有障碍物靠得比较近 
        spin_left();//旋转掉头
        delay(100);
     }else if(Left_Distance > Right_Distance)//左边比右边空旷
      {      
        left();//左转
        delay(300);
        brake();//刹车，稳定方向
      }
      else//右边比左边空旷
      {
        right();//右转
        delay(300);
        brake();//刹车，稳定方向
      }
    }
    else
    {
      run(); //无障碍物，直行     
    }
  } 
}

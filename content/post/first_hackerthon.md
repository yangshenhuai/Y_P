---
title: "My First Hackathon"
date: 2017-08-29T00:00:47+08:00
draft: false
tags: ["Raspberrypi"]
author: "Peter Yang"
---

Last week,I joined Ericsson CGC hackathon 2017. We(team HotLadies) did a simple but interesting project using Raspberrypi,Arduino and several sensors.
# Idea
The Idea is simple. Using `Raspberrypi`,`Arduino` and `sensors` to collect data and send the data to `Wechat` Here's how system works. ![][1]

* A. there's a [Wechat Robot][2] running in Raspberrypi. And the robot can receive the message from my wechat account or sent message to my wechat account
* B. Actually I can not send the data from Raspberrypi to Arduino yet. when I write below python code in Raspberrypi

        import serial  
    ser = serial.Serial('/dev/ttyACM0')  
    ser.write(1)  



and below c code in Arduino

        int incomingBytes  
    void setup() {  
    Serial.begin(9600);  
    }  
    &nbsp;  
    void loop()  
    {  
    while (Serial.available() &gt; 0) {  
    incomingBytes = Serial.read();  
    }  
    Serial.println(incomingBytes);  
    }  



seems the Arduino can not receive the data from Raspberrypi, I am still checking this issue, will update here if find something.
* C. The Arduino will receive data(include Digital and Analog) from Sensors repeatedly
* D. The Arduino will send all the received data to Raspberrypi repeatedly
* E. Here are 2 diffrent scenario in this step

    1. the Raspberrypi receive the data and just store the latest data , so we user ask for a data from the Wechat Robot , it always get the latest data. for example the temperature data.

    2. the Raspberrypi receives the data and check the data , and if the data is abnormal , the Wechat Robot will send the alert message to my Wechat
# Wiring
Here's how Arduino connect with Sensors. As you can see there're four sensors.they're `Gas Sensor`,`Temperature Sensor`,`Light Sensor` and `Flame Sensor`(from left to right) ![][3]![][4]![][5]

![][6]

For `MQ5 Gas Sensor` the Circuit Diagram as below. connect the sensor's `AO` pin to Arduino's `A0` pin, connect Vcc to 5V and GND to GND

![][7]

![][8]

For `AM2302 Temperature Sensor` the Circuit Diagram as below. One thing to highlight is that you should put a 10KΩ resistor between data and Vcc

![][9]

![][10]

For `Flame Sensor` the Circuit Diagram is very simple

VCC............................... 5v

GND...............................GND

A0.................................. Arduino Analog pin

![][11]

For `Light Sensor` the Circuit Diagram is same with `Flame Sensor`
# Code
## Python

the Python [code][12] will run in Raspberrypi. `bot.py` is the entry point and it start a Wechat Robot and register handlers, so when I send the command(for example get templature command) to Robot ,the command can be handled in handlers. And will


    handlers = [msg_handler.TakePicutreHandler(),msg_handler.TemperatureHandler() ,msg_handler.LightHandler(), msg_handler.TulingHandler()] #predifined handlers  
    &nbsp;  
    @bot.register(msg_types=TEXT, except_self=False)  
    def auto_reply_all(msg):                 # register handler and dispatch to correct handlers  
    logger.info('received text message : %s',msg)  
    #msg.chat.send('Hello!')  
    #msg.reply('World!')  
    #tuling.do_reply(msg)  
    for handler in handlers:  
    if handler.match(msg):  
    handler.handle(msg)  
    break  
    &nbsp;

    serial_port.start(master) # start the serial port , 'master' is my  Wechat account  
    &nbsp;  
    bot.join() # block the thread , avoid our process exit  



`serial_port.py` integrate with Arduino data


    def start(master):  
    global m   
    m = master   
    ser = serial.Serial('/dev/ttyACM0') # the /dev/ttyACM0 is Arduino ,serial is from pySerial  
    while True:  
    ser.flushInput()                # Raspberrypi need to read latest data from Arduino every 1 sec, so need to flush Input before read data , otherwise it will read outdated data  
    for i in range(10):             # some times the data from Arduino is truncated , so need read several times, break when read the correct data.  
    data = ser.readline()     # read the data from Arduino  
    handled = handleData(data)    # Handle the data  
    if handled:                    # if read correct data , break the for loop  
    break;  
    time.sleep(1)

    &nbsp;  
    def handleData(data):  
    str_data = data.decode('utf-8')     # the data received from Arduino is bytes object, need produce the string with bytes  
    gas_match = gas_re.search(str_data) # the gas_re is 'gs(d+.d+)ge' , the data from Arduino is 'gs23.22ge'  
    if(gas_match):                        
    global gas_last_notify_time  
    gas_val = gas_match.group(1)   # get the gas value  
    current_gas = float(gas_val)  
    logger.info('gas {}'.format(current_gas))

    if(current_gas &gt;= gas_threadhold  ):   
    current_time = int(time.time())  
    if m is not None and (current_time - gas_last_notify_time) &gt; notify_interval:  
    m.send('ALERT:GAS LEAK!!')            #send alert to my account  
    gas_last_notify_time = current_time  
    logger.info('GAS Alert')    
    return True                              # return True so can break for loop in start method  
    ...                                          #other sensor's code  
    return  False  




## C

The C [code][13] running in Arduino


    int i = 0 ; // use the i as the indicator to determin read from which sensors  
    &nbsp;  
    void setup() {  
    // put your setup code here, to run once:  
    pinMode(gas_sensor,INPUT); #set the mode , default is INPUT, so you can also ignore it  
    Serial.begin(9600);  
    }  
    &nbsp;  
    void loop() {  
    // put your main code here, to run repeatedly:  
    if(i &gt;= 10000) {  
    i = 0 ;// avoid overflow   
    }  
    &nbsp;  
    if(i % 4==1) {  
    Serial.print("gs");  
    Serial.print(readGasSensor());  
    Serial.println("ge"); // the out put will like 'gs33.22ge'  
    } else if(i%4==2)  {  
    hum = dht.readHumidity();  
    temp = dht.readTemperature();  
    Serial.print("hts");  
    Serial.print(hum);  
    Serial.print("-");  
    Serial.print(temp);  
    Serial.println("hte");// the out put will like 'hts54.33-24.59hte'  
    } else if(i%4==3){  
    int range = detectFire();  
    if(range == 0 || range ==1) {  
    Serial.println("Fire");  
    } else {  
    Serial.println("No F"); // the out put will like 'hts54.33-24.59hte'  
    }  
    } else {  
    lightSensorValue = analogRead(lightSensorPin);  
    Serial.print("ls");  
    Serial.print(lightSensorValue);  
    Serial.println("le");  
    }  
    i++;  
    }  


This is very simple but useful stuff. it will more interesting use Arduino control house or something like that

[1]: /img/2017_hackathon_idea.png
[2]: https://github.com/youfou/wxpy
[3]: /img/wire1.jpg
[4]: /img/wire2.jpg
[5]: /img/wire3.jpg
[6]: /img/MQ5.jpg
[7]: /img/Interface_MQ5_Analog_Out.png
[8]: /img/DHT22-PinOut.png
[9]: /img/AM2302.jpg
[10]: /img/flame_sensor.jpg
[11]: /img/light_sensor.jpg
[12]: https://github.com/yangshenhuai/hot_ladies_python
[13]: https://github.com/yangshenhuai/hot_ladies_c/blob/master/Integrated/Integrated.ino

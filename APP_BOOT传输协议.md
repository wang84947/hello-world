# APP->BOOT 通讯协议

|帧头	|包长	|包源	|命令	 |		 |	     |       |和校验|
|-------|------|-------|--------|-------|-------|-------|------|
|0xAA	|0x08	|0x05 	|0x01	|0x00	|0x00	|0x00	|0xB9 |
------------------------------------------------------
### 包源 05 
APP->BOOT ,APP发往boot的数据
####命令
##### **命令42**
>进入BOOT 擦FLAG  ,APP 进入BOOT以9600 波特率发送
>>AA 08 02 FC 4F 4F 54 A2

##### **命令01**  
>死机复位
> >AA 08 05 01 00 00 00 B8

##### **命令02**
校验厂家编码和机型并开启传输帧
>> AA 51 05 02 <u>08 01 88 00</u> <u>00 00 00 01</u> <u>00 01 5C 3C</u> <u>5A 5A 00 00</u> <U>33 32 47 30</u> 
<u>43 49 4D 41 5F 00 00 00 00 00 00 00 00 00 00 00 </u>
<u>31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 </u>
<u>46 65 62 20 20 32 20 32 30 32 31 31 33 3A 31 33 3A 32 30 00 </u>
<u>E0</u> <u>01</u> <u>EF 49</u> E7 



***字段说明***  ^[1]^
><font color=#FF0000>08 01 88 00</font> :反馈程序开始写入的 地址的值

><font color=#00F000>00 00 00 00</font> : boot状态及错误反馈信息
byte1 擦除错误 
byte2 回读错误 
byte3 写错误 
byte4 状态   0:初始状态 1：允许进入擦写状态 2:Ymoderm 传输状态 0xfe:APP跳转前状态

><font color=#0000ff>00 00 00 00</font> :收到的文件大小 

><font color=#ff11ff>5A 5A 00 00</font>  :APP跳转flag flash 地址0x080027C0 的值

><font color=#cc33dd>33 32 47 30</font>   :芯片类别  地址0x080027C4 的值 转ASCII“32G0” 

><font color=#112255>43 49 4D 41 5F 00 00 00 00 00 00 00 00 00 00 00 </font>:厂家编码 转ASCII "CIMA_"

><font color=#00ffff>31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 </font> :机型 转ASCII "123"

><font color=#ff00ff>46 65 62 20 20 32 20 32 30 32 31 31 33 3A 31 33 3A 32 30 00</font> ：升级程序的版本 转ASCII "Feb  2 202113:13:20"

><font color=#ffff20>E0 </font>   
:低4位
>>强制刷机标志:
>>- 值=1.厂家编码一样才能刷;
>>- 值=2.厂家编码机型一样才能刷;
>>- 值=3.强制写入厂家编码,机型; 
>>- 值=其他，强制刷机，不写入厂家编码 机型等信息;

:高4位

>>擦写标志位：
>>- bit4 1.擦APP跳转FLAG,
>>- bit5 1擦除app falsh，
>>- bit6 是否擦除最后8K eeprom

><font color=#ff0f20>01 </font>
>>:高四位
>>>写入协议 
>>>0=Ymoderm 其他=预留

>>:低四位
>>>1=程序写入完成后,有程序写入,回读没有错误,置上APP跳转FLAG

><font color=#1f1300>EF 49</font> 
>>传输数据的CRC校验 确保传输数据的正确
>>###### 校验的数据从第三个字节开始即：02 08 01 88 00 00 00 00 01 00 01 5C 3C 5A 5A 00 00 33 32 47 30 43 49 4D 41 5F 00 00 00 00 00 00 00 00 00 00 00 31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 46 65 62 20 20 32 20 32 30 32 31 31 33 3A 31 33 3A 32 30 00 E0 01

-----------------------------------
### 包源 85
BOOT->APP相关数据  
#### BOOT版本信息
##### **命令5F**
>AA 15 85 5F 42 4F 4F 54 5F 4A 61 6E 20 31 36 20 32 30 32 31 BB

转ASCII“?卂BOOT_Jan 16 2021?”
#### **命令02**
boot状态信息
>>AA 51 85 02 08 01 88 00 00 00 00 01 00 01 5C 3C 5A 5A 00 00 33 32 47 30 43 49 4D 41 5F 00 00 00 00 00 00 00 00 00 00 00 31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 46 65 62 20 20 32 20 32 30 32 31 31 33 3A 31 33 3A 32 30 00 E0 01 EF 49 67 

**字段说明同上^[1]^**

# 说明 

- 1.蓝牙app 发送命令进入boot 命令 AA 08 02 FC 4F 4F 54 A2

- 2.主控板 发送修改波特率命令     AA 08 55 F0 09 00 00 00

- 3.主控板 发送透传命令          AA 08 55 F1 01 00 00 F9

- 4.BOOT 发送 boot版本信息      AA 15 85 5F 42 4F 4F 54 5F 4A 61 6E 20 31 36 20 32 30 32 31 BB

- 5.BOOT 周期发送 boot 状态信息

- 6.蓝牙APP 发送 校验厂家编码和机型并开启传输帧

- 7.上位机Ymodem传输

- 8.下位机Ymodem 回复 'C'.ACK=0X06.NAK=0X15

- 9.传输完成上位机 发送死机复位 

其他

KEIL 移植
 
如何生成APP?
\$KARM\ARMCC\bin\fromelf.exe --bin -o "$L@L.bin" "#L"

编译
CUBEiede？

```flow
st=>start: 开始
cond11=>condition: 是否有APP FLAG?
cond12=>condition: 等待200ms 是否有收
到进入app的命令?
op11=>operation: 运行APP

cond13=>condition: 收到进入
boot命令？
op2=>operation: 以9600的波特率发送修改波特率帧 
delay 
以115200的波特率发送透传帧 
delay 
发送boot版本信息帧
op22=>operation: 进入boot           
op3=>operation: 接收命令
cond41=>condition: 死机复位命令？
cond51=>condition: 校验厂家编码和机型
并开启传输帧?
cond52=>condition: 刷机标志=1？
cond53=>condition: 刷机标志=2？
cond54=>condition: 刷机标志=3？
cond55=>condition: 刷机标志=其他？
cond61=>condition: 厂家编码对比通过？
cond62=>condition: 厂家编码和机型编码
对比的通过？
op61=>operation: 强制刷机并修改厂家编码和机型编码
op62=>operation: 强制刷机保留原有厂家编码和机型编码
cond71=>condition: 是否擦APP FLAG?
op71=>operation: 擦除APP FLAG
cond81=>condition: 是否擦
APP FALSH?
cond91=>condition: 是否擦最后8k flash?
op91=>operation: 保留最后8kflash
cond101=>condition: 进入Ymoderm 传输？
cond111=>condition: 写入的文件过大？
会覆盖最后8k？
op121=>operation: 擦除最后8k
op131=>operation: Ymoderm 写app程序
cond141=>condition: 是否写入FALG？
cond151=>condition: 传输过程中没有回
读校验错误？
op151=>operation: boot状态信息(AA 51 82)
op161=>operation: 写入FLAG
e=>end: 结束框
pa=>parallel: 强制刷机标志 | approved
pa_wr_flg=>parallel: 擦写标志判断
st->cond11->op2->op3->cond41
cond11(yes)->op2
cond11(no)->cond12
cond12(yes)->op22(left)->cond11
cond12(no)->op11(right)->cond13
cond13(yes)->op22(left)->cond11
cond13(no)->op11
cond41(no)->cond51
cond41(yes,right)->op22
cond51(yes)->pa(path1,left)->cond52
cond51(no,right)->op3
cond52(yes)->cond61
cond53(yes)->cond62(left)
pa(path2,bottom)->cond53
pa(path3,right)->cond54
cond54(yes)->op61
op61->pa_wr_flg(path3,right)
op62->pa_wr_flg(path3,right)
pa_wr_flg(path1,bottom)->cond71
cond54(no)->op62
cond62(yes)->pa_wr_flg
cond61(yes)->pa_wr_flg

pa_wr_flg(path3,right)->cond81
cond71(no)->cond101
cond71(yes,right)->op71
cond81(yes)->cond91
cond81(no)->op3
cond91(yes)->cond101
cond91(no,right)->op91
op91->cond101
cond101(yes)->cond111
cond111(yes)->op121
op121(right)->op131
op131(bottom)->cond141
cond111(no)->op131(right)
cond141(yes)->cond151
cond141(no)->op3
cond151(yes)->op161(left)->op3(left)
cond151(no)->op151(right)->op3


```




<font face="Microsoft YaHei UI">

<center><font size=7>北汽高配嵌入式代码说明</font></center>

<!-- TOC -->

- [1. 流程图](#1-%E6%B5%81%E7%A8%8B%E5%9B%BE)
  - [1.1. 主函数](#11-%E4%B8%BB%E5%87%BD%E6%95%B0)
  - [1.2. 进程](#12-%E8%BF%9B%E7%A8%8B)
    - [1.2.1. 网络](#121-%E7%BD%91%E7%BB%9C)
    - [1.2.2. 显示屏](#122-%E6%98%BE%E7%A4%BA%E5%B1%8F)
    - [1.2.3. 监测传感器](#123-%E7%9B%91%E6%B5%8B%E4%BC%A0%E6%84%9F%E5%99%A8)
    - [1.2.4. 电表](#124-%E7%94%B5%E8%A1%A8)
    - [1.2.5. 读卡器](#125-%E8%AF%BB%E5%8D%A1%E5%99%A8)
  - [1.3. 功能模块](#13-%E5%8A%9F%E8%83%BD%E6%A8%A1%E5%9D%97)
    - [1.3.1. moudel0-1：重启获取数据](#131-moudel0-1%E9%87%8D%E5%90%AF%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE)
    - [1.3.2. moudel1-1：系统升级](#132-moudel1-1%E7%B3%BB%E7%BB%9F%E5%8D%87%E7%BA%A7)
    - [1.3.3. moudel2-1：接收屏幕数据并处理](#133-moudel2-1%E6%8E%A5%E6%94%B6%E5%B1%8F%E5%B9%95%E6%95%B0%E6%8D%AE%E5%B9%B6%E5%A4%84%E7%90%86)
    - [1.3.4. moudel2-2：接收并处理读卡器数据](#134-moudel2-2%E6%8E%A5%E6%94%B6%E5%B9%B6%E5%A4%84%E7%90%86%E8%AF%BB%E5%8D%A1%E5%99%A8%E6%95%B0%E6%8D%AE)
    - [1.3.5. moudel2-3：接收智能电表数据（645）](#135-moudel2-3%E6%8E%A5%E6%94%B6%E6%99%BA%E8%83%BD%E7%94%B5%E8%A1%A8%E6%95%B0%E6%8D%AE645)
    - [1.3.6. moudel2-4：接收并处理服务器数据](#136-moudel2-4%E6%8E%A5%E6%94%B6%E5%B9%B6%E5%A4%84%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%B0%E6%8D%AE)
    - [1.3.7. moudel3-1：虚拟时钟](#137-moudel3-1%E8%99%9A%E6%8B%9F%E6%97%B6%E9%92%9F)
    - [1.3.8. moudel3-2：充电枪状态检测](#138-moudel3-2%E5%85%85%E7%94%B5%E6%9E%AA%E7%8A%B6%E6%80%81%E6%A3%80%E6%B5%8B)
    - [1.3.9. moudel3-3：读取枪座温度](#139-moudel3-3%E8%AF%BB%E5%8F%96%E6%9E%AA%E5%BA%A7%E6%B8%A9%E5%BA%A6)
    - [1.3.10. moudel3-4：一键启停（锁）操作](#1310-moudel3-4%E4%B8%80%E9%94%AE%E5%90%AF%E5%81%9C%E9%94%81%E6%93%8D%E4%BD%9C)
    - [1.3.11. moudel4-1：电表数据处理](#1311-moudel4-1%E7%94%B5%E8%A1%A8%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86)
    - [1.3.11. moudel5-1：接收蓝牙数据](#1311-moudel5-1%E6%8E%A5%E6%94%B6%E8%93%9D%E7%89%99%E6%95%B0%E6%8D%AE)

<!-- /TOC -->

# 1. 流程图

## 1.1. 主函数

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI

start
  :设置向量表的位置和偏移量;
  :板级驱动初始化;
  :默认数据初始化;
  :获取外部存储数据;
    note right
    详见 <b>module0-1</b>
    ====
    * reboot_process()
    end note
  :创建启动进程;
  fork
    :进程1：网络;
  fork again
    :进程2：显示屏;
      fork again
    :进程3：监测传感器;
      fork again
    :进程4：电表;
      fork again
    :进程5：读卡器;
      fork again
    :进程6：服务器;
      fork again
    :进程7：led;
  end fork

@enduml
```
</center>

## 1.2. 进程

### 1.2.1. 网络


```plantuml {align="center"}
@startuml

skinparam defaultFontName Microsoft YaHei UI

start
    :关闭充电输出;
    :电磁锁解锁;
    if(蓝牙使能标记) then (true)
        :使能蓝牙;
        else (false)
        :禁用蓝牙;
    endif
    if(重置信号) then(true)
        :清除IC卡信息;
        :蓝牙恢复出厂密码;
        else(false)
    endif
    repeat
    if(升级标志位) then(true)
        :升级BootLoader;
            note right
            详见 <b>moudel1-1</b>
            ====
            * upgrade_bootloader()
            end note
        :删除进程;
        stop
        else (false)
            if(网络类型为非离线且处于离线状态) then (true)
                :更新屏幕网络状态显示;
                :LED显示为离线定义状态;
                :重启网络模块;
                if(选择网络类型) then (3G GSM)
                    :设置网络模块重启超时时间;
                    else (wifi)
                        if(注册网络)then(成功)
                            :更新屏幕显示;
                            else(失败)
                        endif
                endif
            else(false)
            endif
            if(网络为在线状态)then(true)
                if(上报订单信息)then(false)
                    :上报订单信息;
                    else(true)
                    :上传枪的状态和心跳;
                    endif
                if(心跳时间到)then(true)
                :更新枪数据;
                else(false)
                endif
            else(false)    
            endif

            if(网络为离线类型)then(true)
                if(更新订单时间到)then(true)
                    :读取屏幕RTC时间;
                else(false)
            endif    
    endif   
@enduml
```


### 1.2.2. 显示屏



```plantuml {align="center"}
@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    repeat
    if(升级标志位) then(true)
        :删除进程;
        stop
        else (false)
        :充电页面循环显示;
        :接收并处理屏幕数据;
            note 
            详见 <b>moudel2-1</b>
            ====
            * dwin_receive()
            end note
        :接收并处理读卡器数据;
            note 
            详见 <b>moudel2-2</b>
            ====
            * card_data_pro()
            end note
        :接收智能电表数据;
            note
            详见 <b>moudel2-3</b>
            ====
            * meter_rx()
            end note           
        :接收并处理服务器数据;
            note
            详见 <b>moudel2-4</b>
            ====
            * server_wifi_RX()
            end note 
    endif   
@enduml
```

### 1.2.3. 监测传感器
<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    repeat
    if(升级标志位) then(true)
        :删除进程;
        stop
        else (false)
        :虚拟时钟;
            note right
            详见 <b>moudel3-1</b>
            ====
            * virtual_clock()
            end note
        :检测枪的CP状态;
            note right
            详见 <b>moudel3-2</b>
            ====
            * CP_detection()
            end note
        :获取主板温度;
        if(主板温度大于65度)then(true)
            if(主板状态数据正常)then(true)
                :状态数据更新为主板异常;
                :切换显示屏为主板异常界面;
                else(false)
            endif
        else(false)
            if(主板状态数据异常)then(true)
                :状态数据更新为主板正常;
                :切换显示屏回主页;
                else(false)
            endif
        endif
        :获取枪座的温度;
            note left
            详见 <b>moudel3-3</b>
            ====
            * get_temp()
            end note

        if(急停按钮)then(按下)
            if(急停状态数据正常)then(true)
                if(枪状态数据是开启)then(true)
                    :关闭充电枪;
                    :更新枪的状态数据;
                else(flase)
                    :切换显示屏为急停界面;
                    :状态数据更新为急停异常;
                endif
            else(false)
            endif
        else(正常)
            if(急停状态数据异常)then(true)
                :清除急停异常状态数据;
                :切换显示屏为主页;
            else(false)
            endif
        endif
        if(桩数据箱门状态异常)then(true)
            if(箱门被打开)then(true)
                if(箱门标志位)then(false)
                    :更新枪的状态数据;
                    :切换显示屏为门开界面;
                else(ture)
                endif
            else(false)
                if(箱门标志位)then(true)
                    :更新枪的状态数据;
                    :切换显示屏为主页;
                else(false)
                endif
            endif
        endif
    
    :一键开启（锁）检测处理;
        note left
        详见 <b>moudel3-4</b>
        ====
        * charge_button()
        end note
    :检测浪涌;
    endif   
@enduml
```

</center>

### 1.2.4. 电表

<center>

```plantuml
@startuml

skinparam defaultFontName Microsoft YaHei UI

start
    title 进程4：电表
    :显示启动页;
    :读取屏幕的版本;
    :读取屏幕信号显示状态;
    :修改屏幕触摸灵敏度;
    if(屏幕显示启动页)then(true)
    :切换显示屏为主页;
    else(flase)
    endif
    repeat
    if(升级标志位) then(true)
        :删除进程;
        stop
        else (false)
            if(屏幕显示启动页)then(true)
                :切换显示屏为主页;
            else(flase)
            endif
        :电表数据读取与处理;
            note left
            详见 <b>moudel4-1</b>
            ====
            * meter_pro()
            end note
        :延时关闭背光灯;
    endif   
@enduml

```
</center>

### 1.2.5. 读卡器

<center>

```plantuml

@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    title 进程5：读卡器
    repeat
    if(升级标志位) then(true)
        :删除进程;
        stop
        else (false)
            :定时读取屏幕当前显示页数;
            :接收读卡器数据;
            :接收蓝牙数据;
                note left
                详见 <b>moudel5-1</b>
                ====
                * Receive_Bluetooth()
                end note
    endif   
@enduml

```
</center>

## 1.3. 功能模块

### 1.3.1. moudel0-1：重启获取数据
<center>

```plantuml

@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    note
    reboot_process()
    ====
    * main.c
    end note
    :获取实时订单数据地址;
    floating note: get_realtime_bill_address()
    :读出外部存储数据;
    if(订单标志数据)then(错误)
    else(正确)
        if(上次关机是因为系统掉电)then(否)
        else(是)
        :复制实时订单数据到枪数据;
        :将枪数据保存到外部存储离线账单中;
        :清除外部存储的实时订单数据;
        :将临时账单和枪结构体数据清零;
        endif
    endif 
stop
@enduml

```

</center>

### 1.3.2. moudel1-1：系统升级
<center>

```plantuml

@startuml

skinparam defaultFontName Microsoft YaHei UI
start
    note
    upgrade_bootloader()
    ====
    * protocol.c
    end note
    if(升级标志)then(false)
    else(true)
        :禁用串口2、3、4、5、6中断;
        :屏幕切换至启动页;
        :将接收数据存入外部存储;
        note
        upgrade()
        ====
        * bootloader.c
        end note
        :将外部系统更新数据存到内部存储;
        note
        copyfromEFlash()
        ====
        * bootloader.c
        end note
        :清除升级信息结构体数据;
    endif
    :重启系统;
stop
@enduml


```

</center>

### 1.3.3. moudel2-1：接收屏幕数据并处理

<center>

```plantuml

@startuml

skinparam defaultFontName Microsoft YaHei UI
start
    note
    dwin_receive()
    ====
    * dwin.c
    end note
    while(有新数据)is(true)
        if(接收数组位置超出范围)then(false)
        else(true)
        :数据计数置零;
        endif

        :数组位置累加;

        if(协议头)then(错误)
        else(正确)
            :协议头写入数组头部,数据位置设为2;
        endif 
        if(达到接收数据长度)then(false)
        else(true)
            :处理接收数据;
            floating note: dwin_data_pro()
        endif      
    endwhile(false)
stop
@enduml
```
</center>

### 1.3.4. moudel2-2：接收并处理读卡器数据
<center>

```plantuml
@startuml

skinparam defaultFontName Microsoft YaHei UI

start
    note
    card_data_pro()
    ====
    * usart_ic.c
    end note
    while(有新数据)is(true)
        if(是IC卡数据)then(false)
        else(true)
            if(一帧数据接收完成)then(false)
            else(true)
                if(CRC校验)then(false)
                    :蜂鸣器响;
                    else(true)
                    :调用接收数据相应的处理函数;
                        note 
                        new_card_data()
                        end note
                    :回传IC卡串口数据;
                    :清空数据结构体;
                endif
                :清空数据结构体;
            endif   
        endif  
        if(协议头内容及位置)then(错误)
        else(正确)
            :标记是IC卡数据;
        endif 
    
    endwhile(false)
stop
@enduml

```

</center>

### 1.3.5. moudel2-3：接收智能电表数据（645）
<center>

```plantuml
@startuml

skinparam defaultFontName Microsoft YaHei UI
start
    note
    meter_645_re()
    ====
    * meter_645.c
    end note
    while(有新数据)is(true)
        if(接收数组位置超出范围)then(false)
        else(true)
        :数据计数置零;
        endif
        if(步骤为2)then(true)
            :接收数据写入数组;
            if(一帧接收完成)then(false)
            else(true)
                if(帧尾)then(错误)
                else(正确)
                    :数据校验;
                    if(校验)then(错误)
                    else(正确)
                        :接受数据发送到数据队列;
                    endif
                endif
            endif
        elseif(步骤为1)then(true)
            :接收数据写入数组;
            if(接收数据个数达到8并且数据正确)then(true)
            :标记步骤为2;
            endif
        elseif(步骤为0且帧头正确)then(true)
            :协议头写入数组头部，数据位置设为1;
            :标记步骤为1;
        endif    
    endwhile(false)
stop
@enduml

```

</center>

### 1.3.6. moudel2-4：接收并处理服务器数据

<center>

```plantuml
@startuml

skinparam defaultFontName Microsoft YaHei UI

start
    note
    server_wifi_RX()
    ====
    * WiFi.c
    end note
    while(有新数据)is(true)
        if(接收数组位置超出范围)then(false)
        else(true)
        :数据计数置零;
        endif
        :接收数据写入数组;

        if(步骤为1)then(false)
        else(true)
            if(桢尾)then(否)
            else(是)
                :数据类型设为服务器数据;
                :接收数据发送到数据队列;
                :标记步骤为2;
            endif
        endif

        if(步骤为0且帧头正确)then(false)
        else(true)
            :步骤标记为1;
            :协议头写入数组头部，数据位置设为1;
        endif  
        if(步骤为2)then(false)
        else(true)
            :数据计数置零;
        endif 

        if(步骤为0且接收数据为换行回车符)then(false)
        else
            if(网络类型为WIFI)then(true)
            else(false)
                :数据类型设为AT指令;
                :接收数据发送到数据队列;
            endif
            :数据计数置零;
        endif
    endwhile(false)
stop
@enduml

```

</center>

</center>

### 1.3.7. moudel3-1：虚拟时钟

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    note
    virtual_clock()
    ====
    * pile_API.c
    end note
    if(更新费率时间到)then(false)
    else(true)
        if(当前时间在存储的时间段内)then(true)
        :设置费率为所在时间段内的费率值;
        else(false)
        :设置费率为默认费率值;
        endif
        :更新屏幕费率值;
    endif

    if(桩的时间已更新)then(false)
    else(true)
        if(未更新时间大于1s)then(true)
        :更新时间数据;
        elseif(计时器溢出)
        :标记桩的时间未更新;
        endif
    endif
stop
@enduml
```

</center>

### 1.3.8. moudel3-2：充电枪状态检测

> 详细流程请查看国标文档

> ~~描述
| gun_op_step | CP                            | 动作                               | 描述                           |
|-------------|-------------------------------|------------------------------------|--------------------------------|
| 0           |                               |                                    | 空闲，未连接                    |
|             | <font color=red>变为6V</font> | step从0设为1                       | 6V电平状态只在无S2的情况下存在 |
| 1           |                               | 刷卡后step设为2，或低配直接自动充电 | 连接但未充电状态               |
| 2           |                               | 开始充电，step设为4                 |                                |
在温度检测里，3 4 11 是充电状态，需要关闭
5是关闭~~

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start
    note
    CP_detection()
    ====
    * detection_api.c
    end note
stop
@enduml
```
</center>

### 1.3.9. moudel3-3：读取枪座温度

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start

    note
    get_temp()
    ====
    * AD.c
    end note
    if(枪座相数)then(单相)
        :读取单相温度;
    else(三相)
        :读取三相温度;
    endif

    if(温度异常)then(false)
        if(数据标记为温度异常)then(false)
        else(true)
            :清除温度异常报警;
        endif
        if(小于恢复温度)then(true)
            if(数据标记为温度异常)then(true)
            else(true)
                :恢复枪充电;
                :上报告警信息;
            endif
        elseif(大于警告温度)then(true)
            :降低充电功率;
            :上报告警信息;
        endif
    else(true)
        :关闭充电;
        :上报异常;
    endif

stop
@enduml
```

</center>

### 1.3.10. moudel3-4：一键启停（锁）操作

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start

    note
    charge_button()
    ====
    * IO_API.c
    end note

    if(一键开启（锁）状态)then(true)
        if(重置按钮状态)then(true)
            stop
        else(false)
        endif
        :一键开启（锁）状态计数;
    else(false)
        :重置按键状态置零;
        :一键开启（锁）状态计数清零;
        :关闭添加卡功能;
    endif
    if(一键开启（锁）状态计数等于3)then(true)
        if(一键启停功能可用)then(true)
            if(枪在使用)then(true)
                :一键开启（锁）状态计数清零;
            else(false)
            endif
            if(枪状态)then(未使用)
            :更新枪充电模式为自动;
            :发送开始充电到服务器;
            :更新屏幕显示为充电中;
            else(使用中)
            :更新枪为停止充电状态;
            endif
        else(false)
        endif
    else(false)
    endif
    if(一键开启（锁）状态计数大于200)then(true)
        :更新为可添加卡状态;
    else(false)
    endif
stop
@enduml
```

</center>

### 1.3.11. moudel4-1：电表数据处理

<center>

```plantuml
@startuml
skinparam defaultFontName Microsoft YaHei UI
start

    note
    meter_pro()
    ====
    * modbus.c
    end note
    :读取电表数据;
    if(枪在充电状态)then(true)
        :电量计费;
        :判断是否需要停止充电;
            note
            _auto_stop_charge()
            end note
    :间隔存入实时订单;
    :更新屏幕信息;
    elseif(接触器打开状态)then(true)
    :增加逻辑状态错误计数;
    elseif(充电电流大于50)then(true)
    :增加逻辑状态错误计数;
    endif
    if(逻辑状态错误计数大于10)then(true)
    :关闭充电;
    :逻辑状态错误计数清零;
    else(false)
    endif
    :数据记录个数递增;
    if(数据记录个数超过最大枪个数)then(true)
        :数据记录个数清零;
    else(false)
    endif
stop
@enduml
```

</center>

### 1.3.11. moudel5-1：接收蓝牙数据

<center>

```plantuml
@startuml

skinparam defaultFontName Microsoft YaHei UI
start
    note
    Receive_Bluetooth()
    ====
    * debug.c
    end note
    while(有新数据)is(true)
        if(接收数据为正常数据)then(true)
            if(接收完成)then(false)
            else(true)
                if(CRC校验)then(false)
                else(true)
                :数据处理句柄;
                note
                blue_handleProtocol()
                end note
                endif
                :数据计数置零;
            endif
        elseif(接收数据为AT指令)then(true)
            if(数据为断开连接标志)then(true)
                :标记蓝牙状态;
                :清空数据;
            else(false)
            endif
        endif
        if(接收到第三个数据)then(false)
        else(true)
            :接收字符判断;
        endif     
    endwhile(false)
stop
@enduml
```

</center>
<center>

```plantuml

```

</center>




</font>

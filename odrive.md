# $\color{#FF0}{ODRIVE}$

## $\color{#0FF}{配置}$

> + 使用之前必须要把电机参数（极对数）和编码器参数（CPR、PPR）输入给`odrive`

### 设置限制值

```python
odrv0.erase_configuration()  				#将配置重置为出厂默认值。
odrv0.axis1.controller.config.enable_overspeed_error = False	#取消超速报错	

odrv0.axis0.motor.config.current_lim = 10	#设置电流限制（单位为A），不过10A很小，便于使用请设置为60A
										#请注意：重新设置电流限制值后需要重新设置电流采样范围
odrv0.axis0.motor.config.requested_current_range = 90	#然后保存配置并重新启动 ODrive 以生效，因为 ODrive 仅在启动时才													会将电流采样运放增益写入DRV（MOSFET 驱动器）
odrv0.axis0.motor.config.calibration_current	#电机校准电流限制值，即在电机校准时连续流过电机的电流大小，＜电机额定电流
odrv0.axis0.controller.config.vel_limit		#单位为 [turn/s]
```

### 设置硬件参数

```python
odrv0.config.brake_resistance				#设置功率耗散电阻的电阻值，单位为 [Ohm]
odrv0.axis0.motor.config.pole_pairs			#永磁电机中的磁极对数（磁钢个数除以2）
odrv0.axis0.motor.config.motor_type			#设置电机类型，大电流电机（MOTOR_TYPE_HIGH_CURRENT）和云台电机（MOTOR_TYPE_GIMBAL）
odrv0.axis0.encoder.config.cpr				#编码器每转计数 [CPR]，这是每转产生的脉冲数（PPR）值的4倍
odrv0.save_configuration()					#保存配置
```



## $\color{#0FF}{控制}$

> + 
> + 以控制`m0`为例
> + `odrive`有多种控制方式
>   + 位置控制
>   + 速度控制
>   + 平滑位置控
>   + 轨迹控制
>   + 环形位置控制
>   + 转速控制模式
>   + 转速爬升控制模式
>   + 转矩控制模式

```python
odrv0.axis0.requested_state = AXIS_STATE_FULL_CALIBRATION_SEQUENCE	
								#电机的电气特性（即电机相电阻和相电感），然后进行编码器偏移校准
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
								#此时 ODrive 将保持电机转子的位置。如果用手去扭动电机轴，电机会试图阻止转子被扭动
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
								#进入闭环模式（速度环、位置环、电流环都需要这一步！）    
odrv0.axis0.controller.input_pos = 10
								#单位为 [转]，可以为小数
 odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
								#设置进入位置控制状态
```

> + 经过`odrivetool`的实践，验证后正确的电机参数为
>   + `cpr`：8192
>   + `pr`：14

## $\color{#0FF}{报错/除错}$

```python
 odrv0.axis0.clear_errors()		#清楚axis0上的错误
 dump_errors(odrv0) 			#查看是否报错
```

## $\color{#F00}{示例}$​

+ 目前唯一让轮子转起来的示例

  ```python
  odrv0.erase_configuration()  #将配置重置为出厂默认值
  odrv0.axis1.controller.config.enable_overspeed_error = False
  						#取消超速报错
  odrv0.config.dc_max_negative_current = -3 
  						#电源吸收电流的能力 单位(A)
  odrv0.axis0.motor.config.current_lim = 60
  						#调大电机限制电流
  odrv0.axis0.controller.config.vel_limit = 1000
  						#增大电机限制速度，以防进入速度环时速度大于限制值报错
  odrv0.axis0.motor.config.pole_pairs = 14
  						#设置电机极对数
  odrv0.axis0.encoder.config.mode = ENCODER_MODE_INCREMENTAL
  						#设置编码器格式
  odrv0.axis0.encoder.config.cpr = 8192
  						#设置CPR
  odrv0.save_configuration()
  						#保存设置，下次重启不用重新设置
  odrv0.axis0.requested_state = AXIS_STATE_MOTOR_CALIBRATION
  						#校准电机参数，会出现滴的一声，然后电流滋滋声
  odrv0.axis0.motor.config.pre_calibrated = True
  						#保存电机参数
  odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION
  						#校准编码器，会正转一圈，反转一圈
  dump_errors(odrv0) 		  #如果调试过程中现象与预期不一致，说明出现报错，odrivetool停止运行，使用此指令查看错误
  odrv0.axis0.clear_errors() #使用此指令可清楚错误，恢复odrivetool的运行
  
  odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
  						#输入指令，进入闭环（速度环、位置环、电流环都需要这一步！）
  odrv0.axis0.controller.config.control_mode = CONTROL_MODE_POSITION_CONTROL	
  						#位置环设置
  odrv0.axis0.controller.input_pos = 1
  						#设置转到第1圈的位置
  odrv0.axis0.controller.config.control_mode = CONTROL_MODE_VELOCITY_CONTROL
  						#速度环设置
  odrv0.axis0.controller.config.vel_gain = 0.03
  						#速度环增益设置
  odrv0.axis0.controller.input_vel = 5
  						#设置以速度为5运转
  
  ```

## $\color{#0FF}{UART}$

### 串口配置

```python
odrv0.config.uart_a_baudrate 				#更改波特率，默认115200
odrv0.config.enable_uart_a					#禁用/重新启用UART_A(Ture/False)	
#注意
UART_A端口可以运行本机协议或ASCII 协议，但不能同时运行两者。您可以通过设置odrv0.config.uart0_protocol为STREAM_PROTOCOL_TYPE_ASCII_AND_STDOUTASCII 协议或STREAM_PROTOCOL_TYPE_FIBRE本机协议来配置此项。
```

### 禁用/重启UART示例

```python
odrv0.config.enable_uart_a = False
odrv0.config.gpio1_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio2_mode = GPIO_MODE_DIGITAL
odrv0.config.enable_uart_b = True
odrv0.config.gpio3_mode = GPIO_MODE_UART_B
odrv0.config.gpio4_mode = GPIO_MODE_UART_B
odrv0.reboot()
```



## $\color{#0FF}{ASCII协议}$

> + [发送命令](https://docs.odriverobotics.com/v/0.5.6/ascii-protocol.html#sending-commands)
> + [命令格式](https://docs.odriverobotics.com/v/0.5.6/ascii-protocol.html#command-format)
> + [命令参考](https://docs.odriverobotics.com/v/0.5.6/ascii-protocol.html#command-reference)
> + [参数读/写](https://docs.odriverobotics.com/v/0.5.6/ascii-protocol.html#parameter-reading-writing)
> + [系统命令](https://docs.odriverobotics.com/v/0.5.6/ascii-protocol.html#system-commands)

### 发送命令

> + **通过 UART：**将 ODrive 的 TX (GPIO1) 连接到主机的 RX。将 ODrive 的 RX (GPIO2) 连接到主机的 TX。有关详细信息，请参阅[UART 。](https://docs.odriverobotics.com/v/0.5.6/uart.html#uart-doc)
>
>   **Arduino：**您可以使用[ODrive Arduino 库](https://github.com/madcowswe/ODrive/tree/master/Arduino/ODriveArduino)与 ODrive 通信。
>
>   **Windows/Linux/macOS：**您可以使用 FTDI USB-UART 电缆连接到 ODrive。
>
>   ODrive 不回显命令。这意味着当您在类似 的程序中键入命令时`screen`，您键入的字符不会显示在控制台中。

### 命令格式

```python
 command *42 ; comment [new line character]
```

> + `*42`代表 GCode 兼容的校验和，可以省略。当且仅当提供了校验和时，设备还将在响应中包含校验和（如果有）。如果提供了校验和但无效，则该行将被忽略。校验和计算为星号 ( * ) 之前的所有字符的按位异或。有效校验和的示例：
> + `r vbus_voltage *93`
> + 支持注释以实现 GCode 兼容性
> + 一旦遇到换行符，命令就会被解释

### 命令参考

#### 电机轨迹

```python
#格式：
t motor destination #对于一般的轴移动，这是推荐的命令
#示例：
t 0 -2
```

> + `t`为轨迹。
> + `motor`是电机编号，`0`或`1`。
> + `destination`是目标位置，以[回合]为单位。

#### 电机位置

```python
#格式：
q motor position velocity_lim torque_lim #一次发送一个设定值的基本用途
#示例：
q 0 -2 1 0.1
```

> + `q`为了位置。
> + `motor`是电机编号，`0`或`1`。
> + `position`是所需的位置，以[转]为单位。
> + `velocity_lim`是速度限制，单位为[转/秒]（可选）。
> + `torque_lim`是扭矩限制，单位为 [Nm]（可选）。

```python
#格式：
p motor position velocity_ff torque_ff #如果您一个实时控制器，可以传输设定点并跟踪轨迹
#示例：
p 0 -2 0 0
```

> + `p`对于位置
> + `motor`是电机编号，`0`或`1`。
> + `position`是所需的位置，以[转]为单位。
> + `velocity_ff`是速度前馈项，单位为[转/秒]（可选）。
> + `torque_ff`是扭矩前馈项，单位为 [Nm]（可选）。

#### 电机速度

```python
#格式：
v motor velocity torque_ff #一次发送一个设定值的基本用途
#示例：
v 0 1 0
```

> + `v`对于速度
> + `motor`是电机编号，`0`或`1`。
> + `velocity`是所需的速度，单位为 [转/秒]。
> + `torque_ff`是扭矩前馈项，单位为 [Nm]（可选）。

#### 电机电流

```python
#格式：
c motor torque
#示例：
c 0 1 
```

> + `c`对于扭矩
> + `motor`是电机编号，`0`或`1`。
> + `torque`是以 [Nm] 为单位的所需扭矩。

#### 请求反馈

```python
#输入格式：
f motor 
#响应格式：
pos vel
```

> + `c`对于扭矩
> + `motor`是电机编号，`0`或`1`。
> + `torque`是以 [Nm] 为单位的所需扭矩。

#### 更新看门狗

```python
#格式：
u motor #该命令更新电机的看门狗定时器，而不更改任何设定点。
```

> + `u`对于 /u/pdate。
> + `motor`是电机编号，`0`或`1`。

#### 参数读/写

并非所有参数都可以通过 ASCII 协议访问，但至少支持所有浮点和整数类型的参数。

> + 阅读格式：`r [property]`
>   - `property`属性的名称，如 ODrive 工具中所示
>   - 响应：请求值的文本表示
>   - 示例：=> 响应：`r vbus_voltage``24.087744`
> + 书写格式：`w [property] [value]`
>   - `property`属性的名称，如 ODrive 工具中所示
>   - `value`要写入的值的文本表示
>   - 例子：

```python
w axis0.controller.input_pos -123.456
```

#### 系统命令

> + `ss`- 保存配置
> + `se`- 删除配置
> + `sr`- 重启
> + `sc`- 清除错误

  > 参考文章：
  >
  > + [ODrive应用 #1 ODrive入门指南_odrive from .libfibre import domain, objectlosterr-CSDN博客](https://blog.csdn.net/abf1234444/article/details/103325808?spm=1001.2014.3001.5506)
  > + [【电机控制】OdriveFOC-无刷电机控制（指令篇——配置0.5.6版本！）_odrive 设置教程-CSDN博客](https://blog.csdn.net/qq_42681425/article/details/132390245?spm=1001.2014.3001.5506)

## $\color{#F0A}{参考资料}$

+ `000--ODriveTool 0.5.1.post0 指令大全.pdf`

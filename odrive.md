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

  > 参考文章：
  >
  > + [ODrive应用 #1 ODrive入门指南_odrive from .libfibre import domain, objectlosterr-CSDN博客](https://blog.csdn.net/abf1234444/article/details/103325808?spm=1001.2014.3001.5506)
  > + [【电机控制】OdriveFOC-无刷电机控制（指令篇——配置0.5.6版本！）_odrive 设置教程-CSDN博客](https://blog.csdn.net/qq_42681425/article/details/132390245?spm=1001.2014.3001.5506)

## $\color{#F0A}{参考资料}$

+ `000--ODriveTool 0.5.1.post0 指令大全.pdf`

# 一、上电
1. 连接电机再给ODrive通电之前，一定要考虑安全问题，电机转起来可能出现未知的危险
2. 12V-24V for the 24V board variant, _**12V-56V** for the 56V board varian_
3. 状态LED：
	* 蓝色/青色：ODrive处于空闲状态
	* 绿色：ODrive处于活动状态并正在运行
	* 红色：发生错误，ODrive被禁用
---
# 二、连接
1. 接口工具：
	* 基于Python的命令行程序：==odrivetool==
	* 在线GUI：[Web GUI](https://gui.odriverobotics.com/configuration)（***ODrive V3.6不可用***）
	**odrivetool和Web GUI不要同时打开，否则可能没办法识别ODrive设备（真遇到了就重新插拔ODrive供电电池）**



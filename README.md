# 0.96寸OLED的LL库Demo
## 说明
  * 本工程由[MrWei95](https://github.com/MrWei95)开源共享
  * 文本编码使用UTF-8
  * OLED驱动移植自[江协科技](https://jiangxiekeji.com/)，驱动使用硬件IIC
  * 工程由STM32CubeMX生成

## LL库IIC实现
```C
/**
	* 函    数：OLED写命令，向SSD1306写入一字节命令
	* 参    数：cmd：需要发送的命令
	* 说    明：用于OLED屏幕的IIC通讯
	*/
void OLED_WR_Cmd(uint8_t cmd)
{
	uint32_t timeout = 0;
	
	while(LL_I2C_IsActiveFlag_BUSY(I2C1));										// 判断总线是否忙碌
	
	LL_I2C_GenerateStartCondition(I2C1);								
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT))					// 读SR1->SB位，判断起始位是否发送
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_TransmitData8(I2C1,0x78);											// 发送地址
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED))		// 等待EV6完成；读SR1、SR2
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_TransmitData8(I2C1,0x00);											// 发送命令
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED))
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_TransmitData8(I2C1,cmd);												// 发送数据
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED))
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_GenerateStopCondition(I2C1);							
}

/**** 
	* 函    数：OLED写数据，向SSD1306写入数据
	* 参    数：dat：需要发送的数据
	*			len：数据长度
	* 说    明：用于OLED屏幕的IIC通讯
	*/
void OLED_WR_Data(uint8_t *dat,uint8_t len)
{
	uint32_t timeout = 0;
	
	while(LL_I2C_IsActiveFlag_BUSY(I2C1));										// 判断总线是否忙碌
	
	LL_I2C_GenerateStartCondition(I2C1);								
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT))					// 读SR1->SB位，判断起始位是否发送
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_TransmitData8(I2C1,0x78);//发送地址
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED))		// 等待EV6完成；读SR1、SR2
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	LL_I2C_TransmitData8(I2C1,0x40);											// 发送数据
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED))
		{if (++timeout > TIMEOUT_MAX) return;}									// 超时退出
	
	// 循环发送数据
	for (uint8_t i = 0; i < len; i++)
	{
		LL_I2C_TransmitData8(I2C1, dat[i]);										// 发送当前字节
		while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTED))			// 等待字节发送完成
			{if (++timeout > TIMEOUT_MAX) return;}								// 超时退出
	}
	
	LL_I2C_GenerateStopCondition(I2C1);							
}
```

## 字符取模示例
### 汉字取模示例
<img src="./Documents/汉字取模.png" class="" title="汉字取模" >

### 图片取模示例
<img src="./Documents/图片取模.png" class="" title="图片取模" >

# I2C 要点总结

https://mp.weixin.qq.com/s/jlkHgiDsUNqel2R0QSf96A

## 写在前面的话

I2C其实肝的我挺难受的，通讯协议这种规范往往可以抠出很多的细节，看了波叔的文章《万变不离其宗之I2C总线要点总结》，很详细。我打赌我还不会I2C，因为涉及到很多技术细节，在实际项目中往往是会被忽略的问题，于是结合自己以前的项目经验，简单再总结一下I2C，由于认知偏差，写完之后，长吁一口气，感觉自己好像懂了。![img](https://mmbiz.qpic.cn/mmbiz_png/FJtgaC7iaibC7l9AAibb4OSnW6uzPXIsJwQAM1dJnuOdC2ZRX8uD0hnt9DUnl3THFPsk0EbBtnoLCTGDZws0nuKvA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 目录

1. 背景
2. I2C 硬件层
3. I2C 数据传输协议
4. I2C 如何工作
5. 单个主设备连接多个从机
6. 多个主设备连接多个从机
7. I2C 编程
8. 总结



**1. 背景**

I2C（Inter-Integrated Circuit），中文应该叫集成电路总线，它是一种串行通信总线，使用多主从架构，是由飞利浦公司在1980年代初设计的，方便了主板、嵌入式系统或手机与周边设备组件之间的通讯。由于其简单性，它被广泛用于微控制器与传感器阵列，显示器，IoT设备，EEPROM等之间的通信。

**I2C最重要的功能包括：**

1. 只需要两条总线；
2. 没有严格的波特率要求，例如使用RS232，主设备生成总线时钟；
3. 所有组件之间都存在简单的主/从关系，连接到总线的每个设备均可通过唯一地址进行软件寻址；
4. I2C是真正的多主设备总线，可提供仲裁和冲突检测；
5. 传输速度；

- 标准模式：Standard Mode=100 Kbps
- 快速模式：Fast Mode=400 Kbps
- 高速模式：High speed mode=3.4 Mbps
- 超快速模式：Ultra fast mode=5 Mbps

1. 最大主设备数：无限制；
2. 最大从机数：理论上是127；



# 2. I2C 硬件层

**I2C**协议仅需要一个**SDA和SCL**引脚。SDA是**串行数据线的**缩写，而SCL是**串行时钟线的**缩写。这两条数据线需要接上拉电阻。

设备间的连接如下所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkfIPmuV6LSnYS09GsF2ulczicmaWLhmtID91RnWu2ClNzmspMvmjOw8Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用I2C，可以将多个从机（`Slave`）连接到单个主设备（`Master`），并且还可以有多个主设备（`Master`）控制一个或多个从机（`Slave`）。

> 假如希望有多个微控制器（`MCU`）将数据记录到单个存储卡或将文本显示到单个LCD时，这个功能就非常有用。

I2C总线（`SDA`，`SCL`）内部都使用漏极开路驱动器（开漏驱动），因此`SDA`和`SCL` **可以被拉低为低电平，但是不能被驱动为高电平**，所以每条线上都要使用一个上拉电阻，默认情况下将其保持在高电平；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkqNrFFGVlbSAuXNBRSoLdqjeKzn8LXaeNKHdqeToROX6D3ZNkJgoyng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上**拉电阻的**值取决于许多因素。德州仪器TI 建议 使用以下公式来计算正确的上拉电阻值：

$$
R_p(min)=\frac{V_{DD} -V_{OL}(max)}{I_{OL}}
$$

$$
R_p(min)=\frac{t_r}{0.8473xC_b}
$$

其中$V_{OL}$是逻辑低电压；

$I_{OL}$是逻辑低电流；

$t_r$是信号的最大上升时间；

$C_b$是总线（电线）电容；

具体如下所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkhw48d805JTxrIJJmdHqkeHXuKvQdxSANnllV8sagTrpPZkdDmnNu9A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

根据上表，这里不难发现需要在做电阻选择需要满足几个条件；

- 灌电流 最大值为$3mA$；
- 另外I2C总线规范和用户手册还为低电平输出电压设置了最大值为0.4V



$$
R_p(min)=\frac{VCC-0.4V}{3mA}
$$




所以根据上述公式可以计算，对于5V的电源，每个上拉电阻阻值至少1.53kΩ，而对于3.3V的电源，每个电阻阻值至少967Ω。

如果觉得计算电阻值比较麻烦，也可以使用**典型值 4.7kΩ**。

> 上述推导过程可以参考 TI的文档《I2C Bus Pullup Resistor Calculation》 https://www.ti.com/lit/an/slva689/slva689.pdf

最终在调试的时候，当我们测量SDA或SCL信号并且逻辑LOW上的电压高于0.4V时，我们就知道可以知道灌电流太高了；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkOPNIC7uRnXSIa9skjCA2w3DlxaibKiahy1hNWhNNiccRjib9UDvue9eJSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然，这并不意味着每当灌电流超过3mA时，设备就会立即停止工作。但是，在操作超出其规格的设备时，应始终小心，因为它可能导致通信故障，缩短其使用寿命甚至甚至永久损坏设备。



# 3. I2C 数据传输协议

主设备和从设备进行数据传输时遵循以下协议格式。数据通过一条SDA数据线在主设备和从设备之间传输`0`和`1`的串行数据。串行数据序列的结构可以分为，开始条件，地址位，读写位，应答位，数据位，停止条件，具体如下所示；![img](https://mmbiz.qpic.cn/mmbiz_jpg/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkFHiceEF4iaAtMfuya4GGtV5KNxHCv6lZvpErYRuKRvScczQGbq5HgK1w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**开始条件**

当主设备决定开始通讯时，需要发送开始信号，需要执行以下动作；

- 先将SDA线从高压电平切换到低压电平；
- 然后将`SCL`从高电平切换到低电平；

在主设备发送开始条件信号之后，所有从机即使处于睡眠模式也将**变为活动状态**，并**等待接收地址位**。

具体如下图所示；

![img](https://mmbiz.qpic.cn/mmbiz_jpg/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkSHBe8l9C9zuiaDMp4T5grQqk9DAWFv74ByQAt54KXmMwDUPrhwSYsJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**地址位**

通常地址位占7位数据，主设备如果需要向从机发送/接收数据，首先要发送对应从机的地址，然后会匹配总线上挂载的从机的地址；

> I2C还支持10位寻址；

**读写位**

该位指定数据传输的方向；

- 如果主设备需要将数据发送到从设备，则该位设置为 `0`；
- 如果主设备需要往从设备接收数据，则将其设置为 `1` 。

**ACK / NACK**

主机每次发送完数据之后会等待从设备的应答信号`ACK`；

- 在第9个时钟信号，如果从设备发送应答信号`ACK`，则`SDA`会被拉低；
- 若没有应答信号`NACK`，则`SDA`会输出为高电平，这过程会引起主设备发生重启或者停止；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkNoDiagdADCsoe88TPiabu28RmiaIo5eV0RQtE9picrqDr09m7EZbvJmY7w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**数据块**

传输的数据总共有8位，由发送方设置，它需要将数据位传输到接收方。

发送之后会紧跟一个`ACK` / `NACK`位，如果接收器成功接收到数据，则设置为0。否则，它保持逻辑“ 1”。

重复发送，直到数据完全传输为止。

**停止条件**

当主设备决定结束通讯时，需要发送开始信号，需要执行以下动作；

- 先将SDA线从低电压电平切换到高电压电平；
- 再将SCL线从高电平拉到低电平；

具体如下图所示；

![img](https://mmbiz.qpic.cn/mmbiz_jpg/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkRNvyK43CVib7dnARrou7ico5EBuB18icbqMjRw8lMIQnSBHnnq6kDDtdg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#  4. I2C 如何工作？

**第一步：起始条件**

主设备通过将SDA线从高电平切换到低电平，再将SCL线从高电平切换到低电平，来向每个连接的从机发送启动条件 ：

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkRUUQyX9xWoU0PdGInhA9GibFfyq6aeIa18Mt9CXcicvMWPxzI3GXrJ1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**第二步：发送从设备地址**

主设备向每个从机发送要与之通信的从机的7位或10位地址，以及相应的**读/写位**；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkKu7sAfBShVpnsRQuice2OnuCLsbyEDeMPicVmzTooXVZQbzWY3914icmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**第三步：接收应答**

每个从设备将主设备发送的地址与其自己的地址进行比较。如果地址匹配，则从设备通过**将SDA线拉低一位以表示返回一个ACK位**；

如果来自主设备的地址与从机自身的地址不匹配，则**从设备将SDA线拉高，表示返回一个NACK位**；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPk45kx2dQXqSMPqYC4rywicnKXXuGatJXyCqavgL7NSbJibiaVxSOoOvxkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**第四步：收发数据**

主设备发送或接收数据到从设备；

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkcZN9iaEoLqJTCRVBSBz3fB6IzkDNicibmAvbztGXql2nZQfDicibjvsyYFQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**第五步：接收应答**

在传输完每个数据帧后，接收设备将另一个ACK位返回给发送方，以确认已成功接收到该帧：

![img](https://mmbiz.qpic.cn/mmbiz_png/aYs0iczibHlYib7iaKbJm6C7TzxicfzcYCCPkzc568qARQ4YfQhcqwAD66mcl6iaqBol9uvv1MLb8NFibiaiaYMHpYZK8pQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**第六步：停止通信**

为了停止数据传输，主设备将SCL切换为高电平，然后再将SDA切换为高电平，从而向从机发送停止条件；

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#  5. 单个主设备连接多个从机

I2C总线上的主设备使用7位地址对从设备进行寻址，可以使用128（）个从机地址。

> 请使用4.7K上拉电阻将SDA和SCL线连接到Vcc；

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#  6. 多个主设备连接多个从机

多个主设备可以连接到一个或多个从机；

当两个主设备试图通过SDA线路同时发送或接收数据时，同一系统中的多个主设备就会出现问题。

为了解决这个问题，**每个主设备都需要在发送消息之前检测SDA线是低电平还是高电平**；

- 如果SDA线为低电平，则意味着另一个主设备可以控制总线，并且主设备应等待发送消息。

- 如果SDA线为高电平，则可以安全地发送消息。

  > 要将多个主设备连接到多个从机，请使用下图，其中4.7K上拉电阻将SDA和SCL线连接到Vcc：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#  7. I2C 编程

Talk is cheap. Show me the code.

参考了STM32的HAL库中I2C驱动，主设备发送函数`HAL_I2C_Master_Transmit()`具体如下：

```
/**
  * @brief  Transmits in master mode an amount of data in blocking mode.
  * @param  hi2c Pointer to a I2C_HandleTypeDef structure that contains
  *                the configuration information for the specified I2C.
  * @param  DevAddress Target device address: The device 7 bits address value
  *         in datasheet must be shifted to the left before calling the interface
  * @param  pData Pointer to data buffer
  * @param  Size Amount of data to be sent
  * @param  Timeout Timeout duration
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_I2C_Master_Transmit(I2C_HandleTypeDef *hi2c, 
                                          uint16_t DevAddress, 
                                          uint8_t *pData, 
                                          uint16_t Size, 
                                          uint32_t Timeout){
  uint32_t tickstart = 0x00U;

  /* Init tickstart for timeout management*/
  tickstart = HAL_GetTick();

  if(hi2c->State == HAL_I2C_STATE_READY){
    /* Wait until BUSY flag is reset */
    if(I2C_WaitOnFlagUntilTimeout(hi2c, I2C_FLAG_BUSY, SET, I2C_TIMEOUT_BUSY_FLAG, tickstart) != HAL_OK){
      return HAL_BUSY;
    }

    /* Process Locked */
    __HAL_LOCK(hi2c);

    /* Check if the I2C is already enabled */
    if((hi2c->Instance->CR1 & I2C_CR1_PE) != I2C_CR1_PE){
      /* Enable I2C peripheral */
      __HAL_I2C_ENABLE(hi2c);
    }

    /* Disable Pos */
    hi2c->Instance->CR1 &= ~I2C_CR1_POS;

    hi2c->State     = HAL_I2C_STATE_BUSY_TX;
    hi2c->Mode      = HAL_I2C_MODE_MASTER;
    hi2c->ErrorCode = HAL_I2C_ERROR_NONE;

    /* Prepare transfer parameters */
    hi2c->pBuffPtr    = pData;
    hi2c->XferCount   = Size;
    hi2c->XferOptions = I2C_NO_OPTION_FRAME;
    hi2c->XferSize    = hi2c->XferCount;

    /* Send Slave Address */
    if(I2C_MasterRequestWrite(hi2c, DevAddress, Timeout, tickstart) != HAL_OK){
      if(hi2c->ErrorCode == HAL_I2C_ERROR_AF){
        /* Process Unlocked */
        __HAL_UNLOCK(hi2c);
        return HAL_ERROR;
      }else{
        /* Process Unlocked */
        __HAL_UNLOCK(hi2c);
        return HAL_TIMEOUT;
      }
    }

    /* Clear ADDR flag */
    __HAL_I2C_CLEAR_ADDRFLAG(hi2c);

    while(hi2c->XferSize > 0U){
      /* Wait until TXE flag is set */
      if(I2C_WaitOnTXEFlagUntilTimeout(hi2c, Timeout, tickstart) != HAL_OK){
        if(hi2c->ErrorCode == HAL_I2C_ERROR_AF){
          /* Generate Stop */
          hi2c->Instance->CR1 |= I2C_CR1_STOP;
          return HAL_ERROR;
        }else{
          return HAL_TIMEOUT;
        }
      }
      /* Write data to DR */
      hi2c->Instance->DR = (*hi2c->pBuffPtr++);
      hi2c->XferCount--;
      hi2c->XferSize--;

      if((__HAL_I2C_GET_FLAG(hi2c, I2C_FLAG_BTF) == SET) 
         && (hi2c->XferSize != 0U)){
        /* Write data to DR */
        hi2c->Instance->DR = (*hi2c->pBuffPtr++);
        hi2c->XferCount--;
        hi2c->XferSize--;
      }
      /* Wait until BTF flag is set */
      if(I2C_WaitOnBTFFlagUntilTimeout(hi2c, Timeout, tickstart) != HAL_OK){
          
        if(hi2c->ErrorCode == HAL_I2C_ERROR_AF){
          /* Generate Stop */
          hi2c->Instance->CR1 |= I2C_CR1_STOP;
          return HAL_ERROR;
        }else{
          return HAL_TIMEOUT;
        }
      }
    }

    /* Generate Stop */
    hi2c->Instance->CR1 |= I2C_CR1_STOP;

    hi2c->State = HAL_I2C_STATE_READY;
    hi2c->Mode = HAL_I2C_MODE_NONE;
    
    /* Process Unlocked */
    __HAL_UNLOCK(hi2c);

    return HAL_OK;
  }else{
    return HAL_BUSY;
  }
}
```



# 8. 总结

本文主要介绍I2C的入门基础知识，从I2C协议的硬件层，协议层进行了简单介绍；作者能力有限，难免存在错误和纰漏，请大佬不吝赐教。

你和我各有一个苹果，如果我们交换苹果的话，我们还是只有一个苹果。但当你和我各有一个想法，我们交换想法的话，我们就都有两个想法了。

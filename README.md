# OneIM v1.0 协议规范

## 摘要
OneIM 是轻量级即时通讯服务器，它可以通过很少的代码和带宽和远程设备连接。以及在一些自动化或小型设备上，而且由于小巧，省电，协议开销小和能高效的向一和多个接收者传递信息，故同样适用于移动应用设备上。

## 1. 规范简介
本规范分为三个主要部分

1. 数据包的消息格式
2. 每种消息类型的具体消息格式
3. 数据包如何在服务端和客户端之间传递

### 2. 消息格式
每个命令消息的消息头包含一个Fixed Header， 有些消息类型还包含一个Variable Header。 下面的章节描述了消息头部每个部分的格式：

### 2.1 Fixed header
每个数据包都包含一个固定的头部， 下面的表格显示了固定的标题格式：

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>byte 1</td>
			<td colspan="5"> Message Type </td>
			<td>DUP</td>
			<td>QoS</td>
			<td>RETAIN</td>
		</tr>
		<tr>
			<td>byte 2</td>
			<td colspan="8"> Remaining Length </td>
		</tr>
	</tbody>
<table>

#### Byte 1

包含消息类型和一些标记值(DUP flag、QoS level和retain)

#### Byte 2

至少一个字节的长度， 表示剩余部分的长度

在一下几节中描述的字断， 所有的数据都是大端顺序： 高阶字节先于低阶字节序。

#### 消息类型

**Position**：bits byte 1, 7 - 3

这是一个 5-bit 无符号值，这个版本的枚举值如下表:

| 枚举值 | Hex  | COMMAND     | DESCRIPTION         |
| ----- |:----:|:------------|:--------------------|
|  00   | 0x00 | Reserved    | 保留字段 |
|  01   | 0x01 | CONNECT     | 客户端请求连接到服务器 |
|  02   | 0x02 | CONNACK     | 连接请求确认 |
|  03   | 0x03 | PUBLISH     | 发布消息 |
|  04   | 0x04 | PUBACK      | 发布确认 |
|  05   | 0x05 | RECEIVE     | 接收消息 |
|  06   | 0x06 | RECEACK     | 接收确认 |
|  07   | 0x07 | PROPERTY    | 设置设备属性 |
|  08   | 0x08 | PROPACK     | 设置设备属性ack |
|  09   | 0x09 | PING        | PING请求 |
|  10   | 0x0A | PONG        | PONG返回 |
|  11   | 0x0B | CMDREQ      | 扩展命令请求 |
|  12   | 0x0C | CMDRESP     | 扩展命令返回 |
|  13   | 0x0D | DISCONNECT  | 客户端断开连接 |
|  14   | 0x0E | Reserved    |      -      |
|  15   | 0x0F | Reserved    |      -      |
|  16   | 0x10 | MVNODE      | 接入节点变更， 返回新节点地址 |
|  17   | 0x11 | MVNODEACK   | 接入节点变更ACK，之后断开连接 |
|  18   | 0x12 | Reserved    |      -      |
|  19   | 0x13 | Reserved    |      -      |
|  20   | 0x14 | Reserved    |      -      |
|  21   | 0x15 | Reserved    |      -      |
|  22   | 0x16 | Reserved    |      -      |
|  23   | 0x17 | Reserved    |      -      |
|  24   | 0x18 | Reserved    |      -      |
|  25   | 0x19 | Reserved    |      -      |
|  26   | 0x1A | Reserved    |      -      |
|  27   | 0x1B | Reserved    |      -      |
|  28   | 0x1C | Reserved    |      -      |
|  29   | 0x1D | Reserved    |      -      |
|  30   | 0x1E | Reserved    |      -      |
|  31   | 0x1F | Reserved    |      -      |

#### 标记值

剩余的标记位包含字段DUP、QoS和retain. 标识位的含义如下表：

| postion |  NAME  | DESCRIPTION |
|:-------:|:------:|:------------|
|    2    | DUP    | 标识是否为重传的消息 |
|    1    | QoS    | 标记服务质量 |
|    0    | RETAIN | 标识是否序列化消息 |

#### DUP

**Position**: byte 1, bit 2

这个标识设置时，表示这是一条未收到ACK的重新发送的消息。

#### QoS

**Position**: byte 1, bit 1

此标识指示发布消息的交付级别， 下面的表格显示了服务质量的水平。

| QoS | bit | DESCRIPTION |
|:---:|:---:|:------------|
|  0  |  0  | 不需要接收方发送ACK响应 |
|  1  |  1  | 需要接收方发送ACK响应 |

#### RETAIN

**Position**: byte 1, bit 0

此标识表示了服务端是否永久保存消息， 直到被所有的客户端收到。

| retain | bit | DESCRIPTION |
|:------:|:---:|:------------|
|    0   |  0  | 不在服务端序列化消息， 在连接断开的时候丢失消息 |
|    1   |  1  | 在服务端序列化消息， 在连接断开的时候消息不丢失 |

#### 剩余长度

**Position**: byte 2

表示当前消息中的剩余字节数， 包括Variable Header和payload中的数据大小。

在长度小于127的时候，可变长度的编码方案采用1 byte进行编码， 在大于127 的时候， 每个byte 使用7 bit进行编码。 第8 bit表示是否使用下一个字节进行编码。 每增加一个字节编码， 可以表示的长度值增加128倍。 例如，64 可以使用1 byte(0x40)进行编码。再比如，321 ＝ 65 + 2 * 128 使用2 byte进行编码，第一个byte的值是(65 + 128 = 193)。

该协议限制在表示中的字节数最多为4，这表示允许发送的信息最多为435 455 268（256MB）, 这个数字表示为: 0x7F 0xFF 0xFF 0xFF。

字节数增加可以表示的长度值的对应关系如下：

|Digits| From                                 | To     |
|:----:|:-------------------------------------|:------------|
|   1  | 0 (0x00)                             | 127 (0x7F) |
|   2  | 128 (0x80, 0x01)                     | 16 383 (0xFF, 0x7F) |
|   3  | 16 384 (0x80, 0x80, 0x01)            | 2 097 151 (0xFF, 0xFF, 0x7F) |
|   4  | 	2 097 152 (0x80, 0x80, 0x80, 0x01)  | 268 435 455 (0xFF, 0xFF, 0xFF, 0x7F) |

编码一个十进制数（×）到可变长度的编码方案的算法如下：

```
function encode(long x) {
    do {
        int digit = x % 128;
        x = x / 128;
        if (x > 0) {
          digit = digit / 0x80;
        }
        printf(digit);
    } while) (x > 0);
}
```

解码可变长度的算法如下：

```
function decode() {
    multiplier = 1;
    value = 0;
    do {
    	digit = 'next digit from stream' 
    	value += (digit AND 127) * multiplier
    	multiplier *= 128
    } while ((digit AND 128) != 0);
}
```

可变长度编码不是Variable Header的一部分， 用于编码剩余长度的字节数不算入该字段值。 可变长度是Fixed Header的一部分，而不是Variable Headers.

### 2.2 Variable header

某些类型的消息也包含一个Variable Header，它的位置在Fixed Header和payload之间。

在下面的部分描述了Variable Header中的字段格式

#### 协议版本

在CONNECT命令中， 协议版本是必须存在于Variable Header。该字段是一个8位无符号值，该值表示客户端使用的协议的版本。协议当前版本是1 (0x01)， 格式如下表所示：

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> </td>
			<td colspan="8"> Protocol Version </td>
		</tr>
		<tr>
			<td> </td>
			<td>0</td>
			<td>0</td>
			<td>0</td>
			<td>0</td>
			<td>0</td>
			<td>0</td>
			<td>0</td>
			<td>1</td>
		</tr>
	</tbody>
<table>

#### 连接标识

在CONNECT命令中， 连接标识是必须存在于Variable Header。该字段包含Clean session, Will, Will QoS, and Retain flags， 格式如下表：

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> </td>
			<td> Username Flag </td>
			<td> Password Flag </td>
			<td> Clean Session </td>
			<td> Compression </td>
			<td> Will Flag </td>
			<td> - </td>
			<td> - </td>
			<td> - </td>
		</tr>
		<tr>
			<td> </td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
		</tr>
		<tr>
			<td> </td>
			<td colspan="8"> Keep Alive Timer </td>
		</tr>
		<tr>
			<td> </td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
			<td>x</td>
		</tr>
	</tbody>
<table>

**compression**
**Position:** 4 bit

| 枚举值 | Hex  | DESCRIPTION |
| ----- |:----:|:------------|
|  00   | 0x00 | 不压缩 |
|  01   | 0x01 | GZIP压缩 |

**Will Flag**
**Position:** 3 bit

| 枚举值 | Hex  | DESCRIPTION |
| ----- |:----:|:------------|
|  00   | 0x00 | 连接断开后可以清理Client |
|  01   | 0x01 | 连接断开后不可以清理Client |

**Clean Session**

**Position:** 5 bit

如果这个标识没有被设置(0)，在连接完成后， 服务端要继续推送上次这个设备没有接收的retain消息。 如果被设置(1)，则表示这是一个新设备接入，需要服务端在返回的时候返回当前的消息队列位置。

服务器可以提供一个管理机制， 用来清除存储在一个客户机上的信息，当认为客户机永远不会重新连接时。 当Will Flag 设置为0， 并且在64个Keep Alive Timer之后没有重新连接， 则认为设备永远不会重新连接。

**Username and Password**

**Position:** 6 bit and 7 bit

一个客户端可以选择指定一个用户名和密码， 表示在payload中包含了用户名和密码。

如果设置了用户名标识， 则payload中的用户名字段是强制性的， 否则用户名的值将被忽略。
如果设置了密码标识， 则payload中的用户名字段是强制性的， 否则用户名的值将被忽略。但是只提供一个密码， 不提供用户名是无效的。

#### Keep Alive Timer

在CONNECT命令中， 连接标识是必须存在于Variable Header。

该字段单位为秒， 定义了从客户端接收消息的最大时间间隔。 它用于服务器检测到客户端的网络连接是否假死， 从而避开了漫长的TCP/IP超时。 客户端应该在这个时间间隔内发送一个消息到服务端，如果没有有用的信息传输， 则发送一个PING消息， 服务端确认后返回一个PONG消息.

如果服务器在这个1.5倍的间隔内没有收到客户端的消息， 它会断开到客户端的TCP/IP连接。

如果客户端在这个时间间隔内没有收到服务端返回的消息， 它会断开到服务器的TCP/IP连接。

保持生命的计时器是一个11位的值，表示时间段的秒数。 实际值是特定于应用程序，但一个典型值是几分钟。

最大值是2048s，（0）值意味着客户端没有DISCONNECT的操作。

#### 连接返回代码

这个Variable Header包含在CONNACK消息中。 这个字段定义了一一个字节无符号返回码。
值的含义，如下面的表格所示，是特定于消息类型。零（0）的返回码通常表示成功。

| 枚举值 | Hex  | DESCRIPTION         |
| ----- |:----:|:------------|:--------------------|
|  00   | 0x00 | 接受连接 |
|  01   | 0x01 | 连接拒绝：不可接受的协议版本 |
|  02   | 0x02 | 连接拒绝：被拒绝 |
|  03   | 0x03 | 连接拒绝：服务器不可用 |
|  04   | 0x04 | 连接拒绝：用户名或密码错误 |
|  05   | 0x05 | 连接拒绝：未授权 |
| 6-255 |  -   | 保留供以后使用 |

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> </td>
			<td colspan="8"> Return Code </td>
		</tr>
	</tbody>
<table>

### 2.3 Payload

以下类型的消息类型需要有一个Payload:

**CONNECT**
	payload包含一个或多个UTF-8编码的字符串. 包含了一个Client ID，用户名和密码。 其中Client ID是必须的， 用户名和密码是否存在是根据Header中的连接标识决定的。

如果你想有效的压缩payload中的数据，需要在Header中定义压缩的细节。

### 2.4 Message identifiers

消息标示符会出现在需要ACK的命令和ACK命令，并且在服务质量＝1的时候需要Message ID和ACK。

Message ID是一个16-bit的无符号整数，在单个命令的通信中必须是唯一的。它通常会根据消息数递增。

客户端将自己的消息列表保存到服务器所使用的信息系统中。在同一时间内，可能发送一个Message ID＝1的消息和接受一个MessageID=1的消息。

这个两个字节的Message ID的顺序是MSB在前面， LSB在后面.

不要使用0作为Message ID， 因为0被保留为无效的标识。

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> </td>
			<td colspan="8"> Message Identifier MSB </td>
		</tr>
		<tr>
			<td> </td>
			<td colspan="8"> Message Identifier LSB </td>
		</tr>
	</tbody>
<table>

### 2.5 信息编码 UTF-8
UTF-8是一种有效的Unicode编码的字符串进行编码的ASCII字符在基于文本的通信支持。

我们定义了字符串的前两个字节表示字符串的长度，如下表所示。

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> byte 1 </td>
			<td colspan="8"> String Length MSB </td>
		</tr>
		<tr>
			<td> byte 2 </td>
			<td colspan="8"> String Length LSB </td>
		</tr>
		<tr>
			<td> bytes 3 </td>
			<td colspan="8"> Encoded Character Data </td>
		</tr>
	</tbody>
<table>

字符串长度是编码字符串字符的字节数，而不是字符数。例如，字符串otwp是UTF-8编码如下表所示

<table>
	<thead>
		<tr style="background-color: #dedeff;">
			<th>bit</th>
			<th>7</th>
			<th>6</th>
			<th>5</th>
			<th>4</th>
			<th>3</th>
			<th>2</th>
			<th>1</th>
			<th>0</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> byte 1 </td>
			<td colspan="8"> String Length MSB (0x00) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
		</tr>
		<tr>
			<td> byte 2 </td>
			<td colspan="8"> String Length LSB (0x04) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 0 </td>
		</tr>
		<tr>
			<td> byte 3 </td>
			<td colspan="8"> 'O' (0x4F) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 1 </td>
			<td> 1 </td>
			<td> 1 </td>
		</tr>
		<tr>
			<td> byte 4 </td>
			<td colspan="8"> 'T' (0x54) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 0 </td>
		</tr>
		<tr>
			<td> byte 5 </td>
			<td colspan="8"> 'W' (0x57) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 1 </td>
			<td> 1 </td>
		</tr>
		<tr>
			<td> byte 6 </td>
			<td colspan="8"> 'P' (0x50) </td>
		</tr>
		<tr>
			<td> </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 1 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
			<td> 0 </td>
		</tr>
	</tbody>
<table>

在Java语言中 `writeUTF()` 和 `readUTF()` 就是使用这种方式编码的.

### 2.6 未使用的bit
	
所有未使用的bit都应该被设置为0.

## 3. 消息类型

## 4. 工作流程
### 4.1 不同QoS的服务流程
**QoS Level 0**: 最多一次到达，无ACK.

消息传递根据依赖于底层的TCP/IP网络， 不需要ACK相应。并且在协议中没有定义重试语义， 消息最多到达服务器一次。

**QoS Level 1**: 最少一次到达，有ACK.

由服务器接收一个消息并返回ACK确认。如果有一个确定的故障或通信链路或设备发送的确认消息，或不在规定时间内接收，发送方重新发送消息设置邮件头中的重复点。至少一次消息到达服务器。无论是订阅和退订消息使用QoS等级1。

QoS Level 1 的消息在Variable Header中包含一个Message ID

如果客户没有收到puback消息（无论是在应用程序中定义的时间段或，如果检测到故障，通信会话重新启动），客户端可以发送消息的发布与DUP flag。

### 4.2 消息重试

在一般情况下TCP是可以保证交付数据包， 在一些特殊情况下， 可能不会正确到达。在期望响应的情况下(QoS=1的时候)， 如果在一定的时间段内未收到ACK， 发送者可以尝试重新发送， 在重试消息中发件人应该设置DUP flag。

重试超时应该是一个可配置的选项。但是必须小心，以确保消息传递不会超时，而它仍然被发送。例如，在慢速网络上发送一个大消息，自然会比一个快速网络上的小消息要长。多次重试超时消息往往会使事情变得更糟，一个增加超时值在多个重试策略应用。

当客户端重新连接，如果不是明确指定Clean Session，客户端和服务器都应该继续传递以前的消息。

除“重新连接”重试行为，客户端不需要重试消息发送。但应服务端应该重试任何没有收到ACK的消息。

### 4.3 消息排序

消息的顺序受多种因素影响，例如客户端和服务端是否使用多线程，这里只讨论单线程的情况。

假定方案， 每次推送都推送上一次的位置之后产生的消息，到达客户端后进行解包。 在QoS level 1的情况下， 上一组的推送未返回的时候， 不推送下一组消息。

利用客户端的library解包成单个消息

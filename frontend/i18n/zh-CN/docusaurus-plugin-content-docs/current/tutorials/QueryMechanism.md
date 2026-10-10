---
title: SA:MP 查询机制
sidebar_label: SA:MP 查询机制
---

## 机制概述

SA:MP 查询机制是通过 UDP 数据包传输服务器统计信息的标准协议，可获取服务器名称、玩家延迟、语言设置、在线玩家列表等核心数据。

本文详解该协议工作原理，并指导如何在不依赖官方客户端的情况下实现查询功能。

## 查询机制

查询是通过 UDP 协议向服务器地址发送的序列化数据包。

你可能会疑惑："服务器如何区分查询数据包与常规 RakNet 协议数据？" 其原理在于底层 RakNet 套接字层会识别数据包头部的"SAMP"标识（十六进制值：53 41 4D 50），并采用特殊处理流程。[查看源码](https://github.com/openmultiplayer/RakNet/blob/master/Source/SocketLayer.cpp#L371)

## 序列化数据包

传输数据包由以下部分组成：**"SAMP"** + **IP 四元组** + **端口低字节** + **端口高字节** + **操作码**

若对 IP 四元组和端口字节的解析方式存在疑问，可参考：[IP 地址与端口解析指南](http://penta2.ufrgs.br/trouble/ts_ip.htm)

| 字节长度 |     字段      |
| :------: | :-----------: |
|    4     |    "SAMP"     |
|    4     | IP 地址四元组 |
|    1     |  端口低 8 位  |
|    1     |  端口高 8 位  |
|    1     |    操作码     |

[C 语言实现示例](https://github.com/Louzindev/sampquery-c/blob/master/src/packet.c)

## 操作码说明

各操作码对应不同查询类型：

- **0x69 ('i')**：基础信息查询  
  获取服务器密码状态、在线人数、主机名、游戏模式、语言等核心参数

- **0x72 ('r')**：规则查询  
  返回重力值、天气代码、网站链接等自定义规则参数

- **0x63 ('c')**：玩家简表  
  获取玩家昵称与得分的快速列表

- **0x64 ('d')**：玩家详情  
  包含玩家 ID、昵称、得分、延迟等详细数据

- **0x78 ('x')**：RCON 命令  
  远程控制指令通道（需认证）

- **0x70 ('p')**：延迟测试  
  通过四字节随机数计算服务器响应时间

## 响应数据包

如前述，每个操作码将返回特定格式的响应数据。

所有响应数据包前 11 字节为固定包头（与请求包头完全一致），后续字节为具体响应内容：

### `i`, `r`, `c`, `d`, `p` 响应类型数据表

#### 类型 `i` 响应

| 字节        | 键值       | 字节宽度 | 说明                               |
| ----------- | ---------- | -------- | ---------------------------------- |
| 11          | Password   | 1        | 0 表示未设置密码，1 表示已设置密码 |
| 12-13       | Players    | 2        | 当前在线玩家数量                   |
| 14-15       | MaxPlayers | 2        | 服务器最大玩家容量                 |
| 16-19       | (strlen)   | 4        | 服务器主机名字符串长度             |
| 20 + strlen | Hostname   | (strlen) | 服务器主机名                       |
| 21-24       | (strlen)   | 4        | 游戏模式字符串长度                 |
| 25 + strlen | Gamemode   | (strlen) | 服务器游戏模式                     |
| 26-29       | (strlen)   | 4        | 服务器语言字符串长度               |
| 30 + strlen | Language   | (strlen) | 服务器使用语言                     |

#### 类型 `r` 响应

| 字节        | 键值      | 字节宽度 | 说明                 |
| ----------- | --------- | -------- | -------------------- |
| 11-12       | RuleCount | 2        | 服务器提供的规则数量 |
| 13          | (strlen)  | 1        | 规则名称字符串长度   |
| 14 + strlen | Rulename  | (strlen) | 规则名称             |
| 15          | (strlen)  | 1        | 规则值字符串长度     |
| 16 + strlen | RuleValue | (strlen) | 规则值               |

_(从第 13 字节开始循环，共循环 RuleCount 次)_

#### 类型 `c` 响应

| 字节        | 键值        | 字节宽度 | 说明                 |
| ----------- | ----------- | -------- | -------------------- |
| 11-12       | PlayerCount | 2        | 服务器提供的玩家数量 |
| 13          | (strlen)    | 1        | 玩家昵称字符串长度   |
| 14 + strlen | PlayerNick  | (strlen) | 玩家昵称             |
| 15-18       | Score       | 4        | 玩家分数             |

_(从第 13 字节开始循环，共循环 PlayerCount 次)_

#### 类型 `d` 响应

| 字节        | 键值        | 字节宽度 | 说明                      |
| ----------- | ----------- | -------- | ------------------------- |
| 11-12       | PlayerCount | 2        | 服务器提供的玩家数量      |
| 13          | PlayerID    | 1        | 玩家 ID（取值范围 0-255） |
| 14          | (strlen)    | 1        | 玩家昵称字符串长度        |
| 15 + strlen | PlayerNick  | (strlen) | 玩家昵称                  |
| 16-19       | Score       | 4        | 玩家分数                  |
| 20-23       | Ping        | 4        | 玩家到服务器的延迟        |

_(从第 13 字节开始循环，共循环 PlayerCount 次)_

#### 类型 `p` 响应

| 字节 | 键值     | 字节宽度 | 说明                             |
| ---- | -------- | -------- | -------------------------------- |
| 11   | number 1 | 1        | 客户端发送的伪随机序列第一个数字 |
| 12   | number 2 | 1        | 伪随机序列第二个数字             |
| 13   | number 3 | 1        | 伪随机序列第三个数字             |
| 14   | number 4 | 1        | 伪随机序列第四个数字             |

## C 语言实现示例

开源 C 语言库 sampquery-c 实现了完整的查询功能，可作为开发参考：[代码仓库](https://github.com/Louzindev/sampquery-c)

## PHP 语言实现示例

```php
/**
 * Let's generate the string needed for the packet.
 */
$sIPAddr = "127.0.0.1"; // IP address of the server
$iPort = 7777; // Server port.
$sPacket = ""; // Blank string for packet.

$aIPAddr = explode('.', $sIPAddr); // Exploding the IP addr.

$sPacket .= "SAMP"; // Telling the server it is a SA-MP packet.

$sPacket .= chr($aIPAddr[0]); //
$sPacket .= chr($aIPAddr[1]); //
$sPacket .= chr($aIPAddr[2]); //
$sPacket .= chr($aIPAddr[3]); // Sending off the server IP,

$sPacket .= chr($iPort & 0xFF); //
$sPacket .= chr($iPort >> 8 & 0xFF); // Sending off the server port.

$sPacket .= 'i'; // The opcode that you want to send.
// You can now send this to the server.

/**
 * Let's connect now to the server.
 */
$rSocket = fsockopen('udp://'.$sIPAddr, $iPort, $iError, $sError, 2); // Create an active socket.
fwrite($rSocket, $sPacket); // Send the packet to the server.

echo fread($rSocket, 2048); // Get the output from the server

fclose($rSocket); // Close the connection
```

## C# 语言实现示例

```csharp
using System;
using System.IO;
using System.Net;
using System.Net.Sockets;

namespace Query
{
    class RCONQuery
    {
        Socket qSocket;
        IPAddress address;
        int _port = 0;
        string _password = null;
        string[] results = new string[50];
        int _count = 0;

        public RCONQuery(string IP, int port, string password)
        {
            qSocket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
            qSocket.SendTimeout = 5000;
            qSocket.ReceiveTimeout = 5000;

            try
            {
                address = Dns.GetHostAddresses(IP)[0];
            }
            catch
            {
            }

            _port = port;
            _password = password;
        }

        public bool Send(string command)
        {
            try
            {
                IPEndPoint endpoint = new IPEndPoint(address, _port);

                using (MemoryStream stream = new MemoryStream())
                {
                    using (BinaryWriter writer = new BinaryWriter(stream))
                    {
                        writer.Write("SAMP".ToCharArray());

                        string[] SplitIP = address.ToString().Split('.');

                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[0])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[1])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[2])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[3])));

                        writer.Write((ushort)_port);

                        writer.Write('x');

                        writer.Write((ushort)_password.Length);
                        writer.Write(_password.ToCharArray());

                        writer.Write((ushort)command.Length);
                        writer.Write(command.ToCharArray());
                    }

                    if (qSocket.SendTo(stream.ToArray(), endpoint) > 0)
                        return true;
                }
            }
            catch
            {
                return false;
            }

            return false;
        }

        public int Receive()
        {
            try
            {
                for (int i = 0; i < results.GetLength(0); i++)
                    results.SetValue(null, i);

                _count = 0;

                EndPoint endpoint = new IPEndPoint(address, _port);

                byte[] rBuffer = new byte[500];

                int count = qSocket.ReceiveFrom(rBuffer, ref endpoint);

                using (MemoryStream stream = new MemoryStream(rBuffer))
                {
                    using (BinaryReader reader = new BinaryReader(stream))
                    {
                        if (stream.Length <= 11)
                            return _count;

                        reader.ReadBytes(11);
                        short len;

                        try
                        {
                            while ((len = reader.ReadInt16()) != 0)
                                results[_count++] = new string(reader.ReadChars(Convert.ToInt32(len)));
                        }
                        catch
                        {
                            return _count;
                        }
                    }
                }
            }
            catch
            {
                return _count;
            }

            return _count;
        }

        public string[] Store(int count)
        {
            string[] rString = new string[count];

            for (int i = 0; i < count && i < _count; i++)
                rString[i] = results[i];

            _count = 0;

            return rString;
        }
    }

    class Query
    {
        Socket qSocket;
        IPAddress address;
        int _port = 0;
        string[] results;
        int _count = 0;
        DateTime[] timestamp = new DateTime[2];

        public Query(string IP, int port)
        {
            qSocket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
            qSocket.SendTimeout = 5000;
            qSocket.ReceiveTimeout = 5000;

            try
            {
                address = Dns.GetHostAddresses(IP)[0];
            }
            catch
            {
            }

            _port = port;
        }

        public bool Send(char opcode)
        {
            try
            {
                EndPoint endpoint = new IPEndPoint(address, _port);

                using (MemoryStream stream = new MemoryStream())
                {
                    using (BinaryWriter writer = new BinaryWriter(stream))
                    {
                        writer.Write("SAMP".ToCharArray());

                        string[] SplitIP = address.ToString().Split('.');

                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[0])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[1])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[2])));
                        writer.Write(Convert.ToByte(Convert.ToInt32(SplitIP[3])));

                        writer.Write((ushort)_port);

                        writer.Write(opcode);

                        if (opcode == 'p')
                            writer.Write("8493".ToCharArray());

                        timestamp[0] = DateTime.Now;
                    }

                    if (qSocket.SendTo(stream.ToArray(), endpoint) > 0)
                        return true;
                }
            }
            catch
            {
                return false;
            }

            return false;
        }

        public int Receive()
        {
            try
            {
                _count = 0;

                EndPoint endpoint = new IPEndPoint(address, _port);

                byte[] rBuffer = new byte[3402];
                qSocket.ReceiveFrom(rBuffer, ref endpoint);

                timestamp[1] = DateTime.Now;

                using (MemoryStream stream = new MemoryStream(rBuffer))
                {
                    using (BinaryReader reader = new BinaryReader(stream))
                    {
                        if (stream.Length <= 10)
                            return _count;

                        reader.ReadBytes(10);

                        switch (reader.ReadChar())
                        {
                            case 'i': // Information
                            {
                                results = new string[6];

                                results[_count++] = Convert.ToString(reader.ReadByte());

                                results[_count++] = Convert.ToString(reader.ReadInt16());
                                results[_count++] = Convert.ToString(reader.ReadInt16());

                                int hostnamelen = reader.ReadInt32();
                                results[_count++] = new string(reader.ReadChars(hostnamelen));

                                int gamemodelen = reader.ReadInt32();
                                results[_count++] = new string(reader.ReadChars(gamemodelen));

                                int languagelen = reader.ReadInt32();
                                results[_count++] = new string(reader.ReadChars(languagelen));

                                return _count;
                            }

                            case 'r': // Rules
                            {
                                int rulecount = reader.ReadInt16();

                                results = new string[rulecount * 2];

                                for (int i = 0; i < rulecount; i++)
                                {
                                    int rulelen = reader.ReadByte();
                                    results[_count++] = new string(reader.ReadChars(rulelen));

                                    int valuelen = reader.ReadByte();
                                    results[_count++] = new string(reader.ReadChars(valuelen));
                                }

                                return _count;
                            }

                            case 'c': // Client list
                            {
                                int playercount = reader.ReadInt16();

                                results = new string[playercount * 2];

                                for (int i = 0; i < playercount; i++)
                                {
                                    int namelen = reader.ReadByte();
                                    results[_count++] = new string(reader.ReadChars(namelen));

                                    results[_count++] = Convert.ToString(reader.ReadInt32());
                                }

                                return _count;
                            }

                            case 'd': // Detailed player information
                            {
                                int playercount = reader.ReadInt16();

                                results = new string[playercount * 4];

                                for (int i = 0; i < playercount; i++)
                                {
                                    results[_count++] = Convert.ToString(reader.ReadByte());

                                    int namelen = reader.ReadByte();
                                    results[_count++] = new string(reader.ReadChars(namelen));

                                    results[_count++] = Convert.ToString(reader.ReadInt32());
                                    results[_count++] = Convert.ToString(reader.ReadInt32());
                                }

                                return _count;
                            }

                            case 'p': // Ping
                            {
                                results = new string[1];

                                results[_count++] = ((int)timestamp[1].Subtract(timestamp[0]).TotalMilliseconds).ToString();

                                return _count;
                            }

                            default:
                                return _count;
                        }
                    }
                }
            }
            catch
            {
                return _count;
            }
        }

        public string[] Store(int count)
        {
            string[] rString = new string[count];

            for (int i = 0; i < count && i < _count; i++)
                rString[i] = results[i];

            _count = 0;

            return rString;
        }
    }
}
```

```csharp
Query.Query sQuery = new Query.Query("127.0.0.1", 7777);

sQuery.Send('i');

int count = sQuery.Receive();
string[] info = sQuery.Store(count);

/*
 * Variable 'info' might now contain:
 * Password Players Max. players Hostname Gamemode Language
 * { "0", "12", "500", "Query test server", "LVDM", "English" }
 */
```

```csharp
Query.RCONQuery sQuery = new Query.RCONQuery("127.0.0.1", 7777, "changeme");

sQuery.Send("echo Hello from C#");

int count = sQuery.Receive();
string[] info = sQuery.Store(count);

/*
 * Variable 'info' might now contain:
 * { "Hello from C#" }
 */
```

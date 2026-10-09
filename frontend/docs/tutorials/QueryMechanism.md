---
title: "SA:MP Query Mechanism"
sidebar_label: "SA:MP Query Mechanism"
---

## Introduction

The SA:MP Query Mechanism is nothing more than the mechanism for transmitting server statistics and information, such as name, ping, language, online players, etc...

In this article I will document how this mechanism works, as well as teach you how to use it without needing the original client.

## Queries

Queries are pure UDP packets sent to the server address containing serialized data.

You may ask yourself: "But how does the server interpret query packets differently than those from the RakNet protocol?", and the answer is simple: in the low RakNet socket layer, packets that contain 53 41 4D 50 or translated into characters "SAMP" at the beginning, are treated in a different way. **[View Code](https://github.com/openmultiplayer/RakNet/blob/master/Source/SocketLayer.cpp#L371)**

## Serialized Data

The data transmitted in the packet is: **"SAMP"** + **IP octets** + **first port byte\*** + **second port byte** + **OPCODE**

If you have doubts about why to extract and what IP octets and port bytes are, see: [Link](http://penta2.ufrgs.br/trouble/ts_ip.htm).

| Byte size |       Name       |
| :-------: | :--------------: |
|     4     |      "SAMP"      |
|     4     |    IP Octets     |
|     1     |   Port & 0xFF    |
|     1     | Port >> 8 & 0xFF |
|     1     |      OPCODE      |

For example, an `i` query sent to the server `192.168.200.103:7777` consists of these bytes (in hexadecimal): `53 41 4D 50` ("SAMP"), `C0 A8 C8 67` (192, 168, 200 and 103), `61 1E` (7777, which is 0x1E61) and `69` ("i").

[Example in C](https://github.com/Louzindev/sampquery-c/blob/master/src/packet.c)

## OPCODE

OPCODE's are package identifiers, and each one represents a different request.

- **OPCODE "i" or 0x69:** This stands for information. This gets the amount of players in the server, the map name, and all the stuff like that. It's really useful for describing your server without changing anything.

- **OPCODE "r" or 0x72:** This stands for rules. 'Rules' when it comes to SA:MP includes the instagib, the gravity, weather, the website URL, and so on

- **OPCODE "c" or 0x63:** It stands for client list, this sends back to the server the players' name, and then the players' score. Just imagine it as a basic overview of all the players.

- **OPCODE "d" or 0x64:** This stands for detailed player information. With this, you can get everything from the ping to the player, the player ID (useful for admin scripts), the score again, and also the username.

- **OPCODE "x" or 0x78:** This is an RCON command, and it's completely different from all of the other packets.

- **OPCODE "p" or 0x70:** Four psuedo-random characters are sent to the server, and the same characters are returned. You can use the time between sending and receiving to work out the servers' ping/latency.

:::note

If more than 100 players are online, the server does not respond to the `c` (client list) query.

:::

## RCON Packets

The RCON packet (OPCODE "x") carries more data than the other queries: the 11 bytes described above are followed by the RCON password and the command to execute, each prefixed with its length as a 2-byte number. The server only answers RCON packets when the [Remote Console](../server/RemoteConsole) is enabled (`rcon.enable` in [config.json](../server/config.json)).

The table below uses `changeme` as the RCON password and `varlist` as the command; the values and offsets will be different for other passwords and commands.

| Byte  | Key      | Byte Width | Hexadecimal             | Description                 |
| ----- | -------- | ---------- | ----------------------- | --------------------------- |
| 11-12 | (strlen) | 2          | 08 00                   | Length of the RCON password |
| 13-20 | Password | (strlen)   | 63 68 61 6E 67 65 6D 65 | The RCON password           |
| 21-22 | (strlen) | 2          | 07 00                   | Length of the RCON command  |
| 23-29 | Command  | (strlen)   | 76 61 72 6C 69 73 74    | The RCON command            |

The server sends the output of the command back one line per packet: the 11-byte header, followed by the length of the line (2 bytes) and the line itself.

## Response

As stated above, each OPCODE returns information.

the response consists of the same first 11 bytes sent, what we call the Header, then the definitive response.

### Response Tables for `i`, `r`, `c`, `d`, `p`

#### Response Type `i`

| Byte        | Key        | Byte Width | Description                                             |
| ----------- | ---------- | ---------- | ------------------------------------------------------- |
| 11          | Password   | 1          | Either 0 or 1, depending on whether the password is set |
| 12-13       | Players    | 2          | Current number of players online                        |
| 14-15       | MaxPlayers | 2          | Maximum number of players allowed on the server         |
| 16-19       | (strlen)   | 4          | Length of the server’s hostname                         |
| 20 + strlen | Hostname   | (strlen)   | Hostname of the server                                  |
| 21-24       | (strlen)   | 4          | Length of the server’s gamemode                         |
| 25 + strlen | Gamemode   | (strlen)   | Gamemode of the server                                  |
| 26-29       | (strlen)   | 4          | Length of the server’s language                         |
| 30 + strlen | Language   | (strlen)   | Language of the server                                  |

#### Response Type `r`

| Byte        | Key       | Byte Width | Description                            |
| ----------- | --------- | ---------- | -------------------------------------- |
| 11-12       | RuleCount | 2          | Number of rules provided by the server |
| 13          | (strlen)  | 1          | Length of the rule name                |
| 14 + strlen | Rulename  | (strlen)   | Name of the rule                       |
| 15          | (strlen)  | 1          | Length of the rule value               |
| 16 + strlen | RuleValue | (strlen)   | Value of the rule                      |

_(Repeat from Byte 13 for each rule, as many times as `RuleCount`)_

#### Response Type `c`

| Byte        | Key         | Byte Width | Description                              |
| ----------- | ----------- | ---------- | ---------------------------------------- |
| 11-12       | PlayerCount | 2          | Number of players provided by the server |
| 13          | (strlen)    | 1          | Length of the player’s nickname          |
| 14 + strlen | PlayerNick  | (strlen)   | Player’s nickname                        |
| 15-18       | Score       | 4          | Player’s score                           |

_(Repeat from Byte 13 for each player, as many times as `PlayerCount`)_

#### Response Type `d`

| Byte        | Key         | Byte Width | Description                              |
| ----------- | ----------- | ---------- | ---------------------------------------- |
| 11-12       | PlayerCount | 2          | Number of players provided by the server |
| 13          | PlayerID    | 1          | Player’s ID (values 0-255)               |
| 14          | (strlen)    | 1          | Length of the player’s nickname          |
| 15 + strlen | PlayerNick  | (strlen)   | Player’s nickname                        |
| 16-19       | Score       | 4          | Player’s score                           |
| 20-23       | Ping        | 4          | Player’s ping to the server              |

_(Repeat from Byte 13 for each player, as many times as `PlayerCount`)_

#### Response Type `p`

| Byte | Key      | Byte Width | Description                                                   |
| ---- | -------- | ---------- | ------------------------------------------------------------- |
| 11   | number 1 | 1          | First number of the pseudo-random sequence sent by the client |
| 12   | number 2 | 1          | Second number of the pseudo-random sequence                   |
| 13   | number 3 | 1          | Third number of the pseudo-random sequence                    |
| 14   | number 4 | 1          | Fourth number of the pseudo-random sequence                   |

## Example Code in C

A while ago I made a small lib in C, which allows you to perform queries, you can use it as an example. **[See Repository](https://github.com/Louzindev/sampquery-c)**

## Example Code in PHP

The following PHP code builds an `i` query, sends it to the server and prints the raw response:

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

## Example Code in C#

The following C# classes send queries (`Query`) and RCON commands (`RCONQuery`) and read the responses:

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

To query the server information, use the methods like this:

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

To send an RCON command:

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

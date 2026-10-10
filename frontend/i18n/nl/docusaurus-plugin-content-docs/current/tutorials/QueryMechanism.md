---
title: "SA:MP Query-mechanisme"
sidebar_label: "SA:MP Query-mechanisme"
---

## Introductie

Het SA:MP Query-mechanisme verstuurt serverstatistieken en -informatie (naam, ping, taal, online spelers, enz.). Hier staat hoe het werkt en hoe je het zonder de originele client kunt gebruiken.

## Queries

Queries zijn pure UDP-pakketten met geserialiseerde data.

In de lagere RakNet-socketlaag worden pakketten die beginnen met 53 41 4D 50 ("SAMP") anders behandeld. Zie de code: **[View Code](https://github.com/openmultiplayer/RakNet/blob/master/Source/SocketLayer.cpp#L371)**

## Geserialiseerde data

De payload: **"SAMP"** + **IP-octets** + **eerste poortbyte\*** + **tweede poortbyte** + **OPCODE**

Zie uitleg over IP-octets/poortbytes: [Link](http://penta2.ufrgs.br/trouble/ts_ip.htm).

| Byte size | Naam             |
| :-------: | :--------------- |
|     4     | "SAMP"           |
|     4     | IP Octets        |
|     1     | Port & 0xFF      |
|     1     | Port >> 8 & 0xFF |
|     1     | OPCODE           |

[C-voorbeeld](https://github.com/Louzindev/sampquery-c/blob/master/src/packet.c)

## OPCODE

Identifiers voor verzoektypes:

- **"i" (0x69):** Info (players, map, hostname, gamemode, language)
- **"r" (0x72):** Rules (instagib, gravity, weather, website URL, ...)
- **"c" (0x63):** Client list (namen en scores)
- **"d" (0x64):** Detail per player (ping, id, score, naam)
- **"x" (0x78):** RCON command (anders dan de rest)
- **"p" (0x70):** Ping/latency check met 4 pseudo-random bytes

## Response

Elke OPCODE geeft een response terug. De eerste 11 bytes zijn gelijk aan de request-header, daarna volgt de specifieke payload.

### Response-tabellen (`i`, `r`, `c`, `d`, `p`)

#### Type `i`

| Byte        | Key        | Breedte  | Omschrijving                |
| ----------- | ---------- | -------- | --------------------------- |
| 11          | Password   | 1        | 0 of 1 (password ingesteld) |
| 12-13       | Players    | 2        | Aantal online spelers       |
| 14-15       | MaxPlayers | 2        | Maximum aantal spelers      |
| 16-19       | (strlen)   | 4        | Lengte van hostname         |
| 20 + strlen | Hostname   | (strlen) | Hostname van de server      |
| 21-24       | (strlen)   | 4        | Lengte van gamemode         |
| 25 + strlen | Gamemode   | (strlen) | Gamemode                    |
| 26-29       | (strlen)   | 4        | Lengte van language         |
| 30 + strlen | Language   | (strlen) | Taal                        |

#### Type `r`

| Byte        | Key       | Breedte  | Omschrijving           |
| ----------- | --------- | -------- | ---------------------- |
| 11-12       | RuleCount | 2        | Aantal rules           |
| 13          | (strlen)  | 1        | Lengte van rule-naam   |
| 14 + strlen | Rulename  | (strlen) | Rule-naam              |
| 15          | (strlen)  | 1        | Lengte van rule-waarde |
| 16 + strlen | RuleValue | (strlen) | Rule-waarde            |

Herhaal vanaf byte 13 voor elke rule.

#### Type `c`

| Byte        | Key         | Breedte  | Omschrijving        |
| ----------- | ----------- | -------- | ------------------- |
| 11-12       | PlayerCount | 2        | Aantal spelers      |
| 13          | (strlen)    | 1        | Lengte van nickname |
| 14 + strlen | PlayerNick  | (strlen) | Spelernaam          |
| 15-18       | Score       | 4        | Score               |

Herhaal vanaf byte 13 per speler.

#### Type `d`

| Byte        | Key         | Breedte  | Omschrijving        |
| ----------- | ----------- | -------- | ------------------- |
| 11-12       | PlayerCount | 2        | Aantal spelers      |
| 13          | PlayerID    | 1        | Speler-ID (0-255)   |
| 14          | (strlen)    | 1        | Lengte van nickname |
| 15 + strlen | PlayerNick  | (strlen) | Spelernaam          |
| 16-19       | Score       | 4        | Score               |
| 20-23       | Ping        | 4        | Ping                |

Herhaal vanaf byte 13 per speler.

#### Type `p`

| Byte | Key      | Breedte | Omschrijving              |
| ---- | -------- | ------- | ------------------------- |
| 11   | number 1 | 1       | Eerste pseudo-random byte |
| 12   | number 2 | 1       | Tweede                    |
| 13   | number 3 | 1       | Derde                     |
| 14   | number 4 | 1       | Vierde                    |

## Codevoorbeeld in C

Er is een kleine C-lib beschikbaar om queries te doen: **[Repository](https://github.com/Louzindev/sampquery-c)**

## Codevoorbeeld in PHP

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

## Codevoorbeeld in C#

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

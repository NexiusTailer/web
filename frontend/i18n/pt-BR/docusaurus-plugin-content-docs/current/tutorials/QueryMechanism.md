---
title: "Mecanismo de Consulta SA:MP"
sidebar_label: "Mecanismo de Consulta SA:MP"
---

## Introdução

O Mecanismo de Consulta SA:MP nada mais é do que o mecanismo para transmissão de estatísticas e informações do servidor, como nome, ping, idioma, jogadores online, etc...

Neste artigo, vou documentar como esse mecanismo funciona, além de ensinar como usá-lo sem precisar do cliente original.

## Consultas

Consultas são pacotes UDP puros enviados ao endereço do servidor contendo dados serializados.

Você pode se perguntar: "Mas como o servidor interpreta pacotes de consulta de forma diferente daqueles do protocolo RakNet?", e a resposta é simples: na camada de socket de baixo nível do RakNet, pacotes que contêm 53 41 4D 50 ou traduzido para caracteres "SAMP" no início, são tratados de forma diferente. **[Ver Código](https://github.com/openmultiplayer/RakNet/blob/master/Source/SocketLayer.cpp#L371)**

## Dados Serializados

Os dados transmitidos no pacote são: **"SAMP"** + **octetos do IP** + **primeiro byte da porta\*** + **segundo byte da porta** + **OPCODE**

Se você tem dúvidas sobre por que extrair e o que são octetos de IP e bytes de porta, veja: [Link](http://penta2.ufrgs.br/trouble/ts_ip.htm).

| Tamanho em bytes |       Nome        |
| :--------------: | :---------------: |
|        4         |      "SAMP"       |
|        4         |   Octetos do IP   |
|        1         |   Porta & 0xFF    |
|        1         | Porta >> 8 & 0xFF |
|        1         |      OPCODE       |

[Exemplo em C](https://github.com/Louzindev/sampquery-c/blob/master/src/packet.c)

## OPCODE

OPCODE's são identificadores de pacotes, e cada um representa uma requisição diferente.

- **OPCODE "i" ou 0x69:** Significa informação. Obtém a quantidade de jogadores no servidor, o nome do mapa e todas as informações desse tipo. É realmente útil para descrever seu servidor sem alterar nada.

- **OPCODE "r" ou 0x72:** Significa regras. 'Regras' quando se trata do SA:MP inclui o instagib, a gravidade, clima, a URL do site, e assim por diante.

- **OPCODE "c" ou 0x63:** Significa lista de clientes, isso envia de volta ao servidor os nomes dos jogadores e então as pontuações dos jogadores. Imagine apenas como uma visão geral básica de todos os jogadores.

- **OPCODE "d" ou 0x64:** Significa informação detalhada do jogador. Com isso, você pode obter tudo, desde o ping do jogador, o ID do jogador (útil para scripts de admin), a pontuação novamente, e também o nome de usuário.

- **OPCODE "x" ou 0x78:** Este é um comando RCON, e é completamente diferente de todos os outros pacotes.

- **OPCODE "p" ou 0x70:** Quatro caracteres pseudo-aleatórios são enviados ao servidor, e os mesmos caracteres são retornados. Você pode usar o tempo entre enviar e receber para calcular o ping/latência do servidor.

## Resposta

Como dito acima, cada OPCODE retorna informações.

A resposta consiste nos mesmos primeiros 11 bytes enviados, o que chamamos de Cabeçalho, depois a resposta definitiva.

### Tabelas de Resposta para `i`, `r`, `c`, `d`, `p`

#### Tipo de Resposta `i`

| Byte        | Chave      | Largura em Bytes | Descrição                                         |
| ----------- | ---------- | ---------------- | ------------------------------------------------- |
| 11          | Password   | 1                | 0 ou 1, dependendo se a senha está definida       |
| 12-13       | Players    | 2                | Número atual de jogadores online                  |
| 14-15       | MaxPlayers | 2                | Número máximo de jogadores permitidos no servidor |
| 16-19       | (strlen)   | 4                | Comprimento do hostname do servidor               |
| 20 + strlen | Hostname   | (strlen)         | Hostname do servidor                              |
| 21-24       | (strlen)   | 4                | Comprimento do gamemode do servidor               |
| 25 + strlen | Gamemode   | (strlen)         | Gamemode do servidor                              |
| 26-29       | (strlen)   | 4                | Comprimento do idioma do servidor                 |
| 30 + strlen | Language   | (strlen)         | Idioma do servidor                                |

#### Tipo de Resposta `r`

| Byte        | Chave     | Largura em Bytes | Descrição                                 |
| ----------- | --------- | ---------------- | ----------------------------------------- |
| 11-12       | RuleCount | 2                | Número de regras fornecidas pelo servidor |
| 13          | (strlen)  | 1                | Comprimento do nome da regra              |
| 14 + strlen | Rulename  | (strlen)         | Nome da regra                             |
| 15          | (strlen)  | 1                | Comprimento do valor da regra             |
| 16 + strlen | RuleValue | (strlen)         | Valor da regra                            |

_(Repita a partir do Byte 13 para cada regra, quantas vezes for `RuleCount`)_

#### Tipo de Resposta `c`

| Byte        | Chave       | Largura em Bytes | Descrição                                    |
| ----------- | ----------- | ---------------- | -------------------------------------------- |
| 11-12       | PlayerCount | 2                | Número de jogadores fornecidos pelo servidor |
| 13          | (strlen)    | 1                | Comprimento do apelido do jogador            |
| 14 + strlen | PlayerNick  | (strlen)         | Apelido do jogador                           |
| 15-18       | Score       | 4                | Pontuação do jogador                         |

_(Repita a partir do Byte 13 para cada jogador, quantas vezes for `PlayerCount`)_

#### Tipo de Resposta `d`

| Byte        | Chave       | Largura em Bytes | Descrição                                    |
| ----------- | ----------- | ---------------- | -------------------------------------------- |
| 11-12       | PlayerCount | 2                | Número de jogadores fornecidos pelo servidor |
| 13          | PlayerID    | 1                | ID do jogador (valores 0-255)                |
| 14          | (strlen)    | 1                | Comprimento do apelido do jogador            |
| 15 + strlen | PlayerNick  | (strlen)         | Apelido do jogador                           |
| 16-19       | Score       | 4                | Pontuação do jogador                         |
| 20-23       | Ping        | 4                | Ping do jogador ao servidor                  |

_(Repita a partir do Byte 13 para cada jogador, quantas vezes for `PlayerCount`)_

#### Tipo de Resposta `p`

| Byte | Chave    | Largura em Bytes | Descrição                                                          |
| ---- | -------- | ---------------- | ------------------------------------------------------------------ |
| 11   | número 1 | 1                | Primeiro número da sequência pseudo-aleatória enviada pelo cliente |
| 12   | número 2 | 1                | Segundo número da sequência pseudo-aleatória                       |
| 13   | número 3 | 1                | Terceiro número da sequência pseudo-aleatória                      |
| 14   | número 4 | 1                | Quarto número da sequência pseudo-aleatória                        |

## Código de Exemplo em C

Há algum tempo fiz uma pequena lib em C, que permite realizar consultas, você pode usá-la como exemplo. **[Ver Repositório](https://github.com/Louzindev/sampquery-c)**

## Código de Exemplo em PHP

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

## Código de Exemplo em C#

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

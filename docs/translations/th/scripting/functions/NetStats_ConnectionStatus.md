---
title: NetStats_ConnectionStatus
description: Gets the player's current connection status.
tags: []
---

:::warning

This function was added in SA-MP 0.3z and will not work in earlier versions!

:::

## คำอธิบาย

Gets the player's current connection status.

| Name     | Description                                           |
| -------- | ----------------------------------------------------- |
| playerid | The ID of the player to get the connection status of. |

## ส่งคืน

The player's [connection status](../resources/connectionstatus), as an integer value.

## ตัวอย่าง

```c
public OnPlayerCommandText(playerid,cmdtext[])
{
    if (!strcmp(cmdtext, "/connectionstatus"))
    {
        new szString[144];
        format(szString, sizeof(szString), "Your current connection status: %i.", NetStats_ConnectionStatus(playerid));
        SendClientMessage(playerid, -1, szString);
    }
    return 1;
}
```

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [GetPlayerNetworkStats](GetPlayerNetworkStats): Gets a player networkstats and saves it into a string.
- [GetNetworkStats](GetNetworkStats): Gets the servers networkstats and saves it into a string.
- [NetStats_GetConnectedTime](NetStats_GetConnectedTime): Get the time that a player has been connected for.
- [NetStats_MessagesReceived](NetStats_MessagesReceived): Get the number of network messages the server has received from the player.
- [NetStats_BytesReceived](NetStats_BytesReceived): Get the amount of information (in bytes) that the server has received from the player.
- [NetStats_MessagesSent](NetStats_MessagesSent): Get the number of network messages the server has sent to the player.
- [NetStats_BytesSent](NetStats_BytesSent): Get the amount of information (in bytes) that the server has sent to the player.
- [NetStats_MessagesRecvPerSecond](NetStats_MessagesRecvPerSecond): Get the number of network messages the server has received from the player in the last second.
- [NetStats_PacketLossPercent](NetStats_PacketLossPercent): Get a player's packet loss percent.
- [NetStats_GetIpPort](NetStats_GetIpPort): Get a player's IP and port.
- [IsPlayerConnected](IsPlayerConnected): Check if a player is connected to the server.
- [OnPlayerConnect](../callbacks/OnPlayerConnect): Called when a player connects to the server.
- [OnPlayerDisconnect](../callbacks/OnPlayerDisconnect): Called when a player leaves the server.

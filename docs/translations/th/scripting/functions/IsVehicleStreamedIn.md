---
title: IsVehicleStreamedIn
description: Checks if a vehicle is streamed in for a player.
tags: ["vehicle"]
---

:::warning

This function was added in SA-MP 0.3a and will not work in earlier versions!

:::

## คำอธิบาย

Checks if a vehicle is streamed in for a player. Only nearby vehicles are streamed in (visible) for a player.

| Name        | Description                     |
| ----------- | ------------------------------- |
| vehicleid   | The ID of the vehicle to check. |
| forplayerid | The ID of the player to check.  |

## ส่งคืน

0: Vehicle is not streamed in for the player, or the function failed to execute (player and/or vehicle do not exist).

1: Vehicle is streamed in for the player.

## ตัวอย่าง

```c
new streamedVehicleCount;

for(new v = 1; v <= MAX_VEHICLES; v++)
{
    if (IsVehicleStreamedIn(v, playerid))
    {
        streamedVehicleCount++;
    }
}

new szString[144];
format(szString, sizeof(szString), "You currently have %i vehicles streamed in to your game.", streamedVehicleCount);
SendClientMessage(playerid, -1, szString);
```

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [IsPlayerStreamedIn](IsPlayerStreamedIn): Checks if a player is streamed in for another player.
- [OnVehicleStreamIn](../callbacks/OnVehicleStreamIn): Called when a vehicle streams in for a player.
- [OnVehicleStreamOut](../callbacks/OnVehicleStreamOut): Called when a vehicle streams out for a player.
- [OnPlayerStreamIn](../callbacks/OnPlayerStreamIn): Called when a player streams in for another player.
- [OnPlayerStreamOut](../callbacks/OnPlayerStreamOut): Called when a player streams out for another player.

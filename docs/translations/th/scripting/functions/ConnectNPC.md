---
title: ConnectNPC
description: Connect an NPC to the server.
tags: ["npc"]
---

:::warning

This function was added in SA-MP 0.3a and will not work in earlier versions!

:::

## คำอธิบาย

Connect an NPC to the server.

| Name     | Description                                                                              |
| -------- | ---------------------------------------------------------------------------------------- |
| name[]   | The name the NPC should connect as. Must follow the same rules as normal player names.   |
| script[] | The NPC script name that is located in the npcmodes folder (without the .amx extension). |

## ส่งคืน

This function always return 1.

## ตัวอย่าง

```c
public OnGameModeInit()
{
    ConnectNPC("[BOT]Pilot", "pilot");
    return 1;
}
```

## บันทึก

:::tip

NPCs do not have nametags. These can be scripted with Attach3DTextLabelToPlayer.

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [IsPlayerNPC](IsPlayerNPC): Check if a player is an NPC or an actual player.
- [OnPlayerConnect](../callbacks/OnPlayerConnect): Called when a player connects to the server.

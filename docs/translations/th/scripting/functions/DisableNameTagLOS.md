---
title: DisableNameTagLOS
description: Disables the nametag Line-Of-Sight checking so that players can see nametags through objects.
tags: []
---

:::warning

This function was added in SA-MP 0.3a and will not work in earlier versions!

:::

## คำอธิบาย

Disables the nametag Line-Of-Sight checking so that players can see nametags through objects.

| Name | Description |
| ---- | ----------- |


## ตัวอย่าง

```c
public OnGameModeInit()
{
    DisableNameTagLOS();
    return 1;
}
```

## บันทึก

:::warning

This can not be reversed until the server restarts.

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [ShowNameTags](ShowNameTags): Set nametags on or off.
- [ShowPlayerNameTagForPlayer](ShowPlayerNameTagForPlayer): Show or hide a nametag for a certain player.

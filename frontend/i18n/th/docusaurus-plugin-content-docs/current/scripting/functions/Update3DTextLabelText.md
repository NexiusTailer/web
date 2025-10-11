---
title: Update3DTextLabelText
sidebar_label: Update3DTextLabelText
description: Updates a 3D Text Label text and color.
tags: ["3dtextlabel"]
---

## คำอธิบาย

Updates a 3D Text Label text and color.

| Name      | Description                                                   |
| --------- | ------------------------------------------------------------- |
| Text3D:textid | The 3D Text Label you want to update.                         |
| color     | The color the 3D Text Label should have from now on.          |
| text[]    | The new text which the 3D Text Label should have from now on. |

## ส่งคืน

This function does not return any specific values.

## ตัวอย่าง

```c
public OnGameModeInit()
{
    new Text3D:mylabel;
    mylabel = Create3DTextLabel("I'm at the coordinates:\n30.0,40.0,50.0",0x008080FF,30.0,40.0,50.0,40.0,0);
    Update3DTextLabelText(mylabel, 0xFFFFFFFF, "New text.");
    return 1;
}
```

## บันทึก

:::warning

If text[] is empty, the server/clients next to the text might crash!

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [Create3DTextLabel](Create3DTextLabel): Create a 3D text label.
- [Delete3DTextLabel](Delete3DTextLabel): Delete a 3D text label.
- [Attach3DTextLabelToPlayer](Attach3DTextLabelToPlayer): Attach a 3D text label to a player.
- [Attach3DTextLabelToVehicle](Attach3DTextLabelToVehicle): Attach a 3D text label to a vehicle.
- [CreatePlayer3DTextLabel](CreatePlayer3DTextLabel): Create A 3D text label for one player.
- [DeletePlayer3DTextLabel](DeletePlayer3DTextLabel): Delete a player's 3D text label.
- [UpdatePlayer3DTextLabelText](UpdatePlayer3DTextLabelText): Change the text of a player's 3D text label.

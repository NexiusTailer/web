---
title: UpdatePlayer3DTextLabelText
description: Updates a player 3D Text Label's text and color.
tags: ["player", "3dtextlabel"]
---

:::warning

This function was added in SA-MP 0.3a and will not work in earlier versions!

:::

## คำอธิบาย

Updates a player 3D Text Label's text and color

| Name            | Description                                                   |
| --------------- | ------------------------------------------------------------- |
| playerid        | The ID of the player for which the 3D Text Label was created. |
| PlayerText3D:id | The 3D Text Label you want to update.                         |
| color           | The color the 3D Text Label should have from now on.          |
| text[]          | The new text which the 3D Text Label should have from now on. |

## ส่งคืน

This function does not return any specific values.

## บันทึก

:::warning

If text[] is empty, the server/clients next to the text might crash!

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [Create3DTextLabel](Create3DTextLabel): Create a 3D text label.
- [Delete3DTextLabel](Delete3DTextLabel): Delete a 3D text label.
- [Attach3DTextLabelToPlayer](Attach3DTextLabelToPlayer): Attach a 3D text label to a player.
- [Attach3DTextLabelToVehicle](Attach3DTextLabelToVehicle): Attach a 3D text label to a vehicle.
- [Update3DTextLabelText](Update3DTextLabelText): Change the text of a 3D text label.
- [CreatePlayer3DTextLabel](CreatePlayer3DTextLabel): Create A 3D text label for one player.
- [DeletePlayer3DTextLabel](DeletePlayer3DTextLabel): Delete a player's 3D text label.

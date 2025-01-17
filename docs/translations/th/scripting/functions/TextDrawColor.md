---
title: TextDrawColor
description: Sets the text color of a textdraw.
tags: ["textdraw"]
---

## คำอธิบาย

Sets the text color of a textdraw

| Name  | Description                                     |
| ----- | ----------------------------------------------- |
| text  | The ID of the textdraw to change the color of.. |
| color | The color to set the textdraw to.               |

## ส่งคืน

This function does not return any specific values.

## ตัวอย่าง

```c
new Text:Example;

public OnGameModeInit()
{
    Example = TextDrawCreate(123.0, 123.0, "Example");
    TextDrawColor(Example, 0x000000FF);
    return 1;
}
```

## บันทึก

:::tip

If the TextDraw is already shown, it must be re-shown (TextDrawShowForAll/TextDrawShowForPlayer) for the changes of this function to take effect.

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [TextDrawCreate](TextDrawCreate): Create a textdraw.
- [TextDrawDestroy](TextDrawDestroy): Destroy a textdraw.
- [TextDrawBoxColor](TextDrawBoxColor): Set the color of the box in a textdraw.
- [TextDrawBackgroundColor](TextDrawBackgroundColor): Set the background color of a textdraw.
- [TextDrawAlignment](TextDrawAlignment): Set the alignment of a textdraw.
- [TextDrawFont](TextDrawFont): Set the font of a textdraw.
- [TextDrawLetterSize](TextDrawLetterSize): Set the letter size of the text in a textdraw.
- [TextDrawTextSize](TextDrawTextSize): Set the size of a textdraw box.
- [TextDrawSetOutline](TextDrawSetOutline): Choose whether the text has an outline.
- [TextDrawSetShadow](TextDrawSetShadow): Toggle shadows on a textdraw.
- [TextDrawSetProportional](TextDrawSetProportional): Scale the text spacing in a textdraw to a proportional ratio.
- [TextDrawUseBox](TextDrawUseBox): Toggle if the textdraw has a box or not.
- [TextDrawSetString](TextDrawSetString): Set the text in an existing textdraw.
- [TextDrawShowForPlayer](TextDrawShowForPlayer): Show a textdraw for a certain player.
- [TextDrawHideForPlayer](TextDrawHideForPlayer): Hide a textdraw for a certain player.
- [TextDrawShowForAll](TextDrawShowForAll): Show a textdraw for all players.
- [TextDrawHideForAll](TextDrawHideForAll): Hide a textdraw for all players.

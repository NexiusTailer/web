---
title: OnPlayerExitedMenu
sidebar_label: OnPlayerExitedMenu
description: Pozvano kada igrač napusti meni.
tags: ["player", "menu"]
---

## Deskripcija

Pozvano kada igrač napusti meni.

| Ime      | Deskripcija                     |
| -------- | ------------------------------- |
| playerid | ID igrača koji je napustio meni |

## Returns

Uvijek je pozvana prva u gamemode-u.

## Primjeri

```c
public OnPlayerExitedMenu(playerid)
{
    TogglePlayerControllable(playerid,1); // odledi igrača kada napusti meni
    return 1;
}
```

## Srodne Funkcije

- [CreateMenu](../functions/CreateMenu): Kreiraj meni.
- [DestroyMenu](../functions/DestroyMenu): Uništi meni.

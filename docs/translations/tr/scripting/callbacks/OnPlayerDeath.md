---
title: OnPlayerDeath
description: This callback is called when a player dies, either by suicide or by being killed by another player.
tags: ["player"]
---

## Açıklama

Bu callback bir oyuncu öldüğünde, intihar ettiğinde yada başka bir oyuncu tarafından öldürüldüğünde çağırılır.

| İsim     | Açıklama                                                                                    |
| -------- | ------------------------------------------------------------------------------------------- |
| playerid | Ölen oyuncunun ID'si.                                                                       |
| killerid | Öldüren oyuncunun ID'si, eğer öldüren bir oyuncu yoksa INVALID_PLAYER_ID değeri döndürülür. |
| reason   | Oyuncunun ölüm sebebinin ID'si.                                                             |

## Çalışınca Vereceği Sonuçlar

0 - Diğer filterscriptlerin bu callbacki çağırmasını engeller.

1 - Bu callbackin sonraki filterscriptlere geçeceğini belirtir.

Filterscriptlerde her zaman ilk olarak çağırılır.

## Örnekler

```c
new PlayerDeaths[MAX_PLAYERS];
new PlayerKills[MAX_PLAYERS];

public OnPlayerDeath(playerid, killerid, reason)
{
    SendDeathMessage(killerid, playerid, reason); // killfeed(Sağdaki öldüren, silah ve ölen oyuncu) listesine ölen, öldüren ve ölüm nedenini gösterir.

    // killerid ile herhangi bir işlem yapmadan önce, killerid'nin geçerli olup olmadığını kontrol eder.
    if (killerid != INVALID_PLAYER_ID)
    {
        PlayerKills[killerid] ++;
    }

    // Kontrolün dışında, playerid için gerekli işlemleri yapar. (playerid her zaman geçerlidir.)
    PlayerDeaths[playerid] ++;
    return 1;
}
```

## Notlar

:::tip

Herhangi bir ateş kaynağından gelen hasarlarda(örnek: molotov, 18) weaponid değeri 37 (flame thrower) olarak döndürülür. Herhangi bir patlama kaynağından gelen hasarlarda (örnek: roketatar, el bombası) weapondid değeri 51 olarak geri döndürülür. SendDeathMessage fonksiyonunu kullanmadan önce killerid'nin geçerli olup olmadığını kontrol etmenize gerek yok. INVALID_PLAYER_ID geçerli bir killerid ID parametresidir. Yalnızca playerid tarafından çağırılabilir. (anti fake death için bilmeniz iyi olur.)

:::

:::warning

Bir dizi içerisinde kullanmadan önce killerid'nin geçerli olup olmadığını kontrol ETMELİSİNİZ, yoksa OnPlayerDeath scriptinin çökmesine neden olur (Bütün script değil). Bunun nedeni INVALID_PLAYER_ID'nin 65535 olarak belirlenmiş olmasıdır ve bu değer MAX_PLAYERS'dan yüksek olacağı için scripti çökertir.

:::

## Bağlantılı Fonksiyonlar

- [SendDeathMessage](../functions/SendDeathMessage): killfeed'e (Sağdaki öldüren, silah, ölüm listesi.) bir yeni ölüm verisi girer.
- [SetPlayerHealth](../functions/SetPlayerHealth): Oyuncu canını belirler.

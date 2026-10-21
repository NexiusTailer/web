---
title: OnRecordingPlaybackEnd
sidebar_label: OnRecordingPlaybackEnd
description: Bu geri çağırma, StartRecordingPlayback ile oynatılan bir kaydın sonuna gelindiğinde çağrılır.
tags: []
---

## Açıklama

Bu geri çağırma, StartRecordingPlayback ile oynatılan bir kaydın sonuna gelindiğinde çağrılır.

## Örnek

```c
public OnRecordingPlaybackEnd()
{
    StartRecordingPlayback(PLAYER_RECORDING_TYPE_DRIVER, "all_around_lv_bus"); // Bu, kaydın bitiminde tekrar oynatılmasını sağlar.
}
```

## Bağlantılı Fonksiyonlar

Aşağıdaki fonksiyonlar bu geri çağırma ile bir şekilde ilişkilidir ve faydalı olabilirler.

- [StartRecordingPlayback](../functions/StartRecordingPlayback): Kayıtlı bir _.rec_ dosyasını oynatmaya başlar.
- [StopRecordingPlayback](../functions/StopRecordingPlayback): Bir _.rec_ dosyasının oynatılmasını durdurur.

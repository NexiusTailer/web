---
title: OnActorStreamIn
sidebar_label: OnActorStreamIn
description: يتم استدعاء هذا الاستدعاء أو الكال باك عند خروج السيارة بواسطة كلاينت اللاعب
tags: []
---

<VersionWarn name='callback' version='SA-MP 0.3.7' />

<div dir="rtl" style={{ textAlign: "right" }}>


## الوصف

يتم استدعاء هذا الاستدعاء أو الكال باك عند خروج السيارة بواسطة كلاينت اللاعب

| الوصف        |        الإسم                                                |
| ----------- | ------------------------------------------------------------ |
| vehicleid   | ايدي السيارة الذي تم خرجت للاعب.                            |
| forplayerid | هوية اللاعب الذي قام خرجت السيارة                          |

## Returns

دائمًا يتم استدعاؤه أولاً في الفلترسكربتات.

## أمثلة

</div>

```c
public OnVehicleStreamOut(vehicleid, forplayerid)
{
    new string[48];
    format(string, sizeof(string), "Your client is no longer streaming vehicle %d", vehicleid);
    SendClientMessage(forplayerid, 0xFFFFFFFF, string);
    return 1;
}
```

<div dir="rtl" style={{ textAlign: "right" }}>

## Notes

<TipNPCCallbacks/>

## الاستدعاءات او كالباكات ذات الصلة

قد تكون الاستدعاءات التالية مفيدة، حيث أنها ذات صلة بهذا الاستدعاء بطريقة أو بأخرى.

- [OnVehicleStreamIn](OnVehicleStreamIn): يتم استدعاؤه عندما يتم تدفق السبارةإلى كلاينت اللاعب.
- [OnPlayerStreamIn](OnPlayerStreamIn): يتم استدعاؤه عندما يتم تدفق لاعب آخر إلى كلاينت اللاعب.
- [OnPlayerStreamOut](OnPlayerStreamOut): يتم استدعاؤه عندما يتم تدفق لاعب آخر خارج كلاينت اللاعب.

</div>


---
title: SetVehicleAngularVelocity
description: Define a velocidade angular X, Y e Z de um veículo.
tags: ["vehicle"]
---

Esta função foi implementada no SA-MP 0.3b e não funcionará em versões anteriores.

:::info

Esta função está no _world_ space e não no _local_ space. Se você quiser fazer ajustes locais à velocidade angular, deve aplicar uma matriz baseada em [vehicle rotation quat](GetVehicleRotationQuat).

:::

## Descrição

Define a velocidade angular X, Y e Z de um veículo.

| Nome      | Descrição                                        |
| --------- | ------------------------------------------------ |
| vehicleid | O ID do veículo a definir a velocidade.          |
| Float:X   | A quantidade de velocidade na direção angular X. |
| Float:Y   | A quantidade de velocidade na direção angular Y. |
| Float:Z   | A quantidade de velocidade na direção angular Z. |

## Retorno

1: A função foi executada com sucesso.

0: A função falhou ao ser executada. O veículo não existe.

## Exemplos

```c
public OnPlayerCommandText(playerid, cmdtext[])
{
    if (!strcmp("/spin", cmdtext))
    {
        if (IsPlayerInAnyVehicle(playerid))
        {
            SetVehicleAngularVelocity(GetPlayerVehicleID(playerid), 0.0, 0.0, 2.0);
        }
        return 1;
    }
}
```

## Notas

:::warning

Esta função não tem efeito em veículos vazios e não afeta trens.

:::

## Funções Relacionadas

- [SetVehicleVelocity](SetVehicleVelocity): Define a velocidade de um veículo.
- [GetVehicleVelocity](GetVehicleVelocity): Obtém a velocidade de um veículo.

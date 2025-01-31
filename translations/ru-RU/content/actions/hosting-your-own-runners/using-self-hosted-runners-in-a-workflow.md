---
title: Использование локальных средств выполнения в рабочем процессе
intro: 'Чтобы использовать локальные средства выполнения тестов в рабочем процессе, можно использовать метки, чтобы указать тип средства выполнения тестов для задания.'
redirect_from:
  - /github/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow
  - /actions/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
shortTitle: Use runners in a workflow
ms.openlocfilehash: 5c0ff57f5b3eda79e3fcf8b09706ed19f981b8ae
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2022
ms.locfileid: '147573420'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

Сведения о создании пользовательских меток и меток по умолчанию см. в разделе [Использование меток с локальными средствами выполнения](/actions/hosting-your-own-runners/using-labels-with-self-hosted-runners).

## Использование локальных средств выполнения в рабочем процессе

Метки позволяют отправлять задания рабочих процессов в определенные типы локальных средств выполнения в соответствии с их общими характеристиками. Например, если заданию требуется определенный аппаратный компонент или программный пакет, можно назначить средству выполнения пользовательскую метку, а затем настроить выполнение задания только в средствах выполнения с этой меткой.

{% data reusables.actions.self-hosted-runner-labels-runs-on %}

Дополнительные сведения см. в статье [Синтаксис рабочего процесса для {% data variables.product.prodname_actions %}](/github/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idruns-on).

## Использование меток по умолчанию для маршрутизации заданий

Локальное средство выполнения автоматически получает определенные метки при добавлении в {% data variables.product.prodname_actions %}. Они служат для указания операционной системы и аппаратной платформы:

* `self-hosted` — метка по умолчанию, применяемая ко всем локальным средствам выполнения;
* `linux`, `windows` или `macOS` — применяется в зависимости от операционной системы;
* `x64`, `ARM` или `ARM64` — применяются в зависимости от архитектуры оборудования.

Код YAML рабочего процесса можно использовать для отправки заданий в средства выполнения с сочетанием этих меток. В этом примере выполнение задания допускается в локальном средстве выполнения со всеми тремя метками:

```yaml
runs-on: [self-hosted, linux, ARM64]
```

- `self-hosted` — задание выполняется в локальном средстве выполнения.
- `linux` — использовать только средство выполнения на основе Linux.
- `ARM64` — использовать средство выполнения только на основе оборудования ARM64.

Метки по умолчанию являются фиксированными: их нельзя изменять или удалять. Пользовательские метки можно применять, если требуется больший контроль над маршрутизацией заданий.

## Использование пользовательских меток для маршрутизации заданий

Вы можете создавать пользовательские метки и назначать их локальным средствам выполнения в любое время. Пользовательские метки позволяют отправлять задания в определенные типы локальных средств выполнения в зависимости от того, как они помечены. 

Например, если заданию требуется конкретный тип графического оборудования, можно создать пользовательскую метку `gpu` и назначить ее средствам выполнения с этим оборудованием. Выполнение задания допускается в локальном средстве выполнения, которому назначены все соответствующие метки. 

В этом примере показано задание с сочетанием меток по умолчанию и пользовательских меток:

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

- `self-hosted` — задание выполняется в локальном средстве выполнения.
- `linux` — использовать только средство выполнения на основе Linux.
- `x64` — использовать средство выполнения только на основе архитектуры x64.
- `gpu` — эта пользовательская метка была вручную назначена локальным средствам выполнения с установленным GPU. 

Метки работают в совокупности, то есть для обработки задания локальное средство выполнения должно иметь все четыре метки.

## Приоритет маршрутизации для локальных средств выполнения

При маршрутизации задания в локальное средство выполнения {% data variables.product.prodname_dotcom %} ищет средство выполнения, соответствующее меткам `runs-on` задания:

{% ifversion fpt or ghes > 3.3 or ghae or ghec %}
- Если {% data variables.product.prodname_dotcom %} находит в сети бездействующее средство выполнения, соответствующее меткам `runs-on` задания, задание назначается этому средству выполнения и отправляется в него.
  - Если средство выполнения не принимает назначенное задание в течение 60 секунд, задание снова ставится в очередь для поиска нового средства выполнения.
- Если {% data variables.product.prodname_dotcom %} не находит в сети бездействующее средство выполнения, соответствующее меткам `runs-on` задания, задание остается в очереди, пока средство выполнения не появится.
- После нахождения в очереди в течение 24 часов задание завершается сбоем.
{% elsif ghes = 3.3 %}
- {% data variables.product.prodname_dotcom %} сначала ищет средство выполнения на уровне репозитория, затем на уровне отдела, а затем на уровне организации.
- Если {% data variables.product.prodname_dotcom %} находит в сети бездействующее средство выполнения на определенном уровне, соответствующее меткам `runs-on` задания, задание назначается этому средству выполнения и отправляется в него.
  - Если средство выполнения не принимает назначенное задание в течение 60 секунд, задание помещается в очередь на всех уровнях и ожидает появления в сети подходящего средства выполнения любого уровня.
- Если {% data variables.product.prodname_dotcom %} не находит в сети бездействующее средство выполнения ни на одном уровне, задание помещается в очередь на всех уровнях и ожидает появления в сети подходящего средства выполнения любого уровня.
- После нахождения в очереди в течение 24 часов задание завершается сбоем.
{% else %}
1. {% data variables.product.prodname_dotcom %} сначала ищет средство выполнения на уровне репозитория, затем на уровне отдела, а затем на уровне организации.
2. Затем задание отправляется в первое подходящее средство выполнения, которое находится в сети и бездействует.
   - Если все подходящие средства выполнения в сети заняты, задание помещается в очередь на уровне с наибольшим числом подходящих средств выполнения в сети.
   - Если все подходящие средства выполнения не в сети, задание помещается в очередь на уровне с наибольшим числом подходящих средств выполнения не в сети.
   - Если подходящих средств выполнения нет ни на одном уровне, задание завершается сбоем.
   - После нахождения в очереди в течение 24 часов задание завершается сбоем.
{% endif %}

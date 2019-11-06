<!-- ############################################################ -->
## Tarantool =

.center[
![:scale 810px](images/1-database+appserver.svg)
]

.pull-left[.center[
База данных
<br/><br/>
(Транзакции, WAL)
]]

.pull-right[.center[
Сервер приложений (Lua)
<br/><br/>
(Бизнес логика, HTTP)
]]

???

* Вопрос залу - Кто работал? Кто слышал название?
* Тарантул - это ...
* Применяется он в самых разных сценариях:
  как бд кеши, очереди, сессии; и как аппсервер

<!-- ############################################################ -->
---

## Ядро

* 20 разработчиков C
* Развитие продукта

## Команда решений
* 35 разработчиков Lua
* Коммерческие проекты

???

* Коллекив 70 человек
* Преимущественно программисты
* Условно две половинки
* Команда ядра развивает опенсорс платформу
* Команда решений делает коммерческие проекты

--

### Цели

* Меньше багов, больше продуктивность

???

* Делимся опытом и проблемами
* Чтобы не было багов
* Главное - Чтобы разработка велась быстро
* Об этом "быстро" мы и поговорим

<!-- ############################################################ -->
---
## Инструменты разработчика

.override[.center[![:scale 900px](images/stack-1.1.svg)]]
???

* Давайте посмотрим на набор инструментов,
  которые есть у нас в распоряжении
* Самый главный инструмент - тарантул.
  Не только In-memory, не тоьлко no-SQL

--
.override[.center[![:scale 900px](images/stack-1.2.svg)]]
???
* На его основе мы делаем проекты

--
.override[.center[![:scale 900px](images/stack-2.1.svg)]]
???
* Но нужно уметь масштабироваться

--
.override[.center[![:scale 900px](images/stack-2.2.svg)]]
???
* Два инстанса, а между ними пропасть

--
.override[.center[![:scale 900px](images/stack-3.svg)]]
???
* Для этого есть вшард, делает базу распределенной.
* Не первая попытка сделать шардинг. Было ещё две.
* Третья получилась удачной, но всё равно не всемогущей.

<!-- ############################################################ -->
---
## Конфигурация vshard

- Vshard управляется программно:

```lua
sharding_cfg = {
    ['cbf06940-0790-498b-948d-042b62cf3d29'] = {
        replicas = { ... },
    },
    ['ac522f65-aa94-4134-9f64-51ee384f1a54'] = {
        replicas = { ... },
    },
}
```
```lua
vshard.router.cfg(...)
vshard.storage.cfg(...)
```

???
- Во-первых,  дело в удобстве
- Вшард управляется программно

<!-- ############################################################ -->
---
## Vshard - горизонтальное масштабирование в Tarantool

- Vshard группирует данные по виртуальным "ведёркам"
- Ведёрок много, они распределены по серверам

.override[![](images/vshard-p1.png)]
???
- Вшард - опенсорс модуль, который делает БД распределённой
- Вшард группирует записи ... (отсюда название)
- Бакетов много, вшард организует маршрутизацию
- И помогает в балансировке
- Вот здесь на схеме 2 стораджа

<!-- ############################# -->
--
.override[![](images/vshard-p2.png)]
???
- Стало тесно

<!-- ############################# -->
--
.override[![](images/vshard-p3.png)]
???
- Сначала нужно обновить конфиг

<!-- ############################# -->
--
.override[![](images/vshard-p4.png)]
???
- Стартануть новый инстанс. Но старые пока не знают

<!-- ############################# -->
--
.override[![](images/vshard-p5.png)]
???
- И применить этот конфиг к старым инстансам

<!-- ############################# -->
--
.override[![](images/vshard-p6.png)]
???
- Теперь вшард перетащит туда часть данных

<!-- ############################################################ -->
---
## Оркестрация vshard

.override[.center[![:scale 900px](images/stack-4.1.svg)]]
???

- Никто не хочет заниматься этим вручную
- Мы тоже пробовали разные варианты

--
.override[.center[![:scale 900px](images/stack-4.2.svg)]]
???
- Какие сложности:
- - Применять конфигурации на лету
- - Следить за разнородными сервисами
- - Не допустить потери данных

- Мы решали мелкие проблемы
- Cистема становилась неудобной
-
- Благо у нас есть сервер приложений

<!-- ############################################################ -->
---
## Организация бизнес логики

.override[.ilustrate[![](images/roles-p0.svg)]]
???
- Разработка приложения не начинается с деплоя

--
.override[.ilustrate[![](images/roles-p1.svg)]]
--
.override[.ilustrate[![](images/roles-p2.svg)]]
--
.override[.ilustrate[![](images/roles-p2.3.svg)]]
--
.override[.ilustrate[![](images/roles-p2.4.svg)]]

<!-- ############################################################ -->
---
## Как должно выглядеть управление кластером?

- Мы запускаем процессы
- Конфигурацией vshard управляет Tarantool

.override[.ilustrate[![](images/roles-p3.png)]]
???
- пользователь жмёт кнопку одну кнопку в веб-интерфейсе
- Мы не следим за запуском процессов
- Управлять кластером можно с любого узла
  (нет никаких причин делать иначе)
- Нам нужен мониторинг
  (в каком состоянии находится каждый узел)

--
.override[.ilustrate[![](images/roles-p4.2.png)]]

<!-- ############################################################ -->
---
## Конфигурация кластера

- Одинаковая **везде**
- Обновляется двухфазным коммитом
- Самое главное - **топология**:

```yaml
topology:
* servers:                # two servers
    s1:
      replicaset_uuid: A
      uri: localhost:3301
    s2:
      replicaset_uuid: A
      uri: localhost:3302
* replicasets:            # one replicaset
    A:
      roles: ...
```

???

- В отличие от вшарда, у кластера один общий конфиг
- Чтобы он не разъехался, будем применять его двухфазно
- Задача выглядит не сложной, всего лишь 2pc.
- Все инструменты у нас есть,



<!-- ############################################################ -->
---
## Курица или яйцо?

.center[
![:scale 550px](images/egg.jpg)
]
.margin-top-0[
	.pull-left[.pull-right[.center[
	База данных
	<br/>
	`net.box call`
	]]]
	.pull-right[.pull-left[.center[
	Новый процесс
	<br/>
	`box.cfg listen`
	]]]
]

???

- На самом деле нет
- Бинарный протокол Тарантула - это часть базы
- Как сконфигурировать сервер, который ничего не сконфигурировал
- Стандартный способ решить эту проблему - разоравать порочный круг

<!-- ############################################################ -->
---
## Внутренний мониторинг

- Протокол SWIM - распространение **слухов**

--
.override[.ilustrate[![:scale 100%](images/mm-p0.1.svg)]]
--
.override[.ilustrate[![:scale 100%](images/mm-p0.2.svg)]]
--
.override[.ilustrate[![:scale 100%](images/mm-p1.svg)]]
--
.override[.ilustrate[![:scale 100%](images/mm-p2.0.svg)]]

---
## Membership implementation

- SWIM protocol - one of the **gossips** protocols family
- Dissemination speed: O(logN)
- Network load: O(N)

.override[.ilustrate[![:scale 100%](images/mm-p2.2.svg)]]
--
.override[.ilustrate[![:scale 100%](images/mm-p3.svg)]]

???

- Поможет нам в этом мониторинг
- Как именно - станет ясно чуть позже, а пока про сам протокол
- Свим протогол
- Оч классный, меньше всего проблем с ним было

<!-- ############################################################ -->
---
.pull-left-70[
## Bootsrapping new instance

1. New process starts
1. New process joins membership
1. Cluster checks new process is alive
1. Cluster applies configuration
1. New process polls it from membership
1. New process bootstraps
]

.pull-right-30[.center[.margin-top-0[
![:scale 400px](images/seq-join-one.png)
]]]

<!-- ############################################################ -->
---
## Benefits so far

1. Orchestration works

.override[.ilustrate[![](images/topology-p1.png)]]
--
.override[.ilustrate[![](images/topology-p3.png)]]
--
.override[.ilustrate[![](images/topology-p4.png)]]

<!-- ############################################################ -->
---
## Benefits so far

1. Orchestration works
1. Monitoring works

.override[.ilustrate[![](images/topology-p5.png)]]
--
.override[.ilustrate[![](images/topology-p6.png)]]

<!-- ############################################################ -->
---
## Role management

- `function init()`
???
Когда дёргается инит?<br/>
Что можно/нужно делать на ините?
--
- `function validate_config()`
- `function apply_config()`
???
Роли могут пользоваться распределённым конфигом.
Что полезного там можно хранить?
--
- `function stop()`
???
Зачем нужен стоп


<!-- ############################################################ -->
---
.pull-left-70[
## Bootsrapping new instance

1. New process starts
1. New process joins membership
1. Cluster checks new process is alive
1. Cluster applies configuration
1. New process polls it from membership
1. New process bootstraps
]

.pull-right-30[.center[.margin-top-0[
![:scale 400px](images/seq-join-one.png)
]]]

<!-- ############################################################ -->
---
.pull-left-70[
## Bootsrapping new instance

1. New process starts
1. New process joins membership
1. Cluster checks new process is alive
1. Cluster applies configuration
1. New process polls it from membership
1. New process bootstraps
<br/>
<br/>
1. Repeat N times
]

.pull-right-30[.center[.margin-top-0[
![:scale 400px](images/seq-join-one-2.png)<br/>
]]]
???

<!-- ############################################################ -->
---
.pull-left-70[
## Bootsrapping new instance

1. New process starts
1. New process joins membership
1. Cluster checks new process is alive
1. Cluster applies configuration
1. New process polls it from membership
1. New process bootstraps
<br/>
<br/>
1. Repeat N times
]

<br/><br/>
.pull-right-30[.center[.margin-top-0[
![:scale 320px](images/plan-aaa.jpg)<br/>
N = 100
]]]

???

- Но вернёмся к нашей проблеме

<!-- ############################################################ -->
---
## Refactoring the bootstrap process

- Assembling large clusters with 100+ instances is slow
- N two-phase commits are slow
- N config pollings is slow

--

### Solution

- Bootstrap all instances with a single 2pc
- Re-implement binary protocol and reuse port


<!-- ############################################################ -->
---
.pull-left-70[
## Bootsrapping new instance
![:scale 320px](images/seq-join-two.png)<br/>
]


<!-- ############################# -->
---
## Links

- [tarantool.io](https://www.tarantool.io/)
- [github.com/tarantool/tarantool](https://github.com/tarantool/tarantool)
- Telegram -
  [@tarantool](https://t.me/tarantool),
  [@tarantool_news](https://t.me/tarantool_news)
<br/>
<br/>
- Cartridge framework - [github.com/tarantool/cartridge](https://github.com/tarantool/cartridge)
- Cartridge CLI - [github.com/tarantool/cartridge-cli](https://github.com/tarantool/cartridge-cli)
- Posts on Habr - [habr.com/users/rosik/](https://habr.com/users/rosik/)
- This presentation - [rosik.github.io/2019-bigdatadays](https://rosik.github.io/2019-bigdatadays)

## Questions?
class: titlepage
background-size: contain
background-image: url(template/bg-face.svg)

# Переосмысление Picodata <br/>в качестве cluster-first СУБД
## Ярослав Дынников
### Picodata
.footnote[Слайды: https://rosik.github.io/2024-shl]
???
- Привет меня зовут Ярослав Дынников
- Я разрабатываю распределенную СУБД Picodata
- И собираюсь сегодня познакомить с ней _вас_

<!-- ############################################################ -->
---
# План доклада
1. Питч
1. Архитектура и алгоритмы
1. Конкурентные преимущества
1. Расширение функциональности
???
- План доклада такой
- Я сначала запитчу наш продукт
- Потом обрисую на верхнем уровне архитектуру
- Расскажу про применяемые алгоритмы
- И из этого мы сможем сделать выводы о конкурентных отличиях пикодаты
  от других вендоров
- А если этого вам покажется мало, в заключение я расскажу как мы
  планируем дополнительно расширять функциональность при помощи плагинов
  на Rust

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
???
- С точки зрения пользователя мы — достаточно обычная база данных
- У нас есть SQL — `select * from`, `join`, и вот это все
- SQL, естесвенно, распределенный.

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
- Sharding, replication
???
- Пользователь взаимодействует с кластером как с единым ресурсом,
- а под капотом спрятано шардирование и репликация

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
- Sharding, replication
- In-memory
???
- Отличительных особенностей у нас есть две
- Первая заключается в том, что все данные лежат в оперативной памяти
- Этим мы режем косты по латенси за счет экономии на доступе к диску
- Это не значит что мы им не пользуемся, нет
- Запись идет в том числе и на жесткий диск
- Но чтение всегда выполняется из оперативной памяти

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
- Sharding, replication
- In-memory, single-threaded
???
- Вторая особенность состоит в том, что
- Доступ к локальному хранилищу реализован однопоточным, а потому без
  блокировок
- И это снова идет на руку производительности

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
- Sharding, replication
- In-memory, single-threaded
- Горизонтальное масштабирование
???
- Масштабируемся мы исключительно горизонтальным образом
- И даже если речь идет о современных многоядерных системах,
- то утилизация ресурсов достигается за счет развертывания нескольких
  инстансов субд, а не за счет многопоточности

<!-- ############################################################ -->
---
# Picodata — это
- Distributed SQL
- Sharding, replication
- In-memory, single-threaded
- Горизонтальное масштабирование
- ...
???
- Итак, распределенный SQL, оперативная память, низкий латенси.

<!-- ############################################################ -->
---
<br><br><br><br><br>
# Picodata —
# Маленькие, быстрые данные
???
- Пикодата — маленькие, быстрые данные

<!-- ############################################################ -->
---
# Что было до
???
- Ну а я пришел сегодня выступить в роли _экскурсовода_
- И познакомить вас с продуктом со стороны _разработки_
- И начну я экскурсию с исторической справки
- Такие продукты как распределенная субд в вакууме не создаются
- Вообще исторические причины зачастую играют решающую роль в вопросах
  выбора архитектуры

--
## Tarantool
In-memory СУБД и сервер приложений на Lua
???
- И так получилось, что я и мои коллеги в прошлом тесно связаны c
- Tarantool — это ...

--
## Vshard
Модуль шардирования на основе виртуальных бакетов
???
- Vshard — это ...

--
## Cartridge
Фреймворк для разработки распределенных приложений
???
- Cartridge — это ...

<!-- - Я думаю я никого не удивлю, если скажу что -->
<!-- - 24 активных контрибутора в репе, 1 внешний -->
<!-- - Initial commit 22 ноября 2021  -->
<!-- ############################################################ -->
---
# Особенности экосистемы
???
- Это срез на начало 2022

--
## Performance
Быстро, но не всегда предсказуемо
???
- LuaJIT, GC

--
## Разработка
Очень интересно, но сложно
???
- box.begin(); netbox.call(); box.commit()

--
## Эксплуатация
Местами слишком гибко<p>
Но иногда слишком строго
???
- Управление каждым репликасетом в отдельности
- Ansible, genin

<!-- ############################################################ -->
---
# План
1. Заменить 2PC на Raft
2. Переписать все на Rust
3. ???
4. Profit

<!-- ############################################################ -->
---
# Raft
.center[![:scale 1050px](images/raft-1.svg#frame2)]

<!-- ############################################################ -->
---
# Raft
.center[![:scale 1050px](images/raft-1.svg#frame1)]

<!-- ############################################################ -->
---
# Топология
.center[![:scale 1050px](images/multiframe-2.svg#frame1)]

<!-- ############################################################ -->
---
# Топология
.center[![:scale 1050px](images/multiframe-2.svg#frame2)]

<!-- ############################################################ -->
---
# Иерархия
.center[![:scale 1050px](images/pyramid.svg)]

<!-- ############################################################ -->
---
# Описание структуры
<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;float:left;margin-right:60px}
.tg td{border-color:black;border-style:solid;border-width:1px;font-size:20pt;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-size:20pt;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c{border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-l{border-color:inherit;text-align:left;vertical-align:top}
.tg .tg-lb{font-weight:bold;text-align:left;vertical-align:top}
</style>

<table class="tg">
  <thead>
  <tr> <th class="tg-c" colspan="2">INSTANCE</th> </tr>
  </thead>
<tbody>
  <tr> <td></td>                <td class="tg-l">raft_id</td></tr>
  <tr> <td class="tg-l">PK</td> <td class="tg-l">instance_id</td> </tr>
  <tr> <td class="tg-l">FK</td> <td class="tg-l">replicaset_id</td> </tr>
  <tr> <td></td>                <td class="tg-lb">current_state</td> </tr>
  <tr> <td></td>                <td class="tg-lb">target_state</td> </tr>
</tbody>
</table>
.footnote[
Важно: не status, а state!<br>
State: Online ⇆ Offline; `incarnation`<br>
]

--
<table class="tg" style="">
  <thead>
  <tr> <th class="tg-c" colspan="2">REPLICASET</th> </tr>
  </thead>
<tbody>
  <tr> <td class="tg-l">PK</td> <td class="tg-l">replicaset_id</td> </tr>
  <tr> <td class="tg-l">FK</td> <td class="tg-l">tier</td> </tr>
  <tr> <td></td>                <td class="tg-lb">current_master</td> </tr>
  <tr> <td></td>                <td class="tg-lb">target_master</td> </tr>
</tbody>
</table>
--
<table class="tg" style="">
  <thead>
  <tr> <th class="tg-c" colspan="2">TIER</th> </tr>
  </thead>
<tbody>
  <tr> <td class="tg-l">PK</td> <td class="tg-l">tier</td> </tr>
  <tr> <td></td>                <td class="tg-l">replication_factor</td> </tr>
  <tr> <td></td>                <td class="tg-l">can_vote</td> </tr>
</tbody>
</table>

<!-- ############################################################ -->
---
# Единая схема данных
```rust
pub enum Op {
    Nop,
    Dml(Dml),
    DdlPrepare { schema_version: u64, ddl: Ddl },
    DdlCommit,
    DdlAbort,
    Acl(Acl),
}
```

<!-- ############################################################ -->
---
class: finalpage
background-size: contain
background-image: url(template/bg-final.svg)

# Материалы

- Слайды: https://rosik.github.io/2023-shl
- Picodata: https://picodata.io/
- Telegram: [@picodataru](https://t.me/picodataru)

.qr[
  **Обратная связь**:<br/><br/>
  ![:scale 100%](images/qr.gif)
  <a href="https://conf.ontico.ru/online/shl2024/lectures/5409989/to-card">
  https://conf.ontico.ru<br>
  /online/shl2024<br>
  /details/5492379
  </a><br/><br/>
]

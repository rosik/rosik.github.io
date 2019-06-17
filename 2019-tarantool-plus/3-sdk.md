<!-- ############################################################ -->
## Третья штука: SDK
### Для сборки проекта используется утилита **tarantoolapp**

???
Первое что нас интересует - это управление зависимостями.<br/>
Как это делают другие?<br/>
Тарантул энетерпрайз - это СДК.

--
- `tarantoolapp pack rpm` упаковывает всё в один артефакт:
  - зависимости
  - сам проект
  - tarantool (бинарь)
  - systemd сервисы

???
tarantoolapp существует.<br/>
Упаковывает и подготавливает к деплою.

<!-- ############################################################ -->
---
## Третья штука: SDK
### Для управления зависимостями используется **rockspec**

```console
$ tarantoolctl rocks make
```

???
Зависимости перечисляются в рокспеке<br/>
Луарокс<br/>

--
```lua
package = 'manhattan'
version = 'scm-1'
source  = {url = 'git+ssh://gitlab.com/manhattan.git'}

dependencies = {
    'cluster == 0.8.0-1',
}

build = {
    type = 'none' -- or make/cmake
}
```

???
Давайте я покажу.

<!-- ############################# -->
---
## Третья штука: SDK
### Для управления зависимостями используется **rockspec**

```console
$ tarantoolctl rocks make
```

```lua
*package = 'manhattan'
*version = 'scm-1'
*source  = {url = 'git+ssh://gitlab.com/manhattan.git'}

dependencies = {
    'cluster == 0.8.0-1',
}

build = {
    type = 'none' -- or make/cmake
}
```

???
Название проекта, версия, репозиторий

<!-- ############################# -->
---
## Третья штука: SDK
### Для управления зависимостями используется **rockspec**

```console
$ tarantoolctl rocks make
```

```lua
package = 'manhattan'
version = 'scm-1'
source  = {url = 'git+ssh://gitlab.com/manhattan.git'}

*dependencies = {
*   'cluster == 0.8.0-1',
*}

build = {
    type = 'none' -- or make/cmake
}
```

???
Тарантулапп установит зависимости.

<!-- ############################# -->
---
## Третья штука: SDK
### Для управления зависимостями используется **rockspec**

```console
$ tarantoolctl rocks make
```

```lua
package = 'manhattan'
version = 'scm-1'
source  = {url = 'git+ssh://gitlab.com/manhattan.git'}

dependencies = {
    'cluster == 0.8.0-1',
}

*build = {
*   type = 'none' -- or make/cmake
*}
```

???
Большинству проектов хватит type=none.<br/>
Если надо что-то собрать, можно пользоваться make.
<!-- ############################# -->
---

<!-- ############################################################ -->
## Третья штука: SDK
### Весь шаблонный код можно сгенерировать

```console
$ tarantoolapp create --template cluster
Enter project name [myproject]: manhattan
```

- `*.lua`
- `rockspec`
- `git` репозиторий
- `.gitignore`, и прочий boilerplate

???
tarantoolapp create генерирует проект по шаблону.<br/>
После create остаётся только написать код.

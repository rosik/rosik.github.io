<!-- ############################################################ -->
## Lua очень простой

```lua
local function hello(name)
    print('Hello, ' .. name .. '!')
end

local names = {'World', 'Highload', 'Moscow'}

for i, name in pairs(name) do
    hello(name)
end
```

```console
Hello, World!
Hello, Moscow!
Hello, Highload!
```

???

Начну издалека, вдруг кто-то в первый раз к нам в гости зашёл.
Что такое Луа и что в нём привлекательного?
Простой, легко компилировать в голове

<!-- ############################################################ -->
---

## Типизация

- Динамическая

- Не сильная (но почти)

???

Луа относится к языкам с динамической типизацией, и это упрощает жизнь
и ускоряет разработку. Кто пробовал на плюсах парсить жсон, тот знает.
<br/><br/>
К вопросом сильной / слабой типизации мы придём позже, а пока пример
из жизни.

<!-- ############################################################ -->
---

## Пример из жизни


```lua
for uuid, new_leader in pairs(new_leaders) do
    log.info('Replicaset %s: new leader %s, was %s',
        uuid,
        describe(new_leader),
        describe(old_leaders[uuid])
    )
end
```

???

- Нам надо было улучшить логирование

---

## Пример из жизни

```diff
for uuid, new_leader in pairs(new_leaders) do
+   if uuid == my_uuid then
+       uuid = uuid .. ' (me)'
+   end

    log.info('Replicaset %s: new leader %s, was %s',
        uuid,
        describe(new_leader),
        describe(old_leaders[uuid])
    )
end
```

???

- Я решил что этого мало

---

## Пример из жизни

```diff
for uuid, new_leader in pairs(new_leaders) do
+   if uuid == my_uuid then
+       uuid = uuid .. ' (me)'
+   end

    log.info('Replicaset %s: new leader %s, was %s',
        uuid,
        describe(new_leader),
*       describe(old_leaders[uuid]) -- nil (иногда)
    )
end
```

???

- Бабах

---

## Пример из жизни

```diff
for uuid, new_leader in pairs(new_leaders) do
+   local replicaset_name = uuid

    if uuid == my_uuid then
-       uuid = uuid .. ' (me)'
+       replicaset_name = replicaset_name .. ' (me)'
    end

    log.info('Replicaset %s: new leader %s, was %s',
-       uuid,
+       replicaset_name,
        describe(new_leader),
        describe(old_leaders[uuid])
    )
end
```

???

- Надо было так
- Динамическая типизация не означает, что в коде можно
  завести 3 переменные abc и переиспользовать их по
  кругу с разными типами

<!-- ############################################################ -->
---

## Типизация — не сильная

Нельзя

```lua
"k" .. nil -- attempt to concatenate a nil value
{1, 2} + {3} -- attempt to perform arithmetic on a table value
-- Python: [1, 2] + [3] == [1, 2, 3]
-- JS:     [1, 2] + [3] == '1,23'
```

???

- Слабой её назвать язык не поворачивается
- Это классно, потому что Lua легко компилировать в голове

--

Можно


```lua
math.sqrt("144") -- 12 (number)

string.len(1337) -- 4
```

.footnote[
  Lua 5.1 Reference Manual.
  [§2.2.1 Coercion](https://www.lua.org/manual/5.1/manual.html#2.2.1)
]

???

- Нужно: явно использовать tostring, tonumber
- Нужно: проверять типы входных аргументов

<!-- # CHECKS ################################################### -->
---

## Аргументы надо проверять

```lua
function get_stat(uri, opts)
    return http.get('http://' .. uri .. '/stat', opts)
end
```

--

```lua
get_stat(req.uri) -- req.uri == nil
-- error: api.lua:310: attempt to concatenate a nil value
-- Не понятно
```

---

## Аргументы надо проверять

```lua
function get_stat(uri, opts)
    assert(type(uri) == 'string', 'uri must be a string')
    -- ...
end
```

--

```lua
get_stat(req.uri) -- req.uri == nil
-- error: api.lua:310: uri must be a string
-- Уже лучше
```

--

```lua
get_stat('localhost', {timeuot = 1})
--                         ^^ typo
-- Ошибки нет, но поведение не правильное
```

---

## Аргументы надо проверять

```lua
require('checks')

function get_stat(uri, opts)
    checks('string', {timeout = '?number'})
    -- ...
end
```

--

```lua
get_stat()
-- error: bad argument #1 to get_stat (string expected, got nil)
```

--

```lua
get_stat('localhost', {timeuot = 1})
-- error: unexpected argument opts.timeuot to get_stat
```

???

- И раз уж об этом зашла речь, упомяну ещё что в мире существует такой
  класс инструментов как линтеры. Для луа это луачек. Он прям в
  редакторе подсвечивает самые вопиющие ошибки.

<!-- ############################################################ -->
---

## Типизация — простая, как топор

- boolean, string, number

- function, thread

- userdata, .grey[cdata]

- table

- nil

???

- cdata - это тип из luajit, в обычном луа его нет.
- cdata позволяет пользоваться из луашки сишной системой типов.

<!-- ############################################################ -->
---

## number == double

???

В луа 5.1 и 5.2 это единственный вариант. Флагами компилятора можно
настроить на float или long double.

--

```lua
assert_equals(0.1 + 0.2, 0.3)
-- error:
-- expected: 0.3
--   actual: 0.3
```

???

С одной стороны никаких сюрпризов с неявными кастами
signed <-> unsigned. Но сюрпризы даблов все на месте.

--

```lua
2^53 - 2 -- 9007199254740990
2^53 - 1 -- 9007199254740991
*2^53 + 0 -- 9007199254740992
*2^53 + 1 -- 9007199254740992
2^53 + 2 -- 9007199254740994

2^53 + 1 == 2^53 -- true
```

???

Числа от 2^52 до 2^53 представимы с точностью до 1, итд

--

```lua
now = clock.time64() -- 1621068872741010434, ~2^60
ffi.typeof(t) -- ctype<uint64_t>
```

???

Как кусается? Есть например апишка clock.time64 - время в
наносекундах с начала эпохи. Используют как таймстампы, пара
неаккуратных тайпкастов и привет.
<br/><br/>
В луа 5.3 завезли ещё и integer, но это касается только внутреннего
представления

<!-- ############################################################ -->
---
## IEEE 754

.center[![:scale 900px](images/double.svg)]
.center[![:scale 600px](images/double-formula.3.svg)]

???

Это касается не только луа, это стандарт I-triple-E

--

.center[
```console
s 11111111111 00..0 ₂ == ±inf
s 11111111111 xx..x ₂ ==  nan
```
]

---

## Not a Number

```lua
nan ~= nan -- true
```

--

```lua
nan = math.sqrt(-1)
nan = 0 / 0
```

--

```lua
nan > nan -- false
nan < nan -- false
nan == nan -- false
```

--

```lua
nan + 1 -- nan
nan * 2 -- nan
```

--

```lua
1 ^ nan -- 1
nan ^ 0 -- 1
```

---

## Not a Number


```console
$ man pow

SYNOPSIS
    #include <math.h>
    double pow(double x, double y);

RETURN VALUE

    If x is +1, the result is 1.0 (even if y is a NaN).
    If y is 0, the result is 1.0 (even if x is a NaN).
```

.footnote[
  [Linux man page](https://linux.die.net/man/3/pow)
]

???

Вывод здесь не тот, о котором вы могли подумать. Вывод - избегайте
нанов вообще. Наны не для бизнес логики.

<!-- ############################################################ -->
---

## LuaJIT internal tags


```c
// lj_obj.h

// Interpreted as a double these are special NaNs. The FPU only generates
// one type of NaN (0xfff8_0000_0000_0000). So MSWs > 0xfff80000 are available
// for use as internal tags.
//                  ---MSW---.---LSW---
// primitive types |  itype  |         |
// lightuserdata   |  itype  |  void * |
// GC objects      |  itype  |  GCRef  |
// int (LJ_DUALNUM)|  itype  |   int   |
// number           -------double------

#define LJ_TNIL     (~0u)
#define LJ_TFALSE   (~1u)
#define LJ_TTRUE    (~2u)
#define LJ_TSTR     (~4u)
#define LJ_TTAB     (~11u)
```

???

Но наны в сишке могут быть полезными. В луаджите есть одно классное
архитектурное решение, которым я хочу с вами поделиться.

<!-- ############################################################ -->
---
## Таблицы

```lua
t1 = { 'Sunday', 'Monday', 'Im tired' }
```

???

Рассказал. Возвращаемся к простоте типов и опасностям, которые они таят.
Таблица - это единственный композитный тип данных, который используется
для всего на свете. Ну а куда деваться то, других типов нету.

--

```lua
t2 = {
    cat = 'meow',
    dog = 'woof',
    cow = 'moo',
}
```

--

```lua
t3 = {
    'k1', 'k2', 'k3',
    ['k1'] = 'v1',
    ['k2'] = 'v2',
    ['k3'] = 'v3',
}
```

<!-- ############################################################ -->
---

## Таблицы изнутри

```c
typedef struct GCtab {
  /* GC stuff */
  MRef array;     /* Array part. */
  MRef node;      /* Hash part.  */
  uint32_t asize; /* Size of array part (keys [0, asize-1]). */
  uint32_t hmask; /* Hash part mask (size of hash part - 1). */
} GCtab;
```

???

В коде луаджита таблицы выглядят так

---

## Таблицы изнутри

```c
typedef struct GCtab {
  /* GC stuff */
* MRef array;     /* Array part. */
  MRef node;      /* Hash part.  */
* uint32_t asize; /* Size of array part (keys [0, asize-1]). */
  uint32_t hmask; /* Hash part mask (size of hash part - 1). */
} GCtab;
```

???

Есть часть с массивом

---

## Таблицы изнутри

```c
typedef struct GCtab {
  /* GC stuff */
  MRef array;     /* Array part. */
* MRef node;      /* Hash part.  */
  uint32_t asize; /* Size of array part (keys [0, asize-1]). */
* uint32_t hmask; /* Hash part mask (size of hash part - 1). */
} GCtab;
```

---

## Таблицы изнутри

```lua
t = {}
-- table: 0x40eae3a8
--  a[0]:
--  h[1]: nil=nil
```

--

```lua
t["a"] = "A"
t["b"] = "B"
t["c"] = "C"
-- table: 0x40eae3a8
--  a[0]:
--  h[4]: b=B, nil=nil, a=A, c=C
```

---

## Таблицы изнутри

```lua
t1 = {a = 1, b = 2, c = 3}
-- table: 0x40eaeb08
--  a[0]:
--  h[4]: b=2, nil=nil, a=1, c=3

t2 = {c = 3, b = 2, a = 1}
-- table: 0x40ea7e70
--  a[0]:
--  h[4]: b=2, nil=nil, c=3, a=1
```

--

```lua
traverse(pairs, t1)
-- b=2, a=1, c=3

traverse(pairs, t2)
-- b=2, c=3, a=1
```

---

## Таблицы изнутри

```lua
t2["c"] = nil
-- table: 0x411c83c0
--  a[0]:
--  h[4]: b=2, nil=nil, c=nil, a=1
```

???

Давайте дальше попробуем удалить из таблицы значение.
Обратите внимание, что ключ в таблице не удаляется.

--

```lua
next(t2, "c") -- a
next(t2, "d") -- error: invalid key to 'next'
```

???

Это позволяет не только избежать лишних телодвижений, но и не ломать лукап.
А он нам нужен, чтобы в цикле можно было удалять элементы

--

```lua
for k, v in pairs(t) do
    t[k] = v .. '_new' -- можно

    t[k .. '_new'] = nil -- нельзя
end
```

---

## Массивы

```lua
t = {1, 2}
-- table: 0x41735918
--  a[3]: nil, 1, 2
--  h[1]: nil=nil
```

???

Ладно, с итерацией по строковым ключам всё понятно, с них спросу нет.
Давайте теперь поговорим о второй половине таблицы — о массиве:
<br/><br/>
Над Lua часто шутят за то, что индексация в языке начинается с единицы.
Интересно то, что LuaJIT всё равно выделяет память под нулевой элемент.

--

```lua
t = {[2] = 2, 1}
-- table: 0x416a3998
--  a[2]: nil, 1
--  h[2]: nil=nil, 2=2
```

???

Если вам кажется, что с массивами никаких сюрпризов быть не может, то
спешу вас огорчить (или обрадовать) — может:
<br/><br/>
Я немного изменил синтаксис, и один элемент элемент попал в массив, а
второй в хеш-мапу. В общем, я предупредил: бесполезно делать
предположения о внутреннем представлении, они будут нарушены в любой
момент.

--

```lua
t = table.new(4, 4)
for i = 1, 8 do t[i] = i end
-- table: 0x412c6df0
--  a[5]: nil, 1, 2, 3, 4
--  h[4]: 7=7, 8=8, 5=5, 6=6
```

???

Я ни в коем случае не хочу показаться параноиком и сам считаю этот
пример весьма неправдоподобным. В 99.9 % случаев "массивы" в Lua
действительно будут представлены массивами, и порядок итерации будет
последовательным.

<!-- ############################################################ -->
---

## Длина массива — определение

```lua
#{1, 2, 3} -- 3
```

.footnote[
  Lua 5.1 Reference Manual.
  [§3.4.6 – The Length Operator](https://www.lua.org/manual/5.1/manual.html#3.4.6)
]

???

Иными словами, не каждая таблица обладает свойством длины. Сюрприз — в
Lua бывает undefined behavior. У таблиц с "дырками" попросту нет такого
свойства.

--

Undefined behavior:

```lua
*#{nil, 2} -- 2
-- table: 0x410d5528
--  a[3]: nil, nil, 2
--  h[1]: nil=nil

*#{[2] = 2} -- 0
-- table: 0x410d5810
--  a[0]:
--  h[2]: nil=nil, 2=2
```

???

Свойства нет, а оператор есть. LuaJIT ищет в таблице "границу". Если в
массиве есть пропущенные значения, а значит границ существует несколько,
то LuaJIT в зависимости от обстоятельств может найти любую из них, и
будет прав:

<!-- ############################################################ -->
---

## Откуда берутся дырки?

```diff
- t[i] = nil
+ table.remove(t, i)
```

???

Откуда берутся дырки все знают, ? Во-первых, кто-то забывает использовать
table.remove и делает t[i] = nil

--

```lua
function vararg(...)
    local args = {...}
    -- #args == undefined behavior
end

vararg(nil, "err")
```

???
Второй способ - завернуть варарг в таблицу.

---

## Длина массива — применение

```lua
table.sort(t) -- 1, #t
unpack(t) -- 1, #t
```

???

Проблемы начинаются, когда мы с такой таблицей пытаемся работать.

Сортировка таблиц всегда работает в диапазоне от 1 до #t, и нет никакого
способа на это повлиять. Поэтому в тех функциях, которые работают с
таблицей как с массивом, полезно проверять входные значения:

--

```lua
function table.pack(...)
    return {n = select('#', ...), ...}
end

t = table.pack(nil, 2)
unpack(t, 1, t.n) -- nil, 2
```

???

Еще часто встречается функция unpack(t), у нас по крайней мере да во всяких проксях.
По дефолту она тоже распаковывает #то хвост может отвалиться (а может и нет, зависит от везения, это ведь UB).

---

## Итерации

```lua
for i = 1, #t do
end

for i, v in ipairs(t) do
end
```

--

```lua
-- ipairs:
local i = 1
while type(t[i]) ~= 'nil' do
    -- do something
    i = i + 1
end
```

<!-- ############################################################ -->
---

## FFI и cdata

```lua
ffi = require('ffi')
NULL = ffi.new('void*', nil)

type(nil) -- nil
type(NULL) -- cdata
ffi.typeof(box.NULL) -- ctype<void *>
```

---

## FFI и cdata

```lua
ffi = require('ffi')
NULL = ffi.new('void*', nil)

type(nil) -- nil
type(NULL) -- cdata
ffi.typeof(box.NULL) -- ctype<void *>

*NULL == nil -- true
```

---

## FFI и cdata

```lua
ffi = require('ffi')
NULL = ffi.new('void*', nil)

type(nil) -- nil
type(NULL) -- cdata
ffi.typeof(box.NULL) -- ctype<void *>

NULL == nil -- true

*if NULL then
    print('NULL is not nil')
end
-- NULL is not nil
```

---

## FFI и cdata

.center[![:scale 600px](images/nil-false-null.svg)]

???

Я тут нарисовал картинку

<!-- ############################# -->
---
## Выводы

- Старайтесь писать код без багов 😊
- Проверяйте аргументы, пользуйтесь линтерами
- Избегайте NaN
- Не делайте лишних предположений
- Бойтесь дырявых массивов

???

Что ж, давайте подведём итоги.

<br/>
# Вопросы?

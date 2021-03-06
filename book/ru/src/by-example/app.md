# The `app` attribute

Это наименьшая возможная программа на RTIC:

``` rust
{{#include ../../../../examples/smallest.rs}}
```

Все программы на RTIC используют атрибут [`app`] (`#[app(..)]`). Этот атрибут
нужно применять к `const`-элементам, содержащим элементы. Атрибут `app` имеет
обязательный аргумент `device`, в качестве значения которому передается *путь*.
Этот путь должен указывать на библиотеку *устройства*, сгенерированную с помощью
[`svd2rust`] **v0.14.x**. Атрибут `app` развернется в удобную точку входа,
поэтому нет необходимости использовать атрибут [`cortex_m_rt::entry`].

[`app`]: ../../../api/cortex_m_rtic_macros/attr.app.html
[`svd2rust`]: https://crates.io/crates/svd2rust
[`cortex_m_rt::entry`]: ../../../api/cortex_m_rt_macros/attr.entry.html

> **ОТСТУПЛЕНИЕ**: Некоторые из вас удивятся, почему мы используем ключевое слово `const` как
> модуль, а не правильное `mod`. Причина в том, что использование атрибутов на
> модулях требует feature gate, который требует ночную сборку. Чтобы заставить
> RTIC работать на стабильной сборке, мы используем вместо него слово `const`.
> Когда большая часть макросов 1.2 стабилизируются, мы прейдем от `const` к `mod` и в конце концов в атрибуту уровне приложения (`#![app]`).

## `init`

Внутри псевдо-модуля атрибут `app` ожидает найти функцию инициализации, обозначенную
атрибутом `init`. Эта функция должна иметь сигнатуру `[unsafe] fn()`.

Эта функция инициализации будет первой частью запускаемого приложения.
Функция `init` запустится *с отключенными прерываниями* и будет иметь эксклюзивный
доступ к периферии Cortex-M и специфичной для устройства периферии через переменные
`core` and `device`, которые внедряются в область видимости `init` атрибутом `app`.
Не вся периферия Cortex-M доступна в `core`, потому что рантайм RTIC принимает владение
частью из неё -- более подробно см. структуру [`rtic::Peripherals`].

Переменные `static mut`, определённые в начале `init` будут преобразованы
в ссылки `&'static mut` с безопасным доступом.

[`rtic::Peripherals`]: ../../api/rtic/struct.Peripherals.html

Пример ниже показывает типы переменных `core` и `device` и
демонстрирует безопасный доступ к переменной `static mut`.

``` rust
{{#include ../../../../examples/init.rs}}
```

Запуск примера напечатает  `init` в консоли и завершит процесс QEMU.

```  console
$ cargo run --example init
{{#include ../../../../ci/expected/init.run}}```

## `idle`

Функция, помеченная атрибутом `idle` может присутствовать в псевдо-модуле
опционально. Эта функция используется как специальная *задача ожидания* и должна иметь
сигнатуру `[unsafe] fn() - > !`.

Когда она присутствует, рантайм запустит задачу `idle` после `init`. В отличие от
`init`, `idle` запустится *с включенными прерываниями* и не может завершиться,
поэтому будет работать бесконечно.

Когда функция `idle` не определена, рантайм устанавливает бит [SLEEPONEXIT], после чего
отправляет микроконтроллер в состояние сна после выполнения `init`.

[SLEEPONEXIT]: https://developer.arm.com/docs/100737/0100/power-management/sleep-mode/sleep-on-exit-bit

Как и в `init`, переменные `static mut`будут преобразованы в ссылки `&'static mut`
с безопасным доступом.

В примере ниже показан запуск `idle` после `init`.

``` rust
{{#include ../../../../examples/idle.rs}}
```

``` console
$ cargo run --example idle
{{#include ../../../../ci/expected/idle.run}}```

## `interrupt` / `exception`

Как Вы бы сделали с помощью библиотеки `cortex-m-rt`, Вы можете использовать атрибуты
`interrupt` и `exception` внутри псевдо-модуля `app`, чтобы определить обработчики
прерываний и исключений. В RTIC, мы называем обработчики прерываний и исключений
*аппаратными* задачами.

``` rust
{{#include ../../../../examples/interrupt.rs}}
```

``` console
$ cargo run --example interrupt
{{#include ../../../../ci/expected/interrupt.run}}```

До сих пор программы RTIC, которые мы видели не отличались от программ, которые
можно написать, используя только библиотеку `cortex-m-rt`. В следующем разделе
мы начнем знакомиться с функционалом, присущим только RTIC.

Как писать на С в 2016
======================

*Этот проект я написал в начале 2015 года, но у меня никак не доходили руки его
опубликовать. Здесь достаточно сырая версия, поскольку проект не приносит
никакой пользы другим людям, лежа у меня на компьютере. Простейшим изменением
было обновление года с 2015 на 2016 во время публикации. *

При необходимости вносите исправления/улучшения/жалобы. -Matt.

 

Первое правило C состоит в том, чтобы не писать на нем, если этого можно
избежать.

Если вам приходится писать на C, то вам следует придерживаться современных
правил.

С появился где-то в начале 1970-х.

Люди выучивали С в различные этапы его эволюции, но их знания заходили в тупик
после изучения, потому как каждый обладал разными знаниями о С, которые
датировались годом (годами), когда они впервые начали его изучение.

При разработке на С очень важно не мыслить  понятиями, изученными в 80-90-х
годах.

Предполагается, что вы используете современную платформу, удовлетворяющую
современным стандартам, и вам не критична совместимость со старыми платформами.
Мы не должны быть привязаны к устаревшим стандартам, лишь из-за отказа некоторых
компаний обновлять свои системы 20-летней давности.

 

Введение
--------

Стандарт c99 (c99 звучит как “Стандарт С, принятый в 1999 году"; c11 means
"Стандарт С, принятый в 2011 году", таким образом, 11 \> 99).

-   clang, по умолчанию

    -   C99 является реализацией по умолчанию для clang без дополнительных опций

    -   Если вам нужен C11, то необходимо указать `-std=c11`

    -   clang компилирует ваши исходные файлы быстрее, чем gcc

-   gcc требует указания `-std=c99` или `-std=c11`

    -   gcc создает исходные файлы медленнее, чем clang, но *иногда* генерирует
        более быстрый код. Сравнения производительности и регрессионное
        тестирование являются важными этапами при работе над проектом.

    -   gcc-5 по умолчанию указывается как `-std=gnu11`, но вам все равно
        следует указывать non-GNU c99 или c11 для практичного использования.

Оптимизация

-   \-O2, -O3

    -   Обычно вам нужен `-O2`, но иногда требуется `-O3`. Протестируйте каждый
        из уровней (и с помощью компиляторов), после чего выберите оптимальный.

-   \-Os

    -   При оптимизации работы кэша (которая должна быть) используйте `-Os`

Предупреждения

-   `-Wall -Wextra -pedantic`

    -   новые версии компиляторов имеют параметр `-Wpedantic`, но они все еще
        поддерживают старый параметр `-pedantic`, что хорошо для обратной
        совместимости.

-   во время тестирования вы должны добавить `-Werror` и `-Wshadow` на всех
    ваших платформах

    -   развертывание исходников с использованием `-Werror` довольно опасное,
        поскольку платформы, компиляторы и библиотеки могут кидать разнообразные
        предупреждения. Вы же не хотите убить пользователю сборку проекта лишь
        потому, что версия GCC под их платформу бросает неизвестные до этого
        исключения?

-   extra fancy options include `-Wstrict-overflow -fno-strict-aliasing`

    -   Либо указывайте `-fno-strict-aliasing`  или будьте уверены, что к
        объектам будут обращаться только в соответствии с типом, которые был
        указан при их создании. Поскольку на языке С написано много схожего
        кода, который имеет разный тип, то использование `fno-strict-aliasing`
        гораздо безопаснее, если вы, конечно, не контролируете все дерево
        исходного кода

-   в настоящее время, Clang помечает некоторый верный синтаксис как
    предупреждение, поэтому вам следует добавить
    `-Wno-missing-field-initializers`

    -   GCC исправил это лишнее предупреждение после GCC 4.7.0

Сборка

-   Модуль компиляции

    -   Самый распространенный путь сборки проектов на C заключается в
        сопоставлении объектного файла каждому исходному файлу, а затем в
        связывании всех объектов в конце воедино. Эта процедура отлично подходит
        для итеративной разработки, но неприемлема для производительности и
        оптимизации. Ваш компилятор не сможет хорошо оптимизировать ваш код,
        пока ваши файлы разделены таким образом.  
        

-   LTO — Link Time Optimization

    -   LTO исправляет “проблему анализа и оптимизации всего юнита компиляции”
        аннотируя объекты файлов промежуточным представлением, таким образом,
        source-aware оптимизация может осуществляться сразу по компиляции юнитов
        в процессе компоновки(это заметно замедляет процесс компоновки, но
        помогает выполнению `make -j`)

    -   [clang
        LTO](<http://llvm.org/docs/LinkTimeOptimization.html>) ([гайд](<http://llvm.org/docs/GoldPlugin.html>))

    -   [gcc LTO](<https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html>)

    -   В 2016, clang и gcc выпустили поддержку LTO лишь добавлением `-flto` в
        опции вашей командной строки в течение компиляции объекта и финальной
        сборки библиотеки/программы.

    -   Однако `LTO` все еще нуждается в некотором присмотре. Иногда, если если
        в вашей программе есть код, не используемый непосредственно, но
        используемый дополнительными библиотеками, LTO может удалить функции или
        код, поскольку будет обнаружено, глобально при компоновке, что некоторый
        код не используется/не достижим и не нуждается во включении в финальную
        сборку.

Arch

-   `-march=native`

    -   дайте компилятору доступ к использованию полного набора функций CPU

    -   Опять же, тестирование производительности и регрессионное тестирование
        является важным (сравнение результатов нескольких компиляторов и/или
        версий компилятора), чтобы убедиться в том, что любые доступные
        оптимизации не имеют неблагоприятных побочных эффектов.

-   `-msse2` и `-msse4.2` могут быть полезными, если необходимо ориентироваться
    на ваши not-your-build-machine особенности.

Описание кода
-------------

### Типы

Вы делаете ошибку, употребляя в новом коде типы
`char`, `int`, `short`, `long`, `unsigned`.

В современных программах для использования стандартных типов необходимо указать
`#include <stdint.h>`.

Общие стандартные типы:

-   `int8_t`, `int16_t`, `int32_t`, `int64_t` — знаковые целочисленные

-   `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t` — беззнаковые целочисленные

-   `float` — 32-битное число с плавающей точкой

-   `double` — 64-битное число с плавающей точкой

Обратите внимание, что больше нет `char`. `char`, фактически, неверно назван, и
неправильно используется в C. 

Разработчики постоянно злоупотребляют типом `char`, подразумевая под ним “байт”
, даже кода производят беззнаковые байтовые манипуляции. Гораздо прозрачнее
использовать `uint8_t`, подразумевая один беззнаковый байт и `uint8_t *` для
обозначения последовательности беззнаковых байтов.

#### Исключение из правил —  `char`

Единственным допустимым использованием `char` в 2016 является использование уже
существующего API, в котором необходим `char` (например, `strncat`, printf'ing
"%s", …),

или если вы инициализируете строку только для чтения (например, `const char
*hello = "hello";`), потому что типом строковых литералов в C  являются `char
*`.

Также: в C11 имеется встроенная поддержка Юникода; строковые литералы, имеющие
кодировку UTF-8, имеют тип `char *` даже для многобайтовых последовательностей,
таких как `const char *abcgrr = u8"abc😬";`

#### Знаковость

Ни в одном месте вашего кода вы не должны писать слово `unsigned`. Сейчас мы
можем писать код без уродливых соглашений в C, касающейся типов из нескольких
слов, которые препятствуют читаемости, а также использованию.  Кто захочет
писать `unsigned long long int`, когда можно написать `uint64_t`? Типы
`<stdint.h>` более явные, более понятные для восприятия, лучше передают смысл и
более компактные для использования и чтения при выводе.

Но вы можете сказать: “Мне нужно приводить указатели к типу `long` для трюка с
указателем!"

Вы можете так сказать. Но вы будете неправы.

Корректным типом для указателя на число является `uintptr_t` объявленный в
`<stddef.h>`.

Вместо:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
long diff = (long)ptrOld - (long)ptrNew;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Используйте:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ptrdiff_t diff = (uintptr_t)ptrOld - (uintptr_t)ptrNew;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### Системо-зависимые типы

Вы скажете: “На 32-разрядной платформе мне нужен тип 32 bit long, а на
64-разрядной -  64 bit long!"

Если мы опустим ход мыслей, при котором вы умышленно вносите трудности в код,
используя два различных размера в зависимости от платформы, то вы по-прежнему не
хотите использовать `long` для типов, зависимых от системы.

В этой ситуации вам следует использовать `intptr_t` — целочисленный тип, который
определяет размер в зависимости от текущей платформы.

На 32-битных платформах, `intptr_t` является `int32_t`.

На 64-битных платформах `intptr_t` является `int64_t`.

`uintptr_t` также является разновидностью `intptr_t`.

Для смещения указателей, у нас есть емко названный  `ptrdiff_t`, который
является надлежащим типом для хранений значений вычитаемых указателей.

#### Хранители максимальных значений

Вам нужен целочисленный тип, способный удержать любое целое число, используемое
в вашей системе?

Люди склоняются к использованию самого большого известного типа в их случае,
такого как приведение меньшего беззнакового типа к `uint64_t`, но есть более
технически правильный путь, гарантирующий, что любое значение может содержать в
себе любое другое значение.

Безопаснейший контейнер для любых целочисленных типов -
`intmax_t` (также `uintmax_t`).  Вы также можете присваивать или приводить любые
знаковые/беззнаковые целочисленные типы к `intmax_t/uintmax_t` без потери
точности.

#### Другие типы

Самым широко используемым типом, зависящим от системы, является `size_t`.

`size_t` определен как “целочисленный тип, способный хранить максимальную длину
массива”, что также означает, что он способен хранить наибольшее смещение памяти
в вашей программе.

На практике `size_t` является возвращаемым типом оператора `sizeof`.

В любом случае: `size_t`, как правило, определен так, чтобы быть таким же как и`
uintptr_t` на всех современных системах, таким образом в 32-битной системе`
size_t` является `uint32_t`, а на 64-битной - `uint64_t`.

Также есть `ssize_t`, который является знаковым `size_t`, использующийся, как
возвращаемое значение из библиотечных функций, которые при ошибке возвращают
`-1`. (Примечание: `ssize_t` является частью интерфейса POSIX, поэтому его
нельзя применять к интерфейсам Windows)

Таким образом, должны ли вы использовать `size_t` для произвольных
системо-зависимых размеров в аргументах вашей функции? Технически, `size_t`
является типом, возвращаемым `sizeof`, таким образом, любые функции, принимающие
в качестве аргумента значение размера в  байтах,  могут быть `size_t`.

Другие области применения: `size_t` является типом аргумента функции malloc,
 `ssize_t` является возвращаемым типом функций `read()` и `write()` (за
исключением Windows, где  `ssize_t` не существует и возвращаемыми значениями
являются просто `int`).

#### Printing Types

Вы никогда не должны приводить типы при выводе. Следует использовать правильные
типовые спецификаторы.

Вот некоторые примеры:

-   `size_t` - `%zu`

-   `ssize_t` - `%zd`

-   `ptrdiff_t` - `%td`

-   исходное значение указателя - `%p` (выводит шестнадцатиричное
    значение;сначала приведите ваш указатель к `(void *)`)

-   64-битные типы должны выводиться с помощью `PRIu64` (беззнаковый)
    и `PRId64` (знаковый)

    -   на некоторых платформах 64-битное значение является типом `long` ,

    а на других — `long long`

    -   на самом деле, невозможно определить правильный формат
        кросс-платформенной строки без этих форматирующих макросов, потому что
        типы меняются под вас (и помните: приведение значения до вывода не
        является безопасным или логичным)

-   `intptr_t` — `"%" PRIdPTR`

-   `uintptr_t` — `"%" PRIuPTR`

-   `intmax_t` — `"%" PRIdMAX`

-   `uintmax_t` — `"%" PRIuMAX`

Небольшое примечание о спецификаторах форматирования `PRI*`: они являются
макросами, расширяющими соответствующие спецификаторы типа printf на основе
текущей системы. Это означает, что вы не сможете сделать так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
printf("Local number: %PRIdPTR\n\n", someIntPtr);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

но в силу того, что они макросы, вы сможете сделать:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
printf("Local number: %" PRIdPTR "\n\n", someIntPtr);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Обратите внимание, что ‘%’ находится внутри форматирующей строки, но тип
спецификатора находится вне форматирующей строки.

### C99 позволяет объвлять переменные в любом месте

Не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void test(uint8_t input) {
    uint32_t b;

    if (input > 3) {
        return;
    }

    b = input;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void test(uint8_t input) {
    if (input > 3) {
        return;
    }

    uint32_t b = input;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Предостережение: если у вас небольшие итерации, то проверьте расположение ваших
инициализаторов. Иногда разрозненные объявления переменных могут привести к
неожиданным замедлениям. Для регулярно используемого кода, который не является
самым быстрым (а такого в мире больше всего), лучше, чтобы он был как можно
более четким, поэтому определение типа переменной рядом с ее инициализаций
является большим шагом к удобочитаемости.

### C99 allows `for` loops to declare counters inline

### С99 позволяет `for` создать счетчики при объявлении цикла

Не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    uint32_t i;

    for (i = 0; i < 10; i++)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    for (uint32_t i = 0; i < 10; i++)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Единственное исключение: если вам необходимо сохранить значение счетчика после
выхода из цикла, очевидно, что необходимо объявить счетчик перед циклом.

### Современные компиляторы поддерживают `#pragma once`

Не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#ifndef PROJECT_HEADERNAME
#define PROJECT_HEADERNAME
.
.
.
#endif /* PROJECT_HEADERNAME */
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#pragma once
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`#pragma once` говорит компилятору включать ваш заголовок только один раз, и вам
больше не нужно писать три строчки header guards. Эта прагма широко
поддерживается всеми компиляторами на всех платформах и рекомендуется к
использованию вместо header guards.

Для более детального ознакомления смотрите список компиляторов, поддерживающих
[pragma once](<https://en.wikipedia.org/wiki/Pragma_once>).

### C позволяет использовать статическую инициализацию массивов 

Никогда не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    uint32_t numbers[64];
    memset(numbers, 0, sizeof(numbers));
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    uint32_t numbers[64] = {0};
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### C позволяет использовать статическую инициализацию структур

Никогда не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    struct thing {
        uint64_t index;
        uint32_t counter;
    };

    struct thing localThing;

    void initThing(void) {
        memset(&localThing, 0, sizeof(localThing));
    }
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    struct thing {
        uint64_t index;
        uint32_t counter;
    };

    struct thing localThing = {0};
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если вам надо переобъявить уже существующую структуру, то объявите глобальную
нулевую структуру для дальнейшего присваивания:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    struct thing {
        uint64_t index;
        uint32_t counter;
    };

    static const struct thing localThingNull = {0};
    .
    .
    .
    struct thing localThing = {.counter = 3};
    .
    .
    .
    localThing = localThingNull;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### C99 поддерживает инициализацию массивов с переменной длиной

Никогда не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    uintmax_t arrayLength = strtoumax(argv[1], NULL, 10);
    void *array[];

    array = malloc(sizeof(*array) * arrayLength);

    /* remember to free(array) when you're done using it */
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:у

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    uintmax_t arrayLength = strtoumax(argv[1], NULL, 10);
    void *array[arrayLength];

    /* no need to free array */
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Важное примечание:** массивы переменной длины (обычно) располагаются также,
как и обычные массивы. Если вы не хотите создавать статический массив на 3
миллиона элементов, не пытайтесь создать массива на 3 миллиона элемента во время
работы программы, используя этот синтаксис. Это не саморасширяемые списки из
python/ruby. Если вы указываете длину массива переменной длины и длина слишком
большая для вашего стека, то ваша программа будет творить ужасные вещи (вылеты,
проблемы с  безопасностью). Массивы переменной длины удобны для небольших
узкоспециализированных ситуаций, но не следует доверять им при масштабной
разработке программного обеспечения. Если когда-то вам понадобится массив на 3
элемента, а когда-то - на 3 миллиона, определенно не стоит использовать массивы
переменной длины.

Хорошо быть осведомленным о синтаксисе массивов переменной длины в случае
столкновения с ними в жизни (или при использовании его при быстром одноразовом
тестировании), но их применение можно считать опасным анти-паттерном, так как вы
можете довольно просто внести фатальные ошибки в работу вашей программы, просто
забыв проверить условие выхода за границу массива, или забыть, что находитесь на
чужой целевой платформе, где в стеке отсутствует свободное пространство.

Примечание: вы должны быть уверены, что `arrayLength` - это подходящая длина
массива в данной ситуации (то есть меньше, чем несколько килобайт; ваш стек не
должен когда-нибудь стать больше 4 килобайт на сторонних платформах). Вы не
можете создавать огромные массивы (на миллионы элементов), но если вы знаете,
что у вас есть ограниченное количество элементов, то лучше использовать
возможности [C99 VLA], чем выделять память вручную с помощью malloc.

Второе примечание: здесь нет проверки пользовательского ввода, поэтому
пользователь легко может положить вашу программу путем создания большого
динамического массива. [Некоторые люд
](<https://twitter.com/comex/status/68542301698196684>)пошли так далеко, что
называют использование динамических массивов анти-паттернов, но если вы четко
следите за границами массива, то это будет крохотной победой в определенных
ситуациях.

### С99 позволяет аннотировать неперекрывающиеся параметры указателя

Смотри [ключевое слово
restrict](<https://en.wikipedia.org/wiki/Restrict>) (также `__restrict`)

### Типы параметров

Если функция принимает произвольные входные данные и длину, не накладывайте
ограничение на тип аргумента:

Не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void processAddBytesOverflow(uint8_t *bytes, uint32_t len) {
    for (uint32_t i = 0; i < len; i++) {
        bytes[0] += bytes[i];
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void processAddBytesOverflow(void *input, uint32_t len) {
    uint8_t *bytes = input;

    for (uint32_t i = 0; i < len; i++) {
        bytes[0] += bytes[i];
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Тип аргументов вашей функции описывает интерфейс вашего кода, а не то что код
делает с аргументами. Интерфейс кода выше означает: “принимается массив байтов и
его длина”. Но вы же не хотите ограничить вызывающих только потоком байтов
uint8\_t. Может ваши пользователи даже хотят в старом стиле  использовать
значение `char *` или что-то еще более неожиданное. Объявляя тип входных данных
`void *` и используя приведение типов внутри функции, вы избавляете
пользователей от мыслей об абстракциях внутри вашей библиотеки.

 

### Тип возвращаемых параметров 

C99 дает нам силу библиотеки `<stdbool.h>`, которая определяет
`true` как `1` и `false` как `0`.

Для успешно/неуспешно возвращаемых значений, функция должна вернуть
`true` или `false`, а не тип `int32_t` с ручным указанием `1` и `0` (или, что
хуже, `1` и `-1` (или `0` — удача, а `1` — неудача? или `0` — удача, а `-1` —
неудача?))

Если функция меняет аргумент в той степени, что он становится недействительным,
то вместо возвращения измененного указателя ваш API должен создать двойные
указатели в качестве параметров в тех местах, где аргументы могут стать
недействительными. Код, написанный с девизом “для некоторых вызовов функции,
возвращаемое значение может быть несовместимо с входным параметром”, подвержен
ошибкам при массовом использовании.

Не делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void *growthOptional(void *grow, size_t currentLen, size_t newLen) {
    if (newLen > currentLen) {
        void *newGrow = realloc(grow, newLen);
        if (newGrow) {
            /* resize success */
            grow = newGrow;
        } else {
            /* resize failed, free existing and signal failure through NULL */
            free(grow);
            grow = NULL;
        }
    }

    return grow;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо этого делайте так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/* Return value:
 *  - 'true' if newLen < currentLen and attempted to grow
 *    - 'true' does not signify success here, the success is still in '*_grow'
 *  - 'false' if newLen >= currentLen */
bool growthOptional(void **_grow, size_t currentLen, size_t newLen) {
    void *grow = *_grow;
    if (newLen > currentLen) {
        void *newGrow = realloc(grow, newLen);
        if (newGrow) {
            /* resize success */
            *_grow = newGrow;
            return true;
        }

        /* resize failure */
        free(grow);
        *_grow = NULL;

        /* for this function,
         * 'true' doesn't mean success, it means 'attempted grow' */
        return true;
    }

    return false;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Или даже лучше так:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
typedef enum growthResult {
    GROWTH_RESULT_SUCCESS = 1,
    GROWTH_RESULT_FAILURE_GROW_NOT_NECESSARY,
    GROWTH_RESULT_FAILURE_ALLOCATION_FAILED
} growthResult;

growthResult growthOptional(void **_grow, size_t currentLen, size_t newLen) {
    void *grow = *_grow;
    if (newLen > currentLen) {
        void *newGrow = realloc(grow, newLen);
        if (newGrow) {
            /* resize success */
            *_grow = newGrow;
            return GROWTH_RESULT_SUCCESS;
        }

        /* resize failure, don't remove data because we can signal error */
        return GROWTH_RESULT_FAILURE_ALLOCATION_FAILED;
    }

    return GROWTH_RESULT_FAILURE_GROW_NOT_NECESSARY;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Оформление кода

Стиль кода одновременно очень важен и абсолютно бесполезен. Если в вашем проекте
50 страниц посвящено правилам оформления кода, то никто не будет вам помогать.
Но если ваш код нечитаем, то никто не *захочет* помогать вам.

Решение этой проблемы заключается в *постоянном*** **использовании
автокорректора кода.

Для форматирования кода на С в 2016 году стоит использовать [clang-format].
clang-format имеет наиболее подходящие параметры по умолчанию для автокорректор
C и в данное время он продолжает активно развиваться.

Вот мой скрипт для запуска форматирования clang с хорошими параметрами:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/usr/bin/env bash

clang-format -style="{BasedOnStyle: llvm, IndentWidth: 4, AllowShortFunctionsOnASingleLine: None, KeepEmptyLinesAtTheStartOfBlocks: false}" "$@"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь вызовем его (предполагаем, что вы назвали скрипт `cleanup-format`):

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
matt@foo:~/repos/badcode% cleanup-format -i *.{c,h,cc,cpp,hpp,cxx}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Параметр  `-i` позволяет перезаписывать существующие файлы в местах их
изменения, а не записывать изменения в новые файлы или создавать резервные копии
файлов.

Если у вас много файлов, то вы можете рекурсивно обрабатывать все дерево
исходного кода параллельно:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/usr/bin/env bash

# note: clang-tidy only accepts one file at a time, but we can run it
#       parallel against disjoint collections at once.
find . \( -name \*.c -or -name \*.cpp -or -name \*.cc \) |xargs -n1 -P4 cleanup-tidy

# clang-format accepts multiple files during one run, but let's limit it to 12
# here so we (hopefully) avoid excessive memory usage.
find . \( -name \*.c -or -name \*.cpp -or -name \*.cc -or -name \*.h \) |xargs -n12 -P4 cleanup-format -i
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вот новый скрипт для очистки. Содержимое файла `cleanup-tidy`:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/usr/bin/env bash

clang-tidy \
    -fix \
    -fix-errors \
    -header-filter=.* \
    --checks=readability-braces-around-statements,misc-macro-parentheses \
    $1 \
    -- -I.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

[clang-tidy](<http://clang.llvm.org/extra/clang-tidy/>) — это инструмент
управления рефакторингом кода. Указанные выше настройки содержат следующие
изменения:

-   `readability-braces-around-statements` — принуждает все тела
    операторов `if`/`while`/`for` быть заключенными в фигурные скобки

    -   Наличие фигурных скобок в С после конструкций циклов и условий есть
        историческая случайность. Непростительно написать современный код без
        скобок после каждого цикла и условия. Аргументы вроде “Но компилятор
        принимает это” не имеют ничего общего с читаемостью,
        ремонтопригодностью, прозрачностью кода. Вы не программируете, упрашивая
        компилятор, вы программируете, чтобы угодить людям, которые будут
        улавливать ваш ход мысли после того, как все уже забыли, почему в коде
        все устроено именно так.

-   `misc-macro-parentheses` — автоматически добавляет скобки вокруг всех
    параметров, использующихся в телах макросов

`clang-tidy` — это здорово, когда он работает, но для некоторых сложных систем
он может дать сбой. Кроме того `clang-tidy` не является форматом, поэтому вам
необходимо запустить `clang-format` после того, как вы выровняли скобки и
переформатировали текст макросов.

 

### Readability

*the writing seems to start slowing down here...*

#### Комментарии

Логические описание автономных частей кода

#### Структура файла

Старайтесь, чтобы в файле было не больше 1000 строк (1500 строк это не очень
хороший выбор).

Try to limit files to a max of 1,000 lines (1,500 lines in really bad cases).
Если ваши тесты находятся на одной линией с вашим исходным файлом, (для
тестирования статических функций и.т.д.), то отрегулируйте их при необходимости.

### misc thoughts

#### Никогда не используйте `malloc`

Всегда используйте `calloc`. Вы не потеряете в производительности при одинаковых
затратах памяти. Если вам не нравится прототип `calloc(object count, size per
object)`, то вы можете сделать обертку `#define mycalloc(N) calloc(1, N)`.

Читатели отметили несколько вещей:

-   `calloc` более производителен при работе с большими выделениями памяти

-   `calloc` более проивзодителен при работе на других платформах (небольшие
    встраиваемые системы, игровые консоли, аппаратные средства 30-летней
    давности ...)

-   обертывать `calloc(element count, size of each element)` не всегда хорошая
    идея

-   хороший способ избежать использования `malloc()` без проверки переполнения и
    потенциальных угроз безопасности

Это хорошие качества, и именно поэтому мы всегда должны производить тестирование
производительности и регрессионное тестирование  для достижения скорости в обход
компиляторов, платформ, оперативных систем и устройств.

Одно из преимуществ использования `calloc()` непосредственно без обертки, в
отличии от `malloc()`, `calloc()` может проверить переполнение целого числа,
потому что он перемножает свои аргументы, чтобы получить окончательный размер
кластера. Если вы выделяете немного памяти, то обертка  `calloc()` возможна.
Если вы выделяете потенциально неограниченный поток данных, то вы можете
сохранить для использования обертку `calloc(element count, size of each
element)`.

Нет универсальных подходов, но можно попытаться  дать некоторые универсальные
советы, которые будут в конечном итоге, как книга по спецификациям языка.

Для справки о том, как `calloc()` высвобождает память, посмотрите эти заметки:

-   [Benchmarking fun with calloc() and zero pages
    (2007)](<https://blogs.fau.de/hager/archives/825>)

-   [Copy-on-write in virtual memory
    management](<https://en.wikipedia.org/wiki/Copy-on-write#Copy-on-write_in_virtual_memory_management>)

Я остаюсь при своем мнении о постоянном использовании `calloc()` для обычных
задача в 2016 (предположение: x64 ориентированные платформы, данные человеческих
размеров, не включая размеров человеческих геномов). Любые отклонения от
ожидаемого втянут вас в яму отчаяния под названием “предметная область”, о
которых мы сегодня говорить не будем.

Subnote: The pre-zero'd memory delivered to you by `calloc()` is a one-shot
deal. If you `realloc()`your `calloc()` allocation, the grown memory extended by
realloc is *not* new zero'd out memory. Your grown allocation is filled with
whatever regular uninitialized contents your kernel provides. If you need zero'd
memory after a realloc, you must manually `memset()` the extent of your grown
allocation.

#### Не используйте memset (если вы можете этого избежать)

Не используйте  `memset(ptr, 0, len)` когда вы можете статически
инициализировать структуру (или массив) нулем (или сбросить к 0 присваиванием
глобальной нулевой структуры)

Смотрите также
--------------

Also see [Fixed width integer types (since
C99)](<http://en.cppreference.com/w/c/types/integer>)

Also see Apple's [Making Code 64-Bit
Clean](<https://developer.apple.com/library/mac/documentation/Darwin/Conceptual/64bitPorting/MakingCode64-BitClean/MakingCode64-BitClean.html>)

Also see the [sizes of C types across
architectures](<https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=4374>) — unless
you keep that entire table in your head for every line of code you write, you
should use explicitly defined integer widths and never use char/short/int/long
built-in storage types.

Also see [size\_t and ptrdiff\_t](<http://www.viva64.com/en/a/0050/>)

Also see [Secure
Coding](<https://www.securecoding.cert.org/confluence/display/c/SEI+CERT+C+Coding+Standard>).
If you really want to write everything perfectly, simply memorize their thousand
simple examples.

Also see [Modern
C](<http://icube-icps.unistra.fr/img_auth.php/d/db/ModernC.pdf>) by Jens Gustedt
at Inria.

### Заключение

Написание правильного кода в больших масштабах практически невозможно. У нас
есть несколько операционных систем , рантаймов, библиотек и аппаратных платформ,
которые имеют свои проблемы, даже без учета такой вещи, как случайно
инвертированный бит в памяти или отказ устройств с неизвестной нам вероятностью.

Лучшее, что мы можем сделать - это написать простой, понятный код с
минималистичными отклонениями и недокументироваными хитростями настолько,
насколько это возможно.

\-[Matt](<mailto:matt@matt.sh>) — [\@mattsta](<https://twitter.com/mattsta>) — [☁mattsta](<https://github.com/mattsta>)

### Attributions

This made the twitter and HN rounds, so many people helpfully pointed out flaws
or biased thoughts I'm promulgating here.

First up, Jeremy Faller and [Sos
Sosowski](<https://twitter.com/Sosowski/status/685431663501926400>) and Martin
Heistermann and a few other people were kind enough to point out
my `memset()` example was broken and provided the proper fix.

Martin Heistermann also pointed out the `localThing = localThingNull` example
was broken.

The opening quote about not writing C if you can avoid it is from the wise
internet sage [\@badboy\_](<https://twitter.com/badboy_>).

[Remi Gacogne](<https://twitter.com/rgacogne/status/685390620723154944>) pointed
out I forgot `-Wextra`.

[Levi
Pearson](<https://twitter.com/pineal_servo/status/685393454487056384>) pointed
out gcc-5 defaults to gnu11 instead of c89.

[Christopher](<https://twitter.com/shrydar/status/685375992114757632>) pointed
out the `-O2` vs `-O3` section could use a little more clarification.

[Chad
Miller](<https://twitter.com/chadmiller/status/685469896914919424>) pointed out
I was being lazy in the clang-format script params.

[Many](<https://twitter.com/lordcyphar/status/685444198481412096>) people also
pointed out the `calloc()` advice isn't *always* a good idea if you have extreme
circumstances or non-standard hardware (examples of bad ideas: huge allocations,
allocations on embedded jiggers, allocations on 30 year old hardware, etc).

Charles Randolph pointed out I misspelled the world "Building."

Sven Neuhaus pointed out kindly I also do not posess the ability to spell
"initialization" or "initializers." (and also pointed out I misspelled
"initialization" wrong the first time here as well)

[Colm
MacCárthaigh](<https://twitter.com/colmmacc/status/685493166988906497>) pointed
out I forgot to mention `#pragma once`.

[Jeffrey
Yasskin](<https://twitter.com/jyasskin/status/685493531515826176>) pointed out
we should kill strict aliasing too (mainly a gcc optimization).

Jeffery Yasskin also provided better wording around
the `-fno-strict-aliasing` section.

[Chris Palmer](<https://twitter.com/fugueish/status/685503534230458369>) and a
few others pointed out calloc-vs-malloc parameter advantages and the overall
drawback of writing a wrapper for `calloc()` because `calloc()` provides a more
secure interface than`malloc()` in the first place.

Damien Sorresso pointed out we should remind people `realloc()` doesn't zero out
grown memory after an initial zero'd `calloc()` request.

Pat Pogson pointed out I was unable to spell the word "declare" correctly as
well.

[\@TopShibe](<https://twitter.com/TopShibe/status/685505183762223105>) pointed
out the stack-allocated initialization example was wrong because the examples I
gave were global variables. Updated wording to just mean "auto-allocated"
things, be it stack or data sections.

[Jonathan
Grynspan](<https://twitter.com/grynspan/status/685509158024691712>) suggested
harsher wording around the VLA example because they **are** dangerous when used
incorrectly.

David O'Mahony kindly pointed out I can't spell "specify" either.

Dr. David Alan Gilbert pointed out `ssize_t` is a POSIXism and Windows either
doesn't have it or defines`ssize_t` as an *unsigned* quantity which obviously
introduces all kinds of fun behavior when your type is signed on POSIX platforms
and unsigned on Windows.

Chris Ridd suggested we explicitly mention C99 is C from 1999 and C11 is C from
2011 because otherwise it looks strange having 11 be newer than 99.

Chris Ridd also noticed the `clang-format` example used unclear naming
conventions and suggested better consistency across examples.

[Anthony Le
Goff](<https://twitter.com/Ideo_logiq/status/685384708188930048>) pointed us to
a book-length treatment of many modern C ideas called [Modern
C](<http://icube-icps.unistra.fr/img_auth.php/d/db/ModernC.pdf>).

Stuart Popejoy pointed out my inaccurate spelling of deliberately was truly
inaccurate.

jack rosen pointed out my usage of the word 'exists' does not mean 'exits' as I
intended.

Jo Booth pointed out I like to spell compatibility as compatability, which seems
more logical, but English commonality disagrees.

Many people on reddit went apeshit because this article originally
had `#import` somewhere by mistake. Sorry, crazy people, but this started out as
an unedited and unreviewed year old draft when originally pushed live. The error
has since been remedied.

Some people also pointed out the static initialization example uses globals
which are always initialized to zero by default anyway. This is a poor choice of
example on my part, but the concepts still stand for typical usage within
function scopes. The examples were meant to be any generic "code snippet" and
not necessarily top level globals.

A few people seem to have read this as an "I hate C" page, but it isn't. C is
dangerous in the wrong hands (not enough testing, not enough experience when
widely deployed), so paradoxically the two kinds of C developers should only be
novice hobbyists (code failure causes no problems, it's just a toy) or people
who are willing to test their asses off (code failure causes life or financial
loss, it's not just a toy) should be writting C code for production usage.
There's not much room for "casual observer C development." For the rest of the
world, that's why we have Erlang.

Many people have also mentioned their own pet issues as well or issues beyond
the scope of this article (including new C11 only features like [George
Makrydakis](<https://twitter.com/irrequietus/status/685407732464226306>) reminding
us about C11 generic abilities).

Perhaps another article about "Practical C" will show up to cover testing,
profiling, performance tracing, optional-but-useful warning levels, etc.

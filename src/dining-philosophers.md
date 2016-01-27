% Обедающие философы

Для нашего второго проекта мы выбрали классическую задачу с параллелизмом. Она
называется «Обедающие философы». Задача была сформулирована в 1965 году Эдсгером
Дейкстрой, но мы будем использовать версию задачи, [адаптированную][paper] в
1985 году Ричардом Хоаром.

[paper]: http://www.usingcsp.com/cspbook.pdf

> В древние времена богатые филантропы пригласили погостить пятерых выдающихся
> философов. Им выделили каждому по комнате, в которой они могли заниматься
> своей профессиональной деятельностью — мышлением. Также была общая столовая,
> где стоял большой круглый стол, а вокруг него пять стульев. Каждый стул имел
> табличку с именем философа, который должен был сидеть на нем. Слева от каждого
> философа лежала золотая вилка, а в центре стола стояла большая миска со
> спагетти, которая постоянно пополнялась. Как подобает философам, они большую
> часть своего времени проводили в раздумьях. Но однажды они почувствовали голод
> и отправились в столовую. Каждый сел на свой стул, взял по вилке и воткнул её
> в миску со спагетти. Но сущность запутанных спагетти такова, что необходима
> вторая вилка, чтобы отправлять спагетти в рот. То есть философу требовалась
> ещё и вилка справа от него. Философы положили свои вилки и встали из-за стола,
> продолжая думать. Ведь вилка может быть использована только одним философом
> одновременно. Если другой философ захочет её взять, то ему придётся ждать
> когда она освободится.

Эта классическая задача показывает различные элементы параллелизма. Сложность
реализации задачи состоит в том, что простая реализация может зайти в
безвыходное состояние. Давайте рассмотрим простой пример решения этой проблемы:

1. Философ берет вилку в свою левую руку.
2. Затем берет вилку в свою правую руку.
3. Ест.
4. Кладёт вилки на место.

Теперь представим это как последовательность действий философов:

1. Философ 1 начинает выполнять алгоритм, берет вилку в левую руку.
2. Философ 2 начинает выполнять алгоритм, берет вилку в левую руку.
3. Философ 3 начинает выполнять алгоритм, берет вилку в левую руку.
4. Философ 4 начинает выполнять алгоритм, берет вилку в левую руку.
5. Философ 5 начинает выполнять алгоритм, берет вилку в левую руку.
6. ...? Все вилки заняты и никто не может начать есть! Безвыходное состояние.

Есть различные пути решения этой задачи. Мы в этом руководстве покажем своё
решение. Сначала давайте создадим проект с помощью `cargo`:

```bash
$ cd ~/projects
$ cargo new dining_philosophers --bin
$ cd dining_philosophers
```

Теперь мы можем начать моделирование задачи. Начнём с философов в `src/main.rs`:

```rust
struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }
}

fn main() {
    let p1 = Philosopher::new("Джудит Батлер");
    let p2 = Philosopher::new("Рая Дунаевская");
    let p3 = Philosopher::new("Зарубина Наталья");
    let p4 = Philosopher::new("Эмма Гольдман");
    let p5 = Philosopher::new("Анна Шмидт");
}
```

Здесь мы создаём [`struct`][struct], представляющую философа. На данный момент
нам нужно всего лишь имя. Мы выбрали тип [`String`][string], а не `&str` для
хранения имени. Обычно проще работать с типом, владеющим данными, чем с типом,
использующим ссылки.

[struct]: structs.html
[string]: strings.html

Продолжим:

```rust
# struct Philosopher {
#     name: String,
# }
impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }
}
```

Этот блок `impl` позволяет объявить что-либо для структуры `Philosopher`. В
нашем случае мы объявляем «статическую функцию» `new`. Первая строка этой
функции выглядит так:

```rust
# struct Philosopher {
#     name: String,
# }
# impl Philosopher {
fn new(name: &str) -> Philosopher {
#         Philosopher {
#             name: name.to_string(),
#         }
#     }
# }
```

Она принимает один аргумент, `name`, типа `&str`. Это ссылка на другую строку.
Она возвращает новый экземпляр нашей структуры `Philosopher`.

```rust
# struct Philosopher {
#     name: String,
# }
# impl Philosopher {
#    fn new(name: &str) -> Philosopher {
Philosopher {
    name: name.to_string(),
}
#     }
# }
```

Этот код создаёт новый экземпляр `Philosopher` и присваивает его полю `name`
значение переданного аргумента `name`. Но используется не сам аргумент, а
результат вызова его метода `.to_string()`. Этот вызов создаёт копию строки, на
которую указывает наш `&str`, и возвращает новый экземпляр `String`, который и
будет присвоен полю `name` структуры `Philosopher`.

Почему бы сразу не передавать строку типа `String` напрямую? Так легче её
вызывать. Если бы мы принимали тип `String`, а тот, кто вызывает функцию, имел
бы ссылку на строку, `&str`, то ему пришлось бы приводить её к типу `String`
перед каждым вызовом. Это уменьшит гибкость кода, и мы будем вынуждены _каждый
раз_ создавать копию строки. Для этой небольшой программы это не очень важно,
так как мы знаем, что будем использовать только короткие строки.

И последнее на что следует обратить внимание: мы просто объявляем структуру
`Philosopher` и кажется, что ничего больше не делаем. Rust — это язык
программирования, «ориентированный на выражения», что означает, что каждое
выражение возвращает значение. Это верно и для функций, у которых автоматически
возвращается последнее выражение. Так как в нашем примере в последнем выражении
функции мы создаём структуру `Philosopher`, то она и будет возвращена функцией.

Имя функции `new()` не регламентируется Rust. Это просто соглашение об
именовании функций, которые возвращают новые экземпляры структур. Давайте снова
посмотрим на функцию `main()`:

```rust
# struct Philosopher {
#     name: String,
# }
#
# impl Philosopher {
#     fn new(name: &str) -> Philosopher {
#         Philosopher {
#             name: name.to_string(),
#         }
#     }
# }
#
fn main() {
    let p1 = Philosopher::new("Джудит Батлер");
    let p2 = Philosopher::new("Рая Дунаевская");
    let p3 = Philosopher::new("Зарубина Наталья");
    let p4 = Philosopher::new("Эмма Гольдман");
    let p5 = Philosopher::new("Анна Шмидт");
}
```

Здесь мы связываем пять имён переменных с пятью новыми философами. Если бы
мы _не объявили_ свою реализацию функции `new()`, то наш код выглядел бы так:

```rust
# struct Philosopher {
#     name: String,
# }
fn main() {
    let p1 = Philosopher { name: "Джудит Батлер".to_string() };
    let p2 = Philosopher { name: "Рая Дунаевская".to_string() };
    let p3 = Philosopher { name: "Зарубина Наталья".to_string() };
    let p4 = Philosopher { name: "Эмма Гольдман".to_string() };
    let p5 = Philosopher { name: "Анна Шмидт".to_string() };
}
```

Этот код выглядит не слишком изящно. Использование статической функции `new`
имеет и другие преимущества, но даже в этом простом случае, её использование
было оправдано.

Теперь у нас уже есть каркас программы, и можно заняться решением задачи с
обедающими философами. Начнём с конца: сделаем так, чтобы философ сообщал нам,
когда он закончит есть. Для этого потребуется метод, сообщающий нам об окончании
приёма пищи, и цикл, запускающий этот метод для каждого философа.

```rust
struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} закончила есть.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Джудит Батлер"),
        Philosopher::new("Рая Дунаевская"),
        Philosopher::new("Зарубина Наталья"),
        Philosopher::new("Эмма Гольдман"),
        Philosopher::new("Анна Шмидт"),
    ];

    for p in &philosophers {
        p.eat();
    }
}
```

Давайте сначала рассмотрим функцию `main()`. Вместо того чтобы создавать пять
отдельных связанных имён для философов, мы создаём для них `Vec<T>`. `Vec<T>`
называют «вектор», он является расширяемой версией массива. Затем в цикле
[`for`][for] мы перебираем вектор, получая ссылку на очередного философа на
каждой итерации.

[for]: loops.html#for

В теле цикла мы вызываем метод `p.eat()`, который объявлен выше:

```rust,ignore
fn eat(&self) {
    println!("{} закончила есть.", self.name);
}
```

В Rust методы явно получают параметр `self`. Вот почему `eat()` является
методом, а `new` — статической функцией: `new()` не получает параметр `self`.
Для нашей первой версии метода `eat()` мы выводим только имя философа и
сообщение о том, что он закончил есть. Запустив эту программу вы получите:

```text
Джудит Батлер закончила есть.
Рая Дунаевская закончила есть.
Зарубина Наталья закончила есть.
Эмма Гольдман закончила есть.
Анна Шмидт закончила есть.
```

Это было не сложно! Осталось чуть-чуть и приступим к самой задаче.

Дальше нам нужно сделать так, чтобы философы не только заканчивали, но и
начинали есть. Это новая версия программы:

```rust
use std::thread;
use std::time::Duration;

struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} начала есть.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} закончила есть.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Джудит Батлер"),
        Philosopher::new("Рая Дунаевская"),
        Philosopher::new("Зарубина Наталья"),
        Philosopher::new("Эмма Гольдман"),
        Philosopher::new("Анна Шмидт"),
    ];

    for p in &philosophers {
        p.eat();
    }
}
```

Появились некоторые небольшие изменения. Давайте посмотрим, что же изменилось:

```rust,ignore
use std::thread;
```

Конструкция `use` предоставляет доступ к области видимости модуля `thread` из
стандартной библиотеки. Мы собираемся использовать этот модуль далее в коде, и
поэтому нам нужно объявить о его использовании.

```rust,ignore
    fn eat(&self) {
        println!("{} начала есть.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} закончила есть.", self.name);
    }
```

Здесь мы выводим на экран два сообщения и вызываем функцию `sleep` между
ними. Эта функция останавливает рабочий поток на 1000 миллисекунд, что
симулирует процесс приёма пищи философа.

Если вы запустите программу теперь, то увидите, что каждый философ, по очереди,
начинает есть, ест какое-то время и заканчивает есть:

```text
Джудит Батлер начала есть.
Джудит Батлер закончила есть.
Рая Дунаевская начала есть.
Рая Дунаевская закончила есть.
Зарубина Наталья начала есть.
Зарубина Наталья закончила есть.
Эмма Гольдман начала есть.
Эмма Гольдман закончила есть.
Анна Шмидт начала есть.
Анна Шмидт закончила есть.
```

Превосходно! Теперь у нас осталась только одна проблема: наши философы едят по
очереди, а не одновременно, то есть мы пока не решили задачу параллелизма.

Для того, чтобы наши философы начали есть одновременно, нам нужно внести
некоторые изменения в код:

```rust
use std::thread;
use std::time::Duration;

struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} начала есть.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} закончила есть.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Джудит Батлер"),
        Philosopher::new("Рая Дунаевская"),
        Philosopher::new("Зарубина Наталья"),
        Philosopher::new("Эмма Гольдман"),
        Philosopher::new("Анна Шмидт"),
    ];

    let handles: Vec<_> = philosophers.into_iter().map(|p| {
        thread::spawn(move || {
            p.eat();
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

Мы добавили ещё один цикл в функцию `main()`. Теперь она выглядит так:

```rust,ignore
let handles: Vec<_> = philosophers.into_iter().map(|p| {
    thread::spawn(move || {
        p.eat();
    })
}).collect();
```

Тут добавились трудные к пониманию пять строк кода. Давайте разбираться.

```rust,ignore
let handles: Vec<_> =
```

Объявляем новое связанное имя `handles`. Мы задали такое имя, потому что
собираемся создать несколько потоков, в результате чего получим для них
дескрипторы, с помощью которых сможем контролировать их выполнение. Здесь нам
нужно явно указать тип, а зачем это необходимо, мы расскажем чуть позже. `_` -
это заполнитель типа. Мы говорим компилятору «`handles` — это вектор, содержащий
элементы, тип которых Rust должен вывести самостоятельно».

```rust,ignore
philosophers.into_iter().map(|p| {
```

Мы берём наш список философов и вызываем метод `into_iter()`. Этот метод создаёт
итератор, который при каждой итерации забирает право владения на соответствующий
элемент. Это нужно для передачи элемента вектора в поток. Мы берём этот итератор
и вызываем метод `map`, который принимает замыкание в качестве аргумента и
вызывает это замыкание для каждого из элементов итератора.

```rust,ignore
    thread::spawn(move || {
        p.eat();
    })
```

Вот здесь происходит сам параллелизм. Функция `thread::spawn` принимает в
качестве аргумента замыкание и исполняет это замыкание в новом потоке. Это
замыкание дополнительно нуждается в указании ключевого слова `move`, которое
сообщает, что это замыкание получает владение переменными, которые оно
захватывает. В данном случае — переменной `p` функции `map`.

Внутри потока мы всего лишь вызываем метод `eat()` переменной `p`. Также
обратите внимание, что вызов `thread::spawn` не оканчивается точкой с запятой,
что превращает его в выражение. Этот нюанс важен, так как возвращается
правильное значение. Для получения более подробной информации, прочитайте главу
[Выражения и операторы][es].

[es]: functions.html#expressions-vs-statements

```rust,ignore
}).collect();
```

По завершении мы получаем результат вызова `map` и собираем полученный результат
в коллекцию с помощью метода `collect()`. Метод `collect()` создаёт коллекцию
какого-то типа, и для того, чтобы Rust понял, коллекцию какого типа мы хотим
получить, мы указали для `handle` тип принимаемого значения `Vec<T>`. Элементами
коллекции будут возвращаемые из методов `thread::spawn` значения, которые
являются дескрипторами этих потоков. Вот так!

```rust,ignore
for h in handles {
    h.join().unwrap();
}
```

В конце функции `main()` мы в цикле перебираем каждый дескриптор и вызываем для
него метод `join()`, который блокирует дальнейшее исполнение основного потока,
пока не завершится дочерний поток. Это позволяет нам быть уверенными, что потоки
завершат работу до того как произойдёт выход из программы.

Если вы запустите эту программу, то вы увидите, что философы едят не дожидаясь
своей очереди! У нас многопоточность!

```text
Джудит Батлер начала есть.
Рая Дунаевская начала есть.
Зарубина Наталья начала есть.
Эмма Гольдман начала есть.
Анна Шмидт начала есть.
Джудит Батлер закончила есть.
Рая Дунаевская закончила есть.
Зарубина Наталья закончила есть.
Эмма Гольдман закончила есть.
Анна Шмидт закончила есть.
```

Но как же быть с вилками? Их мы пока ещё не смоделировали.

Давайте же начнём. Сначала сделаем новую структуру:

```rust
use std::sync::Mutex;

struct Table {
    forks: Vec<Mutex<()>>,
}
```

Структура `Table` содержит вектор мьютексов (`Mutex`). Мьютекс — способ
управления доступом к данным для параллельно выполняющихся потоков: только один
поток может получить доступ к данным в конкретный момент времени. Это именно то
свойство, которое нужно для реализации наших вилок. В коде мы используем пустой
кортеж, `()`, внутри мьютекса, так как не собираемся использовать это значение,
а мьютекс используется только для организации доступа.

Давайте изменим программу, используя структуру `Table`:

```rust
use std::thread;
use std::time::Duration;
use std::sync::{Mutex, Arc};

struct Philosopher {
    name: String,
    left: usize,
    right: usize,
}

impl Philosopher {
    fn new(name: &str, left: usize, right: usize) -> Philosopher {
        Philosopher {
            name: name.to_string(),
            left: left,
            right: right,
        }
    }

    fn eat(&self, table: &Table) {
        let _left = table.forks[self.left].lock().unwrap();
        thread::sleep(Duration::from_millis(150));
        let _right = table.forks[self.right].lock().unwrap();

        println!("{} начала есть.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} закончила есть.", self.name);
    }
}

struct Table {
    forks: Vec<Mutex<()>>,
}

fn main() {
    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});

    let philosophers = vec![
        Philosopher::new("Джудит Батлер", 0, 1),
        Philosopher::new("Рая Дунаевская", 1, 2),
        Philosopher::new("Зарубина Наталья", 2, 3),
        Philosopher::new("Эмма Гольдман", 3, 4),
        Philosopher::new("Анна Шмидт", 0, 4),
    ];

    let handles: Vec<_> = philosophers.into_iter().map(|p| {
        let table = table.clone();

        thread::spawn(move || {
            p.eat(&table);
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

Много изменений! Однако, с этими изменениями мы получили корректно работающую
программу. Приступим к рассмотрению:

```rust,ignore
use std::sync::{Mutex, Arc};
```

Нам далее понадобится структура `Arc<T>` из модуля стандартной библиотеки
`std::sync`. Мы поговорим о ней чуть позже.

```rust,ignore
struct Philosopher {
    name: String,
    left: usize,
    right: usize,
}
```

Нам понадобилось добавить ещё два поля в нашу структуру `Philosopher`. Каждый
философ должен иметь две вилки: одну — для левой руки, другую — для правой руки.
Мы используем тип `usize` для идентификации каждой вилки. Мы используем его при
создании философа, передавая идентификаторы двух вилок. Эти два значения будут
использоваться полем `forks` структуры `Table`.

```rust,ignore
fn new(name: &str, left: usize, right: usize) -> Philosopher {
    Philosopher {
        name: name.to_string(),
        left: left,
        right: right,
    }
}
```

Мы используем функцию `new()` для задания значений `left` и `right`.

```rust,ignore
fn eat(&self, table: &Table) {
    let _left = table.forks[self.left].lock().unwrap();
    thread::sleep(Duration::from_millis(150));
    let _right = table.forks[self.right].lock().unwrap();

    println!("{} начала есть.", self.name);

    thread::sleep(Duration::from_millis(1000));

    println!("{} закончила есть.", self.name);
}
```

Здесь появились три новые строки. Мы добавили один аргумент, `table`. Мы
получаем доступ к списку вилок через структуру `Table`. Затем используем
идентификаторы вилок `self.left` и `self.right` для получения доступа к вилке по
определённому индексу. В результате чего мы получаем `Mutex`, который регулирует
доступ к вилке, и вызываем для него метод `lock()`, блокируя доступ к вилке.
Если в настоящее время доступ к вилке уже предоставлен кому-то ещё, то мы будем
блокированы, пока вилка не станет доступной. Мы также вызываем `thread::sleep`
между взятием первой и второй вилки, поскольку этот процесс не моментален.

Вызов метода `lock()` может потерпеть неудачу, и если это случается, то мы
аварийно завершаем работу программы. Может возникнуть ситуация, когда поток
аварийно завершит свою работу, а мьютекс при этом останется заблокированным.
Такой мьютекс называется «[отравленным (poisoned)][poison]». Но в нашем случае
это не может произойти, потому как мы просто используем метод `unwrap()`.

[poison]: https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html#poisoning

Результаты выполнения этих двух строк имеют имена `_left` и `_right`
соответственно. Зачем мы используем знаки подчёркивания в начале имён? Это для
того, чтобы сказать компилятору, что мы хотим получить значения, которые далее
_не планируем использовать_. Таким образом Rust не будет выводить предупреждение
о неиспользуемых именах.

Когда же мьютекс будет освобождён? Это произойдёт автоматически, когда `_left` и
`_right` выйдут из области видимости, то есть по окончании работы функции.

```rust,ignore
    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});
```

Далее в `main()` мы создаём новый экземпляр структуры `Table` и оборачиваем его
в `Arc<T>`. Это «атомарный счётчик ссылок» (atomic reference count). Он нужен
для обеспечения доступа к нашей структуре `Table` из нескольких потоков. Когда
он передаётся в новый поток, то счётчик увеличивается, а когда этот поток
завершает работу, то счётчик уменьшается.

```rust,ignore
let philosophers = vec![
    Philosopher::new("Джудит Батлер", 0, 1),
    Philosopher::new("Рая Дунаевская", 1, 2),
    Philosopher::new("Зарубина Наталья", 2, 3),
    Philosopher::new("Эмма Гольдман", 3, 4),
    Philosopher::new("Анна Шмидт", 0, 4),
];
```

Мы добавили наши значения `left` и `right` при создании структуры `Philosopher`.
Здесь есть _очень важная_ деталь, на которую следует обратить внимание.
Посмотрите на последнюю строку создания `Philosopher`. Конструктор Анны Шмидт
должен был бы принимать в качестве аргументов значения `4` и `0`, но вместо
этого он принимает значения `0` и `4`. Это помешает нашей программе попасть в
безвыходное состояние, если каждый возьмёт по одной вилке одновременно. Так что
давайте представим, что один из философов у нас левша! Это один из способов
решить данную проблему, и, на мой взгляд, самый простой. Если вы поменяете
порядок параметров, то программа попадёт в безвыходное состояние.

```rust,ignore
let handles: Vec<_> = philosophers.into_iter().map(|p| {
    let table = table.clone();

    thread::spawn(move || {
        p.eat(&table);
    })
}).collect();
```

Внутри нашего цикла `map()`/`collect()` мы вызываем метод `table.clone()`. Метод
`clone()` структуры `Arc<T>` клонирует значение и инкрементирует счётчик,
который автоматически декрементируется, когда клонированное значение покинет
область видимости. Это необходимо для того, чтобы мы знали, как много ссылок на
`table` существуют в рамках наших потоков на данный момент времени. Если бы у
нас не было подсчёта ссылок, то мы бы не знали, как и когда освободить хранимое
значение.

Вы можете заметить, что здесь мы выполняем новое связывание с именем `table`,
затеняя старое связанное имя `table`. Это позволяет нам не вводить новое
уникальное имя.

Теперь наша программа работает! Только два философа могут обедать одновременно.
После запуска программы вы можете получить такой результат.

```text
Рая Дунаевская начала есть.
Эмма Гольдман начала есть.
Эмма Гольдман закончила есть.
Рая Дунаевская закончила есть.
Джудит Батлер начала есть.
Зарубина Наталья начала есть.
Джудит Батлер закончила есть.
Анна Шмидт начала есть.
Зарубина Наталья закончила есть.
Анна Шмидт закончила есть.
```

Поздравляем! Вы реализовали классическую задачу параллелизма на языке Rust.
# Модульность

## basis.resource

Основой модульности `basis.js` являются ресурсы. Они используются для того, чтобы выносить из кода контент различного типа в отдельные файлы, а также сегментировать (разбивать на меньшие части) сам `JavaScript` код. В общем случае ресурс – это некоторый файл, а точнее, интерфейс, обеспечивающий загрузку и доступ к содержимому файла.

Для определения ресурса используется функция `basis.resource()`, которой передается путь к файлу. Путь может быть абсолютным либо относительным. Абсолютные пути начинаются с `/` и разрешаются от корня адреса. Относительные пути должны начинаться с `./` или `../` и разрешаются относительно `html`-файла (обычно это `index.html`).

> В настоящий момент при указании пути к файлу, который начинается не с `/`, `./` или `../`, в консоли выводится предупреждение. При этом путь разрешается относительно `html`-файла. В будущих версиях такие пути перестанут работать, поэтому необходимо явно указывать один из трех возможных префиксов.

Результатом вызова `basis.resource` является функция, вызов которой возвращает содержимое файла. У такой функции есть ряд дополнительных методов и свойств, а также она перенимает интерфейс у [`basis.Token`](basis.Token.md), и, как следствие, поддерживает механизм [`binding bridge`](bindingbridge.md).

Первый вызов функции приводит к загрузке содержимого файла и его кешированию, а последующие вызовы возвращают закешированный результат. У функции есть метод `fetch`, который делает то же самое, и используется для улучшения читаемости кода. Когда содержимое ресурса загружено и обработано, ресурс считается разрешенным.

```js
var someText = basis.resource('./path/to/file.txt'); // объявление, файл еще не загружен

console.log(someText());        // файл будет загружен, его содержимое будет закешировано и возвращено
console.log(someText.fetch());  // эквивалент, будет возвращено закешированное значение
```

Ресурсы обеспечивают возможность раннего связывания и поздней инициализации. Пример:

```js
var example = basis.resource('./path/to/template.tmpl');
var MyControl = basis.ui.Node.subclass({
  template: example
});

// метод ресурса `isResolved` позволяет определить, был ли разрешен ресурс
console.log(example.isResolved());
// > false

var control = new MyControl();
console.log(example.isResolved());
// > true
```

Здесь создается класс `MyControl`, в котором в качестве значения для шаблона задан ресурс. Такое определение не приводит к загрузке файла, так как его содержимое еще не требуется. Но когда будет создан первый экземпляр этого класса, потребуется создать экземпляр шаблона, и тогда будет загружено и использовано содержимое файла.

Содержимое ресурсов считается частью программы (приложения), как если бы его содержимое был указано непосредственно в коде. По этой причине ресурсы загружаются *синхронно*, не могут быть внешними (по отношению к приложению) или быть динамическими (результатом выполнения серверного скрипта). После выполнения сборки содержимое ресурсов встраивается в основной код приложения и не загружаются отдельно. Особый случай – это [виртуальные ресурсы](#%D0%92%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D1%8B), которые не привязанны к конкретному файлу.

### Свойства

#### url

* тип: `String`

Содержит абсолютный разрешенный путь к файлу.

#### type

* тип: `String`

Хранит тип ресурса, представляющий собой расширение файла, включая точку. Например: `.js` или `.css`.

#### virtual

* тип: `Boolean`

Флаг, показывающий является ли ресурс [виртуальным](#%D0%92%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D1%8B).

### Методы

#### fetch()

Возвращает значение ресурса. Обычно это содержимое файла, но для некоторых [типов](#%D0%A2%D0%B8%D0%BF%D1%8B-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2) вместо содержимого могут возвращаться некоторые интерфейсы.

#### get([source])

Если параметр `source` опущен или не приводится к `true`, то возвращается значение ресурса (то же что и вызов `fetch`), иначе возвращается содержимое файла. Для многих [типов ресурсов](#%D0%A2%D0%B8%D0%BF%D1%8B-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2) эти значения одинаковы, но для некоторых они разные, например, для `.js` или `.css`.

#### update(value)

Метод задает новое содержимое ресурса. В основном, этот метод используется для механизмов синхронизации и для обновления [виртуальных ресурсов](#%D0%92%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D1%8B).

#### reload()

Вызов метода сбрасывает кеш ресурса и загружает содержимое значения заново.

#### isResolved()

Метод возвращает `true`, если ресурс разрешен, то есть был вызван его метод `fetch`.

#### hasChanges()

Метод возвращает `true`, если содержимое ресурса было изменено после того, как он был разрешен, но это содержимое не может быть применено без перезагрузки страницы. На данный момент такими свойством обладают только `.js` ресурсы. В основном, это используется инструментами разработки для определения необходимости перезагрузки страницы.

#### ready(fn[, context])

Метод позволяет добавить функцию `fn` с контекстом `context`, которая будет вызывана при разрешении ресурса (после получения и обработки содержимого ресурса). Если ресурс уже разрешен, то функция выполнится сразу. Если ресурс обновляемый, то функция выполнится при каждом обновлении контента (если ресурс был разрешен).

Такая функция получает в качестве единственного аргумента текущее значение ресурса. То есть то же, что можно получить, вызвав метод `fetch` у ресурса.

> Стоит учитывать, что функции, добавленные таким методом, вызываются, только если меняется значение ресурса. Некоторые типы ресурсов, например, `.css`, при изменении содержимого файла обновляют поля интерфейса, при этом значение ресурса не меняется (то есть экземпляр интерфейса остается тот же). В этом случае функции вызываться не будут.

### Наследие basis.Token

Ресурсы перенимают интерфейс у [`basis.Token`](basis.Token.md). Однако он функционален не в полной мере в рамках экземпляра ресурса. В частности, экземпляры `basis.Token` хранят свое значение в свойстве `value`. Но ресурсы хранят свое значение в замыкании, и оно может быть получено только методом `fetch` или методом `get`. Так же оказывается безполезным метод `deferred`, так как он базируется на изменении `value`.

> В версии `1.4` метод `deferred` полагается на метод `get` и работает, как ожидается. Однако нужно помнить, что обработчики будут срабатывать только при изменении значения ресурса. А значение значение ресурса меняется не при каждом изменении содержимого файла и зависит от типа ресурса (например, `.css` ресурсы не обновляют значение ресурса при изменении содержимого файла).

## basis.require

Несмотря на то, что эта функция появилась в `basis.js` раньше `basis.resource`, ее значимость со временем стала меньше. На данный момент эта функция аналогична объявлению и немедленному разрешению ресурса:

```js
var content = basis.require('./path/to/file.ext');
// эквивалент
var content = basis.resource('./path/to/file.ext').fetch();
```

Тем не менее, у `basis.require` есть особенность. Эта функция умеет разрешать путь к файлу по специальному имени. Ранее такие имена явно соотносились с пространством имен. Но в настоящий момент такие имена (названия неймспейсов) в большей степени используются для короткого именования `.js`-ресурсов и разрешаются согласно конфигурации приложения.

Специальным именем (названием неймспейсом) является строка, состоящая из нескольких частей, объединенных точкой, где каждая часть состоит из латинских букв и цифр и не может начинаться с цифры (близко к определению имени переменной). Например, `basis.data.dataset` или `foo.bar.baz`. Такие имена отличает от имен файлов тот факт, что имена файлов должны начинаться с одного из трех префиксов, содержащих слеш (`./`, `../` или `/`), в то время как в специальных именах нельзя использовать слеши.

Специальные имена проецируются на файловую систему по определенному алгоритму. Существует возможность задать базовый путь в [конфигурации приложения](config.md) (`basis-config`), но только для первой части имени, то есть начальной подстроки, что идет до первой точки (либо целиком строка, если она не содержит точек). Такая подстрока еще называется корневым неймспейсом. Если путь для корневого неймспейса не задан в конфигурации, то его базовым путем будет путь к `html`-файлу. Таким образом, к корневому неймспейсу добавляется его базовый путь, а в остальной части неймспейса точки заменяются на слеши, и в конце добавляется расширение `.js`.

Базовый путь для `basis.js` устанавливается неявно при его подключении (используется атрибут `src` у тега `<script>` с атрибутом `basis-config` или `data-basis-config`). Для остальных корневых неймспейсов необходимо явно указавыть базовые пути в `basis-config` в секции `modules`, если они отличаются от базового пути `html`-файла.

> До версии `1.3` для задания путей использовалась секция `path`, которая была несколько ограничена в возможностях конфигурации. Начиная с версии `1.3`, для модулей (корневых неймспейсов) используется секция `modules`, которая имеет больше гибкости по конфигурации. Подробнее в статье [Конфигурация](config.md).

Допустим, на странице `/myapp/index.html` подключается `basis.js` с такой конфигурацией:

```html
<script src="libs/basisjs/src/basis.js" basis-config="modules: { foo: 'src' }"></script>
```

Несмотря на то, что базовый путь для `basis` не указан, он может быть вычислен автоматически. В этом примере базовым путем для `basis` будет `/myapp/libs/basisjs/src`. Здесь также определен путь для `foo`, после нормализации он примет вид `/myapp/src`.

В данном примере неймспейсы будут разрешаться следущим образом:

```js
basis.require('basis.data.dataset');
// basis -> /myapp/libs/basisjs/src/libs/
// basis.data.dataset -> basis/data/dataset.js
// = /myapp/libs/basisjs/src/libs/basis/data/dataset.js

basis.require('foo');
// foo -> /myapp/src
// foo -> foo.js
// = /myapp/src/foo.js

basis.require('foo.bar');
// foo -> /myapp/src
// foo.bar -> foo/bar.js
// = /myapp/src/foo/bar.js

basis.require('baz.baz');
// baz -> /myapp  (по базовому пути страницы)
// baz.baz -> baz/baz.js
// = /myapp/baz/baz.js
```

Путь к файлу по имени неймспейса можно получить, используя функцию `basis.resolveNSFilename(namespace)`.

```js
console.log(basis.resolveNSFilename('basis.data.dataset'));
// > '/myapp/libs/basisjs/src/libs/basis/data/dataset.js'
```

Функция `basis.require` не позволяет получить доступ к использованому ресурсу, так как возвращает его значение. Но используя `basis.resolveNSFilename`, можно получить необходимый ресурс по имени.

```js
var module = basis.resource(basis.resolveNSFilename('basis.data.dataset'));
console.log(module.fetch() === basis.require('basis.data.dataset'));
// > true
```

## Пространства имен

До появления модульности, основанной на ресурсах, в `basis.js` широко использовались пространства имен. Сейчас их роль незначительна, и все идет к тому, чтобы изъять эту функциональность из фреймворка. Единственная значимая часть, которая широко используется (и сохранится) - это использование названий пространств имен как сокращенных ссылок на файл (ресурс) модуля.

Для описания пространства имен используется класс `Namespace`, а для получения экземпляра функция `basis.namespace(name)`. При этом `basis.namespace(name)`, разрешая имя, создает все промежуточные пространства имен.

```js
var ns = basis.namespace('foo.bar.baz');
// такой вызов создаст пространства имен (если они еще не были созданы)
// foo
// foo.bar
// foo.bar.baz
```

В целом, экземпляры `Namespace` ничем не примечательны, кроме того, что создаются автоматически и хранят неявно объявленные вложенные пространства имен и значения `exports` модуля.

```js
var datasets = basis.require('basis.data.dataset');

// обращение к свойствам, объявленным неявно 
var split = new basis.data.dataset.Split();
```

Неявное объявление является исторической особенностью пространств имен. На данный момент оно считается нежелательным и планируется к удалению из `basis.js` (предположительно в версии `1.7`). Рекомендуется явно объявлять зависимости и использовать только явные определения.

```js
var datasets = basis.require('basis.data.dataset');

// НЕПРАВИЛЬНО
var split = new basis.data.dataset.Split();

// ПРАВИЛЬНО
var split = new datasets.Split();
```

> Нежелательность неявного объявления связана с наличием неоднозначных ситуаций и повышенной сложностью анализа приложения. Отсутствие неявных объявлений поможет устранить ряд проблем и улучшить качество анализа.

На переходный период, начиная с версии `1.4`, вводится опция [`implicitExt`](config.md#implicitext) в конфигурации приложения, которая должна помочь с постепенной миграцией и отказом от использования неявных объявлений.

## Виртуальные ресурсы

Виртуальные ресурсы позволяют создавать ресурсы без привязки к файлу. Эта возможность используется для внутренних механизмов или когда нужно преобразовать некоторый контент, полученный не из файла, например, динамически сгенерированный. 

Для создания виртуальных ресурсов используется функция `basis.resource.virtual`. Создаваемый таким образом ресурс не имеет привязки к файлу, а его свойство `virtual` равно `true`. В остальном его поведение ничем не отличается от обычных ресурсов.

```js
var module = basis.resource.virtual('js', 'module.exports = "hello world";');
console.log(module.fetch());
// > "hello world"
```

> Стоит помнить, что содержимое виртуальных ресурсов не может быть проанализировано сборщиком без дополнительных указаний. То есть необходимо обеспечить дополнительные действия для сборщика, чтобы он мог производить анализ. Это стоит учитывать и использовать виртуальные ресурсы с осторожностью.

## Загрузка и кеш

Ресурсы являются частью приложения, как если бы их содержимое находилось непосредсвенно в коде. В сборке так и происходит – сборщик обрабатывает и внедряет содержимое файлов в основной код. Но в режиме разработки содержимое ресурсов загружается посредством `XMLHttpRequest`.

> Исключением являются `.css` ресурсы, которые не внедряются в сам код при сборке, а выносятся в отдельные файлы (один файл на каждую [тему](basis.template_theme.md)). Это связано с особенностью веб-платформы – для более эффективной загрузки сборки приложения клиентом (`JavaScript` и `CSS` файлы могут загружаться и обрабатываться параллельно).

Загрузка ресурсов осуществляется синхронно. При этом в запросе передается заголовок `X-Basis-Resource`. Этот заголовок позволяет `dev-серверу` из `basisjs-tools` определить, что это не обычный запрос, а запрос ресурса. Файлы, запрашиваемые таким образом, добавляются в специальную карту, по которой строится файл кеша ресурсов – `/basisjs-tools/resourceCache.js`. Этот файл внедряется `dev-сервером`, когда он отдает любую `html`-страницу. Когда иницируется `basis.js`, содержимое этого файла используется для первичного наполнения кеша ресурсов. Таким образом, запросив однажды некоторый файл через `XMLHttpRequest`, `basis.js` в дальнейшем получает его содержимое через первичный кеш, внедряемый в страницу. Это позволяет минимизировать число запросов к серверу и ускорить загрузку (так как нет блокирующих синхронных запросов и сетевых издержек).

Но `dev-сервер` не только запоминает запрошенные ресурсы и обеспечивает кеш – он также начинает отслеживать изменения в запрошенных файлах, и если их содержимое меняется, то уведомляет страницу об этих изменениях. Для того чтобы изменения были применены на стороне клиента, `dev-сервер` встраивает еще один файл в отдаваемые `html`-страницы – `/basisjs-tools/fileSync.js`. Этот скрипт устанавливает соединение через `WebSocket` (используется `socket.io`), обеспечивает коммуникацию с сервером, а также уведомляет `basis.js` об изменениях. По сути, он обновляет содержимое ресурсов либо кеш, если ресурс еще разрешен.

> До версии `1.4` изменения в файлах применял скрипт `/basisjs-tools/fileSync.js`. Начиная с версии `1.4`, он ничего не знает о `basis.js`, а изменения применяет сам `basis.js`. Такое изменение связано с тем, что `basis.js` может быть недоступен в `global scope`, а также может быть несколько экземпляров `basis.js` в рамках одной страницы. Также это помогает сделать `dev-сервер` более универсальным: не зависеть от используемой версии `basis.js` и использовать его не только с `basis.js`.

## Жизненный цикл

Любой ресурс проходит несколько этапов обработки.

При объявлении ресурса (использовании функции `basis.resource`) создается лишь интерфейс. При этом переданный путь к файлу приводится к абсолютному пути (в каноническую форму) и кешируется. Повторный вызов `basis.resource` со значением, которое приводится к тому же виду, что было получено для одного из предыдущих вызовов, возвращает уже существующий ресурс (создание нового не происходит). Другими словами, для одного и того же разрешенного пути к файлу возвращается один и тот же интерфейс.

```js
console.log(
  basis.resource('./path/to/file.txt') === basis.resource('./path/foo/../to/file.txt')
);
// > true
```

Когда ресурс разрешается (вызывается его метод `fetch`), делается запрос в кеш и, если файла нет в кеше, то делается синхронный запрос с серверу. Полученное содержимое при необходимости обрабатывается, результат кешируется и возвращается.

В простом случае содержимое ресурса никак не обрабатывается и возвращается, как есть. Но для некоторых ресурсов могут быть определенны трансформации и применены патчи.

Для определенного типа ресурсов (тип определяется расширением, включающим точку) может быть задана функция трансформации. Если для типа задана такая функция, то значением ресурса будет считаться результат ее выполнения (подробнее в ["Типы ресурсов"](#%D0%A2%D0%B8%D0%BF%D1%8B-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2)).

К значению ресурса также могут быть применены патчи. Патчи для ресурса задаются функцией `basis.patch`.

После того, как произведена трансформация и применены патчи, выполняются функций, добавленные методом ресурса `ready`, после чего возвращается значение ресурса.

Так как ресурсы являются частью программы, то процедура разрушения (удаление ссылок и содержимого) для них не предусмотрена.

> В рамках концепции ресурсов это является неправильным. Однако в ходе разработки, при реструктуризации или исправлении ошибок бывают ситуации, когда ресурс перестает существовать. В этом случае изменения вступят в силу лишь после перезагрузки страницы (устаревший ресурс не будет объявлен или создан). Возможно, в будущем получится сделать так, чтобы для "устаревших" обновляемых ресурсов не требовалась перезагрузка страницы.

В общем виде жизненный цикл выглядит так:

* Объявление ресурса
* Разрешение ресурса
  * Получение содержимого (из кеша или запросом к серверу)
  * Применение трансформации (если определено)
  * Кеширование значения
  * Применение патчей (если определены)
  * Вызов обработчиков `ready` (если определены)
* Возвращение значения ресурса

Когда меняется содержимое файла, вызывается метод `update` и выполняются следующие шаги:

* Обновляется кеш содержимого
* Применение трансформации (если определено)
* Кеширование значения
* Применение патчей (если определены)
* Вызов обработчиков `ready` (если определены)

Но эти действия выполняются только в том случае, если ресурс разрешен и является обновляемым. Если ресурс еще не разрешен, то только обновляется кеш содержимого.

## basis.patch

Для определенного файла можно задать функцию, которая может модифицировать значение ресурса. Результат выполнения этой функции никуда не сохраняется. Поэтому данная операция актуальна, в основном, для типов ресурсов, которые возвращают не значение файла, а некоторый интерфейс к нему. Тогда появляется возможность модифицировать сам интерфейс (некоторый объект). 

Функции передается два параметра: имя файла или неймспейс и функция, которая должна быть вызвана, когда разрешается ресурс. Поведение в отношении первого параметра совпадает с поведением `basis.require`. Задаваемая таким образом функция получает на вход значение ресурса и путь к его файлу. Если в момент вызова `basis.patch` ресурс уже разрешен, то функция вызывается немедленно. Перед выполнением функции всегда пишется уведомляющее сообщение, что к ресурсу применен патч (отдельное сообщение для каждого патча). Функция вызывается при каждом изменении содержимого файла, если ресурс разрешен, даже если не меняется значение самого ресурса (например, для `.css`). Если ресурс не обновляемый, то функция вызовется только при первом разрешении.

```js
basis.patch('basis.data', function(exports, url){
  exports.Object.prototype.myExtension = function(){
    console.log('I\'m extension added via patch');
  };
});

var DataObject = basis.require('basis.data').Object;
var example = new DataObject();
example.myExtension();
// > "I'm extension added via patch"
```

Патчи применимы для контролируемого переопределения или дополнения существующих модулей. Например, если есть необходимость дополнить функциональность стандартного модуля. Или если есть баг в модуле, можно воспользоваться `basis.patch` для горячего исправления, пока он не появится в официальной версии.

Несмотря на то, что патч может быть применен сразу к уже разрешенному ресурсу, стоит стараться добавлять патчи до разрешения ресурса, к которому он применяется. Это позволит избежать проблем с кодом, который может полагаться на патч.

Важной особенностью `basis.patch` является тот факт, что подобное объявление не влияет на информацию о ресурсах. Другими словами, не приводит к разрешению ресурса или регистрации имени файла. Также функции, добавленные `basis.patch`, гарантировано выполняются перед функциями, добавлеными методом `ready`, и, таким образом, последние получают уже пропатченное значение.

Несмотря на возможности, предоставляемые `basis.patch`, стоит прибегать к этой функциональности только в случае крайней необходимости.

## basis.asset

Ресурсы описывают контент, который является частью программы. Но существуют ситуации, когда контент должен остаться отдельным файлом, например, изображения.

В этом случае путь к файлу оборачивается в функцию `basis.asset`. Данная функция ничего не делает, кроме как возвращает переданное значение. Но ее вызов служит подсказкой для сборщика `basisjs-tools`, так он понимает, что значение является путем к файлу, который нужно добавить в сборку. Если не использовать `basis.asset` для путей, то ссылка на файл не будет обнаружена, и файл не будет добавлен в сборку. Необходимо применять функцию только к путям, которые генерируются в `JavaScript`-коде.

```js
var logoImage = new Image();
logoImage.src = basis.asset('path/to/logo.jpg');
```

## Вспомогательные функции

Для управления ресурсами и получения дополнительной информации о них используется набор функций, которые прикреплены к `basis.resource`.

### basis.resource.isResource(value)

Возвращает `true`, если переданное значение является ресурсом.

### basis.resource.exists(filename)

Возвращает `true`, если для имени файла ресурс объявлен (для была вызвана функция `basis.resource`). Перед проверкой имя файла приводится к каноническому виду. Нового ресурса не создается.

### basis.resource.isResolved(filename)

Возвращает `true`, если для переданного имени файла объявлен ресурс и он разрешен (его метод `isResolved` возвращает `true`). Перед проверкой имя файла приводится к каноническому виду. Функция не создает нового ресурса, если он не объявлен.

### basis.resource.get(filename)

Возвращает ресурс для имени файла, если он объявлен. Перед проверкой имя файла приводится к каноническому виду.

### basis.resource.getFiles([cache])

Возвращает массив имен файлов объявленных ресурсов. Если параметр `cache` приводится к `true`, то возвращается список имен файлов, которые есть в кеше (часть из которых может быть получена от `dev-сервер`).

> До версии `1.4` возвращались имена файлов относительно `html`-файла, при этом не игнорировались имена виртуальных ресурсов. Начиная с версии `1.4`, возвращаются абсолютные имена (как они хранятся в словарях), а имена виртуальных ресурсов игнорируются.

### basis.resource.virtual(type, content[, ownerFilename])

Позволяет создать виртуальный ресурс. При этом указывается тип ресурса (расширение *без* точки) и первоначальное содержимое ресурса. Также опционально можно указать `ownerFilename`, тогда это значение будет использовано для формирования виртуального `url`, что может помочь понять, от какого файла был образован виртуальный ресурс при разработке и отладке.

### basis.resource.extensions

Словарь, определяющий функции трансформации содержимого ресурсов. Ключом является тип ресурса (расширение, включая точку), а значением – функция, принимающая на вход содержимое файла, его `url` и предыдущее значение ресурса.

Если у функции есть свойство `permanent`, которое приводится к `true`, то ресурс считается не обновляемым и может быть разрешен только раз. На данный момент такими ресурсами являются только `.js`-ресурсы.

## Типы ресурсов

В зависимости от типа файла (его расширения) может возвращаться не только текстовое значение файла, но и значения других типов. То, что будет возвращаться, определяет post-обработчик, ассоциированный с определенным расширением. В `basis.js` определены обработчики для расширений `.js`, `.json` и `.css`. Можно [определять собственные обработчики](#%D0%94%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D0%BE%D0%B1%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA%D0%BE%D0%B2-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2).

### JavaScript

Содержимое `.js`-файлов оборачивается в специальную функцию и немедленно вызывается с определенными параметрами. Таким образом, для кода всегда создается локальная область видимости, в которой доступно несколько дополнительных значений и функций:

* `exports` – объект экспорта;
* `module` – объект, представляющий модуль (`module.exports` === `exports`);
* `basis` – ссылка на корневое пространство имен `basis.js`;
* `global` – ссылка на глобальную область видимости (в браузере `window`);
* `__filename` – полный путь к файлу;
* `__dirname` – полный путь до директории, содержащей файл;
* `resource` – функция, аналог `basis.resource`, но разрешает относительные пути относительно файла модуля;
* `require` – функция, аналог `basis.require`, но разрешает относительные пути относительно файла модуля.

Пример модуля, допустим, его путь `/src/module/list/index.js`:

```js
var Node = require('basis.ui').Node;

var list = Node({
  template: resource('./template/list.tmpl'),  // путь к файлу /src/module/list/template/list.tmpl
  ...
});

// то, что будет возвращаться при использовании ресурса
module.exports = list;
```

> Функции `resource` и `require` доступны только в рамках модулей подключаемых через `basis.resource` либо `basis.require`.

`JavaScript`-модули работают схожим образом с `node.js`. Значением ресурса будет являться значение `module.exports`.

Использование:

```js
// либо сразу получаем содержимое
var list = basis.require('./src/module/list/index.js');
console.log(list);

// либо объявляем ресурс
var list = basis.resource('./src/module/list/index.js');
console.log(list.fetch());    // а разрешаем потом
```

Следующий код схематично показывает, как оборачивается содержимое `JavaScript`-ресурса:

```js
var __filename = '/src/module/list/index.js';
var __dirname = '/src/module/list'
var module = {
  exports: {}
};
var relResource = function(url){
  return basis.resource(basis.path.resolve(__dirname, url));
};
var relRequire = function(urlOrNamespace){
  return basis.require(urlOrNamespace, __dirname)
};

(function(exports, module, basis, global, __filename, __dirname, resource, require){
  'use strict';

  // содержимое файла /src/module/list/index.js

}).call(module.exports, module.exports, module, basis, this,
  __filename, __dirname, relResource, relRequire);

return module.exports;
```

> Код `JavaScript`-ресурсов выполняется в `strict mode`.

`JavaScript`-ресурсы не обновляемы (`permanent: true`): если такой ресурс разрешен, то его значение не меняется, когда меняется содержимое файла.

### JSON

Если файл имеет расширение `.json`, то содержимое файла преобразуется функцией `JSON.parse` и возвращается результат.

```js
var settings = basis.require('./settings.json');
if (settings.someName) {
  // do something
}
```

Если файл содержит некорректный `json`, то будет выведено сообщение об ошибке в консоли и возвращен `null`.

### CSS

Для файлов с расширением `.css` создается и возвращается специальный интерфейс - экземпляр класса `CssResource`.

Такой интерфейс имеет два основных метода: `startUse` и `stopUse`. При первом вызове `startUse` в документ добавляется элемент `<style>`, содержащий код `.css`-файла. При этом считается количество вызовов метода `startUse`, и если столько же раз будет вызван метод `stopUse`, то элемент `<style>` будет удален из документа. При изменении содержимого `.css`-файла содержимое тега `<style>` также обновляется.

Вставка содержимого в тег `<style>` производится таким образом, чтобы ссылки на ресурсы (`@import`, `url(..)` и т.д.) разрешались относительно папки расположения файла. Также, начиная с версии `1.2.4`, к содержимому добавляется `//@ sourceURL=..`, чтобы инструменты разработки, такие как `Developer Tools` в `Google Chrome`, могли ассоциировать сождержимое `<style>` с оригинальным файлом.

Получить текущий `css`-код можно, обратившись к свойству `cssText`.

```js
var myStyle = basis.require('./style.css');
console.log(myStyle.cssText); // выведет содержимое файла
myStyle.startUse();  // добавит в документ <style>
...
myStyle.stopUse();   // удалит <style> из документа
```

### Добавление собственных обработчиков ресурсов

Чтобы определить собственный обработчик для некоторого расширения, нужно зарегистирировать его в объекте `basis.resource.extensions`, где ключ - это расширение файла, включая точку, а значение - функция, принимающая содержимое файла и путь к нему. Возвращаемое такой функцией значение будет являться значением ресурса.

Допустим, требуется добавить поддержку `CoffeeScript`. Для этого нужно подключить компилятор `CoffeeScript` и назначить обработчик расширений `.coffee`:

```html
<script src="path/to/coffeescript.js"></script>
<script>
  basis.resource.extensions['.coffee'] = function(content, url){
    return basis.resource.extensions['.js'](CoffeeScript.compile(content), url);
  }
</script>
```

Компилятор `CoffeeScript` можно подключить и через механизм ресурсов:

```js
var CoffeeScript = basis.require('./path/to/coffeescript.js').CoffeeScript;

basis.resource.extensions['.coffee'] = function(content, url){
  return basis.resource.extensions['.js'](CoffeeScript.compile(content), url);
}
```

> Не все стороние библиотеки могут быть подключены как ресурс, так как не все они работают в `strict mode` и поддерживают `CommonJS`.

Данное решение будет работать в режиме разработки, но сборщиком код `CoffeeScript` будет восприниматься как строка. Как результат, код такого модуля не будет проанализирован, а его зависимости не будут найдены. Для решения проблемы сборщику нужно скомпилировать `CoffeeScript` в `JavaScript`, для этого в его настройках определяется препроцессор для файлов с расширением `.coffee`:

```json
{
  "build": {
    ...
    "extFileTypes": {
      ".coffee": {
        "type": "script",
        "preprocess": "compile-coffee-script.js"
      }
    }
  }
}
```

Содержимое `compile-coffee-script.js` может иметь такой вид:

```js
var CoffeeScript = require('./path/to/coffeescript.js');

exports.process = function(content, file, baseURI, console){
  console.log('Compile ' + file.relpath + ' to JavaScript');
  file.filename = file.filename.replace(/.coffee$/i, '.js');
  return CoffeeScript.compile(content);
}
```

С таким препроцессором при сборке `CoffeeScript` код будет скомпилирован в `JavaScript`, а расширение файлов изменено с `.coffee` на `.js`.

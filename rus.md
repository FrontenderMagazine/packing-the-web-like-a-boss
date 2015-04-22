# Пакуем для веба как босс

[webpack][3] — это крутая новая утилита для сборки бандлов и оптимизации модулей
JavaScript и других ресурсов для фронтенда.
Если вы уже пользовались [RequireJS][4] или [Browserify][5], то велики шансы,
что вы полюбите webpack так же, как и я.
Эта статья — подробная инструкция для вас.

![Логотип webpack][2]

Чтобы продемонстрировать магию webpack, я в этой статье буду использовать
в качестве примеров очень простой, даже неестественно примитивный, код.
[Он хранится на GitHub, вот тут][6].

Представьте, что вам что-то взбрело в голову и захотелось написать код на
JavaScript, который делает блок текста розовым, и другой код на JavaScript,
который делает блок текста полужирным по нажатию на кнопку.
Очень замысловатая штука, ага. Но как бы вы такое сделали?

## Наивный способ доставки JavaScript на клиент

Если бы мы пилили JavaScript как в старые-добрые беззаботные времена, то вы бы
создали такие файлы:

    function Pinkyfier(id) { // Орозовитель
        this.element = document.getElementById(id);
    }

    Pinkyfier.prototype.pink = function () {
        this.element.style.backgroundColor = "mistyrose";
        this.element.style.color = "hotpink";
    }

[js/Pinkyfier.js][7]

    function Fattyfier(id) { // Ожирнитель
        this.element = document.getElementById(id);
    }

    Fattyfier.prototype.fat = function () {
        this.element.style.fontWeight = "bold";
    }

[js/Fattyfier.js][8]

    var pinkyfier = new Pinkyfier("text"),
        fattyfier = new Fattyfier("text");

    pinkyfier.pink();

    document.getElementById("fat").onclick = function () {
        fattyfier.fat();
    }

[js/main.js][9]

Вы бы подключили эти три файла JS в документ HTML при помощи тегов script, вот
так:

    <!DOCTYPE html>
    <html lang="ru">
        <head>
            <meta charset="utf-8">
        </head>
        <body>
            <p id="text">
                Родился на улице Герцена, в гастрономе № 22. Известный экономист,
                по призванию своему — библиотекарь. В народе — колхозник.
                В магазине — продавец. В экономике, так сказать, необходим.
            </p>
            <button id="fat" type="button">Ожирнить</button>

            <script src="js/Fattyfier.js"></script>
            <script src="js/Pinkyfier.js"></script>
            <script src="js/main.js"></script>
        </body>
    </html>

[index.html][10]

Вы открываете файл HTML в своём браузере ([или смотрите на демо здесь][11]),
вы любуетесь розовым цветом, щёлкаете по кнопке, чтобы сделать его полужирным,
и вы довольны.
Чего ещё желать?

Ну, все функции и переменные, которые вы определили в этих трёх файликах,
находятся в глобальной области видимости.
Как известно каждому программисту, глобальные переменные — полная лажа.
Что, если бы у вас было более сложное приложение со, скажем, сотней фалов JS
вместо трёх?
Что, если бы код в этих файлах зависел от других файлов сильнее,
и изменение порядка подключения скриптов ломало бы страницу?

Если бы вы продолжали делать так, в итоге получилась бы здоровенная неуклюжая
каша из кода. Вам нужна система модулей.

## Способ получше: модули AMD

Прежде чем рассматривать магию webpack и на то, как он всё делает лучше, давайте
взглянем на самые известные системы модулей: AMD, CommonJS и, в ближайшем
будущем, ES6.

Самая распространённая реализация модулей AMD — это [RequireJS][4].
Посмотрим, как мы можем сделать текст на страничке розовым и полужирным при
помощи RequireJS.

Вместо того, чтобы вываливать классы `Pinkyfier` и `Fattyfier` прямо в
глобальную область, мы обернём их в клёвую конструкцию `define`:

    define(function () {
        function Pinkyfier(id) {
            this.element = document.getElementById(id);
        }

        Pinkyfier.prototype.pink = function () {
            this.element.style.backgroundColor = "mistyrose";
            this.element.style.color = "hotpink";
        }

        return Pinkyfier;
    });

[js/Pinkyfier.js][12]

    define(function () {
        function Fattyfier(id) {
            this.element = document.getElementById(id);
        }

        Fattyfier.prototype.fat = function () {
            this.element.style.fontWeight = "bold";
        }

        return Fattyfier;
    });

[js/Fattyfier.js][13]

Вы теперь можете в своём `main.js` запросить те два модуля, которые мы только
что определили:

    require([ "Fattyfier", "Pinkyfier" ], function (Fattyfier, Pinkyfier) {

        var pinkyfier = new Pinkyfier("text"),
            fattyfier = new Fattyfier("text");

        pinkyfier.pink();

        document.getElementById("fat").onclick = function () {
            fattyfier.fat();
        }
    });

[js/main.js][14]

А в коде HTML вместо трёх тегов script, как в примере ранее, вы просто пишете
один тег script, который загружает RequireJS, с data-атрибутом указывающим
на точку входа в ваше приложение, `main.js`:

    <script data-main="js/main.js" src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.17/require.min.js"></script>

[index.html][15]

Вот так с лёгкостью можно строить более сложные приложения, и что самое
приятное, модули подгружаются асинхронно — блокирует загрузку страницы только
загрузка самой библиотеки RequireJS, ваши же скрипты её не блокируют.

[Посмотрите на результат вживую тут][16].

## Как в Node.js: модули CommonJS

С появлением серверного JavaScript на основе [Node.js][17] или [io.js][18]
стала популярной другая система модулей JavaScript, [CommonJS][19].

Хотя она используется в основном для npm-модулей и приложений, выполняющихся
на серверах, при помощи [Browserify][5] ей можно также пользоваться в коде
на клиентской стороне.
Это очень круто, потому что в ваше распоряжение попадает всё изобилие модулей
npm, а также это позволяет использовать один и тот же код как на бэкенде,
так и на фронтенде.

Вот наш маленький примерчик в виде модуля CommonJS:

    function Pinkyfier(id) {
        this.element = document.getElementById(id);
    }

    Pinkyfier.prototype.pink = function () {
        this.element.style.backgroundColor = "mistyrose";
        this.element.style.color = "hotpink";
    }

    module.exports = Pinkyfier;

[js/Pinkyfier.js][20]

    function Fattyfier(id) {
        this.element = document.getElementById(id);
    }

    Fattyfier.prototype.fat = function () {
        this.element.style.fontWeight = "bold";
    }

    module.exports = Fattyfier;

[js/Fattyfier.js][21]

Как вы можете заметить, единственное отличие от вышеупомянутой «наивной»
реализации в последних строчках каждого файла указывающих, какие части кода
следует отдавать клиенту при подгрузке модулей, а именно, классы `Pinkyfier` и
`Fattyfier`.
В этом отношении модули CommonJS проще и менее навязчивы, чем RequireJS.

Чтобы использовать эти модули, добавьте вызов `require` в `main.js`:

    var Pinkyfier = require("./Pinkyfier"),
        Fattyfier = require("./Fattyfier"),

        pinkyfier = new Pinkyfier("text"),
        fattyfier = new Fattyfier("text");

    pinkyfier.pink();

    document.getElementById("fat").onclick = function () {
        fattyfier.fat();
    }

[js/main.js][22]

А как этим пользоваться в браузере? Browserify работает иначе, чем RequireJS.
Вместо того, чтобы подгружать библиотеку, которая всё делает сама, мы при помощи
утилиты на node создаём бандл, файл, содержащий весь ваш код на JavaScript.

Чтобы это сделать, установите Browserify через npm (Я полагаю, что node и npm
у вас установлены):

    npm install -g browserify

А теперь перейдите в папку, где хранятся ваши файлы JS и создайте бандл такой
командой:

    browserify main.js > bundle.js

Теперь достаточно просто подключить его в код HTML тегом скрипт:

    <script src="js/bundle.js"></script>

[index.html][23]

[Посмотрите на результат вживую][24].

## Будущее: модули ES6

Сейчас, когда я пишу эти строки, новый стандарт JavaScript лишь только на
подходе и должен быть выпущен в июне 2015.

ES6 привносит нативную поддержку модулей JavaScript, делая AMD и CommonJS
устаревшими.

Давайте взглянем, как орозовить и ожирнить наш рыбный текст с ES6:

    class Pinkyfier {

        constructor(id) {
            this.element = document.getElementById(id);
        }

        pink() {
            this.element.style.backgroundColor = "mistyrose";
            this.element.style.color = "hotpink";
        }
    }

    export default Pinkyfier;

[js/Pinkyfier.js][25]

    class Fattyfier {

        constructor(id) {
            this.element = document.getElementById(id);
        }

        fat() {
            this.element.style.fontWeight = "bold";
        }
    }

    export default Fattyfier;

[js/Fattyfier.js][26]

Ух ты, а что это было? Ну, в ES6, как вы видите, классы определяются совсем
по-другому, нежели в том старом JavaScript, который мы знаем и любим.
Но речь тут даже не об этом, вся соль в последних строчках с ключевыми словами
`export`, которые выносят определённые классы из файла в клиентский модуль.
Выглядит похоже на пример с CommonJS, не правда ли?

А вот так мы используем модули в ES6:

    import Pinkyfier from "./Pinkyfier";
    import Fattyfier from "./Fattyfier";

    let pinkyfier = new Pinkyfier("text"),
        fattyfier = new Fattyfier("text");

    pinkyfier.pink();

    document.getElementById("fat").onclick = function () {
        fattyfier.fat();
    }

[js/main.js][27]

В ES6 `export` и `import` — части языка и браузеры (из будущего) их понимают, и
могут подгружать модули без участия библиотек вроде RequireJS или сборщиков
вроде Browserify. Вы можете просто подключить файл `main.js` через тег script.

Вы можете [посмотреть на результат работы вживую тут][28]. Хотя нет, не можете,
ну разве только что вы не гость из будущего. В настоящем мой браузер жалуется:
«модули пока не реализованы».

Но webpack может это исправить, и далее в этой статье мы увидим, как.

## Прошу любить и жаловать, webpack

Итак, давайте уже наконец перейдём к webpack. Начнём с примера с AMD/RequireJS
и «вебпакифицируем» его.
webpack поддерживает модули AMD прямо из коробки, так что мы просто используем
модули AMD из примера выше: [Pinkyfier.js][29], [Fattyfier.js][30]
и [main.js][31].

webpack в использовании схож с Browserify, вы устанавливаете утилиту на node
через npm и пользуетесь ей, чтобы собрать один или несколько бандлов.

Установка webpack:

    npm install -g webpack

Чтобы настроить webpack, создайте файл настроек под именем `webpack.config.js`.
В этом простом варианте там будет находиться только код настроек, который
указывает webpack путь, где он должен искать модули (`modulesDirectories`), где
у приложения точка входа (`entry`) и как назвать и куда положить файл бандла на
выходе (`output`).

    var webpack = require("webpack");

    module.exports = {
        entry: "./main",
        resolve: {
            modulesDirectories: [
                "."
            ]
        },
        output: {
            publicPath: "js/",
            filename: "bundle.js"
        }
    };

[js/webpack.config.js][32]

После того, как мы создали в нужном месте файл, мы можем просто набрать
«webpack» в командной строке:

    webpack

Это создаст два файла, `bundle.js` и `1.bundle.js`.

«А почему два?» — спросите вы. Что ж, это из-за того, что мы используем модули
AMD, которые подгружаются асинхронно. `bundle.js` содержит код из `main.js`, а
в `1.bundle.js` — код из `Pinkyfier.js` и `Fattyfier.js`, который грузится
асинхронно. Если бы у нас были модули CommonJS, которые всегда подгружаются
синхронно, на выходе был бы всего один файл. Вы уже начинаете понимать,
насколько дьявольски умная эта утилита?

Далее мы подключаем `bundle.js` через тег script в наш [код HTML][33]
(подключать второй бандл не нужно).

[И смотрим на то, что получилось][34].

## Интересный момент

Это всё хорошо, но в чём тут преимущество по сравнению с использованием
RequreJS? С RequireJS идёт в комплекте оптимизатор (`r.js`), который тоже
может создавать бандлы…

Тут начинается самое интересное: помните, что я говорил про Browserify, что он
позволяет использовать модули npm и на бэкенде, и на фронтенде, а преимущество
RequireJS в асинхронной загрузке? Так вот, с webpack вы можете взять лучшее от
обоих миров. webpack поддерживает *и* модули AMD *и* модули CommonJS
*одновременно*.

Попробуйте сами, вы можете заменить Pinkifier.js в формате AMD на
[версию с CommonJS][35]. Запустите команду webpack ещё раз.

[Посмотрите на пример и результат][36] — всё работает точно так же.

Заметьте, не требуется никакой дополнительной настройки, не нужно говорить
webpack: «Эй, я использую оба формата модулей». webpack достаточно умён, чтобы
понять это самостоятельно.

## Назад в будущее

Вернёмся к нашему примеру с ES6, который я, к сожалению, не могу запустить на
своём браузере образца апреля 2015 года. Может ли нам помочь webpack? Легко!
В webpack есть понятие загрузчиков, дополнительных модулей, которые добавляются
в конфигурацию, чтобы загружать файлы, соответствующие какому-то признаку.
Есть целая огромная куча загрузчиков для самых различных вещей, не только для
JavaScript, а даже для CSS или изображений.

Мы настроим загрузчик Babel для всех файлов JavaScript, добавив такой блок в
`webpack.config.js`:

    module: {
            loaders: [
                {
                    test: /\.js$/,
                    loader: "babel-loader"
                }
            ]
        }

[js/webpack.config.js][37]

Теперь webpack будет загружать все файлы js через загрузчик, который использует
[Babel][38] чтобы транскомпилировать код на ES6 в код старой версии JavaScript,
который смогут понять нынешние браузеры.

Загрузчик Babel не является частью webpack по умолчанию, это лишь дополнение,
поэтому придётся установить его в проект через npm.

Я добавил [package.json][39] в свой код, так что я могу просто запустить
`npm install` в командной строке (из папки проекта), чтобы этого добиться.

После запуска webpack мы получим единственный бандл (ничего асинхронного тут
не происходит), который можно подключить через тег script.

[Посмотрите на этот пример вживую][40].

## Хорошенькая, чистенькая асинхронность

Вернёмся к примеру с AMD + webpack, я писал ранее, что webpack автоматически
создаёт несколько бандлов когда ему попадаются модули AMD. Это приятно, но зачем
это может пригодиться? Мы загружаем орозовитель и ожирнитель асинхронно, но
вся эта асинхронная загрузка происходит сразу после загрузки страницы. Не
очень-то большой выигрыш в скорости загрузки страницы по сравнению с одним
большим бандлом со всем кодом, подключённым через тег script перед закрывающим
тегом body, приёмом, который многими признан полезным.

Но действительно ли нам нужен модуль `Fattifier`? Вообще-то, он нам нужен только
когда пользователь щёлкнет по кнопке «ожирнить». Разве не было бы круто
использовать асинхронную загрузку как преимущество и загружать код ожирнителя
только когда он нам нужен, т.е., когда кнопка была нажата?

С webpack этого очень легко добиться. Мы изменим код `main.js`, как-то так:

    var Pinkyfier = require("./Pinkyfier"),
        pinkyfier = new Pinkyfier("text");

    pinkyfier.pink();

    document.getElementById("fat").onclick = function () {
        require(["./Fattyfier"], function (Fattyfier) {
            var fattyfier = new Fattyfier("text");
            fattyfier.fat();
        });
    }

[js/main.js][41]

Что тут происходит? Я смешал модули в стилях CommonJS и AMD в одном файле:
`require` в стиле CommonJS на строке 1 отвечает за загрузку модуля `Pinkyfier`
синхронно. `require` в стиле AMD на строке 7 загружает модуль `Fattyfier`
асинхронно.

Запустив команду webpack, я получаю на выходе файлы: `bundle.js` с кодом
из `main.js` и `Pinkyfier.js` и `1.bundle.js`, с кодом из `Fattyfier.js`.

Когда я открою страницу в браузере, загрузится только `bundle.js`. И только
после того, как я нажму на кнопку, подгрузится другой бандл.

[Посмотрите на этот пример][42].

Это хороший приём для уменьшения времени загрузки страницы и увеличения скорости
работы сайта. Мы используем его на [странице поиска mobile.de][43], когда я
щёлкаю по кнопке «расширенный поиск» сверху слева, появляется большая старая
форма для поиска. Весь код JavaScript для этой формы, даже код шаблона, который
рендерится на клиентской стороне [шаблонизатором Soy][44], загружается
асинхронно, только после нажатия кнопки.

## Заключение

Итак, теперь у вас должно было появиться хорошее понимание того, что такое
webpack, и как он работает.

Очевидно, это только начало.

Как я уже упоминал, к вашим услугам имеется огромное количество
модулей-загрузчиков и плагинов для различных задач вроде [минификации][45] или
компиляции [SASS][46] или [Less][47] в CSS.
Вы можете сказать webpack генерировать карты кода для более удобной отладки
JavaScript в браузере.
Вы можете запустить webpack как [сервер для разработки][48], и он будет
отслеживать изменения в коде и сразу же обновлять сгенерированные файлы.
Вы можете интегрировать webpack в [Grunt][49] или [Gulp][50] и генерировать
[хэши содержимого][51] (также известные как отпечатки пальцев) для оптимизации
кэширования в браузере.
Вы можете использовать различные бандлы для различных страниц вашего приложения
и позволить webpack автоматически организовать общие модули для этих страниц
в общие бандлы.
Вы можете даже написать [собственный модуль-загрузчик][52], что, кстати говоря,
очень легко.

Развлекайтесь, исследуя возможности!

 [2]: img/webpack.png "Логотип webpack"
 [3]: http://webpack.github.io/
 [4]: http://requirejs.org/
 [5]: http://browserify.org/
 [6]: https://github.com/pahund/webpack-talk
 [7]: https://github.com/pahund/webpack-talk/blob/master/01_global-vars/js/Pinkyfier.js
 [8]: https://github.com/pahund/webpack-talk/blob/master/01_global-vars/js/Fattyfier.js
 [9]: https://github.com/pahund/webpack-talk/blob/master/01_global-vars/js/main.js
 [10]: https://github.com/pahund/webpack-talk/blob/master/01_global-vars/index.html
 [11]: http://pahund.github.io/webpack-talk/01_global-vars/index.html
 [12]: https://github.com/pahund/webpack-talk/blob/master/02_amd/js/Pinkyfier.js
 [13]: https://github.com/pahund/webpack-talk/blob/master/02_amd/js/Fattyfier.js
 [14]: https://github.com/pahund/webpack-talk/blob/master/02_amd/js/main.js
 [15]: https://github.com/pahund/webpack-talk/blob/master/02_amd/index.html#L20
 [16]: http://pahund.github.io/webpack-talk/02_amd/index.html
 [17]: https://nodejs.org/
 [18]: https://iojs.org/
 [19]: http://www.commonjs.org/
 [20]: https://github.com/pahund/webpack-talk/blob/master/03_commonjs/js/Pinkyfier.js
 [21]: https://github.com/pahund/webpack-talk/blob/master/03_commonjs/js/Fattyfier.js
 [22]: https://github.com/pahund/webpack-talk/blob/master/03_commonjs/js/main.js
 [23]: https://github.com/pahund/webpack-talk/blob/master/03_commonjs/index.html#L18
 [24]: http://pahund.github.io/webpack-talk/03_commonjs/index.html
 [25]: https://github.com/pahund/webpack-talk/blob/master/04_es6/js/Pinkyfier.js
 [26]: https://github.com/pahund/webpack-talk/blob/master/04_es6/js/Fattyfier.js
 [27]: https://github.com/pahund/webpack-talk/blob/master/04_es6/js/main.js
 [28]: http://pahund.github.io/webpack-talk/04_es6/index.html
 [29]: https://github.com/pahund/webpack-talk/blob/master/05_webpack_amd/js/Pinkyfier.js
 [30]: https://github.com/pahund/webpack-talk/blob/master/05_webpack_amd/js/Fattyfier.js
 [31]: https://github.com/pahund/webpack-talk/blob/master/05_webpack_amd/js/main.js
 [32]: https://github.com/pahund/webpack-talk/blob/master/05_webpack_amd/js/webpack.config.js
 [33]: https://github.com/pahund/webpack-talk/blob/master/05_webpack_amd/index.html
 [34]: http://pahund.github.io/webpack-talk/05_webpack_amd/
 [35]: https://github.com/pahund/webpack-talk/blob/master/06_webpack_amd-commonjs/js/Pinkyfier.js
 [36]: http://pahund.github.io/webpack-talk/06_webpack_amd-commonjs/
 [37]: https://github.com/pahund/webpack-talk/blob/master/07_webpack-es6/js/webpack.config.js#L14
 [38]: https://babeljs.io/
 [39]: https://github.com/pahund/webpack-talk/blob/master/07_webpack-es6/js/package.json
 [40]: http://pahund.github.io/webpack-talk/07_webpack-es6/
 [41]: https://github.com/pahund/webpack-talk/blob/master/08_webpack_multiple-bundles/js/main.js
 [42]: http://pahund.github.io/webpack-talk/08_webpack_multiple-bundles/
 [43]: http://suchen.mobile.de/fahrzeuge/search.html?isSearchRequest=true&scopeId=C&makeModelVariant1.makeId=&makeModelVariant1.modelDescription=&makeModelVariantExclusions%5B0%5D.makeId=&minFirstRegistrationDate=&maxFirstRegistrationDate=&minMileage=&maxMileage=&minPrice=&maxPrice=&minPowerAsArray=&maxPowerAsArray=&maxPowerAsArray=PS&minPowerAsArray=PS&minCubicCapacity=&maxCubicCapacity=&ambitCountry=&zipcode=&minSeats=&maxSeats=&doorCount=&climatisation=&airbag=&daysAfterCreation=&adLimitation=&export=&vatable=&maxConsumptionCombined=&emissionClass=&emissionsSticker=&damageUnrepaired=NO_DAMAGE_UNREPAIRED&numberOfPreviousOwners=&minHu=&usedCarSeals=&lang=en
 [44]: https://developers.google.com/closure/templates/
 [45]: http://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin
 [46]: https://github.com/jtangelder/sass-loader
 [47]: https://github.com/webpack/less-loader
 [48]: http://webpack.github.io/docs/webpack-dev-server.html
 [49]: https://github.com/webpack/grunt-webpack
 [50]: http://webpack.github.io/docs/usage-with-gulp.html
 [51]: http://webpack.github.io/docs/long-term-caching.html
 [52]: http://webpack.github.io/docs/how-to-write-a-loader.html

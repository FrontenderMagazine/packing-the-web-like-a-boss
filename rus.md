# Пакуем для веба как босс

[webpack][3] — это крутая новая утилита для сборки бандлов и оптимизации модулей
JavaScript и других ресурсов для фронтенда.
Если вы уже пользовались [RequireJS][4] или [Browserify][5], то велики шансы,
что вы полюбите webpack так же, как и я.
Эта статья — подробная инструкция для вас.

Чтобы продемонстрировать магию webpack, я в этой статье буду использовать
в качестве примеров очень простой, даже неестественно примитивный, код.
[Он хранится на GitHub, вот тут][6].

Представьте, что вам что-то взбрело в голову, и вам захотелось написать код на
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

Вы бы поключили эти три файла JS в документ HTML при помощи тегов script, вот
так:

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="utf-8">
        </head>
        <body>
            <p id="text">
                Родился на улице Герцена, в гастрономе № 22. Известный экономист,
                по призванию своему — библиотекарь. В народе — колхозник.
                В магазине — продавец. В экономике, так сказать, необходим.
            </p>
            <button id="fat" type="button">Сделать полужирным</button>
    
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
вместо всего трёх?
Что, если бы код в этих файлах зависел от кода в других файлах более сложным
образом, так что изменение порядка тегов script в HTML разломало бы страницу?

Если бы вы продолжали делать так, в итоге получилась бы здоровенная неуклюжая
каша из кода. Вам нужна система модулей.

## Способ получше: модули AMD

Прежде чем рассматривать магию webpack и на то, как он всё делает лучше, давайте
взглянем на самые известные системы модулей: AMD, CommonJS и, в ближайшем
будущем, ES6.

Самая распространённая релизация модулей AMD — это [RequireJS][4].
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

Хотя она используется в основном для модулей npm и приложений, выполняющихся
на веб-серверах, при помощи [Browserify][5] ей можно также пользоваться в коде
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

Как вы можете заметить, единственное отличие от вышеупомятнутой «наивной»
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
ну разве только что вы не из будущего. В настоящем мой браузер жалуется:
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
нсколько дьявольски умная эта утилита?

Далее мы подключаем `bundle.js` через тег script в наш [код HTML][33]
(подключать второй бандл не нужно).

[И смотрим на то, что получилось][34].

## The Fun Part

That's all very well, but what's the advantage over just using RequireJS?
RequireJS comes with an optimizer
(*r.js*) that can bundle files just as well...

That's where the fun part begins: remember what I said about Browserify, how it
allows you to use npm modules both in the backend and frontend, while the 
advantage of RequireJS is asynchronous loading? Well, with webpack, you can have
the best of both worlds. webpack can handle both AMD modules*and* CommonJS
modules*at the same time*.

Try it out — you can simply swap the AMD Pinkifier.js with the 
[CommonJS version][35]. Run the webpack command again.

[Look at the example and see it in action][36] — it works just the same way.

Note that there's no additional configuration neccessary, you don't need to
tell webpack: "Hey, I'm using both styles of module." webpack is smart enough to
figure this out by itself.

## Back to the Future

Let's go back to our ES6 example, which, sadly, I can't run in my April-2015-
browser. How can webpack help? Easily. webpack has the concept of loaders, 
additional modules that you add to the webpack config to load files with that 
match a certain pattern. There are many, many loaders out there for all kinds of
things, not only loading JavaScript, but even CSS or images.

We'll configure the Babel loader for all JavaScript files by adding this block
to our*webpack.config.js*:

    module: {
            loaders: [
                {
                    test: /\.js$/,
                    loader: "babel-loader"
                }
            ]
        }
    

[js/webpack.config.js][37]

This will run all the JS files through a loader that uses [Babel][38] to
transpile the ES6 code to plain old JavaScript code that current browsers 
understand.

Since the Babel loader is not a part of webpack by default but rather an addon
, you have to install it to your project with npm.

I added a [package.json][39] to my code example so I can just run "npm install
" on the command line (in the js directory) to do that.

After running the webpack command, we get a single bundle (nothing asynchronous
going on here) that we include in our script tag.

[Look at the example to see it in action][40].

## Good, Clean, Asynchronous Fun

Back to the AMD+webpack example — I wrote earlier that webpack automatically
creates multiple bundles when it encounters AMD modules. That's nice, but is it 
really useful? We are loading the fattyfier and pinkyfier asynchronously, but 
the asynchronous loading happens immediately after the page was loaded. There's 
not a very big sitespeed advantage over just having one big bundle file with 
everything in it, if we include the script tag before the closing body tag, 
which is commonly accepted best practice.

But do we really need the fattyfier module? Actually, we only need it when the
user clicks the "make it fat" button. Wouldn't it be cool if we could use 
asynchronous loading to our advantage and only load the farryfier code when we 
actually need it, i.e. when the button is clicked?

With webpack, this is very easy to do. We change our *main.js* code like so:</
section
><section class="text">

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

What's going on here? I'm actually mixing up CommonJS module style and AMD
module style in the same file: The CommonJS style require statement in line 1 is
for loading the pinkyfier module synchronously. The AMD style require statement 
in line 7 is for loading the fattyfier module asynchronously.

When I run the webpack command, I get a *bundle.js* file that contains the code
from*main.js* and *Pinkyfier.js* and a *1.bundle.js* file that contains the
code from*Fattyfier.js*.

When I load my page in the browser, only *bundle.js* is loaded. Only after I
click the button, the other bundle is loaded.

[Look at the example to see it in action][42].

This is great for reducing page loading time and increasing site speed. We use
this for[mobile.de's search page][43] — when I click on the "Detailed Search
" button on the upper left, a big old search form box pops up. All the 
JavaScript code that drives this form, even the template code, which is a client
-side rendered[Soy template][44], is loaded asynchronously, only after the
button has been clicked.

## Conclusion

So that's about it, you should now have a good idea about what webpack is all
about and how it works.

Obviousy, this is just the beginning.

As I wrote above, there's a vast selection of loader modules and plugins at
your disposable to do things like[minification][45] or compilation of 
[SASS][46] or [Less][47] to CSS. You can have webpack generate sitemaps to
easily debug your JavaScript in your browser. You can have webpack run a
[dev server][48] that listens for changes to your code and instantly updates
the generated files. You can integrate webpack in your[Grunt][49] or [Gulp][50]
[chunk hashes][51] (a.k.a. fingerprints) to optimize browser caching. You can
have different bundles for different pages of your application and let webpack 
organize the libs shared by these pages in shared bundles automatically. You can
[write your own loader modules][52], which is actually quite easy.

Have fun exploring the possibilities!</section>

** [javascript][53], [es6][54], [webpack][55], [frontend][56], [webdev][57] 

Share this post on: [Twitter][58] , [Facebook][59] , [G+][60] or [LinkedIn][61]

[ ![monster][63]

![pacman][64]

![we are hiring][65]][65] </article>

 [1]: http://www.technology-ebay.de/
 [2]: img/webpack.png
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
 [53]: http://www.technology-ebay.de/filter/add?facility=TAG&value=javascript
 [54]: http://www.technology-ebay.de/filter/add?facility=TAG&value=es6
 [55]: http://www.technology-ebay.de/filter/add?facility=TAG&value=webpack
 [56]: http://www.technology-ebay.de/filter/add?facility=TAG&value=frontend
 [57]: http://www.technology-ebay.de/filter/add?facility=TAG&value=webdev

 [58]: http://twitter.com/home?status=http%3A%2F%2Fwww.technology-ebay.de%2Fthe-teams%2Fmobile-de%2Fblog%2Fpacking-the-web-like-a-boss.html

 [59]: http://www.facebook.com/sharer/sharer.php?s=100&p%5Burl%5D=http%3A%2F%2Fwww.technology-ebay.de%2Fthe-teams%2Fmobile-de%2Fblog%2Fpacking-the-web-like-a-boss.html&p%5Btitle%5D=Packing+the+Web+Like+a+Boss

 [60]: https://plus.google.com/share?url=http%3A%2F%2Fwww.technology-ebay.de%2Fthe-teams%2Fmobile-de%2Fblog%2Fpacking-the-web-like-a-boss.html

 [61]: http://www.linkedin.com/shareArticle?mini=true&url=http%3A%2F%2Fwww.technology-ebay.de%2Fthe-teams%2Fmobile-de%2Fblog%2Fpacking-the-web-like-a-boss.html&title=Packing+the+Web+Like+a+Boss
 []: http://jobs.ebaycareers.com/articles/english
 [63]: img/PacManMonster.gif
 [64]: img/pacman.gif
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

    function Pinkyfier(id) {
        this.element = document.getElementById(id);
    }
    
    Pinkyfier.prototype.pink = function () {
        this.element.style.backgroundColor = "mistyrose";
        this.element.style.color = "hotpink";
    }

[js/Pinkyfier.js][7]

    function Fattyfier(id) {
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
                Nonne vides quid sit? Tu es ... Jesse me respice. Tu ... blowfish sunt. A blowfish! Cogitare. Statura
                pusillus, nec sapientium panem, nec artificum. Sed predators facile prædam blowfish secretum telum non se
                habet. Non ille? Quid faciam blowfish, Isai. Blowfish quid faciat? In blowfish inflat, purus?
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
способом, так что изменение порядка тегов script в HTML разломало бы страницу?

Если бы вы продолжали делать так, в итоге получилась бы здоровенная неуклюжая
каша из кода. Вам нужна система модулей.

## Способ получше: модули AMD

Before we look at the magic of webpack and how it makes everything better, let'
s do a quick review of the most commonly used module systems out there: AMD, 
CommonJS and, in the near future, ES6.

The most widely used implementation of AMD modules is [RequireJS][4]. Let's
have a look how we can do our pink fat text page using RequireJS.

Instead of throwing our *Pinkyfier* and *Fattyfier* classes out there in the
global scope, we wrap them in nice*define* statements:</section><section class
="text
">

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

In your *main.js*, you can now require those two modules you defined like so:</
section
><section class="text">

    require([ "Fattyfier", "Pinkyfier" ], function (Fattyfier, Pinkyfier) {
    
        var pinkyfier = new Pinkyfier("text"),
            fattyfier = new Fattyfier("text");
    
        pinkyfier.pink();
    
        document.getElementById("fat").onclick = function () {
            fattyfier.fat();
        }
    });
    

[js/main.js][14]

In your HTML code, instead of the three script tags in the above example, you
write just one script tag that loads RequireJS, with a data attribute that 
points it to your application entry point*main.js*:</section><section class="
text
">

    <script data-main="js/main.js" src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.17/require.min.js"></script>
    

[index.html][15]

You can easily build much more complex applications this way, and the nice part
about it is that the modules are loaded asynchronously — only loading the 
RequireJS lib blocks the loading of the page, but none of your JavaScript code.

[Look a the example here to see it in action][16].

## The Node.js Way: CommonJS Modules

With the advent of server-side JavaScript using [Node.js][17] or [io.js][18],
another system of JavaScript modules became popular:[CommonJS][19].

While this is primarily used for npm modules and node applications running on
web servers, you can use[Browserify][5] to use it for client-side code, as well
. This is very cool, because it puts a wealth of npm modules at your disposal 
and allows you to use the same code on the backend and on the frontend.

Here's our lil' example in a CommonJS version:</section><section class="text
">

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

As you can see, the only difference to the "naïve" examples at the beginning
are the last lines on each file, which state which parts of the code should be 
exposed to clients using the modules — the classes*Pinkyfier* and *Fattyfier*.
In that respect, CommonJS modules are simpler and less intrusive than RequireJS.

To use the modules, you add *require* statements to your *main.js*:</section><
section class="text
">

    var Pinkyfier = require("./Pinkyfier"),
        Fattyfier = require("./Fattyfier"),
    
        pinkyfier = new Pinkyfier("text"),
        fattyfier = new Fattyfier("text");
    
    pinkyfier.pink();
    
    document.getElementById("fat").onclick = function () {
        fattyfier.fat();
    }
    

[js/main.js][22]

How do you use this in the browser? Browserify works differently than RequireJS
. Instead of loading a library that does the trick, you use a node tool to 
create a JavaScript bundle file that contains all of your code.

To do so, install Browserify with npm (I'm assuming you have node and npm
installed
):

    npm install -g browserify

You can then go to the directory that contains your JS files and create a
bundle with this command:

    browserify main.js > bundle.js
    

You can then simply include this script bundle in your HTML code with a script
tag:

    <script src="js/bundle.js"></script>
    

[index.html][23]

[Look at the example to see it in action here][24].

## The Future: ES6 Modules

At the time of writing this, the new JavaScript standard ECMAScript 6 is just
around the corner, due to be released in June 2015.

ES6 brings along native support for JavaScript modules, rendering AMD and
CommonJS obsolete.

Let's have a look how to pinkyfy and fatten up our lorem ipsum text with ES6:</
section
><section class="text">

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

Whoa, what happened here? Well, in ES6, as you can see, classes are defined in
a very different way than in the trusty old JavaScript we've grown to love. But 
that's actually not the point here, the interesting part are the last lines, 
with the export statements that expose the classes defined in the modules to the
client modules. Looks similar to the CommonJS example, doesn't it?

Here's how to use modules in ES6:

    import Pinkyfier from "./Pinkyfier";
    import Fattyfier from "./Fattyfier";
    
    let pinkyfier = new Pinkyfier("text"),
        fattyfier = new Fattyfier("text");
    
    pinkyfier.pink();
    
    document.getElementById("fat").onclick = function () {
        fattyfier.fat();
    }
    

[js/main.js][27]

In ES6, *export* and *import* are part of the language and understood by the (
future) browser without the help of a library like RequireJS or a build tool 
like Browserify, so you can simply include the main.js file in your script tag.

You can [look at the example and see it in action here][28]. Or no, you can't,
unless you're from the future. Currently, my browser still complains: "modules 
are not implemented yet
".

But webpack can fix that, as we'll see later in this article.

## Enter webpack

So, finally, let's get started with webpack. We'll start with the AMD/RequireJS
example and "webpackify" it. webpack supports AMD modules without further 
fiddling, so we can simply use the AMD modules from the example above:
[Pinkyfier.js][29], [Fattyfier.js][30] and [main.js][31].

Using webpack is similar to using Browserify: you install a node tool with npm
and use it to compile one or more bundles.

Install webpack:

    npm install -g webpack

To configure webpack, you create a configuration file named webpack.config.js.
In this simple version, it only contains config code that tells webpack the 
paths where to find its stuff
(*modulesDirectories*), what the application's main file is (*entry*) and what
bundle file to create where
(*output*).

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

With the configuration file in place, we can simply enter "webpack" on the
command line:

    webpack

This creates two files, *bundle.js* and *1.bundle.js*.

"Why two?" you ask. Well, because we're using AMD modules, which are loaded
asynchronously.*bundle.js* contains the code from *main.js*, *1.bundle.js*
contains the code from*Pinkyfier.js* and *Fattyfier.js* and is loaded
asynchronously. If we were using CommonJS-style modules, which are always loaded
synchronously, we would get only one bundle file. This gives you a first glimpse
at how fiendishly clever this tool is.

We include *bundle.js* in the script tag in our [HTML code][33] (no need to
include the other bundle
).

[Look at the example here to see it in action][34].

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
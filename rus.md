# Использование SVG-изображений

SVG - формат векторной графики. Буквально, его название значит
**Масштабируемая Векторная Графика** (Scalable Vector Graphics). Попросту
говоря, это то, с чем вы работаете в Adobe Illustrator. SVG можно легко
использовать в вебе, но сперва нужно во многом разобраться.

## Зачем вообще нужен SVG?

* Небольшие размеры файлов, отличное сжатие;
* Масштабирование до любого размера, без потери качества (разве что, при совсем
маленьких размерах);
* Хорошо выглядит на ретине;
* Широкие возможности, которые предоставляют фильтры и интерактивность.

## Создадим изображение SVG, с которым будем работать дальше

Создайте произвольный рисунок в Adobe Illustrator. Вот, например, птица киви на
овале.

![Птица киви][Птица киви, которая стоит на овале]

Обратите внимание, что изображение кадрируется чётко по краям изображения.
Холст в SVG играет не меньшую роль, чем в PNG или JPG.

Adobe Illustrator умеет сохранять в SVG.

![Сохранение][Сохраняем файл с расширением SVG]

При сохранении появится ещё одно диалоговое окно с настройками. Честно говоря, я
не очень в них разбираюсь. Существует целая инструкция по [Профилям SVG][1].
Меня вполне устраивает SVG 1.1.

![Настройки][Настройки сохранения файла с расширением SVG]

Здесь стоит отметить, что у вас есть возможность нажать OK и сохранить файл, или
же нажать кнопку «SVG код...» ("SVG Code..."), которая откроет окно TextEdit (по
крайней мере на Mac) с SVG-кодом.

![SVG-код][окно TextEdit с SVG-кодом]

Оба варианта могут пригодиться.

## Добавляем SVG на страницу с помощью тега `<img>`

Если сохранить изображение SVG в файл, его можно вставить с помощью тега `<img>`.

    <img src="kiwi.svg" alt="Киви на овале">

В Illustrator рабочая область была размером 612px ✕ 502px.

![Рабочая область][Размеры рабочей области в Illustrator]

Именно такие размеры будут у изображения на странице, если их не указать
специально. Его размеры можно изменить, задав атрибуты `width` или `height` для
`img`, опять же, точно так же как для PNG или JPG. Вот [пример][2]:

<pre class="codepen" data-height="300" data-type="result" data-href="lCEux" data-user="chriscoyier" data-safe="true"><code></code><a href="http://codepen.io/chriscoyier/pen/lCEux">Посмотрите на этот пример!</a></pre>

### Поддержка браузерами

SVG по-разному [поддерживается браузерами][3]. Он работает везде, кроме IE до 8
версии и браузерах на Android до версии 2.3.

Если вы хотели бы использовать SVG, но проект поддерживает браузеры, которые
не могут вставлять его через `img`, есть разные варианты. Я описал некоторые
приемы в [нескольких][4] [своих][5] [мастер-классах][6].

Один из вариантов: проверка поддержки через Modernizr и замена `src` для
изображения:

    if (!Modernizr.svg) {
      $(".logo img").attr("src", "images/logo.png");
    }

Дэвид Бушел (David Bushell) предложил очень простой [альтернативный вариант][7],
если вы не имеете ничего против JavaScript в разметке:

    <img src="image.svg" onerror="this.onerror=null; this.src='image.png'">

Еще можно использовать [SVGeezy][8]. Далее мы рассмотрим другие способы деградации.

## Добавляем SVG через `background-image`

Использовать SVG в качестве фона c помощью CSS-свойства `background-image` так
же просто, как и вставка с помощью тега `img`.

    <a href="/" class="logo">
      Kiwi Corp
    </a>

    .logo {
      display: block;
      text-indent: -9999px;
      width: 100px;
      height: 82px;
      background: url(kiwi.svg);
      background-size: 100px 82px;
    }

Обратите внимание, что для селектора `.logo` задан размер `background-size`. Это
необходимо, иначе будет видна только верхняя левая часть изображения SVG, у
которого исходный размер намного больше. Эти размеры прописаны с учётом
соотношения сторон изображения в оригинале. Можно также использовать для
`background-size` значение `contain` чтобы убедиться в том, что изображение
поместится в родительский контейнер, если вам не известно какого размера оно
должно быть.

### Поддержка браузерами

Вставка SVG через свойство `background-image` по-разному [поддерживается
браузерами][9], но в общем дела обстоят так же, как и с `img`. Проблемой являются
IE до 8 версии и браузеры на Android до версии 2.3.

В этом случае нам может помочь Modernizr, даже более эффективно, чем при
использовании `img`. Если заменить `background-image` на изображение,
формат которого поддерживается, на сервер будет отправлен один HTTP-запрос,
а не два. Modernizr добавляет класс "no-svg" для html-элемента,
если SVG не поддерживается:

    .main-header {
      background: url(logo.svg) no-repeat top left;
      background-size: contain;
    }

    .no-svg .main-header {
        background-image: url(logo.png);
    }

## Общая проблема при использовании `<img>` и `background-image`...

Проблема состоит в том, что вы не можете управлять внутренностями SVG с помощью
CSS так, как сможете при использовании двух приёмов описанных ниже. Читайте дальше!

## Добавляем SVG непосредственно в документ

Помните, как при необходимости можно получить SVG-код прямо при сохранении
изображения в Illustrator? (Также можно просто открыть SVG-файл в текстовом
редакторе и скопировать его код.) Этот код можно вставить прямо в HTML-документ,
и SVG-изображение будет отображаться точно так же, как если бы его вставили с
помощью тега `img`.

    <body>

       <!-- вставьте код SVG и появится изображение! -->

    </body>

Этот приём может быть полезным, так как изображение встроено прямо в документ и
для его загрузки не происходит дополнительный HTTP-запрос. У этого метода те же
преимущества, как и у использования [Data URI][10]. И недостатки у него те же,
не обольщайтесь. Среди них: вероятность получения очень тяжелого документа,
блоки SVG кода в нем и невозможность кэширования.

Если вы используете серверный язык, который позволяет получить содержимое файла
и вставить его в документ, вы, по крайней мере, сможете очистить свой документ
от блоков SVG кода. Вот так:

    <?php include("kiwi.svg"); ?>

### Сначала оптимизируем

Без сомнений, для вас не станет сюрпризом то, что SVG, полученные в Adobe
Illustrator, не самые оптимальные. Они содержат DOCTYPE, примечания
генератора, и тому подобный мусор. У SVG, в общем, и так небольшой размер, но
почему бы не уменьшить его еще больше, если есть возможность? Питер
Коллингридж (Peter Collingridge) создал [SVG Optimiser][11], инструмент для
онлайн-оптимизации SVG. Загружаете старый файл, скачиваете новый. В [своём
видео][12] Кайл Фостер заходит ещё дальше и удаляет даже переносы строки
в процессе оптимизации.

Если вы еще более суровы, вот вам [инструмент на Node JS][13], с помощью
которого можно оптимизировать изображения самостоятельно.

### Затем управляем с помощью CSS

Видите, насколько сильно теперь SVG похоже на HTML? Это потому, что они оба
не что иное, как XML (проименованные теги, в квадратных скобках, и всякая
всячина внутри). В нашем проекте есть два составляющих элемента, `<ellipse>` и
`<path>`. Можно просто открыть код и присвоить им классы, как любому другому
элементу HTML.

    <svg ...>
      <ellipse class="ground" .../>
      <path class="kiwi" .../>
    </svg>

Теперь эти отдельные элементы можно контролировать с помощью специального CSS
для SVG. Не обязательно добавлять CSS в сам SVG, его можно разместить где
угодно, даже в файле с глобальными стилями. Обратите внимание, что для элементов
SVG есть специальный набор свойств CSS. Например, нельзя использовать
`background-color`, вместо него есть `fill`. Однако кое что стандартное тоже
можно использовать, например, :hover.

    .kiwi {
      fill: #94d31b;
    }
    .kiwi:hover {
      fill: #ace63c;
    }

Даже круче того, для SVG можно использовать всякие навороченные фильтры. Например,
размытие. Прицепите к своему `<svg>` фильтр:

    <svg ...>
      ...
      <filter id="pictureFilter" >
        <feGaussianBlur stdDeviation="5" />
      </filter>
    </svg>

Потом его можно применить через CSS, когда нужно:

    .ground:hover {
      filter: url(#pictureFilter);
    }

[Пример][14], что может получиться:

<pre class="codepen" data-height="300" data-type="result" data-href="evcBu" data-user="chriscoyier" data-safe="true"><code></code><a href="http://codepen.io/chriscoyier/pen/evcBu">Check out this Pen!</a></pre>

* [Подробнее о применении фильтров для SVG][15]
* [Самый лучший список CSS-свойств для SVG, который мне удалось найти][16]
(разработанный для Opera)
* [Песочница для тестирования эффектов от применения фильтров для SVG][17]
(от Microsoft)

## Поддержка браузерами

Строчное добавление SVG по-разному [поддерживается браузерами][18], однако,
опять же, все сводится к отсутствию поддержки IE младше 8 и браузерам на Android
до версии 2.3 <a href="#note-1" id="ref-1" class="reference">1</a>.

Для этого способа вставки SVG можно использовать следующие приемы деградации:

    <svg> ... </svg>
    <div class="fallback"></div>

И снова используем Modernizr:

    .logo-fallback {
      display: none;
      /* Убедитесь, что размер соответствует размеру SVG */
    }
    .no-svg .logo-fallback {
      background-image: url(logo.png);
    }

## Добавляем SVG на страницу с помощью тега `<object>`

Если по какой-либо причине вариант со вставкой SVG непосредственно в документ
вам не нравится (он все же имеет парочку недостатков, например, кэширование
практически невозможно), можно подключить SVG-файл используя `<object>` и
сохранить возможность управлять его частями посредством CSS.

    <object type="image/svg+xml" data="kiwi.svg" class="logo">
      Kiwi Logo <!-- запасное изображение в CSS -->
    </object>

На тот случай, если это не поддерживается, реализуем деградацию используя класс,
который добавляет Modernizr:

    .no-svg .logo {
      width: 200px;
      height: 164px;
      background-image: url(kiwi.png);
    }

При таком подходе не возникают проблемы с кэшированием и, вообще, он
поддерживается браузерами *лучше* чем другие. Но если использовать внешний файл
со стилями или `<style>` встроенный в документ, CSS-навороты работать не будут,
нужно добавить элемент `<style>` в сам SVG-файл.

    <svg ...>
      <style>
        /* специальные CSS-фишки для SVG */
      </style>
      ...
    </svg>

### Внешние файлы со стилями для SVG, вставленного с помощью `<object>`

Все же есть способ использовать для SVG внешний файл со стилями, если такая
возможность необходима для проекта, кэширования или еще чего-то. По результатам
моих экспериментов он работает только для SVG-файлов встроенных посредством
`<object>`. Это нужно добавить в SVG-файл перед открывающим тегом `<svg>`:

    <?xml-stylesheet type="text/css" href="svg.css" ?>

Если попробовать добавить этот код в HTML, страница ругнётся и даже не подумает
его подгружать. Если подключить SVG-файл, в котором предложенный код заменяет
`<img>` или `background-image`, система ругаться не будет, но и работать такой
код не будет (SVG, однако, отобразится).

## Использование Data URI для SVG

SVG-файл можно уменьшить еще сильнее, если конвертировать его в Data URI. На
[Mobilefish.com][42] если для этого [онлайн-конвертер][19]. Просто скопируйте
содержимое SVG-файла и заполните форму, результат конвертирования можно будет
скопировать с текстового поля. Не забудьте удалить переносы строки в полученном
коде. Выглядеть он будет как полнейшая тарабарщина:

![Data URI][Код, полученный в результате конвертации SVG-файла в Data URI]

Его можно использовать в любом из приёмов которые мы рассмотрели (кроме вставки
`<svg>` непосредственно в документ, поскольку это попросту нелогично). Просто
скопируйте всю полученную тарабарщину вместо `[data]` в следующих примерах.

### Добавление на страницу с использованием тега `<img>`

    <img src="data:image/svg+xml;base64,[data]>

### Добавление на страницу в качестве фона с использованием CSS

    .logo {
      background: url(data:image/svg+xml;base64,[data]);
    }

### Добавление на страницу с использованием тега `<object>`

    <object type="image/svg+xml" data="data:image/svg+xml;base64,[data]>
      fallback
    </object>

И да, если если добавить `<style>` в SVG до перекодирования в base64, [он будет
работать][20] при добавлении на страницу с использованием тега `<object>`!

И для по-настоящему суровых разработчиков, компания Filament group предлагает
инструмент [grunticon][21], который автоматизирует этот процесс.

Консольные штучки для перекодирования SVG в base64:

<blockquote class="twitter-tweet"><p>@<a href="https://twitter.com/chriscoyier">chriscoyier</a> @<a href="https://twitter.com/hkfoster">hkfoster</a> maybe you could take a shortcut with &gt;&gt;&gt; echo -n `cat logo.svg` | base64 | pbcopy</p>&mdash; Benny Schudel (@bennyschudel) <a href="https://twitter.com/bennyschudel/status/307963605998006273">March 2, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Или же [Матиас Биненс (Mathias Bynens) предлагает свои приёмы][22]:

> Используйте `openssl base64 < path/to/file.png | tr -d '\n' | pbcopy` или
`cat path/to/file.png | openssl base64 | tr -d '\n' | pbcopy` чтобы пропустить
запись в файл и просто скопировать выходные данные в кодировке base64 в буфер
без переносов строки.

## Материалы для дальнейшего чтения

* Дэвид Бушел: [Букварь фронтендера по SVG-хакерству][23]
* Дэвид Бушел: [Независимость от разрешения с SVG][24]
* [MDN про SVG][25]
* Поддержка браузеров для [всяких всячин повязанных с SVG][26].
* Питер Гестон (Peter Gasston): [Создание лучших SVG-спрайтов при помощи
анкерных идентификаторов][27]
* simuari: [SVG стеки][28]
* [SVG.js][29] - «Упрощённая библиотека для манипуляции и анимации SVG»
* В Emmet есть [отличный способ][30] генерировать data URI для SVG прямо в
редакторе кода
* В Compass также [есть вспомогательное средство][31] для data URI
* Adobe: [Стилизация SVG][32]
* Эндрю Дж. Бейкер (Andrew J. Baker): [Приручаем зверя SVG][33]
* Альтернатива Illustrator: [Inkscape][34], [Sketch][35]
* Кристер Кари (Krister Kari): [Использование SVG-изображений для мобильных
браузеров][36]

Нельзя не упомянуть видео Кайла Фостера «[Последовательность оптимизации SVG][37]»:

<iframe width="480" height="360" src="http://www.youtube.com/embed/iVzW3XuOm7E?rel=0" frameborder="0" allowfullscreen></iframe>

... взгляните также на [это видео][38] + [слайды][39].

---

### Примечания

<a href="#ref-1" id="note-1" class="note">1.</a> Говоря о браузере Android 2.3,
[вот][40]. Но если вам никак не обойтись без поддержки родного браузера, [вот][41].

[1]: http://www.w3.org/TR/SVGMobile/
[2]: http://codepen.io/chriscoyier/pen/evcBu
[3]: http://caniuse.com/#feat=svg-img
[4]: http://css-tricks.com/workshop-notes-webstock-13/
[5]: http://css-tricks.com/workshop-notes-from-incontrol-hawaii/
[6]: http://css-tricks.com/deseret-digital-workshop/
[7]: http://dbushell.com/2013/02/04/a-primer-to-front-end-svg-hacking/
[8]: http://benhowdle.im/svgeezy/
[9]: http://caniuse.com/#feat=svg-css
[10]: http://css-tricks.com/data-uris/
[11]: http://petercollingridge.appspot.com/svg_optimiser
[12]: http://www.youtube.com/watch?v=iVzW3XuOm7E&feature=youtu.be
[13]: https://github.com/svg/svgo
[14]: http://codepen.io/chriscoyier/pen/evcBu
[15]: https://developer.mozilla.org/en-US/docs/Applying_SVG_effects_to_HTML_content
[16]: http://www.opera.com/docs/specs/presto25/svg/cssproperties/
[17]: http://ie.microsoft.com/testdrive/graphics/hands-on-css3/hands-on_svg-filter-effects.htm
[18]: http://caniuse.com/#feat=svg-html5
[19]: http://www.mobilefish.com/services/base64/base64.php
[20]: http://codepen.io/chriscoyier/pen/ioCjk
[21]: https://github.com/filamentgroup/grunticon
[22]: http://superuser.com/questions/120796/os-x-base64-encode-via-command-line#comment280484_120815
[23]: http://dbushell.com/2013/02/04/a-primer-to-front-end-svg-hacking/
[24]: http://coding.smashingmagazine.com/2012/01/16/resolution-independence-with-svg/
[25]: https://developer.mozilla.org/en-US/docs/SVG
[26]: http://caniuse.com/#search=svg
[27]: http://www.broken-links.com/2012/08/14/better-svg-sprites-with-fragment-identifiers/
[28]: http://simurai.com/post/20251013889/svg-stacks
[29]: http://svgjs.com/
[30]: http://docs.emmet.io/actions/base64/
[31]: http://compass-style.org/reference/compass/helpers/inline-data/
[32]: http://blogs.adobe.com/webplatform/2013/01/08/svg-styling/
[33]: http://buildnewgames.com/taming-the-svg-beast/
[34]: http://inkscape.org/
[35]: http://www.bohemiancoding.com/sketch/#4
[36]: http://kristerkari.github.com/adventures-in-webkit-land/blog/2013/03/08/dealing-with-svg-images-in-mobile-browsers/
[37]: http://www.youtube.com/watch?v=iVzW3XuOm7E&feature=youtu.be
[38]: http://www.youtube.com/watch?v=1AdX8odLC8M&feature=youtu.be
[39]: http://kylefoster.me/svg-slides/
[40]: https://twitter.com/paul_irish/status/309037258638512129
[41]: http://www.kendoui.com/blogs/teamblog/posts/12-02-17/using_svg_on_android_2_x_and_kendo_ui_dataviz.aspx
[42]: http://Mobilefish.com/

[Птица киви, которая стоит на овале]: img/kiwi.png?raw=true&amp;repo=using-svg
[Сохраняем файл с расширением SVG]: img/save-as-svg.png?raw=true&amp;repo=using-svg
[Настройки сохранения файла с расширением SVG]: img/svg-options.png?raw=true&amp;repo=using-svg
[окно TextEdit с SVG-кодом]: img/svg-code.png?raw=true&amp;repo=using-svg
[Размеры рабочей области в Illustrator]: img/artboard.png?raw=true&amp;repo=using-svg
[Код, полученный в результате конвертации SVG-файла в Data URI]: img/base64-data.png?raw=true&amp;repo=using-svg

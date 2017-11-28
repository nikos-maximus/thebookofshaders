## Узоры

Шейдерные программы исполняются попиксельно, поэтому вне зависимости от количества повторений фигуры объём вычислений не изменяется. Это значит, что фрагментные шейдеры хорошо справляются с повторяющимися узорами.

[ ![Нина Вормердам - Проект IMPRINT (2013)](warmerdam.jpg) ](../edit.php#09/dots5.frag)

В этой главе мы собираемся применить весь ранее изученный материал, и повторить это в изображении несколько раз. Как и в предыдущих главах, наша стратегия будет основана на умножении пространственных координат (между 0.0 и 1.0), так чтобы фигуры, которые мы рисуем между 0.0 и 1.0 повторялись несколько раз, образуя решётку.

*«Регулярная решётка - это то, с чем человеческой интуиции и изобретательности проще всего работать. Повторяющиеся элементы вступают в контраст с хаосом мироздания и создают ощущение порядка. Люди всегда старались украсить и разнообразить окружающее пространство с помощю повторяющихся элементов. Это прослеживается от доисторических узоров на керамике до геометрических мозаик римских бань.»* [*10 PRINT*, Издательство MIT, (2013)](http://10print.org/)

Для начала давайте вспомним функцию [```fract()```](../glossary/?search=fract). Она возвращает дробную часть числа, то есть работает как взятие остатка от деления на единицу ([```mod(x,1.0)```](../glossary/?search=mod)). Другими словами, [```fract()```](../glossary/?search=fract) возвращает число справа от точки. Переменная с нормализованными координатами (```st```) уже пробегает значения от 0.0 до 1.0, поэтому нет смысла делать что-то вроде:

```glsl
void main(){
	vec2 st = gl_FragCoord.xy/u_resolution;
	vec3 color = vec3(0.0);
    st = fract(st);
	color = vec3(st,0.0);
	gl_FragColor = vec4(color,1.0);
}
```

Но если мы увеличим масштаб нормализованной системы координат, скажем, в три раза, то получится три отрезка линейной интерполяции между 0.0 и 1.0: между 0 и 1, между 1 и 2, и наконец между 2 и 3.

<div class="codeAndCanvas" data="grid-making.frag"></div>

Теперь давайте нарисуем что-нибудь в каждом подпространстве, раскомментировав строку 27. Соотношение сторон при этом не изменится и фигуры не будут искажены, ибо мы умножаем по обеим осям.

Для более полного понимания попробуйте выполнить следующее:

* Поумножайте пространственные координаты на различные числа. Поэкспериментируйте с дробными числами и с различными множителями для x и y.

* Сделайте функцию создания узоров, пригодную для повторного использования.

* Разделите пространство на 3 строки и 3 столбца. Придумайте как узнать в какой строке и каком столбце находится текущий поток, и изменяйте фигуру в зависимости от этого. Нарисуйте игру в крестики-нолики.

### Применение матриц к узорам

Каждая клетка решётки является уменьшенной версией нормализованной системы координат, которую мы использовали ранее, поэтому мы можем применить к этим клеткам матричные преобразования переноса, поворота и масштаба.

<div class="codeAndCanvas" data="checks.frag"></div>

* Придумайте интересные анимации для этого узора. Анимируйте цвет, форму и движение. Сделайте три различных анимации.

* Создайте более сложные узоры, совмещая разные формы.

[![](diamondtiles-long.png)](../edit.php#09/diamondtiles.frag)

* Создайте узор [шотландского тартана](https://www.google.com/search?q=scottish+patterns+fabric&tbm=isch&tbo=u&source=univ&sa=X&ei=Y1aFVfmfD9P-yQTLuYCIDA&ved=0CB4QsAQ&biw=1399&bih=799#tbm=isch&q=Scottish+Tartans+Patterns), совмещая узоры в несколько слоёв.

[ ![Векторный шотландский узор от Kavalenkava](tartan.jpg) ](http://graphicriver.net/item/vector-pattern-scottish-tartan/6590076)

### Узоры со сдвигом

Давайте сымитируем кирпичную стену. Каждый ряд кирпичей в стене смещён на полкирпича по оси x относительно предыдущего ряда. Как это сделать?

![](brick.jpg)

Для начала нам нужно узнать чётность номера строки, над которой работает данный поток. Таким образом мы сможем понять, нужно ли делать сдвиг по x в этой строке.

Чтобы определить чётность строки, используем взятие по модулю 2.0 с помощью функции [```mod()```](../glossary/?search=mod), и сравним результат с единицей. Посмотрите на формулы ниже и раскомментируйте две последние строки.

<div class="simpleFunction" data="y = mod(x,2.0);
// y = mod(x,2.0) < 1.0 ? 0. : 1. ;
// y = step(1.0,mod(x,2.0));"></div>

Как видите, мы могли бы использовать [тернарный оператор](https://ru.wikipedia.org/wiki/%D0%A2%D0%B5%D1%80%D0%BD%D0%B0%D1%80%D0%BD%D0%B0%D1%8F_%D1%83%D1%81%D0%BB%D0%BE%D0%B2%D0%BD%D0%B0%D1%8F_%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F) для сравнения значения по модулю ```2.0``` с единицей, но того же эффекта можно достичь с помощью [```step()```](../glossary/?search=step), которая работает быстрее. Почему? Хотя мы и не знаем как графические карты оптимизирует код, безопаснее будет предположить что встроенные функции работает быстрее, чем не встроенные. Если у вас есть возможность использовать встроенную функцию - используйте!

Теперь у нас есть формула вычисления чётности и мы можем сдвинуть нечётные строки для создания эффекта кирпичной стены. В 14 строке следующего кода мы используем эту формулу для «обнаружения» нечётных строк. Как видите, для чётных строк функция возвращает ```0.0```, что при умножении на сдвиг ```0.5``` так же даёт ```0.0```, а значит чётные строки не сдвигаются. В нечётных же строках ```0.5``` умножается на ```1.0```, поэтому в них пространство сдвигается на ```0.5``` по оси x.

Теперь попробуйте раскомментировать строку 32. Это сделает соотношение сторон похожим на соотношение сторон кирпича. А раскомментировав 40 строку вы увидите отображение координат в красный и зелёный цвета.

<div class="codeAndCanvas" data="bricks.frag"></div>

* Попробуйте анимировать этот пример, изменяя сдвиг в зависимости от времени.

* Создайте ещё одну анимацию, где чётные строки движутся налево, а нечётные - направо.

* Можете ли вы повторить такой же эффект для столбцов?

* Сделайте сдвиг по x и y одновременно, чтобы получить что-то вроде этого:

<a href="../edit.php#09/marching_dots.frag"><canvas id="custom" class="canvas" data-fragment-url="marching_dots.frag"  width="520px" height="200px"></canvas></a>

## Плитка Труше

Теперь, когда мы научились определять чётность строки и столбца для каждой клетки, мы можем многократно использовать один и тот же элемент в зависимости от его расположения. Рассмотрим [плитку Труше](https://ru.wikipedia.org/wiki/%D0%9F%D0%BB%D0%B8%D1%82%D0%BA%D0%B0_%D0%A2%D1%80%D1%83%D1%88%D0%B5), где один элемент дизайна может быть представлен четырьмя различными способами:

![](truchet-00.png)

Изменяя рисунок в зависимости от расположения плитки, можно создать бесконечно много сложных изображений.

![](truchet-01.png)

Обратите особое внимание на функцию ```rotateTilePattern()```, которая разделяет пространство на четыре части и задаёт угол поворота каждой из них.

<div class="codeAndCanvas" data="truchet.frag"></div>

* Закомментируйте, раскомментируйте и продублируйте строки с 69 по 72, чтобы скомпоновать новые изображения.

* Замените чёрно-белый треугольник на какой-нибудь другой элемент, например полукруг, повернутые квадраты или линии.

* Напишите другие узоры с элементами, повёрнутыми в зависимости от их расположения.

* Создайте узор, элементы которого меняют другие свойства в зависимости от расположения.

* Придумайте ещё какое-нибудь изображение (не обязательно повторяющийся узор), в котором можно применить принципы из этой главы, например, [Гексаграммы И цзин](https://ru.wikipedia.org/wiki/%D0%93%D0%B5%D0%BA%D1%81%D0%B0%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0_(%D0%98_%D1%86%D0%B7%D0%B8%D0%BD)).

<a href="../edit.php#09/iching-01.frag"><canvas id="custom" class="canvas" data-fragment-url="iching-01.frag"  width="520px" height="200px"></canvas></a>

## Создавайте свои собственные правила

Создание процедурных узоров - это умственное упражнение по поиску минимальных повторно используемых элементов. Это древняя практика. Мы, как биологический вид, издавна использовали узоры для украшения тканей, пола и границ объектов: вспомните извилистые узоры древней Греции или решётчатые узоры из Китая. Магия повторений и вариаций всегда захватывала наше воображение. Рассмотрите [декоративные](https://archive.org/stream/traditionalmetho00chririch#page/130/mode/2up) [узоры](https://www.pinterest.com/patriciogonzv/paterns/) и обратите внимание как художники и дизайнеры балансируют между предсказуемостью порядка и внезапностью изменчивости хаоса. От арабских геометрических узоров и до бесподобных африканских тканей раскинулась целая вселенная узоров, среди которых есть что изучить.

![Франц Сейлс Мейер - Учебник орнамента (1920)](geometricpatters.png)

Этой главой мы заканчиваем раздел об алгоритмическом рисовании. В следующих главах мы привнесём в шейдеры немного энтропии для создания генеративного дизайна.
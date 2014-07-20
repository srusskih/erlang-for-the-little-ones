Базовый синтаксис функций
=========================

В прошлой главе мы рассмотрели как объявлять функции и как объединять их в модули. В этой главе мы рассмотрим синтаксис функций более подробно.

Сопоставление с образцом
------------------------

Для начала давайте напишем функцию, которая будет приветствовать пользователя и текст приветствия будет зависеть от его пола. В виде псевдокода наша функция будет выглядеть следующим образом:
```
function greet(Gender,Name)
  if Gender == male then
    print("Hello, Mr. %s!", Name)
  else if Gender == female then
    print("Hello, Mrs. %s!", Name)
  else
    print("Hello, %s!", Name)
end
```
Если вместо классической конструкции `if then else` использовать сопоставление с образцом, можно сэкономить кучу шаблонного кода. Вот так эта функция будет выглядеть на Erlang если использовать сопоставление с образцом:
```erlang
greet(male, Name) -> 
    io:format("Hello, Mr. ~s!", [Name]);
greet(female, Name) ->
    io:format("Hello, Mrs. ~s!", [Name]).
```
Функция `io:format()` используется для форматированного вывода в терминал. Здесь мы использовали сопоставление с образцом в описании списка аргументов функции. Это позволило нам одновременно присвоить входные значения и выбрать ту часть функции, которая должна быть выполнена. Зачем сначала присваивать значения, а потом сравнивать их в теле функции, если можно сделать это одновременно и в "более декларативном" стиле?

В общем виде объявление такой функции выглядит следующим образом:
```erlang
fnct_name(X) ->
  Extpression;
fnct_name(Y) ->
  Expression;
fnct_name(_) ->
  Expression.
```
Каждая ветвь функции объявляется как полноценная функция но заканчивается точкой с запятой(`;`), следом за ней объявляется следующая. После последней части ставиться точка.

Обратите внимание на последний образец. Что будет, если мы вызовем нашу функцию `greet()` и укажем непредусмотренный пол? Мы получим исключение о том, что входные параметры не подходят ни под один из образцов:
```erlang
1> chapter03:greet(someOther, "Haru").
** exception error: no function clause matching chapter03:greet(someOther, "Haru")
```
Поэтому важно включать в объявление образец, который подойдет под любое значение. При этом он должен быть последним. В противном случае образцы объявленные после него никогда не будут обработаны.
Давайте перепишем нашу функцию, так что бы она корректно обрабатывала неверные входные значения:
```erlang
greet(male, Name) ->
    io:format("Hello, Mr. ~s!", [Name]);
greet(female, Name) ->
    io:format("Hello, Mrs. ~s!", [Name]);
greet(_, Name) ->
    io:format("Hello, ~s!", [Name]).
````
 
Но сопоставление с образцом в объявлении функций приносит намного больше пользы, чем просто сокращение объема кода. Вспомним списки: список состоит из головы и остальной части. Давайте напишем две функции, которые будут возвращать первый и второй элементы полученного списка.
```erlang
first([X|_]) -> 
    X.
second([_,X|_) -> 
    X.
```
Достаточно просто, не так ли? Есть еще один интересный прием, основанный на том факте, что переменные в Erlang можно присвоить только один раз. 
```erlang
same(X,X) ->
    true;
same(_,_) ->
    false.
```
Эта функция возвращает `true`, если ее аргументы одинаковы, иначе возвращает `false`. Как это работает? При вызове функции `chapter03::same(one, two).` переменной `X` присваивается значение `one`, затем производится попытка присвоить этой же переменной значение `two`. Из-за того, что значение уже было присвоено, попытка заканчивается неудачей и шаблон отбрасывается как неподходящий. Так как во втором случае мы не указываем явно каким переменным нужно присвоить значения, шаблон подходит и функция возвращает `false`. Если же передать в функцию одинаковые значения `chapter03:same(3, 3).`, то первый шаблон подойдет и функция вернет `true`.


Охранные выражения
------------------

У сопоставления с образцом есть один большой недостаток: оно недостаточно выразительно. С его помощью нельзя указать тип данных, диапазон и другие подобные обобщения. Каждый образец является конкретным случаем. Для решения этой проблемы в Erlang есть Охранные выражения (или сторожевые условия). Для начала давайте напишем функцию, которая будет принимать наш ИМТ (индекс массы тела) и оценивать наше состояние. ИМТ человека равен его весу разделенному на квадрат роста.
```erlang
bmi_tell(Bmi) when Bmi =< 18.5 ->
    "You're underweight.";
bmi_tell(Bmi) when Bmi =< 25 ->
   "You're supposedly normal.";
bmi_tell(Bmi) when Bmi =< 30 ->
   "You're fat.";
bmi_tell(_) ->
    "You're very fat.".
```
Здесь при обращении к функции происходит проверка первого условия, находящегося после слова `when` (`Bmi =< 18.5`). Если это выражение возвращает `true` будет выполнена соответствующая ветвь кода. Иначе происходит проверка следующего условия и так до конца. И тут мы тоже добавили в конец условие, под которое подойдет любое значение.

В общем случае объявление функции с использованием охранных выражений выглядит следующим образом:
```erlang
fnct_name(Arg_1, Arg_1, ..., Arg_n) when rule_1 ->
    Expression;
fnct_name(Arg_1, Arg_1, ..., Arg_n) when rule_2 ->
    Expression.
```
Так же мы не ограничены одним выражением в условии. Мы можем провести несколько проверок. Если для прохождения проверки должны быть выполнены оба условия (аналог `andalso`), между ними ставится запятая(`,`).
```erlang
lucky_number(X) when 10 < X, X > 20 ->
    true;
lucky_number(_) ->
    false.
```
Если достаточно хотя бы одного (аналог `orelse`), они разделяются точкой с запятой(`;`).
```erlang
lucky_atom(X) when X == atom1; X == anot2 ->
    true;
lucky_atom(_) ->
    false.
```
В охранных выражениях можно использовать функции. Вот функция, которая делит одно число на другое, но перед этим проверяет, что бы переданные аргументы были числами и что бы `Y` не был равен нулю:
```erlang
safe_division(X, Y) when is_integer(X), is_integer(Y), Y /= 0 ->
    X / Y;
safe_division(_, _) ->
    false.
```

Ветвление
---------

**Оператор `if`**

Помимо рассмотренных выше выражений в Erlang существует и обычный оператор ветвления `if`. Давайте перепишем нашу функцию `bmi_tell()` с использованием этого оператора.
```erlang
if_bmi_tell(Bmi) ->
    if Bmi =< 18.5 -> "You're underweight.";
       Bmi =< 25   -> "You're supposedly normal.";
       Bmi =< 30   -> "You're fat.";
       true        -> "You're very fat."
    end.
```
В общем случае оператор ветвления `if` выглядит следующим образом:
```
if Rule_1 -> Expression;
   Rule_2 -> Expression;
   ...
   true -> Expression;
end.
```
Хочу напомнить, что в отличии от Haskell, здесь отступы не играют никакой роли кроме декоративной и код отформатирован так только того, что бы его было легче воспринимать. Вы вольны форматировать его так, как вам будет удобно. 
Здесь выражения `rule_1` и `rule_2` - это одно или несколько условий. В случае успешного выполнения условия, будет выполнен блок кода `Expression` (может содержать одну или несколько команд) идущий за ним. Таких логических ветвей может быть сколько угодно.

Обратите внимание на последний блок `true -> "You're very fat.`. Что это за условие? Это альтернатива оператору `orelse`. Блок кода идущий за ним будет выполнен если ни одно условие не пройдет проверку. Важно запомнить, что этот оператор обязателен. Ведь, как мы помним, в Erlang любое выражение должно возвращать результат. Если не описать этот блок, то модуль скомпилируется, но когда поток выполнения проверит все условия и не обнаружит блок `true`, будет сгенерированно исключение.
```erlang
** exception error: no true branch found when evaluating an if expression
     in function  chapter03:if_bmi_tell/1 
```

**Оператор `case ... of`**

Помимо `if` в Erlang есть еще один оператор ветвления - `case of`. И этот оператор подобен целой функции вставленной в другую функцию. Он позволяет делать выбор не только на основании условия, но так же дает возможность использовать сопоставления с образцом и охранные выражения. В общем виде он выглядит так:
```erlang
case Rule of
    Val_1 -> Expression;
    Val_2 -> Expression
    ...
    Val_n -> Expression
end.
```
Здесь `Rule` - это переменная или выражение, результат которого будет проверяться. Дальше следуют уже знакомые нам блоки "условие - выражение" (`Val_1 -> Expression;`). В условии можно использовать сопоставления с образцом и охранные выражения, что делает эту конструкцию невероятно гибкой.

Давайте для примера напишем функцию, которая будет принимать кортеж состоящий из температуры и названия шкалы ее измерения и на основании этих данных оценивать ее.
```erlang
assessment_of_temp(Temp) ->
    case Temp of
        {X, celsius} when 20 =< X, X =< 45 ->
            'favorable';
        {X, kelvin} when 293 =< X, X =< 318 ->
            'scientifically favorable';
        {X, fahrenheit} when 68 =< X, X =< 113 ->
            'favorable in the US';
        _ ->
            'not the best tempperature'
  end.
```
При выполнении этой функции наш кортеж будет сопоставлен с переменной `Temp`, затем будет найден паттерн, под который этот кортеж подойдет. Затем будет проведена проверка охранными выражениями и если она будет пройдена - функция вернет соответствующую строку.
Здесь мы так же как и раньше добавляем в конец выражение, которое будет перехватывать все неподошедшие варианты.

Как мы видим, выражение `case of` почти полностью можно переместить в область объявление функции. Так где же лучше размещать условия? Ответ прост: там где вам больше нравиться. Различия между этими двумя конструкциями минимальны. Поэтому используйте тот вариант, который вам легче читать.

Заключение
----------

В этой главе мы рассмотрели то, как можно управлять потоком выполнения внутри функций. Узнали, какие для этого существуют конструкции и как их использовать. Так же мы познакомились с охранными выражениями.

В следующей главе мы более пристально рассмотрим систему типов в языке Erlang.
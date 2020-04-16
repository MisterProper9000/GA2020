# GA2020

# Описание проекта
## Постановка задачи
По описанию дерева на языке, порождаемом заданной грамматикой, сгенерировать код на языке представления графов DOT и вывести результат его визуализации.

## План работы
- Построена и зафиксирована грамматика языка, на котором описываются деревья.
- С помощью плагина ANTLR v4 grammar plugin сгенерирован код лексера и парсера на Java. Результатом работы парсера является дерево синтаксического разбора, которое в дальнейшем используется для генерации кода на DOT.
- Для проверки корректности работы парсера, написан ретранслятор (класс *Repeater*), который по дереву синтаксического разбора обратно собирает строку, по которой оно было построено.
- Реализован генератор кода на DOT (класс *DotGenerator*). 
- Написаны Python - скрипты *DOT_URL_generator.py* и *random_tree_generator.py*. Первый принимает в качестве аргумента входной файл с DOT и открывает в браузере сайт dreampuf.github.io/GraphvizOnline, дописывая в конец запроса закодированный в URL DOT-код. Используется для визуализации сгенерированного DOT-кода. Второй генерирует описание случайного дерева и записывает его в файл, который принимает в качестве аргумента. Используется для генерации тестовых данных, на которых можно проверить корректность работы программы. Оба скрипта при желании могут быть использованы отдельно от основной программы.

## Описание входного языка
Входной язык описания деревьев порождается следующей грамматикой: 
![grammar](https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/grammar.jpg "grammar")

Таким образом, дерево представляет из себя структуру, заключенную в круглые скобки ```()```, состоящую из узла в квадратных скобках ```[]```, и возможно поддеревьев, соединённых с корнем связями в угловых скобках ```<>```. У каждого узла или связи должно быть непустое название. За названием может один или несколько комментариев, заключенных в фигурные скобки```{}```. Рассмотрим пример описания дерева на таком языке:

```([+{это имя узла}]<0{это имя дуги}>([A])<1>([*]<0>([B])<1>([C])))```

Комментарии можно использовать для того, чтобы указывать атрибуты отрисовки, такие как цвет связи или форма узла дерева. Атрибуты имеют синтаксис ```name=value```, где ```name``` и ```value``` произвольные строки, не содержащие ```=```. Любые комментарии, удовлетворяющие данному синтаксису, будут интерпретированы как атрибуты, и напрямую записаны в выходной DOT-файл без дополнительной проверки на валидность значений ```name``` и ```value``` (так как список атрибутов и их возможных значений достаточно велик, и к тому же отличается в разных программах для отрисовки DOT-файлов). Значение ```value``` следует указывать без кавычек, они будут автоматически добавлены при генерации DOT-файла. Любые другие комментарии, которые не интерпретируются как атрибуты отрисовки, будут проигнорированы. Рассмотрим пример описания того же самого дерева с атрибутами в качестве комментариев:

```([+{shape=box}]<0{color=blue}>([A])<1{color=red}>([*{shape=box}]<0{color=blue}>([B])<1{color=red}>([C])))```

В качестве имён узлов и связей могут использоваться любые наборы символов, за исключением ```()[]<>{}```, а так же, возможно, некоторых, зарезервированных в языке DOT.

## Описание алгоритма генерации DOT
DOT-строка, которая затем записывается в файл генерируется методом класса *DotGenerator* за один полный обход синтаксического дерева.
Класс *DotGenerator* представляет из себя конечный автомат со стеком состояний. В начале обхода поддерева на стек кладётся состояние в соответствии с правилом, лежащим в корне поддерева. Когда обход поддерева заканчивается, со стека убирается верхнее состояние. Оно должно совпадать с тем, которое было положено на стек в начале обхода. Если состояния не соответствуют, это является признаком некорректной работы алгоритма. Так же, кроме стека состояний, генератор хранит стек с узлами дерева. Необходимость одновременно хранить в памяти не только последний узел, но и часть предыдущих, обусловлена следующим фактом: во входном языке имя каждого узла встречается лишь единожды, в то время как в DOT нужно указывать имя узла-родителя каждый раз при описании его связи с другим узлом-ребенком. Например, описание простейшего дерева с корнем и двумя сыновьями на входном языке выглядит так: ```([root]<1>([son1])<2>([son2]))```, в то время как на DOT оно выглядит так: ```graph Tree {root -- son1 [ label="1" ] root -- son2 [ label="2" ]}```. Таким образом, при обработке узла *son2* требуется информация, что его родителем был *root*.

Чтобы иметь возможность задавать имена узлов произвольными символами, а также иметь несколько разных узлов с одинаковыми именами, набор символов, указанный во входном языке в качестве имени узла, кодируется в DOT в качестве значения атрибута ```label```, в то время как сами узлы в DOT именуются возрастающими целочисленными идентификаторами. В случае, если явно указать в комментарии входного языка значение атрибута ```label```, то в сгенерированном DOT-коде будет использоваться оно, а имя узла будет проигнорировано.

## Результаты работы
### Пример 1
Рассмотрим, как работает алгоритм генерации на примере, приведенном выше:

```([+{это имя узла}]<0{это имя дуги}>([A])<1>([*]<0>([B])<1>([C])))```

Сгенерированный DOT-код:

![DOT1](https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example1_DOT.png "DOT example 1")

Результат отрисовки в GraphvizOnline:

![example 1](https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example1.png "example 1")

### Пример 2
Тот же пример с атрибутами:

```([+{shape=box}]<0{color=blue}>([A])<1{color=red}>([*{shape=box}]<0{color=blue}>([B])<1{color=red}>([C])))```

Сгенерированный DOT-код:

![DOT2](https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example2_DOT.png "DOT example 2")

Результат отрисовки в GraphvizOnline:

![example 2](https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example2.png "example 2")

### Пример 3
Рассмотрим пример работы на случайном дереве, сгенерированном скриптом *random_tree_generator.py*:

```  
([P{shape=ellipse}{fontcolor=blue}]
<2{style=filled}{color=blue}{fontcolor=black}>([J{shape=circle}{fontcolor=black}])
<2{style=bold}{color=goldenrod}{fontcolor=black}>([p{shape=diamond}{fontcolor=sienna}]
<0{style=filled}{color=black}{fontcolor=red}>([i{shape=ellipse}{fontcolor=red}]
<2{style=filled}{color=red}{fontcolor=blue}>([a{shape=circle}{fontcolor=green}]
<7{style=dotted}{color=blue}{fontcolor=black}>([o{shape=circle}{fontcolor=goldenrod}]
<0{style=filled}{color=black}{fontcolor=red}>([K{shape=octagon}{fontcolor=black}])
<4{style=bold}{color=red}{fontcolor=green}>([o{shape=diamond}{fontcolor=blue}])
<1{style=dotted}{color=goldenrod}{fontcolor=sienna}>([i{shape=ellipse}{fontcolor=goldenrod}]))))
<7{style=filled}{color=blue}{fontcolor=black}>([i{shape=circle}{fontcolor=sienna}]
<1{style=dotted}{color=black}{fontcolor=black}>([A{shape=circle}{fontcolor=blue}]
<7{style=bold}{color=red}{fontcolor=red}>([j{shape=circle}{fontcolor=green}]
<5{style=filled}{color=green}{fontcolor=red}>([s{shape=ellipse}{fontcolor=goldenrod}]))
<2{style=bold}{color=sienna}{fontcolor=sienna}>([v{shape=circle}{fontcolor=sienna}])
<3{style=filled}{color=goldenrod}{fontcolor=red}>([Y{shape=diamond}{fontcolor=black}]
<4{style=bold}{color=red}{fontcolor=goldenrod}>([a{shape=octagon}{fontcolor=sienna}])
<6{style=filled}{color=sienna}{fontcolor=blue}>([q{shape=octagon}{fontcolor=black}])
<5{style=filled}{color=blue}{fontcolor=red}>([D{shape=polygon}{fontcolor=goldenrod}]))))))
```

Сгенерированный DOT-код:

![DOT5(https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example5_DOT.png "DOT example 5)

Результат отрисовки в GraphvizOnline:

![example 5(https://github.com/sergeevgk/GA2020/blob/ChartPS/pictures/example5.png "example 5)


# Структура проекта
- Файл *ChartPC.jar* - результат компиляции программы, при запуске преобразует описание дерева из входного файла в DOT, который записывает в выходной файл. Из этого файла вызываются скрипты *DOTURLGenerator.py* и *random_tree_generator.py*.
- Файл *DOT_URL_generator.py* - по коду на DOT из входного файла генерирует URL запрос и открывает результат его выполнения в браузере.
- Файл *random_tree_generator.py* - генерирует файл с описанием случайного дерева, который используется в качестве входного файла по умолчанию для *ChartPC.jar*.
- Папка *src* содержит файлы с исходным кодом на Java.
- Папка *gen* содержит файлы с кодом на Java, сгенерированные с помощью плагина ANTLR v4 grammar plugin.
- Папка *out* содержит результаты компиляции Java - классов.
- Папка *grammar* содержит файл с правилами грамматики языка, на котором описываются деревья.
- Папка *libs* - директория со сторонними библиотеками и jar-файлами, использующимися в проекте.
- Папка *examples* - содержит текстовые файлы с описаниями деревьев.
- Папка *gen_dot* - содержит результаты работы программы на файлах из папки *examples*.

Для запуска и работы программы необходимы только первые 3 файла.


# Установка и запуск
Для запуска программы (на Windows) достаточно запустить исполняемый файл ChartPC.jar с помощью команды:

```java -jar ChartPC.jar INPUT_FILE OUTPUT_FILE```

(команда должна выполняться из директории, в которой лежит сам файл *ChartPC.jar*, а так же скрипты *DOT_URL_generator.py* и *random_tree_generator.py*)

Первый аргумент запуска ```INPUT_FILE``` будет интерпретироваться как полное имя входного файла. Если его опустить, то в качестве входного файла будет сгенерирован файл *random_generated_tree.txt* с описанием случайного дерева. Второй аргумент ```OUTPUT_FILE_PATH``` будет интерпретироваться как полное имя файла, в который будет записан результат (файл будет создан, или перезаписан, если он уже существует). Если его опустить, то результат будет записан в файл *generated_dot.txt*, он будет создан в активной папке, из которой вызывается команда. Остальные аргументы будут игнорироваться. 

Входной файл должен содержать синтаксически корректное описание одного дерева. В случае успешной работы программы, в выходной файл будет записан код на DOT, соответствующий описанию дерева из входного файла. Также, в браузере будет открыта страница онлайн версии утилиты Graphviz, на которой можно будет посмотреть и скачать изображение дерева, отрисованное по сгенерированному коду.

Для запуска необходимы установленные Java версии 1.8.0_251 или выше и Python версии 3.6 или выше. В случае отсутствия Python (невозможности вызвать команду ```python``` из командной строки), программу всё ещё можно запустить и сгенерировать код на DOT, но генерация случайных деревьев и визуализация в браузере работать не будут.


# Создание и настройка проекта IntelliJ IDEA
Ниже описан один из способов создать и собрать проект в IntelliJ IDEA (инструкция актуальна для версии Community Edition 2019)
1. Скачать содержимое репозитория
2. Открыть папку с содержимым как проект в IntelliJ IDEA
3. Назначить SDK для проекта (во вкладке File->Project Structure->Project выбрать SDK из выпадающего списка в графе Project SDK)
4. Выбрать папку для результатов компиляции (во вкладке File->Project Structure->Project в графе Project compiler output выбрать папку out (или любую другую))
5. Добавить папку libs в зависимости проекта (во вкладке File->Project Structure->Modules->Dependencies нажать на +, выбрать JARs or directories и в открывшемся окне выбрать папку libs)
6. Пометить папки src и gen, как директории с исходниками (в окне со структурой проекта: правый клик по папке src->Mark Directory as->Sources Root, правый клик по папке gen->Mark Directory as->Generated Sources Root)

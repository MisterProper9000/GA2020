# Easy-Pattern-Constructor
IntelliJ IDEA Plugin to simplify design pattern usage. Using DSL method.

ФАН: автор, дата, статус

Ответ авторов:

**Авторы**: Грицаенко Н.Д. и Мельник П.В.

**Дата**: 06.04.2020

**Статус**: В рамках курса реализована поддержка кодогенерации для трёх паттернов проектирования. Вне рамок курса есть простор для улучшений (добавление других паттернов).

## Запуск и установка

1. Клонировать ветку "E=pc"
2. Открыть папку с проектом в IntelliJ IDEA
3. Подождать, пока gradle закончит сборку
4. В gradle выбрать task "runIDE", запустить его
5. Готово. Открыто новое окно IntelliJ IDEA с установленным плагином. Теперь можно его протестировать (см. примеры ниже)

ФАН: Хм. Сначала купить, а потом начать думать, зачем это нужно ...

Ответ авторов:

Ничего покупать не нужно - это проект с открытым исходным кодом. К тому же мы не планируем использовать его в коммерческих целях. Установить плагин, или нет - решает пользователь после просмотра демо-роликов.

## Постановка задачи
Пользователь (программист на java) устанавливает плагин. В процессе разработки он сможет быстро выбрать один из готовых шаблонов проектирования, а затем дополнить его для своей задачи. 

ФАН: неточные слова. Программист на Java за всю свою жизнь хорошо если создаст хоть один свой шаблон проектирования. Обычно он использует готовые.

Ответ авторов: 

исправили на то, что пользователь использует готовые шаблоны.


*Шаблоном проектирования* будем называть подмножество *design patterns* и *idioms* (терминология согласно [UML3](https://uml3.ru/library/design_patterns/design_patterns.html))

Пользователь может:
- выбрать шаблон проектирования
- [выбрать параметры шаблона проектирования] 

Список поддерживаемых паттернов:
- Singleton
- Builder
- Visitor

## Диаграмма использования
![](https://github.com/sergeevgk/GA2020/blob/E-%3D-pc/diagrams/uml_use_case.JPG)

ФАН: это скорее нечто вроде блок-схемы... Какой смысл имеют пунктирные стрелочки? 

Ответ авторов: 

Смысл таков: кодогенерация включает в себя предварительный выбор паттерна и его параметров. Выбор паттерна включает в себя просмотр списка паттернов. Таким образом, пунктирные стрелки отображают зависимость *include*

## Описание GUI пользователя

Пример:

1. Нажать ПКМ
2. Выбрать паттерн из выпадающего списка
3. [Выбрать параметры паттерна]
4. Получить сгенерированный код, дополнить его в своих целях

## Блок-схема работы плагина
![](https://github.com/sergeevgk/GA2020/blob/E-%3D-pc/diagrams/blok-skhema.jpg)

ФАН: Ой! Это ни в коем случае не диаграмма состояний, это блок-схема в обозначениях ГОСТа 1978 года. Ужас! 

Ответ авторов: назвали эту схему "Блок-схема работы плагина". Её основная цель была в том, чтобы авторы (то есть мы) поняли, в каком порядке плагин будет взаимодейстовать с пользователем.  

## Примеры

### Singleton
![](https://github.com/sergeevgk/GA2020/blob/E-%3D-pc/gifs/singleton.gif)

### Builder
![](https://github.com/sergeevgk/GA2020/blob/E-%3D-pc/gifs/builder.gif)

### Visitor
![](https://github.com/sergeevgk/GA2020/blob/E-%3D-pc/gifs/visitor.gif)


ФАН: демо ролики - превосходно! Но где же "method DSL" ??? Как происходит генерация кода?

Ответ авторов:

Генерация кода происходит следующим образом:
1. Программист заполняет необходимы для выбранного паттерна поля, например, в случае Builder, он выбирает необходимые поля, а также необязательные поля, но которые будет включать в себя Builder.
2. Далее программист нажимает кнопку *Ok*.
3. После этого в нашу программу поступают поля, выбранные пользователем, далее в зависимости от выбранного паттерна, эти поля по-своему обрабатываются и за счёт встроенных в IDEA средств, в данном случае PsiClass собирается структура, которая соответствующая java классу, которая затем преобразуется в код засчёт встроенного в IDEA класса JavaCodeGenerator.
4. Как итог пользователь видит сгенерированный код и может его использовать.

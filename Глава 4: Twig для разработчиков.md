Эта глава описывает API Twig, а не сам язык шаблона. Наиболее полезна она будет тем, кто разрабатывает интерфейс шаблона для приложения, нежели для тех, кто создаёт сами шаблоны.

### **Основы**

Центральный объект в Twig - окружение (объект класса ```Twig_Environment```). Экземпляр этого класса используется для хранения конфигураций и расширений, а так же для загрузки файлов шаблонов из файловой системы или любого другого места.

Большинство приложений будут создавать единственный экземпляр класса ```Twig_Environment``` при инициализации приложения и использовать его для загрузки файлов шаблонов. Однако бывает необходимо иметь несколько экземпляров класса, если используются различные настройки окружения одновременно.

Самый простой способ настроить Twig для загрузки файлов шаблонов в приложении выглядит следующим образом:

```php
require_once '/path/to/vendor/autoload.php';

$loader = new Twig_Loader_Filesystem('/path/to/templates');
$twig = new Twig_Environment($loader, array(
  'cache' => '/path/to/compilation_cache'
));
```

После выполнения этого кода будет создана переменная окружения со стандартными настройками и файловый загрузчик, который произведён поиск файлов шаблонов в директории ```/path/to/templates```. Возможны различные конфигурации файловых загрузчиков - от стандартных до собственных разработок загружающих шаблоны из базы данных или любых других источников.

Обратите внимание, что вторым параметров при создании экземпляра класса ```Twig_Environment``` является массив опций. Опция ```cache``` указывает на директорию в которой хранятся закэшированные скомпилированные шаблоны, чтобы избежать процесса компилирования при каждом запросе к ресурсам. Этот кэш отличается от того кэша о котором вы можете подумать по результатирующему выводу, для кэширования отображаемых данных воспользуйтесь любой доступной PHP-библиотекой.

### **Отображение шаблонов**

Для загрузки файла шаблона из окружения Twig воспользуйтесь вызовом функции ```load()```, которая возвратит экземпляр класса ```Twig_TemplateWrapper```:

```php
$template = $twig->load('index.html');
```

Для отображения шаблона с переданными переменными воспользуйтесь функцией ```render()```:

```php
echo $template->render(array('name' => 'Andrew', 'city' => 'Vladimir'));
```

Метод ```display()``` сокращенный вариант для непосредственного вывода результата обработки шаблона.

Есть возможность одним махом загрузить и отобразить шаблон:

```php
$this->render('index.html', array('name' => 'Andrew', 'city' => 'Vladimir'));
```

Если шаблон определяет блоки, то вывод определенного блока можно осуществить следующим образом:

```php
$template->renderBlock('block_name', array('price' => 100.0, 'total' => 2000.0));
```

### **Настройки окружения**

Создавая экземпляр класса окружения ```Twig_Environment``` можно передать дополнительные параметры конфигурации вторым параметров в конструктор:

```php
$twig = new Twig_Environment($loader, array('debug' => true));
```

Доступны следующие параметры конфигурации:

- ```debug``` boolean

При установленном флаге - ```true```, сгенерированные шаблоны реализуют метод ```__toString()```, используя который можно вывести содержание сгенерированных узлов (веток). По-умолчанию - ```false```.

- ```charset``` string (по-умолчанию ```utf-8```)

Кодировка используемая шаблоном.

- ```base_template_class``` string (по-умолчанию ```Twig_Template```)

Базовый класс используемый для сгенерированных шаблонов.

- ```cache``` string или ```false```

Абсолютный путь для хранения скомпилированных шаблонов или ```false``` для отключения кэширования.

- ```auto_reload``` boolean

Разрабатывая с использованием Twig полезно регенерировать ранее скомпилированные шаблоны каждый раз, когда изменяется исходный код шаблона. Если эта опция не установлена, то её значение будет зависеть от значения опции ```debug```.

- ```strict_variables``` boolean

При установленном значении в ```false```, Twig будет автоматически подменять недействительные переменные (переменные и/или свойства и методы классов) на значение ```null```. При установленном значении в ```false``` Twig будет выбрасывать исключение при обнаружении недействительных переменных.

- ```autoescaping``` string

Устанавливает используемую стратегию экранирования (```name```, ```html```, ```js```, ```css```, ```url```, ```html_attr``` или PHP-коллбэк, который получает на вход название файла шаблона и возвращает наименование стратегии экранирования). Значение ```false``` отключает любое экранирование. Стратегия ```name``` определяет фактическую стратегию экранирования по наименованию файла шаблона и используемого расширения (использование этой стратегии никоим образом не влечёт за собой увеличение скорости обработки шаблонов т.к. экранирование происходит на стадии компиляции).

- ```oprimizations``` integer

Флаг определяющий какие оптимизции применять (```-1``` - по-умолчанию применяются все оптимизации, ```0``` - оптимизации отключены).

### **Загрузчики**

Загрузчики отвечают за загрузку файлов шаблонов, например, из файловой системы.

**Кэш компиляции**

У всех загрузчиков шаблонов есть возможность хранить скомпилированный вариант шаблонов для последующего использования. Это позволяет значительно ускорить работу Twig, так как компиляция осуществляется единожды. Еще большего ускорения можно добиться используя [APC](https://ru.wikipedia.org/wiki/%D0%90%D0%BA%D1%81%D0%B5%D0%BB%D0%B5%D1%80%D0%B0%D1%82%D0%BE%D1%80_PHP) в PHP. Подробная информация о флагах ```cache``` и ```auto_reload``` окружения ```Twig_Environment``` пригодится при тюнинге приложения.

**Встроенные загрузчики**

Вот список загрузчиков, которые поддерживаются:

###### Twig_Loader_Filesystem

```Twig_Loader_Filesystem``` загружает файлы шаблонов из файловой системы. Этот загрузчик способен искать файлы шаблонов в директориях и загружать их, является рекомендуемым к использованию:

```php
$loader = new Twig_Loader_Filesystem($templateDir);
```

Так же есть возможность поиска файлов шаблонов по массиву переданных директорий:

```php
$loader = new Twig_Loader_Filesystem(array($templateDir1, $templateDir2));
```

При такой конфигурации Twig начнет поиск файлов шаблонов с директории ```$templateDir1```, если их не существует, то поиск продолжится в директории ```$templageDir2```.

При необходимости можно добавлять пути используя методы ```addPath()```, ```prependPath()```:

```php
$loader->addPath($templateDir3);
$loader->prependPath($template4);
```

Файловый загрузчик так же поддерживает объявления пространст имён шаблонов. Объявление шаблонов по пространствам имён позволяет использовать настраиваемые пути для каждой группы шаблонов.

Используя методы ```setPaths()```, ```addPath()``` и ```prependPath()```, в качестве второго параметра можно передавать пространство имён (если пространство имён не указано, то используется пространство имён "main").

```php
$loader->addPath($templateDir6, 'admin');
```

Шаблоны в различных пространствах имён доступны через нотацию ```@namespace_name/template_path```:

```php
$twig->render('@admin/index.html', array());
```

```Twig_Loader_Filesystem``` поддерживает абсолютные и относительные пути. Использование относительных путей предпочтительнее, т.к. ключи кэша будут независимы от корневой директории проекта:

```php
$loader = new Twig_Loader_Filesystem('templates', getcwd() . '/..');
```

Если не передавать в качестве второго параметра корневой путь, то Twig использует ```getcwd()``` для формирования относительных путей.

###### Twig_Loader_Array

```Twig_Loader_Array``` загружает шаблоны из PHP-массива, который представляет собой комбинации ключ + значение, где ключ - наименование файла шаблона, а значение - исходный код самого шаблона.

```php
$loader = new Twig_Loader_Array(array(
  'index.html' => 'Hello, {{ name }}'
));

$twig = new Twig_Environment($loader);
echo $twig->render('index.html', array('name' => 'Andrew'));
```

Данный загрузчик крайне полезен при Unit-тестировании и разработке небольших проектов, где небольшое число шаблонов проще и оптимальнее хранить в массиве.

Стоит помнить, что при использовании ```Array```-загрузчика с активным механизмом кэширования, кэш-ключ генерируется новый при каждом изменении содержимого шаблона. Если размер кэша значительно увеличивается при использовании такого подхода, то его очистка ложится на ваши плечи.

###### Twig_Loader_Chain

```Twig_Loader_Chain``` делегирует загрузку шаблонов другим загрузчикам:

```php
$loader1 = new Twig_Loader_Array(array(
  'base.html' => '{% block content %}{% endblock %}'
));

$loader2 = new Twig_Loader_Array(array(
  'index.html' => '{% extends "base.html" %}{% block content %}Hello {{ name }}{% endblock %}',
  'base.html' => 'Will never be loaded'
));

$loader = new Twig_Loader_Chain(array($loader1, $loader2));

$twig = new Twig_Environment($loader);
```

При поиске шаблонов, Twig будет использовать загрузчики поочередно и, при нахождении запрашиваемого шаблона, прекращать дальнейший поиск. При обработке ```index.html``` из приведенного выше примера, Twig загрузит шаблон из ```$loader2```, а вот шаблон ```base.html``` из ```$loader1```. 

```Twig_Loader_Chain``` принимает в качестве загрузчика любой класс реализующий интерфейс ```Twig_LoaderInterface```.

Добавление загрузчиков возможно с использованием метода ```addLoader()```.

###### Создаём собственный загрузчик

Все загрузчики реализуют ```Twig_LoaderInterface```:

```php
interface Twig_LoaderInterface
{
  public function getSourceContext($name);
  public function getCacheKey($name);
  public function isFresh($name, $time);
  public function exists($name);
}
```

```isFresh()```-метод должен вернуть ```true```, если текущий кэш шаблона является актуальным (свежим) относительного переданного времени изменения, либо ```false```.

```getSourceContext()```-метод должен возвращать экземпляр класса ```Twig_Source```.

### **Используем расширения**

Расширения Twig это блоки, которые добавляют новые возможности в Twig. Использование расширений - это просто, как вызов ```addExtension()```-метода:

```php
$twig->addExtension(new Twig_Extension_Sandbox());
```

Следующий список расширений доступен в Twig по-умолчанию:

- ```Twig_Extension_Core```: Описывает все ключевые возможности Twig.
- ```Twig_Extension_Escaper```: Реализует автоматическое экранирование кода и возможность экранирование/неэкранирования определенных блоков кода.
- ```Twig_Extension_Sandbox```: Добавляет режим "песочницы" в Twig, позволяя выполнять внешний (сторонний) код.
- ```Twig_Extension_Profiler```: Реализует встроенный в Twig профилировщик.
- ```Twig_Extension_Oprimizer```: Оптимизирует дерево выражений перед компиляцией.

```*_Core```, ```*_Escaper```, ```*_Optimizer``` не требуют ручного добавления в ```Twig_Environment```, т.к. регистрируются автоматически.

### **Встроенные расширения**

Эта секция расскажет подробне о возможностях встроенных расширений.

###### Core-расширение

Расширение ```core``` определяет ключевые возможности Twig:

- теги
- фильтры
- функции
- тесты

###### Escaper-расширение

Расширение ```escaper``` реализует возможность автоматического экранирования. Это расширение определяет ```autoescape```-тег и ```raw```-фильтр.

Создавая (настраивая в ручном режиме) ```Escaper```-расширение есть возможность включить или выключить стратегию автоматического экранирования:

```php
$escaper = new Twig_Extension_Escaper('html');
$twig->addExtension($escaper);
```

Если передаваемое значение равно ```html```, то все переменные в шаблоне экранируются, кроме тех, которые используют ```raw```-фильтр:

```php
{{ article.to_html|raw }}
```

Для локальных изменений стратегии экранирования воспользуйтесь ```autoescape```-тегом:

```php
{% autoescape 'html' %}
  {{ var }}
  {{ var|escape }} {# переменная var не будет дважды экранирована #}
  {{ var|raw }} {# переменная var не будет экранирована #}
{% endautoescape %}
```

**Внимание!**

```autoescape```-тег никак не влияет на подключаемые файлы (через ```include```-тег).

Правила экранирования:

- Литеральные константы (числа, строки, массивы и т.д.) используемые в шаблоне в качестве переменных или аргументов функций никогда не экранируются:

```twig
{{ 'Twig <br />' }} {# не экранируется #}

{% set text = 'Twig <br />' %}
{{ text }} {# экранируется #}
```

- Выражения, результат которых всегда является литеральной константой или переменные отмеченные безопасными, никогда не экранируются автоматически:

```twig
{{ foo ? "Twig<br />" : "<br />Twig" }} {# не экранируется #}

{% set text = "Twig<br />" %}
{{ foo ? text : "<br />Twig" }} {# экранируется #}

{% set text = "Twig<br />" %}
{{ foo ? text|raw : "<br />Twig" }} {# не экранируется #}

{% set text = "Twig<br />" %}
{{ foo ? text|escape : "<br />Twig" }} {# результат выражение не будет экранирован #}
```

- Экранирование применяется до вывода результата работы всех фильтров, функций:

```twig
{{ var|upper }} {# эквивалентно var|upper|escape #}
```

- ```raw```-фильтр должен применяться **всегда** последним:

```twig
{{ var|raw|upper }} {# экранируется переменная var #}
{{ var|upper|raw }} {# не экранируется #}
```

- Автоматическое экранирование не будет применено, если последний фильтр в цепочке отмечен как безопасный для данного контекста (например html или js). ```escape``` и ```escape('html')``` являются безопасными для HTML, ```escape('js')``` является безопасным для JavaScript, ```raw``` является безопасным для всего.

```twig
{% autoescape 'js' %}
  {{ var|escape('html') }} {# будет экранирован для HTML и JavaScript #}
  {{ var }} {# будет экранирован для JavaScript #}
  {{ var|escape('js') }} {# будет экранирован единожды для JavaScript #}
{% endautoescape %}
```

**Внимание!**

Автоматическое экранирование имеет свои ограничения, т.к. применяется к результату всего выражения, а не его отдельным частям. Так образом результат выполнения ```{{ var|raw ~ bar }}``` будет не таким, каким вы его ожидали бы увидеть. 

###### Sandbox-расширение

```sandbox```-расширение может быть использовано для анализа недоверенного кода. Доступ к небезопасным аттрибутам и методам запрещён. Безопасность песочницы определяется экземпляром политики безопасности. По-умолчанию Twig содержит единственный класс политики безопасности - ```Twig_Sandbox_SecurityPolicy```. Этот класс позволяет добавить некоторые теги, фильтры, методы, функции в список разрешенных:

```php
$tags = array('if');
$filters = array('upper');
$methods = array('Article' => array('getTitle', 'getBody'));
$properties = array('Article' => array('title', 'body'));
$functions = array('range');

$policy = new Twig_Sandbox_SecurityPolicy($tags, $filters, $methods, $properties, $functions);
```

При такое конфигурации будет разрешен только тег ```if```, фильтр ```upper```, методы класса ```Article``` - ```getTitle()``` и ```getBody()```, публичные свойства класса ```Article``` - ```title``` и ```body```, функция ```range```. Всё остально запрещено и при попытках обращения будет генерировать исключение ```Twig_Sandbox_SecurityError```.

Объект политики безопасности передаётся первым параметром при инициализации класса ```Twig_Extension_Sendbox```:

```php
$sandbox = new Twig_Extension_Sandbox($policy);
$twig->addExtension($sandbox);
```

По-умолчанию режим песочницы отключен и должен быть активирован при использовании подключений внешнего (недоверенного) кода с использованием ```sandbox```-тега:

```twig
{% sandbox %}
  {% include 'user.html' %}
{% endsandbox %}
```

Чтобы все шаблоны обрабатывались в песочнице по-умолчанию необходимо в качестве второго параметра в конструктор класса ```Twig_Extension_Sandbox``` передать ```true```:

```php
$sandbox = new Twig_Extension_Sandbox($policy, true);
```

###### profiler-расширение (профилировщик)

Расширение ```Profiler``` позволяет подключить профилировщик к Twig; использовать его стоит только на этапе тестирования и отладки:

```php
$profile = new Twig_Profiler_Profile();
$twig->addExtension(new Twig_Extension_Profiler($profile));

$dumper = new Twig_Profiler_Dumper_Text();
echo $dumper->dump($profile);
```

Профилировщик содержит информацию о времени выполнения и потребляемой памяти шаблонов, блоков и макросов. 

###### optimizer-расширение (оптимизация)

```optimizer```-расширение оптимизирует дерево узлов перед комиляцией:

```php
$twig->addExtension(new Twig_Extension_Optimizer());
```

По-умолчанию все оптимизации активны. Перечень оптимизаций, которые вы хотите активировать, можно передать через конструктор:

```php
$optimizer = new Twig_Extension_Optimizer(Twig_NodeVisitor_Optimizer::OPTIMIZER_FOR);

$twig->addExtension($optimizer);
```

Twig поддерживает следующие оптимизации:

- ```Twig_NodeVisitor_Optimizer::OPTIMIZE_ALL```: активирует все оптимизации.
- ```Twig_NodeVisitor_Optimizer::OPTIMIZE_NONE```: выключает все оптимизации. Эта опция сокращает время компиляции, но увеличивает время выполнения и потребляемую память.
- ```Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR```: оптимизирует цикл ```for``` исключая создание переменной ```loop``` где это только возможно.
- ```Twig_NodeVisitor_Optimizer::OPTIMIZE_RAW_FILTER```: исключает использование ```raw``` фильтра где это возможно.
- ```Twig_NodeVisitor_Optimizer::OPTIMIZE_VAR_ACCESS```: упрощает создание и доступ к переменным создаваемым в шаблоне где это возможно.

###### Исключения

Twig может выбрасывать следующие исключения:

- ```Twig_Error```: базовое исключение.
- ```Twig_Error_Syntax```: выбраcывается в том случае, если в шаблоне ошибка синтаксиса.
- ```Twig_Error_Runtime```: выбрасывается на этапе выполнения, например, если фильтр не существует.
- ```Twig_Error_Loader```: выбрасывается на этапе загрузки файла шаблонов.
- ```Twig_Sandbox_SecurityError```: выбрасывается в том случае, если происходит вызов к запрещенному тегу, фильтру, методу в ```sandbox``` режиме.

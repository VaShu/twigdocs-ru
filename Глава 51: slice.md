Фильтр ```slice``` извлекает часть из последовательности, хэша или строки:

```twig
{% for i in [1, 2, 3, 4, 5]|slice(1, 2) %}
  {# итерации пройду по значениям 2 и 3 #}
{% endfor %}

{{ '1234'|slice(1, 2) }}
{# выведет '23' #}
```

Возможно использование любого действительного выражения для определения начала последовательности и её завершения:

```twig
{% for i in [1, 2, 3, 4, 5]|slice(start, length) %}
  ...
{% endfor %}
```

Так же, в качестве удобства использования можно использовать ```slice``` через ```[]```:

```twig
{% for i in [1, 2, 3, 4, 5][start:length] %}
  ...
{% endfor %}

{{ '1234'[1:2] }}
{# выведет 23 #}

{# можно опустить первый аргумент - по-умолчанию он равен 0 #}
{{ '1234'[:2] }}
{# выведет 12 #}

{# можно опустить второй аргумент - по-умолчанию он равен длине последовательности #}
{{ '1234'[2:] }}
{# выведет 34 #}
```

Фильтр ```slice``` работает так же, как PHP-функция ```array_slice``` для массивов и ```mb_substr``` для строк (при отсутствии поддержки этой функции - ```substr```).

Если ```start``` положительное значение - оно используется в качестве начала последовательности; если ```start``` отрицательное значение - отсчет начального индекса происходит с конца последовательности.

Если ```length``` положительное значение, то длина последовательности будет равна этому значению, либо, если ```length``` больше фактической длины последовательности - все элементы последовательности начиная со ```start``` будет возвращены. Если ```length``` отрицательное значение - длина конечной последовательности будет равна ```start``` плюс оставшееся количество символов за вычетом ```length```. Если значение ```length``` не передано - новая последовательность определяется с индекса ```start``` до конца исходной последовательности.

Фильтр так же работает с объектами реализующими интерфейс ```Traversable```.

###### Аргументы

- ```start```: индекс начала новой последовательности
- ```length```: размер новой последовательности
- ```preserve_keys```: сохранять ли значения ключей или нет (в случае массивов)

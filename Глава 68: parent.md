При использовании наследования в шаблонах есть возможность отобразить содержимое блока родительского шаблона с помощью функции ```parent()```:

```twig
{% extends 'base.html' %}

{% block sidebar %}
  <h3>Hello</h3>
  ...
  {{ parent() }}
{% endblock %}
```

Вызов функции ```parent()``` вернёт содержимое родительского блока ```sidebar``` из шаблона ```base.html```.

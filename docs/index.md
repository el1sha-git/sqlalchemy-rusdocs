title: SQLAlchemy начало
<p align="center">
  <a href="#"><img src="https://blog.desdelinux.net/wp-content/uploads/2023/02/sqlalchemy.png" alt="SQL Alchemy"></a>
</p>
<p align="center">
    <em>SQLAlchemy: open source SQL-набор инструментов и объектно-реляционный преобразователь</em>
</p>
<p align="center">
<a href="https://blog.desdelinux.net/wp-content/uploads/2023/02/sqlalchemy.png" target="_blank">Исходник картинки(desdelinux.net)</a>
</p>

---

**Оригинальная документация**: <a href="https://docs.sqlalchemy.org/en/20/tutorial/index.html" target="_blank">
sqlalchemy.org</a>

**Исходный код SQLAlchemy**: <a href="https://github.com/sqlalchemy/sqlalchemy" target="_blank">SQLAlchemy Github</a>

---

## SQLAlchemy: Описание основных компонентов

SQLAlchemy - это открытый набор инструментов для работы с SQL и объектно-реляционным отображением (ORM). Он содержит две
ключевые части - Object Relational Mapper (ORM) и Core.


<figure markdown>
  ![Image title](https://docs.sqlalchemy.org/en/20/_images/sqla_arch_small.png){ width="300" }
  <figcaption>Структура sqlalchemy</figcaption>
</figure>

### Core

Core содержит большинство SQL-инструментов и сервисов интеграции баз данных SQLAlchemy, включая язык выражений SQL - SQL
Expression Language.

SQL Expression Language - это отдельный набор инструментов, независимый от ORM, который предоставляет систему создания
SQL-выражений, представленных композируемыми объектами, которые могут быть "выполнены" на целевой базе данных в рамках
конкретной транзакции, возвращая набор результатов. Вставки, обновления и удаления (т.е. DML) выполняются путем передачи
объектов выражений SQL, представляющих эти операции, вместе со словарями, представляющими параметры, которые будут
использоваться с каждым выражением.

### ORM

ORM использует Core для работы с моделью объектов домена, отображенной на схему базы данных. При использовании ORM
SQL-запросы строятся в основном так же, как и при использовании Core. Однако задача DML (т.е. сохранение объектов
бизнес-модели в базе данных) автоматизируется с помощью шаблона, называемого unit of work. Этот шаблон переводит
изменения состояния мутабельных(изменяемых) объектов в конструкции INSERT, UPDATE и DELETE, которые затем вызываются относительно
этих объектов. Выборка данных также улучшена автоматизацией, специфичной для ORM, и возможностями запросов,
ориентированными на объекты.

В отличие от работы с Core и SQL Expression Language, которые представляют схематическое представление базы данных,
вместе с программной парадигмой, ориентированной на неизменяемость, ORM строит на этом модель объектов домена,
представляющую схему базы данных с программной парадигмой, более явно ориентированной на объекты и основанной на
мутабельности. Поскольку реляционная база данных сама является изменяемым сервисом, разница в том, что Core / SQL
Expression Language ориентированы на выполнение команд, тогда как ORM ориентирован на состояние.


---

## Обзор документации {#documentation-overview}

Документация разделена на четыре секции:

- **[SQLAlchemy Unified Tutorial](tutorial)** - это полностью новое руководство для серии 1.4/2.0 SQLAlchemy, которое
  вводит библиотеку в целом, начиная с описания Core и продвигаясь все дальше к концепциям ORM. Новым пользователям, а
  также пользователям, переходящим с версии 1.x SQLAlchemy, следует начать здесь.

- **[SQLAlchemy ORM](/orm)** - в этом разделе представлена справочная документация для ORM.

- **[SQLAlchemy Core](/core)** - здесь представлена справочная документация для всего остального, что находится в
  Core. Здесь также описаны SQLAlchemy engine, connection и pooling services.

- **[Диалекты](/dialects)** - предоставляет справочную документацию для всех реализаций диалектов, включая
  конкретику DBAPI.

---

## Примеры кода {#code-examples}

Рабочие примеры кода, в основном, относятся к ORM и включены в дистрибутив SQLAlchemy. Описание всех примеров приложений
находится на странице [ORM Examples](orm/).

На вики также есть множество примеров, касающихся как основных конструкций SQLAlchemy, так и ORM.
Смотрите <a href="https://github.com/sqlalchemy/sqlalchemy/wiki/UsageRecipes" target="_blank">Theatrum Chemicum</a>.



---

## Установка {#installation-guide}

### Поддерживаемые платформы {#supported-platforms}

SQLAlchemy поддерживает следующие платформы:

- cPython 3.7 и выше
- Совместимые с Python-3 версии PyPy

Начиная с версии 2.0: SQLAlchemy теперь поддерживает Python 3.7 и выше.

### Поддержка AsyncIO {#asyncio-support}

Поддержка asyncio в SQLAlchemy зависит от проекта greenlet. Эта зависимость будет установлена по умолчанию на
распространенных платформах, однако не поддерживается на всех архитектурах и может не устанавливаться по умолчанию на
менее распространенных архитектурах. См. раздел Asyncio Platform Installation Notes (включая Apple M1) для получения
дополнительной информации о том, как обеспечить наличие поддержки asyncio.

### Поддерживаемые методы установки {#supported-installation-methods}

Установка SQLAlchemy осуществляется стандартными методологиями Python, основанными на setuptools, либо путем прямой
ссылки на setup.py, либо используя pip или другие подходы, совместимые с setuptools.

### Установка через pip {#install-via-pip}

При наличии pip распространение можно загрузить с PyPI и установить за один шаг:

```sh
pip install SQLAlchemy
```

Эта команда загрузит последнюю выпущенную версию SQLAlchemy из Python Cheese Shop и установит ее на вашу систему. Для
большинства распространенных платформ будет загружен файл Python Wheel, который предоставляет нативные расширения
Cython / C, предварительно скомпилированные.

Чтобы установить последнюю предварительную версию, например, 2.0.0b1, требуется использовать флаг --pre:

```sh
pip install --pre SQLAlchemy
```

В случае, если самая последняя версия является предварительной, она будет установлена вместо последней выпущенной
версии.

### Установка вручную из исходного кода {#installing-manually-from-the-source-distribution}

Если необходимо установить из исходного кода, можно использовать скрипт setup.py:

```sh
python setup.py install
```

Установка исходного кода не зависит от платформы и будет установлена на любой платформе, независимо от того, установлены
ли инструменты сборки Cython / C или нет. Как указано в следующем разделе, при наличии возможности setup.py будет
пытаться скомпилировать с помощью Cython / C, но в противном случае выполняется чистая установка Python.

### Сборка расширений Cython {#building-the-cython-extensions}

SQLAlchemy включает расширения Cython, которые обеспечивают дополнительное ускорение в различных областях, в настоящее
время с акцентом на скорость результатов Core.

Изменено в версии 2.0: Расширения SQLAlchemy на C были переписаны с использованием Cython.

```setup.py``` автоматически соберет расширения, если на вашей платформе обнаружен соответствующий компилятор и установлен
пакет Cython. Полная ручная сборка выглядит так:


```shell
# перейдите в директорию с SQLAlchemy
cd path/to/SQLAlchemy

# установите Cython
pip install cython

# По желанию соберите расширения Cython перед установкой
python setup.py build_ext

# Запустите установку
python setup.py install
```

Исходные сборки также могут быть выполнены с использованием техник PEP 517, например, используя build:

```shell
# перейдите в директорию SQLAlchemy
cd путь/к/SQLAlchemy

# установите build
pip install build

# соберите исходные файлы / директории wheel
python -m build
```
Если сборка расширений Cython не удалась из-за отсутствия установленного Cython, отсутствия компилятора или другой
проблемы, процесс установки выведет предупреждающее сообщение и завершит сборку без расширений Cython, сообщив
окончательный статус.

Чтобы запустить сборку/установку, не пытаясь даже скомпилировать расширения Cython, можно указать переменную среды
DISABLE_SQLALCHEMY_CEXT. В таких случаях это может потребоваться в особых тестовых условиях или в редких случаях
совместимости/проблем сборки, которые не могут быть устранены обычным механизмом "пересборки":

``` shell
export DISABLE_SQLALCHEMY_CEXT=1; python setup.py install.
```

## Установка API базы данных {#installing-a-database-api}

SQLAlchemy разработан для работы с DBAPI, созданным для конкретной базы данных, и включает поддержку наиболее популярных
баз данных. В отдельных разделах для каждой базы данных в Dialects перечислены доступные DBAPI, включая внешние ссылки.

## Проверка установленной версии SQLAlchemy {#checking-the-installed-sqlalchemy-version}

Эта документация охватывает версию SQLAlchemy 2.0. Если вы работаете на системе, на которой уже установлен SQLAlchemy,
проверьте версию из своей оболочки Python следующим образом:

```
>>> import sqlalchemy
>>> sqlalchemy.__version__  
2.0.0
```

## Следующие шаги {#next-steps}

С установленным SQLAlchemy новые и старые пользователи могут [перейти к руководству SQLAlchemy](/tutorial).
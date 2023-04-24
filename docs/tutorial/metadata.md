title: Работа с метаданными базы данных

## Работа с метаданными базы данных {#metadata-introduction}

После изучения работы с движками и выполнения SQL-запросов мы готовы начать работу с Alchemy. Центральным элементом как
для SQLAlchemy Core, так и для ORM является язык выражений SQL, который позволяет свободно и компонуемо строить
SQL-запросы. Основу этих запросов составляют объекты Python, которые представляют концепции базы данных, такие как
таблицы и столбцы. Эти объекты известны совместным названием - метаданные базы данных.

Самые распространенные объекты для метаданных базы данных в SQLAlchemy - это MetaData, Table и Column. Ниже мы
проиллюстрируем, как эти объекты используются как в стиле Core, так и в стиле ORM.

!!! tip "ORM пользователи, оставайтесь с нами!"

    Как и в других разделах, пользователи Core могут пропустить разделы ORM, но ORM пользователи лучше ознакомиться с этими
    объектами с обеих сторон. Объект Table, обсуждаемый здесь, объявляется более косвенным (и также полностью
    типизированным) способом при использовании ORM, однако в конфигурации ORM по-прежнему есть объект Table.

## Настройка метаданных с Table объектами {#setting-up-metadata-with-table-objects}

При работе с реляционной базой данных основной структурой, которую мы запрашиваем, является таблица. В SQLAlchemy база
данных "таблица" в конечном итоге представляется объектом на Python, также названым Table.

Для начала использования SQLAlchemy Expression Language мы захотим, чтобы были созданы объекты Table, представляющие все
таблицы базы данных, с которыми мы хотим работать. Table создается программно, либо непосредственно с помощью
конструктора Table, либо косвенно с помощью ORM Mapped классов (описанных позже в разделе Использование
ORM-декларативных форм для определения метаданных таблицы). Также есть возможность загрузить некоторую или всю
информацию о таблице из существующей базы данных, называемой отражением.

Каким бы способом мы ни подошли к делу, мы всегда начинаем с коллекции, куда поместим наши таблицы, называемой объектом
MetaData. Этот объект, по сути, является фасадом вокруг словаря Python, который хранит серию объектов Table, ключевыми
для которых являются их строковые имена. В то время как ORM предоставляет некоторые варианты, где можно получить эту
коллекцию, мы всегда можем просто создать ее напрямую, что выглядит следующим образом:

```python
from sqlalchemy import MetaData

metadata_obj = MetaData()
```

После создания объекта MetaData, мы можем объявить несколько объектов Table. В этом руководстве мы начнем с классической
модели SQLAlchemy, которая имеет таблицу с названием user_account, которая хранит, например, пользователей веб-сайта, и
связанную с ней таблицу address, которая хранит адреса электронной почты, связанные с записями в таблице user_account.
Если мы не используем ORM Declarative модели вовсе, мы создаем каждый объект Table напрямую, обычно присваивая каждому
переменную, которая будет ссылкой на таблицу в приложении:

```python
from sqlalchemy import Table, Column, Integer, String

user_table = Table(
    "user_account",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("name", String(30)),
    Column("fullname", String),
)
```

С приведенным выше примером, когда мы хотим написать код, который относится к таблице user_account в базе данных, мы
будем использовать переменную Python user_table для обращения к ней.

!!! tip "Когда следует создавать объект MetaData в программе?"

    Создание единого объекта MetaData для всего приложения является наиболее распространенным случаем и представляется в
    виде переменной уровня модуля в одном месте в приложении, часто в пакете типа "models" или "dbschema". Также очень
    распространено использование реестра, ориентированного на ORM, или базового класса Declarative Base, чтобы этот же
    объект MetaData был доступен для Table объектов, объявленных как ORM, так и Core.
    
    Также возможно использование нескольких коллекций MetaData; Table объекты могут ссылаться на объекты Table из других
    коллекций без ограничений. Однако для группы связанных между собой Table объектов практически гораздо проще их
    устанавливать в рамках одной коллекции MetaData, как с точки зрения их объявления, так и с точки зрения генерации
    SQL-запросов DDL (например, CREATE и DROP) в правильном порядке.

### Компоненты таблицы {#components-of-table}

Мы можем заметить, что конструкция Table в написанном на Python коде напоминает оператор SQL CREATE TABLE; сначала
задается имя таблицы, затем перечисляются столбцы, каждый из которых имеет имя и тип данных. Объекты, которые мы
используем выше, включают:

- Table - представляет таблицу в базе данных и присваивает себе коллекцию MetaData.
- Column - представляет столбец в базе данных и присваивает себе объект Table. Колонка обычно включает строковое имя и
  объект типа. Коллекция объектов Column относительно родительской таблицы обычно доступна через ассоциативный массив,
  расположенный в Table.c:

    ``` python
    >>> user_table.c.name
    Column('name', String(length=30), table= < user_account >)
    
    >>> user_table.c.keys()
    ['id', 'name', 'fullname']
    ```

- Integer, String - эти классы представляют SQL типы данных и могут быть переданы в Column с или без необходимости
  создания экземпляра. Выше мы хотим задать длину «30» для столбца «name», поэтому мы создали экземпляр String(30). Но
  для «id» и «fullname» мы не указали это, поэтому мы можем отправить сам класс.

???+ info "См. также"

    Справочная и API-документация по MetaData, Table и Column находится в разделе Описание баз данных с MetaData. Справочная
    документация по типам данных находится в разделе Объекты SQL Datatype.

В следующем разделе мы продемонстрируем одну из основных функций Table, которая состоит в генерации DDL на определенном
подключении к базе данных. Но сначала мы объявим вторую таблицу.

### Объявление простых ограничений {#declaring-simple-constraints}

Первый столбец в примере `user_table` включает параметр `Column.primary_key`, который является сокращенной техникой
указания, что этот столбец должен быть частью первичного ключа для этой таблицы. Сам первичный ключ обычно объявляется
неявно и представляется конструкцией `PrimaryKeyConstraint`, которую мы можем увидеть в атрибуте `Table.primary_key`
объекта `Table`:

``` python
>>> user_table.primary_key
PrimaryKeyConstraint(Column('id', Integer(), table=<user_account>, primary_key=True, nullable=False))
```

Ограничение, которое обычно объявляется явно, - это объект `ForeignKeyConstraint`, который соответствует ограничению
внешнего ключа базы данных. Когда мы объявляем таблицы, которые связаны между собой, SQLAlchemy использует наличие этих
объявлений ограничений внешнего ключа не только для того, чтобы они были выведены в операторы `CREATE` базы данных, но
также для помощи в создании SQL-выражений.

`ForeignKeyConstraint`, которое включает только один столбец на целевой таблице, обычно объявляется с использованием
сокращенной записи на уровне столбца через объект `ForeignKey`. Ниже мы объявляем вторую таблицу `address`, которая
будет иметь ограничение внешнего ключа, ссылочное на таблицу `user`:

``` python
>>> from sqlalchemy import ForeignKey
>>> address_table = Table(
...     "address",
...     metadata_obj,
...     Column("id", Integer, primary_key=True),
...     Column("user_id", ForeignKey("user_account.id"), nullable=False),
...     Column("email_address", String, nullable=False),
... )
```

В таблице выше также присутствует третий вид ограничения, который в SQL представляет собой ограничение «NOT NULL»,
указанное выше с помощью параметра `Column.nullable`.

!!! tip "Совет"

    При использовании объекта `ForeignKey` в определении столбца мы можем опустить тип данных для этого столбца; он
    автоматически выводится из связанного столбца, в приведенном выше примере это целочисленный тип данных
    столбца `user_account.id`.

В следующем разделе мы выведем завершенный DDL для таблиц `user` и `address`, чтобы увидеть окончательный результат.

### Отправка DDL {#emitting-ddl-to-the-database}

Мы создали объектную структуру, которая представляет две таблицы базы данных в базе данных, начиная с корневого объекта
MetaData, затем двух объектов Table, каждый из которых удерживает коллекцию объектов Column и Constraint. Эта объектная
структура будет в центре большинства операций, которые мы будем выполнять как с Core, так и с ORM.

Первое полезное, что мы можем сделать с этой структурой, это извлечь операторы CREATE TABLE, или DDL, в нашу базу данных
SQLite, чтобы мы могли вставлять и извлекать данные из них. У нас уже есть все инструменты, необходимые для этого,
вызывая метод MetaData.create_all() нашего объекта MetaData и передавая ему Engine, который ссылается на целевую базу
данных:

``` python
# python
>>> metadata_obj.create_all(engine)

# SQL
BEGIN (implicit)
PRAGMA main.table_...info("user_account")
...
PRAGMA main.table_...info("address")
...
CREATE TABLE user_account (
    id INTEGER NOT NULL,
    name VARCHAR(30),
    fullname VARCHAR,
    PRIMARY KEY (id)
)
...
CREATE TABLE address (
    id INTEGER NOT NULL,
    user_id INTEGER NOT NULL,
    email_address VARCHAR NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES user_account (id)
)
...
COMMIT

```

Процесс создания DDL выше включает некоторые SQLite-специфические операторы PRAGMA, которые проверяют наличие каждой
таблицы перед созданием. Полный набор шагов также включен в пару BEGIN/COMMIT для обеспечения транзакционной DDL.

Процесс создания также заботится о том, чтобы эмитировать операторы CREATE в правильном порядке. Например, FOREIGN
KEY-ограничение зависит от существования таблицы user, поэтому таблица адреса создается второй. В более сложных случаях
зависимости FOREIGN KEY-ограничений также могут быть применены к таблицам после того, как они были созданы с помощью
ALTER.

Объект MetaData также имеет метод MetaData.drop_all(), который будет создавать операторы DROP в обратном порядке, чем
CREATE, чтобы удалить элементы схемы.

!!! tip "Инструменты миграции обычно подходят"

    В целом, функция CREATE / DROP в MetaData полезна для наборов тестов, небольших и / или новых приложений и приложений,
    которые используют базы данных с коротким сроком жизни. Однако для управления схемой базы данных приложения в течение
    длительного времени инструмент управления схемой, такой как Alembic, который основан на SQLAlchemy, вероятно, является
    лучшим выбором, поскольку он может управлять и оркестрировать процесс инкрементного изменения фиксированной схемы базы
    данных со временем, когда проектирование приложения меняется.

## Использование ORM-декларативных форм для определения метаданных таблицы {#using-orm-declarative-forms-to-define-table-metadata}

!!! info "Еще один способ создания объектов таблиц?"

    В предыдущих примерах мы рассмотрели прямое использование объекта Table, который лежит в основе того, как SQLAlchemy
    обращается к таблицам базы данных при создании SQL-выражений. Как упоминалось ранее, SQLAlchemy ORM предоставляет фасад
    вокруг процесса объявления таблиц, который называется "декларативная таблица". Декларативная таблица достигает той же
    цели, которую мы имели в предыдущем разделе, а именно создания объектов Table, но также включает в этот процесс еще
    что-то, называемое ORM-отображаемым классом или просто "отображаемым классом". Отображаемый класс является наиболее
    распространенной основной единицей SQL при использовании ORM, и в современном SQLAlchemy также может быть использован
    вместе с Core-центрическим подходом.
    
    Некоторые преимущества использования декларативной таблицы:
    
    - Более краткий и 'питонический' стиль определения столбцов, где для представления типов SQL, используемых в базе
      данных, могут использоваться типы Python.
    
    - Результирующий отображаемый класс может использоваться для формирования SQL-выражений, которые во многих случаях
        поддерживают информацию о типизации PEP 484, которую можно использовать в инструментах статического анализа, таких как
        Mypy и проверщики типов IDE.
    
    - Позволяет объявить метаданные таблицы и отображаемый класс ORM, используемый в операциях сохранения/загрузки объектов,
        все сразу.
    
    В этом разделе мы продемонстрируем, как метаданные таблицы, созданные в предыдущих разделах, могут быть сконструированы
    с использованием декларативной таблицы.

При использовании ORM процесс объявления метаданных таблиц обычно сочетается с процессом объявления отображаемых
классов (mapped classes). Отображаемый класс - это любой класс Python, который мы хотим создать, и который затем будет
иметь атрибуты, связанные с колонками в таблице базы данных. Хотя существует несколько способов достижения этой цели,
наиболее распространенным стилем является декларативный, который позволяет нам объявлять наши классы, определять
метаданные таблицы и отображаемый класс, используемый в операциях сохранения / загрузки объектов, одновременно.

### Создание декларативной базы {#establishing-a-declarative-base}

При использовании ORM MetaData остается на месте, однако она связана с конструктом, используемым только для ORM, который
обычно называется декларативной базой. Самым быстрым способом получить новую декларативную базу является создание нового
класса, который является подклассом класса DeclarativeBase из SQLAlchemy:

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

В приведенном выше примере класс Base - это декларативная база. Когда мы создаем новые классы, которые являются
подклассами Base и содержат соответствующие директивы на уровне класса, каждый из них обычно (но не исключительно)
ссылается на определенный объект таблицы.

Декларативная база ссылается на коллекцию MetaData, которая автоматически создается для нас, если мы не предоставили ее
извне. Эта коллекция MetaData доступна через атрибут класса DeclarativeBase.metadata. При создании новых отображаемых
классов каждый из них будет ссылаться на таблицу в этой коллекции MetaData:

``` python
>>> Base.metadata
MetaData()
```

Декларативная база также ссылается на коллекцию под названием registry, которая является центральным блоком "
конфигурации маппера" в ORM SQLAlchemy. Хотя это объект редко используется напрямую, он является центральным для
процесса конфигурации маппера, поскольку набор отображаемых классов будет координироваться друг с другом через этот
registry. Как и в случае с MetaData, наша декларативная база также создала registry для нас (опять же, с возможностью
передачи своего собственного registry), к которому мы можем получить доступ через переменную класса
DeclarativeBase.registry:

``` python
>>> Base.registry
<sqlalchemy.orm.decl_api.registry object at 0x...>
```

!!! note ""

    Существуют и другие способы отображения с помощью registry, включая декораторно-ориентированные и императивные способы
    отображения классов. Также существует полная поддержка создания Python-классов данных при отображении. В документации по
    ссылке ORM Mapped Class Configuration можно найти подробную информацию обо всех подходах.

### Определение отображаемых классов {#declaring-mapped-classes}

С установленным классом Base мы теперь можем определить ORM отображаемые классы для таблиц `user_account` и `address` в
виде
новых классов `User` и `Address`. Ниже мы приводим наиболее современную форму `Declarative`, которая основана на
аннотациях
типов _PEP 484_ с использованием специального типа `Mapped`, который указывает на атрибуты, которые должны быть
отображены
как определенные типы:

```python
from typing import List
from typing import Optional
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship


class User(Base):
    __tablename__ = "user_account"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30))
    fullname: Mapped[Optional[str]]
    addresses: Mapped[List["Address"]] = relationship(back_populates="user")

    def __repr__(self) -> str:
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"


class Address(Base):
    __tablename__ = "address"
    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id = mapped_column(ForeignKey("user_account.id"))
    user: Mapped[User] = relationship(back_populates="addresses")

    def __repr__(self) -> str:
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"
```

Оба класса, `User` и `Address`, теперь называются `ORM Mapped Classes` и доступны для использования в ORM операциях
сохранения
и запроса, которые будут описаны позже. Некоторые детали этих классов:

- Каждый класс ссылается на объект Table, который был сгенерирован в процессе декларативного отображения и назван с
  помощью строки, присвоенной атрибуту DeclarativeBase.\_\_tablename__. После создания класса этот сгенерированный Table
  доступен через атрибут DeclarativeBase.\_\_table__.
- Как упоминалось ранее, этот вид называется Declarative Table Configuration. Один из нескольких альтернативных способов
  объявления был бы построить объект Table напрямую и присвоить его непосредственно DeclarativeBase.\_\_table__. Этот
  стиль называется Declarative with Imperative Table.
- Чтобы указать столбцы в таблице, мы используем конструкцию mapped_column(), в сочетании с типами аннотаций,
  основанными на типе Mapped. Этот объект будет генерировать объекты Column, которые будут применяться при создании
  таблицы.
- Для столбцов с простыми типами данных и без других параметров мы можем указать только тип Mapped, используя простые
  типы Python, такие как int и str для означения Integer и String. Настройка того, как интерпретируются типы Python в
  процессе декларативного отображения, является очень гибкой; см. разделы Using Annotated Declarative Table (Type
  Annotated Forms for mapped_column()) и Customizing the Type Map для получения дополнительной информации.
- Столбец может быть объявлен как «nullable» или «not null» на основе наличия типа аннотации Optional[<typ>] (или его
  эквивалентов, <typ> | None или Union[<typ>, None]). Можно также явно использовать параметр mapped_column.nullable (он
  не обязательно должен соответствовать параметру опциональности аннотации).
- Использование явных типовых аннотаций является полностью необязательным. Мы также можем использовать mapped_column()
  без аннотаций. При использовании этой формы мы будем использовать более явные типы объектов, такие как Integer и
  String, а также nullable=False, при необходимости, в каждой конструкции mapped_column().
- Два дополнительных атрибута, User.addresses и Address.user, определяют другой тип атрибута, называемый relationship(),
  который имеет схожие стили конфигурации, основанные на аннотациях. Конструкция relationship() подробно рассмотрена в
  разделе Working with ORM Related Objects.
- Если мы не определяем свой собственный init() метод, то классы автоматически получают этот метод. Форма по умолчанию
  этого метода принимает все имена атрибутов в качестве необязательных ключевых аргументов.

    ```python
    sandy = User(name="sandy", fullname="Sandy Cheeks")
    ```

Чтобы автоматически создать полнофункциональный метод `__init__()` , который предоставляет позиционные аргументы, а
также
аргументы со значением по умолчанию, можно использовать функционал dataclasses, введенный в Declarative Dataclass
Mapping. Конечно, всегда можно использовать явный метод `__init__()`.

Методы `__repr__()` добавлены, чтобы мы получили читаемый строковый вывод; нет необходимости, чтобы эти методы были
здесь.
Как и в случае с `__init__()`, метод `__repr__()` можно автоматически создать с помощью функционала dataclasses.

!!! info "Где же остался старый декларативный способ?"

    Пользователи SQLAlchemy 1.4 или более ранних версий заметят, что приведенное выше отображение использует сильно
    отличающуюся форму, чем раньше; в нем используется mapped_column() вместо Column в декларативном отображении, а также
    используются аннотации типов Python для получения информации о столбцах.
    
    Чтобы обеспечить контекст для пользователей "старого" способа, декларативные отображения все еще могут быть созданы с
    использованием объектов Column (а также с использованием функции declarative_base() для создания базового класса), и эти
    формы будут продолжать поддерживаться без планов на их удаление. Причиной того, что эти два механизма заменены новыми
    конструкциями, является в первую очередь интеграция с инструментами PEP 484, включая среды разработки, такие как VSCode,
    и проверяющие типы, такие как Mypy и Pyright, без необходимости использования плагинов. Во-вторых, происхождение
    объявлений из аннотаций типов является частью интеграции SQLAlchemy с Python dataclasses, которые теперь могут быть
    созданы непосредственно из отображений.
    
    Для пользователей, которым нравится "старый" способ, но все же хотят, чтобы их среды разработки не ошибочно сообщали об
    ошибках типизации для их декларативных отображений, конструкция mapped_column() является заменой Column в декларативном
    отображении ORM (обратите внимание, что mapped_column() предназначен только для декларативных отображений ORM; он не
    может использоваться внутри конструкции Table), а аннотации типов являются необязательными. Наше отображение выше может
    быть записано без аннотаций следующим образом:
    
    ```python
    class User(Base):
        __tablename__ = "user_account"
    
        id = mapped_column(Integer, primary_key=True)
        name = mapped_column(String(30), nullable=False)
        fullname = mapped_column(String)
    
        addresses = relationship("Address", back_populates="user")
    
        # ... definition continues```
    ```

    Вышеуказанный класс имеет преимущество перед классом, который использует `Column` напрямую, в том, что класс `User`, а также
    экземпляры `User` будут указывать правильную типовую информацию для инструментов типизации без использования плагинов.
    `mapped_column()` также позволяет использовать дополнительные ORM-специфические параметры для настройки поведения, таких
    как отложенная загрузка столбца, для которой ранее необходимо было использовать отдельную функцию `deferred()` с Column.
    
    Также есть пример преобразования класса Declarative старого стиля в новый стиль, который можно увидеть в разделе ORM
    Declarative Models в руководстве What’s New in SQLAlchemy 2.0?

!!! quote "Смотрите также"

    - ORM Mapping Styles - полный обзор различных стилей конфигурации ORM.
    - Declarative Mapping - обзор маппинга классов Declarative.
    - Declarative Table with mapped_column() - подробное описание использования mapped_column() и Mapped для определения
      столбцов в  таблице, которые будут отображены при использовании Declarative.
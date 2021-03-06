Конфигурация
=============

Первое, что нужно знать об Idiorm - *вам не нужно объявлять какие-либо классы модели для её использования*. С почти любой другой ORM, первым шагом является установка моделей и их связка с таблицами из базы данных
(через переменные конфигурации, XML файлы и тому подобное). С Idiorm,
вы можете приступить к использованию ORM сразу же.

Установка
~~~~~

Первым делом, укажите файл исходного кода Idiorm в ``require``:

.. code-block:: php

    <?php
    require_once 'idiorm.php';

Затем передайте *Data Source Name* строку соединения в метод ``configure``
класса ORM. Он используется PDO для соединения с вашей базой данных. Для более детальной информации, смотри `документацию PDO`_.

.. code-block:: php

    <?php
    ORM::configure('sqlite:./example.db');

Вы так же можете передать имя пользователя и пароль в драйвер базы данных, используя параметры конфигурации ``username`` и ``password``.
Например, если вы используете MySQL:

.. code-block:: php

    <?php
    ORM::configure('mysql:host=localhost;dbname=my_database');
    ORM::configure('username', 'database_user');
    ORM::configure('password', 'top_secret');

Смотрите так же секцию “Конфигурация” ниже.

Конфигурация
~~~~~~~~~~~~~

Помимо передачи DSN строки для подключения к базе данных (смотри выше), метод ``configure`` можно использовать и для установки некоторых других простых настроек ORM класса. Изменение настроек подразумевает передачу в метод ``configure`` пар ключ/значение, представляющих название параметра, который вы хотите поменять, и значение, которое хотите ему установить.

.. code-block:: php

    <?php
    ORM::configure('название_параметра', 'значение_для_параметра');

Следующий метод используется для передачи множества пар ключ/значение за раз.

.. code-block:: php

    <?php
    ORM::configure(array(
        'название_параметра_1' => 'значение_для_параметра_1', 
        'название_параметра_2' => 'значение_для_параметра_2', 
        'etc' => 'etc'
    ));

Для чтения текущей конфигурации, используйте метод ``get_config``.

.. code-block:: php

    <?php
    $isLoggingEnabled = ORM::get_config('logging');
    ORM::configure('logging', false);
    // какой-то бешенный цикл, который мы не хотим добавлять в лог
    ORM::configure('logging', $isLoggingEnabled);

Подробности аутентификации в базе данных
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметры: ``username`` и ``password``

Некоторые адаптеры баз данных (такие как MySQL) требуют раздельной передачи имени пользователя (username) и пароля (password) в DSN строке. Эти параметры позволяют вам передать их. Типичная настройка соединения MySQL может выгялдеть так:

.. code-block:: php

    <?php
    ORM::configure('mysql:host=localhost;dbname=my_database');
    ORM::configure('username', 'database_user');
    ORM::configure('password', 'top_secret');

Или вы можете соединить настройку соединения в одну строку, используя массив конфигурации:

.. code-block:: php

    <?php
    ORM::configure(array(
        'connection_string' => 'mysql:host=localhost;dbname=my_database', 
        'username' => 'database_user', 
        'password' => 'top_secret'
    ));

Наборы результатов
^^^^^^^^^^^

Параметр: ``return_result_sets``

Коллекция результатов данных может быть возвращена в качестве массива (по-умолчанию) или в виде результирующего набора.
Смотрите документацию о `find_result_set()` для более подробной информации.

.. code-block:: php

    <?php
    ORM::configure('return_result_sets', true); // возвращает результирующий набор


.. примечание::

   Рекомендуется настроить ваши проекты для работы с результирующими наборами, так как они более гибкие.

PDO Параметры драйвера
^^^^^^^^^^^^^^^^^^

Параметр: ``driver_options``

Некоторые адаптеры базы данных требуют (или позволяют использовать) массив параметров конфигурации для конкретного драйвера. Это позволяет вам передавать эти параметры через конструктор PDO. Для более подробной информации, смотрите `документацию PDO`_. Например, чтобы заставить драйвер MySQL использовать кодировку UTF-8 для соединения:

.. code-block:: php

    <?php
    ORM::configure('driver_options', array(PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8'));

PDO Режим ошибок
^^^^^^^^^^^^^^

Параметр: ``error_mode``

Можно использовать для установки параметра ``PDO::ATTR_ERRMODE`` у класса соединения с базой данных, используемой Idiorm. Должна быть передана одна из объявленных в классе констант PDO. Пример:

.. code-block:: php

    <?php
    ORM::configure('error_mode', PDO::ERRMODE_WARNING);

Параметром по-умолчанию является ``PDO::ERRMODE_EXCEPTION``. Для более подробной информации о доступных режимах ошибок, смотри `документация PDO - присвоение атрибута`_.

PDO доступ к объекту
^^^^^^^^^^^^^^^^^

Если он когда-то потребуется, то объект PDO, используемый в Idiorm может быть получен напрямую через ``ORM::get_db()``, или установлен напрямую через ``ORM::set_db()``. Однако это редкое явление.

После того, как заявление было выполнено с помощью любых средств, таких, как ``::save()``
или ``::raw_execute()``, испольуземый экземпляр ``PDOStatement`` может быть получен через ``ORM::get_last_statement()``. Это может быть полезно для доступа к ``PDOStatement::errorCode()``, если исключения в PDO отключены, или для доступа к методу ``PDOStatement::rowCount()``\, который возвращает различные результаты, относительно основной базы данных. Чтобы узнать больше, смотри `PDOStatement документация`_.

Идентификатор символа кавычек
^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметр: ``identifier_quote_character``

Устанавливает символ, используемый для идентификации кавычек (например имени таблицы, имени столбца). Если не задан, то будет определен автоматически в зависимости от драйвера базы данных, используемого в PDO.

ID столбца
^^^^^^^^^

По-умолчанию, ORM предполагает, что у всех ваших таблиц есть колонка первичного ключа(primary key)
 с именем ``id``. Есть два способа это переопределить: для всех таблиц в базе данных, или на основе каждой таблицы.

Параметр: ``id_column``

Этот параметр используется для конфигурации имени столбца первичного ключа (primary key) всех таблиц. Если ваша ID-колонка имеет название ``primary_key``, используйте:

.. code-block:: php

    <?php
    ORM::configure('id_column', 'primary_key');

Вы можете указать `составной первичный ключ`_, используя массив:

.. code-block:: php

    <?php
    ORM::configure('id_column', array('pk_1', 'pk_2'));

Примечание: Если вы используете столбец с auto-increment в составном первичном ключе, то он должен быть объявлен первым в массиве.

Параметр: ``id_column_overrides``

Этот параметр используется для указания имени столбца первичного ключа для каждой таблицы отдельно. Он принимает ассоциативный массив, описывающий название таблиц и имен столбцов. Например, если, имена ID-столбца включают имя таблицы, то вы можете использовать следующую настройку:

.. code-block:: php

    <?php
    ORM::configure('id_column_overrides', array(
        'person' => 'person_id',
        'role' => 'role_id',
    ));

Как и параметр ``id_column``, вы можеите указать составной первичный ключ, используя массив.

Limit clause style
^^^^^^^^^^^^^^^^^^

Параметр: ``limit_clause_style``

Вы можете установить стиль ограничения в конфигурации. Это делается для облегчения ограничений в стиле MS SQL, использующий синтаксис ``TOP``\.

Допустимыми значениями являются ``ORM::LIMIT_STYLE_TOP_N`` и ``ORM::LIMIT_STYLE_LIMIT``.

.. примечание::

    Если драйвер PDO, который вы используете: sqlsrv, dblib или mssql то Idiorm
    автоматически выберет значение ``ORM::LIMIT_STYLE_TOP_N``\, пока вы его не переопределите.

Лог запросов
^^^^^^^^^^^^^

Параметр: ``logging``

Idiorm может записывать в лог все вызываемые запросы. Для включения лога запросов, установите параметр
``logging`` на ``true`` (по-умолчанию он имеет значение ``false``).

Когда лог запросов включен, вы можете использовать два статических метода для доступа к логу. ``ORM::get_last_query()`` возвращает самый последний вызванный запрос. ``ORM::get_query_log()`` возвращает массив всех вызванных запросов.

Logger запросов
^^^^^^^^^^^^

Параметр: ``logger``

.. примечание::

    Вы должны включить ``logging`` для использования этого параметра.

Можно передать в этот параметр конфигурации ``callable``\, который будет вызываться для каждого запроса, вызываемого idiorm. В PHP ``callable`` зовется все, что может быть вызвано, как если бы это была функция. Чаще всего это будет представляться в виде анонимной функции.

Это полезный параметр, если вы хотите отслеживать лог запросов с помощью внешней библиотеки, так как он позволяет получить все то, что происходит внутри функций.

.. code-block:: php

    <?php
    ORM::configure('logger', function($log_string, $query_time) {
        echo $log_string . ' in ' . $query_time;
    });

Кэширование запросов
^^^^^^^^^^^^^

Параметр: ``caching``

Idiorm может кэшировать выполняемые запросы во время обращения. Для включения кэширования запросов, установите параметру ``caching`` значение ``true`` (по-умолчанию имеет значение ``false``\).

.. code-block:: php

    <?php
    ORM::configure('caching', true);
    
    
Параметр: ``caching_auto_clear``

По-умолчанию, кэш Idiorm никогда не очищается. Если вы хотите, чтобы он очищался после сохранения, установите параметр ``caching_auto_clear`` на значение ``true``

.. code-block:: php

    <?php
    ORM::configure('caching_auto_clear', true);

Когда включено кэширование запросов, Idiorm будет кэшировать результаты каждой выборки
``SELECT``\, которая была вызвана. Если Idiorm сталкивается с запросом, который уже был вызван, то он извлечет результаты прямо из кэша и не будет обращаться к базе данных.

Предупреждения и подводные камни
''''''''''''''''''''

-  Обратите внимание что для кэширования используется внутренняя память, сохраняющаяя данные в пределах одного запроса. Это *не* замена для существующих систем кэширования вроде `Memcached`_.

-  Кэш Idiorm построен очень просто, и не пытается стать недействительным, если данные изменились. Это означает, если вы выполните запрос для извлечения каких-то данных, измените их и сохраните, а далее вызовите такой же запрос снова, то результаты будут устаревшими (т.е., они не будут отражать изменений). Это может привести к потенциальным трудоуловимым ошибкам в вашем приложении. Если у вас включено кэширование и вы заметили странное поведение, отключите его и попробуйте снова. Если вам все-таки нужно выполнять такие операции, но вы хотите оставь кэш включенным, то можно вызвать метод ``ORM::clear_cache()`` для очистки всех кэшированных запросов.

-  Включение кэширования увеличивает расход памяти для приложения, так как все строки базы данных, полученные по время каждого запроса будут храниться в памяти. Если вы работаете с большими объемами данных, то лучше будет отключить кэш.

Произвольное кэширование
''''''''''''''

Если вы хотите использовать произвольные функции кэширования, вы можете задать их в параметрах конфигурации. 

.. code-block:: php

    <?php
    $my_cache = array();
    ORM::configure('cache_query_result', function ($cache_key, $value, $table_name, $connection_name) use (&$my_cache) {
        $my_cache[$cache_key] = $value;
    });
    ORM::configure('check_query_cache', function ($cache_key, $table_name, $connection_name) use (&$my_cache) {
        if(isset($my_cache[$cache_key])){
           return $my_cache[$cache_key];
        } else {
        return false;
        }
    });
    ORM::configure('clear_cache', function ($table_name, $connection_name) use (&$my_cache) {
         $my_cache = array();
    });

    ORM::configure('create_cache_key', function ($query, $parameters, $table_name, $connection_name) {
        $parameter_string = join(',', $parameters);
        $key = $query . ':' . $parameter_string;
        $my_key = 'my-prefix'.crc32($key);
        return $my_key;
    });


.. _документацию PDO: http://php.net/manual/ru/pdo.construct.php
.. _документация PDO - присвоение атрибута: http://php.net/manual/ru/pdo.setattribute.php
.. _PDOStatement документация: http://php.net/manual/en/class.pdostatement.php
.. _Memcached: http://www.memcached.org/
.. _составной первичный ключ: https://ru.wikipedia.org/wiki/%D0%9F%D0%B5%D1%80%D0%B2%D0%B8%D1%87%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BB%D1%8E%D1%87#.D0.9F.D1.80.D0.BE.D1.81.D1.82.D1.8B.D0.B5_.D0.B8_.D1.81.D0.BE.D1.81.D1.82.D0.B0.D0.B2.D0.BD.D1.8B.D0.B5_.D0.BA.D0.BB.D1.8E.D1.87.D0.B8

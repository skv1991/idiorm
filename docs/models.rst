Модели
======

Получение данных из объектов
~~~~~~~~~~~~~~~~~~~~~~~~~

Итак, когда вы получили набор записей (объектов) в результате запроса, то можете получить доступ к свойствам у этих объектов (значениям, которые хранятся в столбцах соответствующих таблиц) двумя способами: используя метод ``get``\, или напрямую обращаясь к свойству объекта:

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->find_one(5);

    // Следующие два варианта записи являются эквивалетными
    $name = $person->get('name');
    $name = $person->name;

Так же есть возможность получить все данные внутри экземпляра ORM используя метод ``as_array``\. Он вернет ассоциативынй массив, описывающий имена столбцов (ключи) с их значениями.

Метод ``as_array`` принимает в качестве необязательного аргумента названия столбцов. Если был передан один или более таких аргументов, будут возвращены только совпадающие имена столбцов.

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->create();

    $person->first_name = 'Fred';
    $person->surname = 'Bloggs';
    $person->age = 50;

    // Вернет array('first_name' => 'Fred', 'surname' => 'Bloggs', 'age' => 50)
    $data = $person->as_array();

    // Вернет array('first_name' => 'Fred', 'age' => 50)
    $data = $person->as_array('first_name', 'age');

Обновление записей
~~~~~~~~~~~~~~~~

Для обновления базы данных, измените одно или более свойств объекта, затем вызовите метод ``save`` для принятия измененных данных. Опять же, можно изменять значения свойств объектов используя метод ``set`` или напрямую указать значение свойства. Используя метод ``set`` можно так же обновить множество свойств за раз, передав в метод ассоциативный массив:

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->find_one(5);

    // Следующие два вида записи эквивалентны
    $person->set('name', 'Bob Smith');
    $person->age = 20;

    // А это эквивалентно двум присваиваниям выше
    $person->set(array(
        'name' => 'Bob Smith',
        'age'  => 20
    ));

    // Синхронизируем объект с базой данных
    $person->save();

Свойства, содержащие выражения
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Можно задать свойства модели, содержащие выражения из базы данных, используя метод ``set_expr``\.

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->find_one(5);
    $person->set('name', 'Bob Smith');
    $person->age = 20;
    $person->set_expr('updated', 'NOW()');
    $person->save();

Значение у ``updated`` столбца будет вставлено в запрос в чистом виде, позволяя тем самым базе данных выполнить любую нужную нам функцию - в данном случае ``NOW()``\.

Создание новых записей
~~~~~~~~~~~~~~~~~~~~

Для добавления новой записи, сначала нужно создать "пустой" экземпляр объекта. Затем задать значения объекта как у обычного, и сохранить их.

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->create();

    $person->name = 'Joe Bloggs';
    $person->age = 40;

    $person->save();

После сохранения объекта, вы можете вызвать его метод ``id()`` для определения сгенерированного первичного ключа, который был присвоен базой данных.

Проверка, было ли изменено свойство
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для проверки, было ли свойство объекта изменено с тех пор, как он был создан (или последний раз сохранен), вызовите метод ``is_dirty``\:

.. code-block:: php

    <?php
    $name_has_changed = $person->is_dirty('name'); // Вернет true или false

Удаление записей
~~~~~~~~~~~~~~~~

Для удаления объекта из базы данных, вызовите его метод ``delete``\.

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->find_one(5);
    $person->delete();

Для удаления более одного объекта из базы данных, постройте запрос:

.. code-block:: php

    <?php
    $person = ORM::for_table('person')
        ->where_equal('zipcode', 55555)
        ->delete_many();


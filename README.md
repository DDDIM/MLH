
Это форк плагина https://github.com/etcipnja/MLH с целью перевода.

# Задача:

Этот плагин придет на помощь когда нужно выполнить одно и то же действие для множества растений.  

# Решение:

Выполнять следующие действия:

- Применить фильтр для выбора нужных растений (например, выбрать все морковки (carrots) со статусом "Запланировано" (planned));
- Выполнить начальную функцию (например, взять насадку для посадки);
- Для каждого растения:
    - Выполнить функцию перед движением к координате растения (например, схватить семечку);
    - Переместиться к координате растения;
    - Выполнить функцию, находять на координате растения;
    - Обновить мета-данные растения (например, сменить статус морковок на "Посажено"  (planted));
- Выполнить заключительную функцию (например, вернуть на базу насадку для посадки);

# Подробное параметров:

- Фильтр растений по имени
    - Фильтровать растения по названиям (разделяем запятыми). Например: "Carrot, Beets", не зависит от регистра, '*' означает "Выбрать все"
    - Можно инвертировать вывод, используя '!' как перывый элемент. Например "!, Carrot, Beets" позволяет выбрать все, кроме моркови и свеклы
- Фильтр растений по мета-данным
    - Мета-данные - это информация о растении в формате (Ключ:Значение), привязанная к данному экземпляру растения. К сожалению, её пока невозможно увидеть где-нибудь в Дизайнере грядки (кроме статуса). 
- Начальная функция
    - Имя функции, которая будет исполнена в самом начале. К сожалению, на данный момент, поддерживаются только англоязычные имена. Если ничего не должно исполняться, впишите "None".
- Фукция перед движением к следующей точке
    - Функция, которую следует исполнить перед перемещением головы FarmBot-а к следующему растению (исполняется для каждого растения);
- Фукция после движения к следующей точке
    - Функция, которая будет исполнена на координате растения (исполняется для каждого растения);
- Заключительная функция
    - Заключительная функция  (исполняется единоразово)
- Мета-данные для записи
    - У каждого растения будут обновлены мета-данные после успешного завершения функции, которая исполняется на координате растения. Более подробно про формат мета-данных описано в примере ниже;
- Значение Z-координаты при движении
    - Z-координата при движении к очередному растению;
- Действие (test - тест, real - исполнить)
    - Впишите "test" для запуска плагина в тестовом режиме. При этом робот никуда не двинется и мета-данные обновлены не будут. Плагин просто проверит корректность введенных параметров; 
    - Или впишите "real" - для запуска плагина на исполнение;

Формат полей мета-данных строго определен. На языке Python это называется список пар кортежей (list of tupple pairs). Вот пример:

```
[ ( ‘key1’, ‘value1’ ) , ( ‘key2’, ‘value2’ ) ]
```

Вы можете использовать любые строки в качестве Ключа и Значения. 

Пользуясь списками пар Ключ и Значение, можно фильтровать растения по нескольким критериям, либо обновлять сразу несколько полей мета-данных. Если вы не хотите использовать мета-данные, впишите "None" в соответствующую строку.

Есть несколько Ключей и Значений со специальным назначением:
- Ключ 'plant_stage' не имеет отношения к мета-данным, вместо этого он работает с видимым для пользователя атрибутом "Статус";
- Ключ 'planted_at' - тоже не является мета-данными, 

There are special keys and values with special meaning:
- key ‘plant_stage’ will not deal with metadata, instead it works with the attribute “Status” visible
in Farm Designer for every plant. Only valid values that go with this key are ‘planned’, ‘planted’ and ‘harvested’
- key 'planted_at' - also not stored in meta - this is the date when plant was planted. It's value shall be in the format
YYYY-MM-DD (Example: "2018-04-15")
- key ‘del’ in SAVE filed causes to delete existing meta data for this plant. If ‘value' is ‘*’ - all metadata is
deleted, otherwise only one key specified in ‘value’ is deleted
- value ‘today’ is replaced with actual today’s date. In FILTER you can write ‘!today’ which means “not today’.

# Examples:

Seed all "planned" Carrots and mark them "planted"
```
- FILTER BY PLANT NAME:             Carrot
- FILTER BY META DATA:              [('plant_stage','planned')]
- INIT SEQUENCE NAME:               Pickup seeder  (or whatever the name you have)
- SEQUENCE NAME BEFORE NEXT MOVE:   Pickup a seed
- SEQUENCE NAME AFTER MOVE:         Plant a seed
- END SEQUENCE NAME:                Return seeder
- SAVE IN META DATA:                [('plant_stage','planted')]
```

Please note that if you interrupt this sequence and restart it - it won't start seeding again from the beginning because
already seeded plants are marked as "planted" and won't be selected again in the next run.


Water all "planted" Carrots that have not been watered today
```
- FILTER BY PLANT NAME:             Carrot
- FILTER BY META DATA:              [('plant_stage','planted'), ('last_watering','!today')]
- INIT SEQUENCE NAME:               Pickup watering nozzle
- SEQUENCE NAME BEFORE NEXT MOVE:   None
- SEQUENCE NAME AFTER MOVE:         Water light
- END SEQUENCE NAME:                Return watering nozzle
- SAVE IN META DATA:                [('last_watering','today')]
```

Delete all meta data from all plants (does not affect plant_stage)
```
- FILTER BY PLANT NAME:             *
- FILTER BY META DATA:              None
- INIT SEQUENCE NAME:               None
- SEQUENCE NAME BEFORE NEXT MOVE:   None
- SEQUENCE NAME AFTER MOVE:         None
- END SEQUENCE NAME:                None
- SAVE IN META DATA:                [('del','*')]
```
Delete watering tag from all plants that were watered today
```
- FILTER BY PLANT NAME:             *
- FILTER BY META DATA:              [('last_watering','today')]
- INIT SEQUENCE NAME:               None
- SEQUENCE NAME BEFORE NEXT MOVE:   None
- SEQUENCE NAME AFTER MOVE:         None
- END SEQUENCE NAME:                None
- SAVE IN META DATA:                [('del','last_watering')]
```

Sets up the date when the plants were planted
```
- FILTER BY PLANT NAME:             Carrots
- FILTER BY META DATA:              [('plant_stage','planted')
- INIT SEQUENCE NAME:               None
- SEQUENCE NAME BEFORE NEXT MOVE:   None
- SEQUENCE NAME AFTER MOVE:         None
- END SEQUENCE NAME:                None
- SAVE IN META DATA:                [('planted_at','2018-04-01')    #YYYY-MM-DD
```


# Intelligent watering (iWatering):

Intelligent watering tries to solve a problem that watering shall depend of:
- plant size
- plant age
- weather condition

To engage iWatering mode you need to provide AFTER sequence name that has 'water' and 'mlh' in it (For example:
"Water [MLH]"). This sequence shall be doing the following
- opening watering valve
- waiting 1 sec
- closing watering valve

It can also do whatever you want, the only important part is "waiting"
The idea is that Farmware will update the "waiting" duration basing on its understanding of how much water this
particular plant needs. Note: My assumptions about this may be different from yours - if you are not happy with it -
fork my project and help yourself.

Weather reading is taken from my other farmware "Netatmo". Please note that MLH doesn't call Netatmo explicitly. I
recommend to create a sequence like "Water All" and call Netatmo and MLH from there one after another.

iWatering skips the watering today if:
- plant was already watered today
- there was a rain today >1mm
- there was a rain yesterday >10mm
- there was a rain 2 days ago >20mm

IMPORTANT: The amount of watering is calculated basing on plant's age - make sure your planted_at date is set correctly.
See above for example.

Algorithm to calculate the amount of watering:
- get plant spread from openfarm
- adjust spread to plant's age
- convert spread (mm) into volume (ml) using my best guess
- convert ml into ms assuming that water nozzle produces 80ml every 1000ms
- update the delay in watering sequence

If you want to provide custom sequence for watering of your particular plant name - call it so it has 'water', 'mlh' and
<your_plant_name> in its name. In this case this sequence will be called once for all plants of this name and no
built-in watering will be performed. For example if farmware finds "Water [MLH] - Carrot" - this sequence will be called
when all carrots have to be watered and "Water [MLH]" will NOT be called.

# Installation:

Use this manifest to register farmware
https://raw.githubusercontent.com/etcipnja/MLH/master/MLH/manifest.json

# Bugs:

I noticed that if you change a parameter in WebApplication/Farmware form - you need to place focus on some other
field before you click "RUN". Otherwise old value is  passed to farmware script even though the new value
is displayed in the form.

In Intelligent Watering mode I need to update  watering sequence and sync it from within the farmware.
Sync function doesn't work stable sometimes. As the result the correct duration is not pushed to the bot and watering
will be wrong. This is pretty important problem as it may ruin your plants with excessive watering. I plan to submit
ticket to support as it is a problem of the platform, not the farmware.

# Credits:

The original idea belongs to @rdegosse with his Loop-Plants-With-Filters. https://github.com/rdegosse/Loop-Plants-With-Filters/blob/master/README.md

Thank you,
Eugene


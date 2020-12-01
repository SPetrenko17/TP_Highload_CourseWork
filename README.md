# TP_Highload_CourseWork
# Проектирование высоконагруженных систем. Курсовой проект


<h2>1. Тема</h2>
Shazam. Сервис распознавания музыки. 

<h2>2. Оценка диапазона нагрузки</h2>
Достаточно очевидно, что музыка, ради которой человек пользуется сервисом распознавания - будет
скачана/добавлена на устройство. 

Сравним информацию из разных источников:

- Активная аудитория Shazam - 170 миллионов пользователей в мире

- Согласно [оценке](https://musically.com/2020/02/19/spotify-apple-how-many-users-big-music-streaming-services) консалтинговой компании Midia Research, которая регулярно обновляет свои оценки, 
на конец марта 2020 года музыкальными сервисами ежедневно пользуются 400 миллионов людей.

- 5% всей скачиваемой/добавляемой на стриминговых сервисах музыки, приходится на предложенную данным сервисом. [Источник](https://expandedramblings.com/index.php/shazam-statistics)

- Сервисом пользуется более 150 миллионов людей ежемесячно. [Источник](https://expandedramblings.com/index.php/shazam-statistics/)

- Активные пользователи Shazam совершают более 20 миллионов запросов на поиск музыки ежедневно. [Источник](https://expandedramblings.com/index.php/shazam-statistics/)

5% от 400 000 000 - 20 000 000.

Можно сделать вывод, что 20 миллионов запросов в день на распознавание музыки - нормальная оценка для данного сервиса
    

Источники:         

http://coding-geek.com/how-shazam-works/
https://blog.acrcloud.com/how-does-shazam-work
https://vk.com/blog/arhitektura-i-algoritmy-indeksatsii-audiozapisey

<h2>3. Планируемая нагрузка</h2>
Исходя из статистики и указанных выше фактов определяем такой диапазон:
- Активная аудитория - 170 миллионов
- Активных пользователей в день -  20 миллионов
- Средний RPS = 231 (20 000 000 / (24 * 60 * 60))
- Нагрузка распределена равномерно по всем часовым поясам


(СВЕЖИЕ ПРАВКИ)


<h3> (СВЕЖИЕ ПРАВКИ) </h3>

В секунду имеем 231 соединение. В пункте 5 берем время на распознавание трека - 30 секунд.

Рассчитаем среднюю и пиковую нагрузки на БД:

Из одной секунды аудио можем получить 88 блоков с хешами => имеем 88 запросов на одну секунду аудио-контента.

Передаем по веб-сокету аудио в течение n-секунд.

Поисковой запрос по хешу в худшем случае занимает 2 секунды.

Каждую секунду открывем 231 соединение.

Для каждого соединения в течение одной секунды обрабатываем 88 запросов к БД.

В итоге имеем: 231 * 88 = 20328(Средняя нагрузка) и 1300 * 88 = 11400(Пиковая) Запросов к бд в секунду (Для совпадения по уникальному хешу)


<h3> (СВЕЖИЕ ПРАВКИ) </h3>
<h3>Очень высокая нагрузка</h3>

Рассмотрим ситуацию с высоким количеством коллизий:

Минимальное время поиска хеша в заполненной на 5 миллионов треков БД - 20мс

Имеем 2^32 различных хешей на блок и 2^22 различных треков.

Рассмотрим почти невероятный случай (8.8817842e-16), когда мы слушаем трек из нового альбома 
Леди Гаги и 5 из 5 блоков (1с = 20мс * 5) 1 секунды этой песни выдают совпадение с кучей треков Мадонны.
В таком случае на каждую секунду имеем в 5 больше запросов к БД с клиента.

На массовой вечеринке 1000 человек разом решили найти трек Гаги, который играет на сцене.

1000 * 88 * 5 = 440 000 запросов в секунду

DynamoDB способна обрабатывать до 500 000 запросов в секунду




<h2>4. Логическая схема базы данных</h2>
Начнем с необходимой для распознавания музыки информации:
Для упрощения поиска музыкальных композиций их сигнатуры используются 
как ключи в хэш-таблице. Ключам соответствуют значения времени,
когда набор частот, для которых найдена сигнатура, 
появился в произведении, и идентификатор самого произведения
(название песни и имя исполнителя, например).
Вот вариант того, как подобные записи могут выглядеть в базе данных.

| Хеш отрывка  | Время начала отрывка  | Песня|
| :------------ |:---------------:| -----:|
| 30 51 99 121 195    | 53.52 | **Песня A исполнителя A**|
| 33 56 92 151 185     | 12.32        |   Песня B исполнителя B |
| 39 26 89 141 251 | 15.34        |    Песня C исполнителя C |
| 32 67 100 128 270| 78.43       |    Песня D исполнителя D |
| 30 51 99 121 195 | 10.89        |    `Песня E исполнителя E` |
| 34 57 95 111 200 | 54.52        |    **Песня A исполнителя A** |
| 34 41 93 161 202 | 11.89        |    `Песня E исполнителя E` |

Если обработать таким способом некую библиотеку музыкальных записей,
можно будет построить базу данных с полными сигнатурами каждого произведения.


В данном примере для большего понимания алгоритма имеется столбец со временем начала отрывка.
Алгоритмы комбинаторного хеширования позволяют хранить хеш более оптимальным образом,
а так же лишают необходимости хранения времени начала отрывка.


Исходя из того, что подобная система нуждается в масштабировании, а главная сущность - музыка с хешами, имеет смысл 
рассмотреть нереляционные базы данных
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/hashTable.png?raw=true)



У одного исполнителя может быть большое количество треков, выпущенных совместно с другими исполнителями. 
Почти все стриминговые сервисы предпочитают связь "песня-исполнитель", как "многие к одному"
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/Meladze.png?raw=true)


В добавок ко всему, поиск всегда будет осуществляться по хешу определенного временного сектора трека.
NoSQL решения подойдут лучше.

Основные типы запросов:

 - Добавление записи о песне в базу с хешом
 
 - Удаление записи о песен по хешу
 
 - Поиск записи песне по хешу
 


https://habr.com/ru/company/wunderfund/blog/275043/

<h2>5. Физическая система хранения</h2>
[Кратко про техническую часть](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamIndustrialAudioSearch.pdf)


Используя комбинаторное хеширование можно ускорить поиск до 10 тысяч раз,
но потребуется в 10 раз больше места для хранения хешей.

Кроме того, каждый хэш может быть упакован в 32-битное целое число без знака.
Каждый хэш также связан с временным смещением от начала соответствующего файла
до его опорной точки(момент с которого распознаётся отрывок аудио-файла - блок),
это позволяет избежать лишних вычислений при поиске.

Например, если мы записываем двухканальный звук с размером образца равным 16 бит
и с частотой дискретизации 44100 Гц, одна секунда такого звука займёт
352 Кб памяти (44100 образцов * 32 бита * 2 канала)

В документации говорится о блоках с размером 4 Кб,
тогда на каждую секунду потребуется 88 блоков данных (Просто для статистики и пункта о запросах).

В среднем, длина трека составляет 3 минуты или же 180 секунд.
Тогда для хранения хешей всех блоков одного аудио-файла потребуется 180 сек * 352 Кб = 64 Мб
Для сравнения: 180 сек * 2 канала * 44100 Гц * 8 бит/сек = 15 Мб - размер исходного трека

Для хранения дополнительной информации об треках и исполнителях выделим 5мб, с

Предположим, что в плане "Минимум" цель - догнать половину базу Shazam (5 миллионов треков), как размер 1/20 библиотеки треков "Вконтакте"
Просьба не пугаться, т.к. библиотека "Вконтаке" содержит минимум по 100 копий или ExtraBass/VasyaMix версий разных треков.

5 000 000 * 64мб = 320 Тб - Хранение хешей для 11 миллионов треков
11 000 000 * 2мб = 22 Tб - Хранение информации о треках

Имеем 231 активных пользователей в секунду. Возьмем с запасом на случай внезапной тусовки на 1000 человек, где каждый захочет распознать музыку - 1300 активных пользователей
Для каждого пользователя выделяем 30 секунд на распознавание трека и получение информации о нем.(Учитываем медленных клиентов)
Тогда, имеем при среднем размере записи об сессии - 1 Мб:
1300 * 60 * 1 Мб = 39 Гб на хранение сессий пользователей.


Для уменьшения количества инстансов будем использовать массивы RAID-1. 
C учетом бекапов данных потребуется около 1 Пб.

Для этого сервиса я выбрал DynamoDB от Amazon по многим причинам:
- Позволяет реализовать более 500 000 операций в секунду.
- Имеется встроенный функционал для работы с хранением пользовательских сессий (DynamoDB Session Handler)
- Использование DynamoDB для хранения сеансов устраняет проблемы, возникающие при обработке сеансов в распределенном веб-приложении, путем перемещения сеансов из локальной файловой системы в общее расположение.
- Является быстрым, масштабируемым, простым в настройке инструментом и автоматически выполняет репликацию данных.

<h3> (СВЕЖИЕ ПРАВКИ) </h3>

<h3>Оценка стоимости</h3>

Общее количество операций записи при индексировании:
5 миллионов треков, на индексирование потребуется около месяца.
5 000 000 * (88 блоков/сек * 180cек) + 5 000 000 записей треков = 79 205 000 000

Предположим, что в месяц будет появляться около 5 000 новых композиций.
5 000 * (88 блоков/сек * 180cек) + 5 000 записей треков = 79 205 000 000

Имеем 20 000 000 запросов в день.
20 000 000 * 30 = 600 000 000 
600 000 000 * 2 = 1 200 000 000 запросов к базе

| Период  | Общее количество операций записи  | Общее количество операций чтения|
| :------------ |:---------------:| -----:|
| Индексирование(первый месяц) | 79 205 000 000 | | не требуется
| Индексирование(каждый n-ый месяц) | 79 205 000 | | не требуется
| Поиск треков(месяц) | не требуется | 1 728 000 000  |



Актуальная стоимость операций AmazonDB: 
- 1,25 USD за 1 миллион операций записи.
- 0,25 USD за 1 миллион операций чтения

Тогда:

- 99 000 $ - Плата за первый месяц во время старта индексирования, чтобы попытаться догнать шазам

- 100$ - Ежемесечная плата за добавление новых треков в базу.

20000 * 60 * 60 * 24 * 0.25$/PerM / 10^6 =1728$
- 1728$ - Ежемесячная плата за поиск 

Суммарно: около 100 000$

Воспользовался калькулятором:

Unit conversions
 - Inbound:
 - Internet: 300 TB per month x 1024 GB in a TB = 307200 GB per month
Outbound:
Internet: 100 TB per month x 1024 GB in a TB = 102400 GB per month
Pricing calculations
Inbound:
Internet: 307200 GB x 0 USD per GB = 0.00 USD
Outbound:
- Internet: Tiered pricing for 102400 GB:
- 10239 GB x 0.09 USD per GB = 921.51 $
- 40960 GB x 0.085 USD per GB = 3481.60 $
- 51200 GB x 0.07 USD per GB = 3584.00 $
- Data Transfer cost (ежемесячно): 7,987.11 $


Стоимость сервисов(c учетом заполненной базы и максимальной нагрузки):

Amazon EC2 Instance Savings Plans (ежемесячно) - 34,304.16 $

Amazon Elastic Block Storage (EBS)(ежемесячно) - 22,119.30 $


Всего(без учета стоимости операций): 64,410.57 $

Всего: около 165 000$ в первый месяц и 65 000$ в каждый последующий

<h2>6. Прочие технологии</h2>
<h3>6.1 Языки программирования</h3>

Backend - Go. Будут использоваться все возможности языка:

- Горутины.

- Пулы Воркеров.

- Параллельность работы с сетью.

Frontend/Mobile - TypeScript.
В качестве формальных языков разметки - css3 + html5

<h3>6.2 Фреймворки</h3>

В качестве основного фреймворка на бекенде будет BeeGo. 

- Полноценная ORM

- Кеш

- Работа с сессиями

- Интернационализация (i18n)

- Мониторинг и перегрузка приложения при разработке

- Инструменты для деплоя

- Роутинг

Для работы с базой данных dynamoDB будет использоваться GoAMZ. 
Это хороший опен-сорс продукт, позволяющий удобно работать с сервисами Amazon. 

Для реализации кросс-платформенного приложения подойдет фреймворк React Native в связке с TypeScript.
React Native позволяет писать полноценные мобильные приложения, применяя все возможности фронтенд разрабоки.
React Native, как и React обладает рядом полезных особенностей:

- Поддержка JSX/TSX синтаксиса

- Виртуальное DOM-дерево, позволяющее экономить ресурсы клиента и оптимизировать рендеринг.

- Компонентный подход

- Универсальность и совместимость с любыми js-библиотеками

- Отрисовка приложения будет происходить сразу в нативные контролы операционной системы, под которую будет собрано приложение.

<h3>6.3 Протоколы взаимодействия</h3>
Протоколы связи между приложением и бекендом - https и Websocket.
Websocket позволит осуществить передачу аудио-контента пока пользователь имеет
возможность распознать окружающую его музыку. Реализовать такой стриминг 
на мобильных можно с помощью SwiftWebSocket для IOS и базовых инструментов Java/Kotlin.

На случай реализации кросс-платформенного на React Native, передача по websocket будет даже удобнее.


<h2>7. Расчет нагрузки и потребного оборудования</h2>
Согласно документации Shazam, для базы данных из примерно 200 тысяч треков
, время поиска составляет порядка 5-500 миллисекунд, в зависимости от нагрузки. 
На момент 


Согласно [статье](https://vk.com/blog/arhitektura-i-algoritmy-indeksatsii-audiozapisey/) об индексировании музыкальной библиотеки ВК, насчитывающей 
около 400 миллионов треков. Просьба не пугаться этого числа,
В "Вконтакте", движок индексирования и поиска работает на 32 машинах, написан на чистом Go.
Используются все современные возможности Go.
Данные оптимизированы посредством использования структур данных container/heap вместо массивов.
По данным на 2017 год с версией Go 1.6.2, реализация сервиса, целью которого является индексация всей библиотеки треков
позволила сделать это за 3 месяца.
Исходя из выше описанных проблем с композициями вольных музыкантов, сократим время до пары недель.


Ранее отмечается, что использование комбинаторного хеширования дает в 10 000 раз более быстрый результат поиска совпадений в БД
На данный момент сервис Shazam позволяет найти совпадение с аудио "радио качества"
менее чем за 10 миллисекунд, с вероятной целью оптимизации, достигающей 1 миллисекунды на запрос.
См [Пункт 3.2](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamIndustrialAudioSearch.pdf?raw=true)


<h3>Расчет потребного оборудования</h3>

По [данным](https://www.tyan.com/EN/corporation/success_story_isp_GT24B2891) на 2008-2012 год, Shazam использовал сервера TYAN GT24 B2891 с двухъядерными процессорами
AMD серии Opteron 2000 и оперативной памятью DDR400 на 16 Гб. 
На тот момент у Shazam было минимум 10 миллионов пользователей из 150 стран мира. 
Ежедневно сервисом пользовались 2 миллиона человек.\
За последние 8 лет можно отметить несколько изменений:

 - Количество активных пользователей в день увеличилось в 10 раз
 
 - Скорость поиска аудио увеличилась в 30 раз
 
 - Потребность в индексировании аудиозаписей и их количестве не изменилась из-за оптимизации алгоритмов
 
Исходя из выше перечисленного, будем искать более производительные решения. Как вариант - использование облачных решений на более
производительным железом (процессоры AMD EPYC, Intel Xeon). Облачные решения позволят сэкономить на обслуживании.

В связи с развитием музыкальной индустрии, потребуется объемное хранилище данных.
Для реализации отказоустойчивого и быстрого хранилища потребуется использование RAID-1 массивов.
По RAID-1 следует отметить:

- простота реализации

- простота восстановления массива в случае отказа, благодаря копированию

- высокое быстродействие для приложений с большим RPS


Конфигурация машин, для основых сервисов приложения.

| Сервис | Количесто ядер | ОЗУ (Гб) | Количество машин |
|:------:|:------:|:------:|:------:|
| Индексирование треков| 16 | 64 | 5 |
| Поиск по хешу | 8 | 32 | 10 |
| Фронтенд | 16 | 16 | 15 |


<h2>8. Выбор хостинга / облачного провайдера и расположения серверов</h2>
Последовав примеру Shazam в выборе облачных решений, имеет смысл отказаться от 
поддержки хостинга на своей стороне.
Выбор пал на Amazon Web Service (AWS)
AWS предлагает хорошие решения Amazon EC2 для масштабируемых облачных вычислений с поддержкой высоких нагрузок.
Одним из решений является AWS с инстансами [Amazon EC2 C5](https://aws.amazon.com/ru/ec2/instance-types/c5/).
EC2 C5 отличается:
- высокопроизводительные вычисления
- пакетная обработка данных
- Минимальное решение обеспечивает пропускную способность сети до 25 Гбит/с и отлично подходит для приложений, работающих с аудио и видео контентом с минимальными задержками

По умолчанию, [Amazon AWS](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) позволяет иметь не более 20 инстансов на регион. Увеличение количества инстансов рассматривается после обращения в поддержку.
EC2 работает на основе крупной инфраструктуры с 69 зонами доступности в 22 регионах (от 2 до 5 зон доступности в каждом).
- Регион Восток США (Огайо)
- Регион Запад США (Орегон)
- Регион Запад США (Северная Калифорния)
- Регион GovCloud (Запад США)
- Регион GovCloud (Восток США)
- Регион: Канада (Центр)
- Регион Южная Америка (Сан-Паулу)
- Регион Европа (Франкфурт)
- Регион Европа (Лондон)
- Регион Европа (Париж)
- Регион Ближний Восток (Бахрейн)
- Регион Европа (Ирландия)
- Регион Европа (Милан)
- Регион Европа (Стокгольм)
- Регион AWS Африка (Кейптаун)
- Регион Континентальный Китай (Пекин)
- Регион Азия и Тихий океан (Сидней)
- Регион Азия и Тихий океан (Токио)
- Регион Азия и Тихий океан (Сеул)
- Регион Континентальный Китай (Нинся)
- Локальный регион Азия и Тихий океан (Осака)
- Регион Азия и Тихий океан (Мумбаи)
- Регион Азия и Тихий океан (Гонконг)

AWS Auto Scaling самостоятельно выполняет мониторинг приложений и
автоматически настраивает ресурсы для поддержания стабильной производительности.
При помощи консоли Route53 будет возможность изменять регионы нацеленно.

<h2>9. Схема балансировки нагрузки (входящего трафика и внутрипроектного, терминация SSL)</h2>
Балансировка нагрузки будет осуществляться с помощью AWS Elastic Load Balancing
В данном случае, с Network Load Balancer. Высокая производительность будет более приоритетна чем гибкость

В консоли управления AWS по аналогии с AmazonCloudFront достпны те же технологии маршрутизации, что и 
Amazon Route53.

Из особенностей Network Load Balancer от AWS ELB следует отметить:

- Встроенные возможности SSL/TLS‑расшифровки и управления сертификатами и терминации SSL

- Централизованное управление настройками SSL на балансировщике нагрузки

- Балансировка нагрузки на уровне 4 или уровне 7, что позволяет работать с https и websocket

- Решение проблем с медленными клиентами

<h2>10. Обеспечение отказоустойчивости</h2>
С помощью Amazon ELB создается имя хоста DNS, и любые запросы,
отправленные хост, будут делегироваться к пулу экземпляров Amazon EC2.

![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/BalanceAWS.png?raw=true)

Для обеспечения отказоустойчивости хранилищ данных, помимо инструменов AWS и DynamoDB используется 
дисковый массив с дублированием или зеркалированием.
Избыточность структуры данного массива обеспечивает его высокую отказоустойчивость.

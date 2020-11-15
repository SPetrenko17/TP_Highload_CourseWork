# TP_Highload_CourseWork
# Проектирование высоконагруженных систем. Курсовой проект


<h2>1. Тема</h2>
Shazam. Сервис распознавания музыки. 

<h2>2. Оценка диапазона нагрузки</h2>
Достаточно очевидно, что музыка, ради которой человек пользуется сервисом распознавания - будет
скачана/добавлена на устройство. 
Сравним информацию из разных источников:
- Активная аудитория Shazam - 170 миллионов пользователей в мире
- Согласно ![оценке](https://musically.com/2020/02/19/spotify-apple-how-many-users-big-music-streaming-services?raw=true) консалтинговой компании Midia Research, которая регулярно обновляет свои оценки, 
    на конец марта 2020 года музыкальными сервисами ежедневно пользуются 400 миллионов людей.
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/MIDIA_STAT.png?raw=true)
- 5% всей скачиваемой/добавляемой на стриминговых сервисах музыки, приходится на предложенную данным сервисом.![Источник](https://expandedramblings.com/index.php/shazam-statistics/?raw=true)
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamPercentage.png?raw=true)
- Сервисом пользуется более 150 миллионов людей ежемесячно.![Источник](https://expandedramblings.com/index.php/shazam-statistics/?raw=true)
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamMonthly.png?raw=true)
- Активные пользователи Shazam совершают более 20 миллионов запросов на поиск музыки ежедневно.![Источник](https://expandedramblings.com/index.php/shazam-statistics/?raw=true)
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamPerDay.png?raw=true)

5% от 400 000 000 - 20 000 000.

Можно сделать вывод, что 20 миллионов запросов в день на распознавание музыки - нормальная оценка для данного сервиса
    

Источники:         

http://coding-geek.com/how-shazam-works/
https://blog.acrcloud.com/how-does-shazam-work


https://vk.com/blog/arhitektura-i-algoritmy-indeksatsii-audiozapisey

<h2>3. Планируемая нагрузка</h2>
Исходя из статистики и указанных выше фактов определяем такой диапазон:
- Активная аудитория - 
- Активных пользователей в день -  
- Средний RPS = 231 (20 000 000 / (24 * 60 * 60))
- Нагрузка распределена равномерно по всем часовым поясам


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
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/hashTable.jpg?raw=true)



У одного исполнителя может быть большое количество треков, выпущенных совместно с другими исполнителями. 
Почти все стриминговые сервисы предпочитают связь "песня-исполнитель", как "многие к одному"
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/Meladze.png?raw=true)


В добавок ко всему, поиск всегда будет осуществляться по хешу определенного временного сектора трека.
NoSQL решения подойдут лучше.

https://habr.com/ru/company/wunderfund/blog/275043/

<h2>5. Физическая система хранения</h2>
![Кратко про техническую часть](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamIndustrialAudioSearch.pdf?raw=true)


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
тогда на каждую секунду потребуется 88 блоков данных (Просто для статистики).

В среднем, длина трека составляет 3 минуты или же 180 секунд.
Тогда для хранения хешей всех блоков одного аудио-файла потребуется 180 сек * 352 Кб = 64 Мб
Для сравнения: 180 сек * 2 канала * 44100 Гц * 8 бит/сек = 15 Мб - размер исходного трека

Предположим, что в плане "Минимум" цель - догнать базу Shazam (11 миллионов треков), как размер 1/36 библиотеки треков "Вконтакте"
Просьба не пугаться, т.к. библиотека "Вконтаке" содержит минимум по 100 копий или ExtraBass/VasyaMix версий разных треков.
400 000 000 * 64мб

Для этого сервиса я выбрал DynamoDB от Amazon по многим причинам:
- Позволяет реализовать более 500 000 операций в секунду.
- Имеется встроенный функционал для работы с хранением пользовательских сессий (DynamoDB Session Handler)
- Использование DynamoDB для хранения сеансов устраняет проблемы, возникающие при обработке сеансов в распределенном веб-приложении, путем перемещения сеансов из локальной файловой системы в общее расположение.
- Является быстрым, масштабируемым, простым в настройке инструментом и автоматически выполняет репликацию данных.

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
Согласно документации Shazam, для базы данных из примерно 20 тысяч треков
, время поиска составляет порядка 5-500 миллисекунд, в зависимости от нагрузки.


Согласно ![статье](https://vk.com/blog/arhitektura-i-algoritmy-indeksatsii-audiozapisey) об индексировании музыкальной библиотеки ВК, насчитывающей 
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
См ![Пункт 3.2](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/ShazamIndustrialAudioSearch.pdf?raw=true)


<h3>Расчет потребного оборудования</h3>

По ![данным](https://www.tyan.com/EN/corporation/success_story_isp_GT24B2891) на 2008-2012 год, Shazam использовал сервера TYAN GT24 B2891 с двухъядерными процессорами
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



<h2>8. Выбор хостинга / облачного провайдера и расположения серверов</h2>
Последовав примеру Shazam в выборе облачных решений, имеет смысл отказаться от 
поддержки хостинга на своей стороне.
Выбор пал на Amazon Web Service (AWS)
AWS предлагает хорошие решения Amazon EC2 для масштабируемых облачных вычислений с поддержкой высоких нагрузок.
Одним из решений является AWS с инстансами ![Amazon EC2 C5](https://aws.amazon.com/ru/ec2/instance-types/c5/).
EC2 C5 отличается:
- высокопроизводительные вычисления
- пакетная обработка данных
- Минимальное решение обеспечивает пропускную способность сети до 25 Гбит/с и отлично подходит для приложений, работающих с аудио и видео контентом с минимальными задержками

По умолчанию, ![Amazon AWS](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) позволяет иметь не более 20 инстансов на регион. Увеличение количества инстансов рассматривается после обращения в поддержку.
EC2 работает на основе крупной инфраструктуры с 69 зонами доступности в 22 регионах (от 2 до 5 зон доступности в каждом).
- Регион Восток США (Огайо)
- Регион Запад США (Орегон)
- Регион Запад США (Северная Калифорния)
- Регион GovCloud (Запад США)-
- Регион GovCloud (Восток США-)
- Регион: Канада (Центр-)
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
- Локальный регион Азия и Тихий океан (Осака)1
- Регион Азия и Тихий океан (Мумбаи)
- Регион Азия и Тихий океан (Гонконг)

AWS Auto Scaling самостоятельно выполняет мониторинг приложений и
автоматически настраивает ресурсы для поддержания стабильной производительности.

<h2>9. Схема балансировки нагрузки (входящего трафика и внутрипроектного, терминация SSL)</h2>
Балансировка нагрузки будет осуществляться с помощью AWS Elastic Load Balancing
В данном случае, с Network Load Balancer. Высокая производительность будет более приоритетна чем гибкость
![](https://github.com/SPetrenko17/TP_Highload_CourseWork/blob/main/media/BalanceAWS.png?raw=true)
Из особенностей Network Load Balancer от AWS ELB следует отметить:
- Встроенные возможности SSL/TLS‑расшифровки и управления сертификатами и терминации SSL
- Централизованное управление настройками SSL на балансировщике нагрузки
- Балансировка нагрузки на уровне 4 или уровне 7, что позволяет работать с https и websocket
- Решение проблем с медленными клиентами

<h2>10. Обеспечение отказоустойчивости</h2>
С помощью Amazon ELB создается имя хоста DNS, и любые запросы,
отправленные хост, будут делегироваться к пулу экземпляров Amazon EC2.
Для обеспечения отказоустойчивости хранилищ данных, помимо инструменов AWS и DynamoDB используется 
дисковый массив с дублированием или зеркалированием.
Избыточность структуры данного массива обеспечивает его высокую отказоустойчивость.

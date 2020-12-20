# Курсовой проект по курсу highload в технопарке

## 1. **Выбор темы**
Сервис по планированию путешествий (TripAdvisor)

## 2. **Определение возможного диапазона нагрузок подобного проекта**
### TripAdvisor
Месячная аудитория- 490 миллионов [Источник](https://review42.com/tripadvisor-statistics/)
Среднее время проведения на подобных сайтах- 3 минуты [Источник](https://www.similarweb.com/website/tripadvisor.com/)

## 3. **Выбор планируемой нагрузки**
Планируемая нагрузка - 50% от доли TripAdvisor, т.е
- Месячная аудитория:  
    ```490/2=245 миллиона человек в месяц```

Учитывая специфику сайта (пользователи редко возвращаются на сайт в течение одного месяца, так как выбор путешествия скорее всего дело одного захода), можем принять число ежедневно активных юзеров=245 млн/30=8.1 млн человек
    
Оценивая себя как среднестатистического пользователя, подсчитал сколько пользовательского трафика займет стандартный workflow пользователя, а именно скролл ленты:
![alt-текст](https://github.com/EvilBorsch/booking-highload/blob/main/Скриншоты%20(2).png "lenta")



За 1 минуту 40 секунд минут пользования сайтом:
На динмику:  

- Если исключить лишние запросы, идущие в рекламные сервисы и внешние сервисы аналитики, пришлось 0.46 мб трафика и 32 запроса.  
Собственно основной объем трафика приходится на статику
На статику: 

- На картинки: 3.7 мб и 386 запросов.
- На остальную статику: 0,4 Мб и 142+414=556 запросов

За сутки пользования сервисом в среднем пользователем будет израсходованно
- Для динамики: 
```0.46 * (3/1,4)=1,3 мб трафика```  

```32 * (3/1.4)=68 запросов ```  

- Для картинок:
```3.7 * (3/1,4)=7.9 мб трафика```  

```386 * (3/1.4)=827 запросов ``` 

- Для остальной статики: 
```0.4 * (3/1,4)=0.85 мб трафика```  

```556  * (3/1.4)=1191 запросов ``` 

Расчитаем средний трафик для динамики и статики: 
- Динамика: (1,3 мб * 8 100 000) * 8/1024 / (24 * 60 * 60) = 0.93 Гб/с
- Картинки: (7.9 мб * 8 100 000) * 8/1024/ (24 * 60 * 60) = 5.80 ГБ/c
- Остальная статика: (0.85 мб * 8 100 000) * 8/1024 / (24 * 60 * 60) = 0,62 ГБ/c

Расчитаем средний RPS для динамики и статики: 
- Динамика: (68 * 8 100 000) / (24 * 60 * 60) = 6400 RPS
- Картинки: (827 * 8 100 000) / (24 * 60 * 60) =  77000 RPS
- Остальная статика: (1191 * 8 100 000) / (24 * 60 * 60) = 111000 RPS

## 4. **Логическая схема базы данных**
![alt-текст](https://github.com/EvilBorsch/booking-highload/blob/main/Снимок%20экрана%202020-10-21%20в%2016.38.54.png "Схема бд")

## 5. **Физическая системы хранения**

Данные о пользователе, отелях и отзывы имеют для нас наибольший приоритет, потому что без них наше приложение потеряет ключевой функционал. Для хранения этих данных я выбрал PostgreSQL, как наиболее функциональную и надежную реляционную базу данных.

Шардинг: 
- Пользователей и отелей будем шардировать по ID.
- Каждой картинке также будет присвоенный собственный идентификатор, по которому мы и будем шардировать.
- <b> В отелях, у каждого отеля есть массив фотографий, каждый элемент которого представлен в виде id-url на фотографию на первом сервере_id-url на фотографию на втором_id-url на фотографию на 3. Если надо достать картинку, достаем из бд, сплитим по _ и будем round robinом ходить по серверам </b> 

Активные сессии пользователя будем хранить в redis. Redis имеет встроенный API для работы с Memcached, что позволяет использовать эффективное кэширование данных. А так же неблокирующую master-slave репликацию.

<b> Распределение машин и таблиц для системы хранения написано в пункте 7 </b>

## 6. **Выбор прочих технологий**

#### Backend
Golang. Перспективный язык программирования, со статической типизацией, эффективной многопоточностью, использующий аналог корутин(горутины), а так же удобным менеджером зависимостей (gomod).

#### Frontend
CSS, HTML , TypeScript - как стандарт Frontend разработки.
Webpack — Это сборщик модулей JavaScript с открытым исходным кодом он принимает модули с зависимостями и генерирует статические ресурсы, представляющие эти модули.
Sass - метаязык на основе CSS, предназначенный для увеличения уровня абстракции CSS-кода и упрощения файлов каскадных таблиц стилей.
React - обеспечивающий модульность, быстрый рендеринг, высокую run-time производительность.

#### Протоколы взаимодействия
Протокол связи между фронтендом и бэкендом - https, данные будут передаваться в формате json.
Общение между микросервисами на бэкенде будет осуществляться по протоколу gRPC, данные будут передаваться в формате protobuf

#### Обеспечение качества
Как на фронтенде, так и на бэкенде будут использованы статические и динамические анализаторы кода, интеграционные и юнит тесты. Их запуск будет автоматизирован в Gitlab CI.

#### Протоколы взаимодействия

Протокол связи между фронтендом и бэкендом - https, данные будут передаваться в формате json. 

Общение между микросервисами на бэкенде будет осуществляться по протоколу gRPC, данные будут передаваться в формате protobuf.

## 7. **Расчет нагрузки и потребного оборудования**

### Балансировка и раздача статики
Конфигурация
| CPU  (cores)  |      RAM (gb)    |  SSD (Gb) |
|:----------:|:-------------:|:------:|
| 32 |  64 | 256 | 

Исходя из размера пакета в 10KB и из документации nginx [1.](https://www.nginx.com/blog/testing-the-performance-of-nginx-and-nginx-plus-web-servers/) 
[2.](https://www.nginx.com/blog/nginx-websockets-performance/)
Для раздачи картинок: ```77000/10000 CPS= 10 серверов nginx```

<b> Rps на одну машину: 7700 RPS</b> 
<b> Трафик на одну машину: 0.58 Гб/с </b> 

Для балансировки и раздачи остальной статики нам будет достаточно двух nginx(так как наш RPS не превышает 700000), с двумя слейвами у каждого.

<b> Rps на одну машину: 27750 RPS</b> 
<b> Трафик на одну машину: 0.155 Гб/с </b> 


### Рассчет памяти
##### Активные сессии

Для хранения всех сессий пользователей(245 млн) согласно [статье](https://medium.com/@lucasmagnum/redistip-estimate-the-memory-usage-for-repeated-keys-in-redis-2dc3f163fdab) нам примерно потребуется: 245 000 000 * 220 = 54 00 000 000 байт = 54 Гб

##### Пользователи

Одна запись пользователя в таблице занимает примерно (4+30+30+30+30) 124 байта. Тогда общий объем занимаемой памяти составит 245 млн *124 б= 30 Гб.

##### Отели

Одна запись пользователя в таблице занимает примерно (4+30+30+30+30+150+4+4+30) 312 байт. Согласно статье [Статистика](https://expandedramblings.com/index.php/tripadvisor-statistics/) Число отелей= 1.2 миллиона Тогда общий объем занимаемой памяти составит 1.2 млн * 312 б= 0.374 Гб.

##### Отзывы

Одна запись комментария в таблице занимает примерно (4+4+4+200+4+8+4+4) 232 байта. Согласно статье [Статистика](https://expandedramblings.com/index.php/tripadvisor-statistics/) Число отзывов= 870 миллионов => Общий объем памяти = 870млн * 232 = 201 Гб

##### Фото

В среднем одна фотография весит около 500 кб. 
Если считать что у каждого пользователя есть аватарка, и к каждому отелю приложено в среднем 5 фото. 
Объем информации: 
245 млн* 0,5 Мб + 1,2 млн (число отелей [Статистика](https://expandedramblings.com/index.php/tripadvisor-statistics/) ) *0,5 Мб * 5= 52000 Гб
### Расчет оборудования и таблица серверов

#### Изображения: 
Исходя из приведенных выше данных нам потребуется 52 тб. для хранения всей нужной информации. 
Будем учитывать что в пункте 8 мы приняли решения создать два своих датацентра.
Будем использовать стойки под 12 SSD дисков по 2Тб для хранение картинок:

| CPU  (cores)  |      RAM (gb)    |  SSD (Tb) |
|:----------:|:-------------:|:------:|
| 32 |  64 | 24 |

Возьмем под хранилище фотографий 52 Тб/24=3 машины,с запасом 4, у каждой машины будет по 2 slave с копиями картинок, для отказоустойчивости.

<b> Итого 8 машин для фотографий</b>

#### Доступ к данным: 
Для хранения таблиц, воспользуемся менее мощными машинами, так как будем брать их с избытком

| CPU  (cores)  |      RAM (gb)    |  SSD (Tb) |
|:----------:|:-------------:|:------:|
| 16 |  32 | 1 |

Учитывая рост пользователей, оптимальным решением будет каждую таблицу хостить на двух шардах(по id), каждый шард будет находится в своем дата центре, у каждого шарда будет по 2 слейва для обеспечения отказоустойчивости. 
<b> Итого 18 машин для баз данных </b>

Так как основная нагрузка на сайт приходится на статический контент, и суммарный RPS на динамику составляет всего 6000  RPS, будем использовать машинны с данной конфигурацией: 

| CPU  (cores)  |      RAM (gb)    |  SSD (Gb) |
|:----------:|:-------------:|:------:|
| 32 |  64 | 256 | 

Разместим по машине с двумя слейвами для обеспечения отказоустойчивости в каждый датацентр.

<b> Итого 6 машин для сервисов </b>

## 8. **Выбор хостинга / облачного провайдера и расположения серверов**
Мы будем использловать свои дата центры, так как цены на облачные решения слишком велики для больших сервисов. Клиенты обращаются к нам в основном с территории Европы и США (для китайского и азиатского рынка создаются отдельные продукты, такие как daodao.com) расположим один дата центр в США(Нью-Йорк), а другой в Европе(Франфуркт).

![alt-текст](https://github.com/EvilBorsch/booking-highload/blob/main/xevVvVnHoWU.jpg)

### Сводная таблица
Задача | Количество | CPU  (cores)  |      RAM (gb)    |  SSD (Gb) |
|:----------:|:----------:|:----------:|:-------------:|:------:|
nginx|7/датацентр| 32 |  64 | 256 |
Сервисы|3/датацентр| 32 |  64 | 256 |
Данные|9/датацентр| 16 |  32 | 1000 |
Изображения|9/датацентр| 32 |  64 | 24000 |
L4 балансировщики|2/датацентр| 32 |  64 | 256 |

<b> Суммарно: 32 машины, по 16 в каждый датацентр </b>
## 9. **Схема балансировки нагрузки (входящего трафика и внутрипроектного, терминация SSL)**
Для балансировки нагрузки будем использовать nginx схемы L7 встроенными настройками SSL терминации. Для балансировки нагрузки на сервера nginx используется L4 с общим виртуальным ip (via IP Tunneling). Балансировка нагрузки по классическому Round Robin алгоритму - нет зависимости от времени пакета, более менее равномерное распределение ресурсов.

## 10. **Обеспечение отказоустойчивости**
Отказоустойчивость обеспечивается несколкими компонентами:
1) Микросервисная архитектура с поднятыми несколькими экземплярами каждого сервиса
2) Несколько балансировщиков нагрузки, позволяющих в случае падения одного из них, перекинуть запросы на другие.
3) Использоваине технологии RAID 10 для хранения данных пользователей и медиа информации, допустимое количество вышедших из строя дисков от 1 до N/2 дисков. Информация не потеряется, если выйдут из строя диски в пределах разных зеркал.
4) Мониторинг метрик, с автоматическим оповещением сотрудников в случае нештатных ситуаций.

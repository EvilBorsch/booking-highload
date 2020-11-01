# Курсовой проект по курсу highload в технопарке

## 1. **Выбор темы**
Сервис по планированию путешесвий (TripAdvisor)

## 2. **Определение возможного диапазона нагрузок подобного проекта**
### TripAdvisor
Месячная аудитория- 490 миллионов [Источник](https://review42.com/tripadvisor-statistics/)
Среднее время проведения на подобных сайтах- 3 минуты [Источник](https://www.similarweb.com/website/tripadvisor.com/)

## 3. **Выбор планируемой нагрузки**
Планируемая нагрузка - 50% от доли TripAdvisor, т.е
- Месячная аудитория:  
    ```490/2=245 миллиона человек в месяц```

Учитывая специфику сайта (пользователи редко возвращаются на сайт в течение одного месяца, так как выбор путешествия скорее всего дело одного захода), можем принять число ежедневно активных юзеров=245 млн/30=8.1 млн человек
    
В среднем за минуту на TripAdvisor отправляется 270 запросов от пользователей Источник (без учета картинок)
Что вполне совпадает с экспериментом:
Оценивая себя как среднестатистического пользователя, подсчитал сколько пользовательского трафика займет стандартный workflow пользователя, а именно скролл ленты:
За 1 минуту 40 секунд было израсходовано 21 Мб траффика, 
![alt-текст](https://github.com/EvilBorsch/booking-highload/blob/main/Скриншоты%20(2).png "lenta")  

```8,1 млн ежедневных пользователей * 270/60 RPS * ((3/60)/24 - плотность распределения) = 75937 RPS```

## 4. **Логическая схема базы данных**
![alt-текст](https://github.com/EvilBorsch/booking-highload/blob/main/Снимок%20экрана%202020-10-21%20в%2016.38.54.png "Схема бд")


## 5. **Физическая системы хранения**

Активные сессии пользователя будем хранить в redis. Redis отличается от существующих решений тем, что API для работы с Memcached (MemcacheDB) позволяет хранить массивы, но эти массивы будут сериализованы и сохранены как строки, таким образом атомарные операции над такими массивами не возможны. Redis позволяет хранить как строки, так и массивы, к которым можно применять атомарные операции pop / push, делать выборки из таких массивов, выполнять сортировку элементов, получать объединения и пересечения массивов, что придет большую гибкость нашему приложению.

Высокая скорость работы Redis обеспечивается тем, что данные хранятся в оперативной памяти и сохраняются на диск либо через равные промежутки времени, либо при превышении определённого количества не сохранённых запросов. Из этого вытекает, что используя Redis, можно потерять результаты нескольких последних запросов, что вполне приемлемо для большинства веб-приложений, учитывая, что обращение к Redis по скорости сравнимо с обращением к оперативной памяти. Тем не менее, потерь можно избежать через избыточность - Redis поддерживает неблокирующую master-slave репликацию.

Данные о пользователе, отелях и отзывы имеют для нас наибольший приоритет, потому что без них наше приложение потеряет ключевой функционал. Для хранения этих данных я выбрал PostgreSQL, потому что по сравнению с другими реалиционными аналогами она имеет наиболее обширный функционал, эффективную репликацию и достаточно удобную поддержку.

Для повышения отказоустойчивости нашего приложения будем хранить пользователе, отелях и отзывы на разных шардах, что сделает компоненты нашей системы независимыми и выход из стоя одной компоненты не приведет к полной деградации всего сервиса.

## 6. **Выбор прочих технологий**

#### Backend
Golang. Язык предоставляет эффективный параллелизм, такой как C, C ++, Java, в то же время параллелизм в Go осуществляется намного проще благодаря горутинам, каналам и сборке мусора. Он имеет об обширную стандартную библиотеку, а также утилиты для форматирования кода, тестирования и расчета покрытия прямо из коробки. В нем очень удобно построена интеграция с внешними зависимостями посредствам go.mod. Go быстрый, кроссплатформенный, open-source с относительно низким порогом вхождения.

#### Frontend
HTML, CSS, TypeScript. Для  TypeScript будем использовать фреймворк React, который обеспечивает модульность, быстрый рендеринг, высокую run-time производительность. В сочетании с ES7 ReactTS может легко работать при высоких нагрузках. Также он имеет virtual DOM, которая позволяет упорядочивать документы форматов HTML, XHTML или XML в дерево, которое лучше всего подходит браузерам для анализа различных элементов веб-приложения.

#### Протоколы взаимодействия
Протокол связи между фронтендом и бэкендом - https, данные будут передаваться в формате json.
Общение между микросервисами на бэкенде будет осуществляться по протоколу gRPC, данные будут передаваться в формате protobuf

#### Обеспечение качества
Как на фронтенде, так и на бэкенде будут использованы статические и динамические анализаторы кода, интеграционные и юнит тесты. Их запуск будет автоматизирован в Gitlab CI при открытии merge request в master.

#### Релиз процесс
Будем собирать rpm пакеты для наших сервисов. Каждый пакет будет иметь собственную уникальную версию. Сервисы будут деплоиться в kubernetes. Пайплайн раскладки в k8s будет состоять из нескольких этапов: dev, staging, prod, на каждом из которых можно убедиться в корректности работы сервиса, правильности заправленных хостов и.п.
	
#### Протоколы взаимодействия

Протокол связи между фронтендом и бэкендом - https, данные будут передаваться в формате json. 

Общение между микросервисами на бэкенде будет осуществляться по протоколу gRPC, данные будут передаваться в формате protobuf.

## 7. **Расчет нагрузки и потребного оборудования**

##### Активные сессии

Для хранения всех сессий пользователей(245 млн) согласно [статье](https://medium.com/@lucasmagnum/redistip-estimate-the-memory-usage-for-repeated-keys-in-redis-2dc3f163fdab) нам примерно потребуется: 245 000 000 * 220 = 54 00 000 000 байт = 54 Гб

##### Пользователи

Одна запись пользователя в таблице занимает примерно (4+30+30+30+30) 124 байта. Тогда общий объем занимаемой памяти составит 245 млн *124 б= 30 Гб.

##### Отзывы

Одна запись комментария в таблице занимает примерно (4+4+4+200+4+8+4+4) 232 байта. Согласно статье [Статистика](https://expandedramblings.com/index.php/tripadvisor-statistics/) Число отзывов= 870 миллионов => Общий объем памяти = 870млн * 232 = 201 Гб

##### Фото

В среднем одна фотография весит около 500 кб. 
Если считать что у каждого пользователя есть аватарка, и к каждому отелю приложено в среднем 5 фото. 
Объем информации: 
245 млн* 0,5 Мб + 1,2 млн (число отелей [Статистика](https://expandedramblings.com/index.php/tripadvisor-statistics/) ) *0,5 Мб= 123 000 Гб
##### Расчет оборудования

Исходя из приведенных выше данных нам потребуется 123,3 тб. для хранения всей нужной информации. При закупке машин со следующими характеристиками:

| CPU  (cores)  |      RAM (gb)    |  SSD (gb) |
|:----------:|:-------------:|:------:|
| 32 |  64 | 4096 |

с учетом 30% ежегодного прироста пользователей, нам потребуется 31 машина под хранилища, 5 под микросервисы на бэкенде и 2 для фронтенда.

## 8. **Выбор хостинга / облачного провайдера и расположения серверов**
При покупке облачных серверов нет необходимости в аппаратной поддержке и найме сис. админов поэтому я решил смотреть в сторону облачных решений и в качестве провайдера выбрал MCS, потому что он обеспечивает весь нужный функционал:

- поддержка kubernetes из коробки
- быстрая миграция без остановки приложения
- удобное масштабирование

## 9. **Схема балансировки нагрузки (входящего трафика и внутрипроектного, терминация SSL)**
Будем использовать nginx для балансировки нагрузки с использованием схемы L7. Это позволит равномерно распределить нагрузку и решит проблемы медленных клиентов. Также nginx имеет внутренние инструменты для настройки SSL терминации.

## 10. **Обеспечение отказоустойчивости**
Поскольку запросов на чтение в нашем сервисе приходит в разы больше чем, на изменение, то для обеспечения отказоустойчивости баз данных будет резонно поднять для каждого мастера по два слейва. Отказоустойчивость наших сервисов обеспечивается за счет развертки в k8s, так как внутри него можно настроить автоматическую балансировка нагрузки с помощью постоянного мониторинга сведений о производительности и использовании ресурсов и соответствующее распределение работающих приложений по всему виртуальному кластеру.

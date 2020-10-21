# Technopark-Highload
homework №2 at the course of the technopark highload

### 1. Выбор темы
Сервис по бронированию жилья(Booking.com,Airbnb)

## 2. **Определение возможного диапазона нагрузок подобного проекта**
Самые крупные сервисы:
Booking
===
- Месячная нагрузка- 288 миллионов просмотров сайта в месяц [Источник](https://www.similarweb.com/website/booking.com/#pro)
- Месячная аудитория - 14,5 миллионов пользователей [Источник](https://www.chinatravelnews.com/article/120426)

Airbnb
===
- Месячная нагрузка (без учета региональных доменов)- 57 миллионов просмотров сайта в месяц [Источник](https://www.similarweb.com/website/airbnb.com/)
- Месячная аудитория- 13 миллионов просмотра сайта в месяц [Источник](https://www.chinatravelnews.com/article/120426)

Среднее время проведения на подобных сайтах- 8 минут [Источник](https://www.similarweb.com/website/booking.com/#pro)
### 3. Выбор планируемой нагрузки
Планируемая нагрузка - 50% от доли Airbnb и Booking, т.е
Месячная аудитория: 
```(14,5+13)/2=13,75 миллиона человек в месяц=1,15 миллиона человек в день```
Месячная нагрузка: 
``` (288+57)/2=172 миллиона просмотра сайта в месяц=14,4 миллиона просмотров в день```



### 4. Логическая схема базы данных



### 5. Физическая системы хранения



### 6. Выбор прочих технологий

##### Языки программирования

Backend - Golang. Язык предоставляет эффективный параллелизм, такой как C, C ++, Java, в то же время параллелизм в Go осуществляется намного проще благодаря горутинам, каналам и сборке мусора. Он имеет об обширную стандартную библиотеку, а также утилиты для форматирования кода, тестирования и расчета покрытия прямо из коробки. В нем очень удобно построена интеграция с внешними зависимостями посредствам go.mod. Go быстрый, кроссплатформенный, open-source с относительно низким порогом вхождения.

Frontend - HTML, CSS, TypeScript. Для  TypeScript будем использовать фреймворк React, который обеспечивает модульность, быстрый рендеринг, высокую run-time производительность. В сочетании с ES7 ReactTS может легко работать при высоких нагрузках. Также он имеет virtual DOM, которая позволяет упорядочивать документы форматов HTML, XHTML или XML в дерево, которое лучше всего подходит браузерам для анализа различных элементов веб-приложения.
	
##### Протоколы взаимодействия

Протокол связи между фронтендом и бэкендом - https, данные будут передаваться в формате json. 

Общение между микросервисами на бэкенде будет осуществляться по протоколу gRPC, данные будут передаваться в формате protobuf.

### 7. Расчет нагрузки и потребного оборудования

### 8. Выбор хостинга / облачного провайдера и расположения серверов

### 9. Схема балансировки нагрузки (входящего трафика и внутрипроектного, терминация SSL)

### 10. Обеспечение отказоустойчивости

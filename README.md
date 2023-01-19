# A-B-test

## Входные данные
Дана таблица в ClickHouse некоторого приложения "Лента новостей" (feed_action), состоящей их данных:

feed_action (лента новостей):
- user_id - id пользователя,
- time - время,
- action - действие (like or view),
- age (возраст пользователя),
- city - город,
- country - страна,
- exp_group - номер экспериментальной группы для AB и AA тестов,
- gender - гендер пользователя (0- мужчина, 1- женщина),
- os - операционная система (ios, android),
- post_id - номер поста,
- source - трафик (ads - реклама, organic).

# Часть 1: Проведение AA-теста

## Задание
В наших данныхуже есть разбивка пользователей на экспериментальные группы, это столбец exp_group (0 и 1 - контрольные группы, 2 и 3 - тестовые группы).
Предположим, что на экспериментальных группах 2 и 3 мы хотим развернуть AB-тест двух новых систем рекомендации постов.
Перед тем как проводить AB-тест, нам необходимо проверить корректно ли работает наша система сплитования.
Для этого нам надо провести симмуляцию 10 тыс. AA-тестов, для каждой симмуляции посчитать p-value, построить гистограмму распределения значений полученных p-value и посчитать процент p-value, которые меньше задданого уровня значимости α=0.05.

## Решение
Сравнивались группы 0-3 и 1-2. Проведена симмуляция 10 тыс. AA-тестов, расчитаны p-value, построены гистограммы.
У групп 0-3 0.0481 p-value выходят за уровень значимости α=0.05.
У групп 1-2 0.0481 p-value выходят за уровень значимости α=0.05.

## Вывод
- Oколо 5% p-value наших 10 тыс тестов АА между группами 0 и 3 выходят за границы 5% уровня значимости. Поэтому мы можем принять, что наша система сплитования у групп 0 и 3 корректно работает.
- Около 5% pvalue наших 10 тыс тестов АА между группами 1 и 2 выходят за границы 5% уровня значимости. Поэтому мы можем принять, что наша система сплитования у групп 1 и 2 корректно работает.
Можем принять нашу систему сплитования и проводить AB-тест.

# Часть 2: Проведение AB-теста

AB-тест проводился на группах: 1 - контрольная группа, 2 - тестовая группа.
Метрикой AB-теста была CTR=likes/views.
Распределение CTR выглядит следующим образом:

![ctr](https://user-images.githubusercontent.com/122218714/211630897-bb9587a4-7c0b-47b3-98a9-86fa5bb058b9.png)

Были проведены:
- тест Манна-Уитни поверх CTR,
- тест Манна-Уитни на сглаженном ctr,
- Пуассоновский бутстреп,
- тест Манна-Уитни поверх бакетного преобразования,
- тест Манна-Уитни поверх Линеаризованного CTR.

Вывод: между ctr двух выборок статистически установлена разница, контрольная группа показала лучший ctr, чем группа с новым алгоритмом рекомендации постов.

Рекомендация: не раскатывать новый алгоритм рекомендации постов на всех новых пользователей.

Дальнейшее исследование: 2 горб распределения тестовой группы (2) показал результаты выше, чем обычный алгоритм рекомендации постов. Дополнительно необходимо проверить, почему появились 2 горба, нет ли влияния каких-нибудь непредвиденных факторов на проведения A/B-теста

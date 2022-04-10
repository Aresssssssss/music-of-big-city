# Задачи по SQL (23 штуки):
1.	Посчитайте, сколько компаний закрылось.

Решение:

SELECT COUNT(name)

FROM company

WHERE status = 'closed'

2.	Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .

Решение:

SELECT funding_total
FROM company
WHERE country_code = 'USA'
  AND category_code = 'news'
ORDER BY funding_total DESC

3.	Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

Решение:

SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
  AND EXTRACT(YEAR FROM CAST(acquired_at AS timestamp)) BETWEEN '2011' AND '2013'

4.	Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.

Решение:

SELECT first_name,
       last_name,
       twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%'

5.	Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.

Решение:

SELECT *
FROM people
WHERE twitter_username LIKE '%money%'
  AND last_name LIKE 'K%'

6.	Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

Решение:

SELECT SUM(funding_total),
       country_code
FROM company
GROUP BY country_code
ORDER BY SUM(funding_total) DESC

7.	Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.


Решение:

SELECT funded_at,
       MIN(raised_amount),
       MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount) != 0
   AND MIN(raised_amount) != MAX(raised_amount)

8.	Создайте поле с категориями:
•	Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
•	Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
•	Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.

Решение:

SELECT *,
       CASE
           WHEN invested_companies > 100 THEN 'high_activity'
           WHEN invested_companies >= 20 AND invested_companies <= 100 THEN 'middle_activity'
           WHEN invested_companies < 20 THEN 'low_activity'
       END
FROM fund

9.	Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.

Решение:

SELECT ROUND(AVG(investment_rounds)),
       CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity
FROM fund
GROUP BY activity
ORDER BY ROUND(AVG(investment_rounds))

10.	Выгрузите таблицу с десятью самыми активными инвестирующими странами. Активность страны определите по среднему количеству компаний, в которые инвестируют фонды этой страны.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды, основанные с 2010 по 2012 год включительно.
Исключите из таблицы страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Отсортируйте таблицу по среднему количеству компаний от большего к меньшему.
Для фильтрации диапазона по годам используйте оператор BETWEEN.

Решение:

SELECT country_code,
       AVG(invested_companies),
       MIN(invested_companies),
       MAX(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM CAST(founded_at AS timestamp)) BETWEEN '2010' AND '2012'
GROUP BY country_code
HAVING MIN(invested_companies) > 0
ORDER BY AVG(invested_companies) DESC
LIMIT 10;

11.	Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

Решение:

SELECT p.first_name,
       p.last_name,
       e.instituition
FROM people AS p
LEFT JOIN education AS e ON p.id = e.person_id

12.	Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

Решение:

SELECT c.name,
       COUNT(DISTINCT e.instituition)
FROM company AS c
INNER JOIN people AS p ON p.company_id = c.id
LEFT JOIN education AS e ON p.id = e.person_id
GROUP BY c.name
ORDER BY COUNT(DISTINCT e.instituition) DESC
LIMIT 5;

13.	Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

Решение:

SELECT DISTINCT c.name
FROM company AS c
WHERE status = 'closed'
  AND id IN (SELECT company_id
            FROM funding_round
            WHERE is_first_round = 1
              AND is_last_round = 1)

14.	Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

Решение:

SELECT DISTINCT id
FROM people
WHERE company_id IN (SELECT DISTINCT c.id
                    FROM company AS c
                    WHERE status = 'closed'
                          AND id IN (SELECT company_id
                                    FROM funding_round
                                    WHERE is_first_round = 1
                                    AND is_last_round = 1))

15.	Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

Решение:

SELECT DISTINCT p.id,
       e.instituition
FROM people AS p
INNER JOIN education AS e ON p.id = e.person_id
WHERE company_id IN (SELECT DISTINCT c.id
                    FROM company AS c
                    WHERE status = 'closed'
                          AND id IN (SELECT company_id
                                    FROM funding_round
                                    WHERE is_first_round = 1
                                    AND is_last_round = 1))

16.	Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания.

Решение:

SELECT DISTINCT p.id,
       COUNT(e.instituition)
FROM people AS p
INNER JOIN education AS e ON p.id = e.person_id
WHERE company_id IN (SELECT DISTINCT c.id
                    FROM company AS c
                    WHERE status = 'closed'
                          AND id IN (SELECT company_id
                                    FROM funding_round
                                    WHERE is_first_round = 1
                                    AND is_last_round = 1))
GROUP BY p.id

17.	Дополните предыдущий запрос и выведите среднее число учебных заведений, которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

Решение:

SELECT AVG(res.count)
FROM (SELECT DISTINCT p.id,
             COUNT(e.instituition)
     FROM people AS p
     INNER JOIN education AS e ON p.id = e.person_id
     WHERE company_id IN (SELECT DISTINCT c.id
                         FROM company AS c
                         WHERE status = 'closed'
                           AND id IN (SELECT company_id
                                     FROM funding_round
                                     WHERE is_first_round = 1
                                     AND is_last_round = 1))
     GROUP BY p.id) AS res

18.	Напишите похожий запрос: выведите среднее число учебных заведений, которые окончили сотрудники компании Facebook.

Решение:

SELECT AVG(res.count)
FROM (SELECT DISTINCT p.id,
             COUNT(e.instituition)
     FROM people AS p
     INNER JOIN education AS e ON p.id = e.person_id
     WHERE company_id IN (SELECT DISTINCT c.id
                         FROM company AS c
                         WHERE c.name = 'Facebook')
     GROUP BY p.id) AS res

19.	Составьте таблицу из полей:
•	name_of_fund — название фонда;
•	name_of_company — название компании;
•	amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

Решение:

SELECT f.name AS name_of_fund,
       c.name AS name_of_company,
       fr.raised_amount AS amount
FROM investment AS i
INNER JOIN fund AS f ON i.fund_id = f.id
INNER JOIN company AS c ON i.company_id = c.id
INNER JOIN funding_round AS fr ON i.funding_round_id = fr.id
WHERE c.milestones > 6
  AND EXTRACT(YEAR FROM CAST(funded_at AS timestamp)) BETWEEN '2012' AND '2013'
GROUP BY name_of_fund, name_of_company, amount

20.	Выгрузите таблицу, в которой будут такие поля:
•	название компании-покупателя;
•	сумма сделки;
•	название компании, которую купили;
•	сумма инвестиций, вложенных в купленную компанию;
•	доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в алфавитном порядке. Ограничьте таблицу первыми десятью записями.

Решение:

SELECT c_ing.name,
a.price_amount,
c_ed.name,
c_ed.funding_total,
ROUND(a.price_amount / c_ed.funding_total)
FROM acquisition AS a
JOIN company AS c_ing ON a.acquiring_company_id = c_ing.id
JOIN company AS c_ed ON a.acquired_company_id = c_ed.id
WHERE a.price_amount != 0
AND c_ed.funding_total != 0
ORDER BY a.price_amount DESC, c_ed.name
LIMIT 10;

21.	Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год. Выведите также номер месяца, в котором проходил раунд финансирования.

Решение:

SELECT c.name,
       EXTRACT(MONTH FROM CAST(fr.funded_at AS timestamp))
FROM company AS c
INNER JOIN funding_round AS fr ON c.id = fr.company_id
WHERE EXTRACT(YEAR FROM CAST(fr.funded_at AS timestamp)) BETWEEN '2010' AND '2013'
  AND c.category_code = 'social'

22.	Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
-	номер месяца, в котором проходили раунды;
-	количество уникальных названий фондов из США, которые инвестировали в этом месяце;
-	количество компаний, купленных за этот месяц;
-	общая сумма сделок по покупкам в этом месяце.

Решение:

WITH
mf AS
  (SELECT EXTRACT(MONTH FROM CAST(fr.funded_at AS date)) AS month,
          COUNT(DISTINCT f.name) AS count_of_fund
   FROM funding_round AS fr
   JOIN investment AS i ON fr.id=i.funding_round_id
   JOIN fund AS f ON i.fund_id = f.id
   WHERE EXTRACT(YEAR FROM CAST(fr.funded_at AS date)) >= 2010
           AND EXTRACT(YEAR FROM CAST(fr.funded_at AS date)) <= 2013
           AND f.country_code = 'USA'
   GROUP BY month),
 
a AS 
    (SELECT EXTRACT(MONTH FROM CAST(acquired_at AS date)) AS month,
            COUNT(acquired_company_id) AS count_of_company,
            SUM(price_amount) AS total_sum
     FROM acquisition
     WHERE EXTRACT(YEAR FROM CAST(acquired_at AS date)) >= 2010
           AND EXTRACT(YEAR FROM CAST(acquired_at AS date)) <= 2013
     GROUP BY month)
 
SELECT mf.month,
    mf.count_of_fund,
    a.count_of_company,
    a.total_sum
FROM mf JOIN a ON mf.month = a.month;

23.	Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.

Решение:

WITH

y_2011 AS (SELECT AVG(funding_total) AS avg_2011,
                  country_code
          FROM company
          WHERE EXTRACT(YEAR FROM founded_at) = '2011'
          GROUP BY country_code),
          
y_2012 AS (SELECT AVG(funding_total) AS avg_2012,
                  country_code
          FROM company
          WHERE EXTRACT(YEAR FROM founded_at) = '2012'
          GROUP BY country_code),
          
y_2013 AS (SELECT AVG(funding_total) AS avg_2013,
                  country_code
          FROM company
          WHERE EXTRACT(YEAR FROM founded_at) = '2013'
          GROUP BY country_code)
          
SELECT y_2011.country_code,
       y_2011.avg_2011,
       y_2012.avg_2012,
       y_2013.avg_2013
FROM y_2012 INNER JOIN y_2011 ON y_2011.country_code = y_2012.country_code
INNER JOIN y_2013 ON y_2011.country_code = y_2013.country_code
ORDER BY y_2011.avg_2011 DESC

# Задание 2

## Краткое описание задания

 1. Считать данные из `training.csv`. Проверить является ли ряд стационарным в широком смысле. Это можно сделать двумя способами: 
    1. Провести визуальную оценку, отрисовав ряд и скользящую статистику(среднее, стандартное отклонение). Построить график на котором будет отображен сам ряд и различные скользящие статистики.
    2. Провести тест Дики - Фуллера.
    Сделать выводы из полученных результатов. Оценить достоверность статистики. (*25 баллов*)
  3. Разложить временной ряд на тренд, сезональность, остаток в соответствии с аддитивной, мультипликативной моделями. Визуализировать их, оценить стационарность получившихся рядов, сделать выводы. (*15 баллов*)
  4. Проверить является ли временной ряд интегрированным порядка `k`. Если является, применить к нему модель ARIMA, подобрав необходимые параметры с помощью функции автокорреляции и функции частичной автокорреляции. Выбор параметров обосновать. Отобрать несколько моделей. Предсказать значения для тестовой выборки. Визуализировать их, посчитать `r2 score` для каждой из моделей. Произвести отбор наилучшей модели с помощью информационного критерия Акаике. Провести анализ получившихся результатов. (*50 баллов*)
  
## Теория

**Опр.** *Временной ряд* - совокупность значений какого-либо показателя за определенное время.

Параметры временного ряда:
1. Период времени
2. Уровни ряда - значения показателя

**Опр.** Временной ряд y называется *стационарным*, если E[y] = Const, D[y] = Const, Corr[yk, y(k-1)] = Const * k - т.е если эти параметры не зависят от времени. Другое определение стационарности - это отсутствие тренда.

**Опр.** *Трендом* временного ряда называется изменение, определяющее общее направление развития, т.е. грубо говоря общее направление графика ряда (возрастает/убывает/не изменяется)

Каким образом проводится *визуальная оценка*? Мы смотрим на наличие у данного графика тренда. Тренд есть - ряд не стационарный, тренда нет - ряд стационарен. Стоит отметить, что эта оценка довольно грубая и во многих случаях из внешнего вида графика нельзя сказать, стационарен ряд или нет.

**Опр.** *Скользящая средняя (Moving Average - MA)* - среднее арифметическое значений исходной функции за установленный период.

Зачем нужно?
* Сглаживает краткосрочные колебания
* Помогает выделить основные тенденции

**Опр.** *Стандартное отклонение* - показывает, на сколько в среднем отклонился ряд от средней вариации ряда (от среднего арифметического, в нашем случае).

Скользящая средняя вместе со стандартным отклонением составляют *скользящие статистики*.

### Тест Дики-Фуллера
**Опр.** *Тест Дики-Фуллера* используется для проверки ряда на стационарность. Он проверяет ряд на наличие так называемых единичных корней.

**Опр.** Временной ряд имеет *единичный корень* (хотя бы один), если его первые разности образуют стационарный ряд.

*Обозн.* y(t) ~ I(1), т.е. Δy(t)=y(t)-y(t-1) ~ I(0), где Δ - разностный оператор, I(j) - означает, что ряд является интегрированным порядка j, I(0) - ряд стационарен. Определение интегрированности порядка k дается далее.

Вообще говоря, тест Дики-Фуллера проверяет значение коэффициента `a` в авторегрессионном уравнении 1-го порядка. Оно имеет вид:

y(t) = a * y(t-1) + ε(t), ε(t) - ошибка
* a=1   => есть единичные корни => стационарности нет
* |a|<1 => нет единичных корней => есть стационарность
* |a|>1 => не свойственно для временных рядов, которые встречаются в реальной жизни - требуется более сложный анализ

Преобразуем уравнение:

y(t) = a * y(t-1) + ε(t) => y(t) - y(t-1) = a * y(t-1) - y(t-1) + ε(t) => Δy(t) = (a-1) * y(t-1) + ε(t) => **Δy(t) = b * y(t-1) + ε(t)**, где b=a-1 - для удобства

* Основная гипотеза: H0: b=0 - процесс нестационарен
* Альтернативаная гипотеза: H1: b<0 - процесс стационарен

Как происходит проверка на наличие единичных корней? Для получения ответа, сравниваем найденное с помощью функции `sm.tsa.adfuller(ts)` `p-value` - достоверность статистики с `critical_values` - критическими значениями с уровнем значимости 5%:
* `p-value` > `critical_values` => единичный корень есть и ряд не стационарен
* Иначе стационарен

**Опр.** Достоверность статистики (`p-value`) - мера уверенности в "истинности" результата. Чем p-value меньше, тем больше доверия. Как только p-value становится больше опредленного параметра - `critical_values`, то мы понимаем, что доверять нельзя и говорим, что единичные корни есть.

**Опр.** *Уровень значимости* - показывает (обычно в процентном выражении) степень отклонения от гипотезы. Грубо говоря, если наша достоверность превысила, скажем, 5% уровень значимости, то гипотеза отвергается => процесс нестационарен.

### Тренд, сезональность, остаток. Аддитивная и мультипликативная модели.

**Опр.** *Трендом* временного ряда называется изменение, определяющее общее направление развития, т.е. грубо говоря общее направление графика ряда (возрастает/убывает/не изменяется)

**Опр.** *Сезональностью* временного ряда называются периодические колебания, наблюдаемые во временных рядах.

**Опр.** *Остатком* временного ряда называется разница между предсказанным и наблюдаемым значением.

* Общий вид *аддитивной модели*: Y = T + S + E;
* Общий вид *мультипликативной модели*: Y = T * S * E;
* T - тренд, S - сезональность, E - остаток

Рассмотрим аддитивную модель. 

Как найти *сезональность*? Необходимо найти скользящее среднее, от скользящего среднего найти еще раз скользящее среднее - получим центрированное скользящее среднее. Тогда 

`Сезональность = Временной ряд - Центрированное скользящее среднее`

Как найти *тренд*? С помощью метода наименьших квадратов приближается временной ряд, что и позволяет найти тренд временного ряда.

Как найти *остаток*?

`Остаток = Временной ряд - Тренд - Сезональность`

Рассмотрим мультипликативную модель

Аналогично с аддитивной моделью находим центрированное скользящее среднее.

`Сезональность = Временной ряд / Центрированное скользящее среднее`

Аналогично находим тренд

`Остаток = Временной ряд / Сезональность / Тренд`

### Порядок интегрированности

**Опр.** *Интегрированный временной ряд* - нестационарный временной ряд, разности некоторого порядка от которого являются стационарным рядом.

**Опр.** Временной ряд называется *интегрированным порядка `k`*, если разности ряда k-го порядка `Δ^k x(t)` являются стационарными, а разности меньшего порядка (и сам временной ряд) не являются стационарными рядами.
* Δ yk   = y(k+1) - yk
* Δ^2 yk = Δ y(k+1) - Δ yk = y(k+2) - 2y(k+1) + yk
* Δ^m yk = Δ^m-1 y(k+1) - Δ^m-1 yk, где Δ - разностный оператор

*Обозн.* yk ~ I(k) - интегрированный временной ряд порядка k

### Модель ARIMA

**Опр.** *ARIMA (autoregressive integrated moving average model)* - интегрированная модель авторегрессии скользящего среднего - модель анализа временных рядов. Это расширение моделей ARMA для нестационарных временных рядов.

**Опр.** *ARMA (autoregressive moving average model)* - математическая модель, используемая для анализа и прогнозирования стационарных временных рядов. Обобщает 2 более простые модели: авторегрессии (AR) и скользящего среднего (MA).

А как построить модель ARMA? Нужно найти коэффициенты p, q - порядки для моделей AR(p) и MA(q). Это помогут сделать функция автокорреляции и функция частичной автокорреляции.

**Опр.** *AR (autoregressive model)* - авторегрессионная модель временных рядов, в которой значения временого ряда в данных моемент зависит от предыдущих значений этого же ряда.

**Опр.** *MA (moving average model)* - модель скользящего среднего, в которой моделируемый уровень временного ряда можно представить как линейную функцию прошлых ошибок, т.е. разностей между прошлыми фактическими и теоретическими уровнями.

Таким образом, необходимо построить ARIMA(p, d, q), где p - порядок AR(p), d - порядок интегрированности, q - порядок MA(q)

`ARIMA(p, d, q) = AR(p) + MA(q) ~ I(d)`

### AIC - информационный критерий Акаике

**Опр.** *AIC (an information criterion)* - информационный критерий Акаике - критерий, применяющийся исключительно для выбора из нескольких статистических моделей.


## Подход к решению

1. Чтение данных из `training.csv`
* Чтение производится `командой pd.read_csv('training.csv')`

2. Проверка на стационарность
    1. Визуальная оценка
        * Скользящее среднее
            > ts.rolling().mean().plot()
        * Стандартное отклонение
            > ts.rolling().std().plot()
        * Строим вывод о стационарности ряда по визуальной оценке
    2. Тест Дики-Фуллера
        * `sm.tsa.adfuller(ts)` - проводит тест Дики-Фуллера
        * Анализируем полученные из функции параметры
        * Строим вывод о стационарности данного ряда

3. Разложение временного ряда на тренд, сезональность остаток в соответствии с аддитивной и мультипликативной моделями
    * `models(data, model_s, title)` реализует разложение на соответствующие модели
		+ data - временной ряд
		+ model_s - параметр для функции decomp, которая осуществляет разложение временного ряда
		+ title - название для графика
		+ С помощью функции `seasonal_decompose` раскладываем временной ряд на тренд, сезональность и остаток, после чего отрисовываем их и прогоняем через тест Дики-Фуллера, который анализирует стационарность рядов
    * `seasonal_decompose(data.Value, model)`
		+ data.Value - столбец значений временного ряда
		+ model - 'additive' или 'multiplicate'
		+ возвращает decompose
			* decompose.trend - тренд исходного ряда
			* decompose.resid - остаток исходного ряда
			* decompose.seasonal - сезонность исходного ряда
	* Строим вывод на основании полученных результатов
4. Поиск коэффициента интегрируемости ряда
	* `findOrder(ts, isPrint)` - возвращает коэффициент интегрированности ряда
		+ ts - временной ряд
		+ isPrint - флаг: если True - печать в тесте Дики-Фуллера, если False - ничего не выводит
5. Подбираем нужные параметры с помощью функции автокорреляции и функции частичной автокорреляции
	* `acf(ts, nlags)` - находит автокорреляцию временного ряда
		+ ts - временной ряд
		+ nlags - число лагов для автокорреляции
		+ Зачем нужна? Помогает найти порядок q модели MA(q) для построения модели ARIMA(p, d, q) 
	* `pacf(ts, nlags)` - находит частичную автокорреляцию временного ряда
		+ ts - временной ряд
		+ nlags - число лагов для частичной автокорреляции
		+ Зачем нужна? Помогает найти порядок p модели AR(p) для построения модели ARIMA(p, d, q)
	* Функции автокорреляции и частичной автокорреляции - дискретные, число различных значений равно лагу (в обоих случаях по 20). Соответствующие коэффициенты определяются по правилу - берется целая часть наибольшего значения. В обоих случаях наибольшее значение равно 1.0 и достигается оно на нулевом элементе (в обоих случаях).
6. ARIMA-модель
	* `ARIMA(ts, order=(p, k, q))` - строит модель ARIMA
		+ ts - временной ряд
		+ order - порядки для модели
		+ Возвращает объект ARIMA
	* `model.fit()` - Прогоняем ARIMA-модель через фильтр Калмана
		+ **Опр.** Фильтр Калмана - мощнейщий инструмент фильтрации данных. При фильтрации используется информация о физике самого явления.
		+ Возвращает объект ARIMAResults - содержит также и результаты предсказаний, которые можно выводить и использовать в дальнейших прогнозах.
	* `model.predict(typ='levels')` - проверяем правильность предсказания на тренировочной выборке.

## Инструкции по запуску

Cell -> Run All

## Необходимое ПО

### Библиотеки:
1) **Pandas**
* Для работы с временными рядами
2) **Numpy** 
* Для работы с массивами
* Функция `log()` - натуральный логарифм
* Функция `exp()` - экспонента
3) **Matplotlib.pylab**
* Для построения графиков функций
4) **Statsmodels**
* Используются статистические функции: `acf()`, `pacf()`,
`seasonal_decompose()`
* Для постороения модели ARIMA

### Программы:
1) Jupyter Notebook

## Участники 

1) Горбунов Александр - 312 гр.

2) Гулиев Юрий - 312 гр.

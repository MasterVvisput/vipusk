В ходе работы над проектом мы выполнили ряд работ:

**Подготовили окружение**

- загрузили библиотеки
- установили константы
- вспомогательные функции для построения графиков 
- Настроили подключение к базе данных 

**Выполнили исследовательский аналих данных** 

Для начала выполнили общий анализ таблиц, убкдились в их наличии и в том ,что они содержат данные. 

- мы написали функцию выполнения запроса к базе данных и сохранении запроса в датафрейм
- мы сделали запрос к sqlite_master для получения имён всех таблиц и сравнили результат с условием задачи. На основе этого мы смогли сделать вывод, что в базе таблиц больше чем в условии задачи, но при этом все таблицы из условия задачи в базе имеются
- мы запросили table_info к каждой таблице из базы даных и получили информацию о её полях. Полученный результат выведен в проект.
- мы запросили количество строк для каждой таблицы. На основе полученных данных мы делаем вывод о том, что все таблицы содержат данные.

Выполнили крос анализх колонки кей

- это дало нам понимание о том, сколько партий описаны в таблицах
- В каждой таблице разное количество уникалньных значений в поле key. Пересечений по ключам 3022 во всех таблицах. 

Выполнили анализ и предобработку данных

data_arc

- переименовали колонки, привели данные к корректным типам
- Согласно максимальным и минимальным значениям признака arc_heating_start_dttm, у нас данные за 4 месяца.
- Также признак reactive_power имеет станадртное отклонение в разы больше чем у active_power, а все остальне метрики (медина, средние, 25%, 50%, 75%) отличаются не в разы. Возможно в данных есть выбросы.
- избавились отвыбросов путём их удаления

data_bulk и data_bulk_time

- проанализировали пропуски. Совпадение количества пропускорв говорит о том, что некоторые сыпучие материалы в партии не использовались. В тоже время мы можем предположить, что всегда при указании объёма сыпучего материала указывается время его добавления без пропусков.
- выполнили предобработку: Объединили объём и время добавления материала с идентефикатором материала, удалили малоиспользуемые, привели дату к корректному формату. 
- Таким образом мы получили Entity-Attribute-Value структуру данных о подаче сыпучих материалов, которая не содержит пропусков и избавлена от малоиспользуемых материалов.

data_gas

- Данные о продувке газом не содержат привыязки ко времени. Судя по тому, что количество записей уникальным значением key равно количеству всех записей, можем сделать вывод о том, что продувка газом совершалась один раз в партии в неопределенный момент времени, указанным объёмом газа.

- Наиболее популярный объём газа в прартии от 9 до 10 единиц.

steel.data_wire и steel.data_wire_time

- Структура данных об использовании проволочных материалов очень похожа на данные о сыпучих материалов. Выполнили их предобработку схожим образом.
- Из графика видим, что есть проволочные материалы, который используются редко. Мы считаем, что от них необходимо избавиться для повышения качества модели.
- выполнили предобработку: Объединили объём и время добавления материала с идентефикатором материала, удалили малоиспользуемые, привели дату к корректному формату.
-Таким образом мы получили Entity-Attribute-Value структуру данных о подаче проволочных материалов, которая не содержит пропусков и избавлена от малоиспользуемых материалов.

data_temp

- переименовали колонки, привели типы данных к корректному формату
- мы оставили только те партии, в которых полдностью зивестны замеры температуры

Выполнили создание признаков

Для начала построили хронологию движения одной партии. И на основе этого разработали новые признаки: 

- Последняя предыдущая известная температура на момент текущего измерения
- Значение продувки газом при изготовлении партии
- Минут с начала производства партии до момента измерения температуры
- Количество итераций нагрева дугой до момента измерения температуры
- Суммарная работа нагрева на момент измерения температуры
- Серия столбцов - Данные о всех сессиях нагрева (активная мощность)
- Серия столбцов - Данные о всех сессиях нагрева (реактивная мощность)
- Серия столбцов - Данные о подаче сыпучих материалов с момента начала до момента измерения времени
- Серия столбцов - Данные о подаче проволочных материалов с момента начала до момента измерения времени

Предварительно изучили корреляцию признаков. 
- Похоже, что Последняя предыдущая известная температура на момент текущего измерения сильно кррелирует с таргетом. 

**Выполнили обучение моделей**

Для каждой партии мы отнесли все измерения температуры, за исключением последнего, в обучающую выборку. Последнее измерение, в свою очередь, попадет в тестовую выборку. Таким образом, наша цель - предсказать последнее измерение температуры для каждой партии.

Важно отметить, что партии, у которых после очистки данных осталось только одно измерение температуры, исключаются из рассмотрения.

Выполнили масштабирование признаков

Получили метрики константной модели: MAE модели со стратегией среднего: 10.29596519346655

И приступили к обучению

- Линейная регрессия. 
Средний MAE линейной регрессии на обучающей выборке с кросс-валидацией на 5 выборках: 8.00
MAE линейной регрессии на обучающей/тестовой выборке: 7.93/7.28

Не дотягивает до нужного результата

- Ридж регрессия. 
MAE ридж регрессии на обучающей/тестовой выборке: 7.93/7.27

Тоже недостающий результат

- Случайный лес
MAE случайного леса на обучающей/тестовой выборке: 7.88/7.00

Также выше необходимого порога. 

- Модель CatBoostRegressor. Лучший результат 7.07

- Нейронные сети
Удалось добиться Score: 6.16 на тестовой выборке. 
Это результат ниже необходимого порога. 

В сравнении лучшей и константной модели, побеждает модель основанная на нейросети: 
MAE константной модели со стратегией среднего: 10.29596519346655
MAE нейросети на тестовой выборке:  6.162301707004046

**Исследование важности признаков показало, что* 

- признак key_last_known_temperature имеет прямую кореляцию с целевым признаком.

# Формулы DAX для дашборда
## Вычисляемые столбцы

### Длительность ремонта
```dax
Длительность ремонта = 
IF(
    OR(ISBLANK('data'[Дата]), ISBLANK('data'[Дата отдачи])),
    BLANK(),
    DATEDIFF('data'[Дата], 'data'[Дата отдачи], DAY)
)
```

### Группа по обращениям
```dax
Группа по обращениям = 
VAR Orders_count = CALCULATE(COUNTROWS('data'), ALLEXCEPT('data', 'data'[User_id]))
RETURN
SWITCH(
    TRUE(),
    Orders_count = 1, "1 обращение",
    Orders_count = 2, "2 обращения",
    Orders_count >= 3, "3+ обращений"
)
```

### Группа чека
```dax
Группа чека = 
SWITCH(
    TRUE(),
    'data'[Сумма ремонта] < 3000, "До 3 тыс",
    'data'[Сумма ремонта] < 6000, "3-6 тыс",
    'data'[Сумма ремонта] < 9000, "6-9 тыс",
    "Более 9 тыс"
)
```

## Основные меры

### KPI
```dax
Кол-во заказов = COUNTROWS('data')

Кол-во клиентов = DISTINCTCOUNT('data'[User_id])

Сумма заказов = SUM('data'[Сумма ремонта])

Средний чек = DIVIDE([Сумма заказов], [Кол-во заказов], 0)

Первое обращение = MIN('data'[Дата])

Последнее обращение = MAX('data'[Дата])
```

### Время
```dax
Среднее время отдачи = AVERAGE('data'[Длительность ремонта])

Ср заказов в рабочий день = 
AVERAGEX(
    FILTER(
        VALUES('Календарь'[Date]),
        WEEKDAY('Календарь'[Date], 2) <= 5
    ),
    CALCULATE([Кол-во заказов])
)
```

### Проценты
```dax
% от общего кол-ва заказов = 
DIVIDE(
    [Кол-во заказов],
    CALCULATE([Кол-во заказов], ALLSELECTED('data'))
) * 100

% проблем внутри бренда = 
DIVIDE(
    [Кол-во заказов],
    CALCULATE([Кол-во заказов], ALLEXCEPT('data', 'data'[Бренд']))
) * 100
```

### Топы
```dax
Частая проблема = 
VAR Top_problem =
    TOPN(
        1,
        SUMMARIZE('data', 'data'[Тип неисправности], "Кол", COUNTROWS('data')),
        [Кол],
        DESC
    )
RETURN
    MAXX(Top_problem, 'data'[Тип неисправности])
```

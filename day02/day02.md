## Day 02

#### Exercise 00 - Move to the LEFT, move to the RIGHT

| Exercise 00: Move to the LEFT, move to the RIGHT |

```sql
SELECT 
    pz.name AS pizzeria_name,
    pz.rating
FROM 
    pizzeria pz
LEFT JOIN 
    person_visits pv ON pz.id = pv.pizzeria_id
WHERE 
    pv.pizzeria_id IS NULL;
```


#### Exercise 01 - Find data gaps

| Exercise 01: Find data gaps|
```sql
SELECT 
    missing_date::date
FROM 
    generate_series('2022-01-01', '2022-01-10', INTERVAL '1 day') AS missing_date
LEFT JOIN 
    (SELECT DISTINCT visit_date FROM person_visits WHERE person_id IN (1, 2)) AS visited_dates
    ON missing_date = visited_dates.visit_date
WHERE 
    visited_dates.visit_date IS NULL
ORDER BY 
    missing_date;
```

#### Exercise 02 - FULL means ‘completely filled’

| Exercise 02: FULL means ‘completely filled’|
```sql
SELECT 
    COALESCE(p.name, '-') AS person_name,
    pv.visit_date,
    COALESCE(pz.name, '-') AS pizzeria_name
FROM 
    person p
FULL JOIN 
    person_visits pv ON p.id = pv.person_id AND pv.visit_date BETWEEN '2022-01-01' AND '2022-01-03'
FULL JOIN 
    pizzeria pz ON pv.pizzeria_id = pz.id
ORDER BY 
    person_name, visit_date, pizzeria_name;
```

## Exercise 03 - Reformat to CTE

| Exercise 03: Reformat to CTE |
```sql
WITH date_series AS (
    SELECT generate_series('2022-01-01', '2022-01-10', INTERVAL '1 day')::date AS missing_date
)
SELECT 
    ds.missing_date
FROM 
    date_series ds
LEFT JOIN 
    (SELECT DISTINCT visit_date FROM person_visits WHERE person_id IN (1, 2)) AS visited_dates
    ON ds.missing_date = visited_dates.visit_date
WHERE 
    visited_dates.visit_date IS NULL
ORDER BY 
    ds.missing_date;
```

#### Exercise 04 - Find favourite pizzas

| Exercise 04: Find favourite pizzas |    
```sql
SELECT 
    m.pizza_name,
    pz.name AS pizzeria_name,
    m.price
FROM 
    menu m
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
WHERE 
    m.pizza_name IN ('mushroom pizza', 'pepperoni pizza')
ORDER BY 
    m.pizza_name, pz.name;
```

#### Exercise 05 - Investigate Person Data

| Exercise 05: Investigate Person Data |
```sql
SELECT 
    name
FROM 
    person
WHERE 
    gender = 'female' AND age > 25
ORDER BY 
    name;
```

#### Exercise 06 - favourite pizzas for Denis and Anna

| Exercise 06: favourite pizzas for Denis and Anna |
```sql
SELECT 
    m.pizza_name,
    pz.name AS pizzeria_name
FROM 
    person_order po
JOIN 
    person p ON po.person_id = p.id
JOIN 
    menu m ON po.menu_id = m.id
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
WHERE 
    p.name IN ('Denis', 'Anna')
ORDER BY 
    m.pizza_name, pz.name;
```

#### Exercise 07 - Cheapest pizzeria for Dmitriy

| Exercise 07: Cheapest pizzeria for Dmitriy |
```sql
SELECT 
    pz.name AS pizzeria_name
FROM 
    person_visits pv
JOIN 
    person p ON pv.person_id = p.id
JOIN 
    pizzeria pz ON pv.pizzeria_id = pz.id
JOIN 
    menu m ON pz.id = m.pizzeria_id
WHERE 
    p.name = 'Dmitriy' 
    AND pv.visit_date = '2022-01-08'
    AND m.price < 800;
```

#### Exercise 08 - Continuing to research data

| Exercise 08: Continuing to research data |
```sql
SELECT DISTINCT
    p.name
FROM 
    person p
JOIN 
    person_order po ON p.id = po.person_id
JOIN 
    menu m ON po.menu_id = m.id
WHERE 
    p.gender = 'male'
    AND p.address IN ('Moscow', 'Samara')
    AND m.pizza_name IN ('pepperoni pizza', 'mushroom pizza')
ORDER BY 
    p.name DESC;
```

#### Exercise 09 - Who loves cheese and pepperoni?

| Exercise 09: Who loves cheese and pepperoni? |  
```sql
SELECT 
    p.name
FROM 
    person p
JOIN 
    person_order po ON p.id = po.person_id
JOIN 
    menu m ON po.menu_id = m.id
WHERE 
    p.gender = 'female'
    AND m.pizza_name IN ('cheese pizza', 'pepperoni pizza')
GROUP BY 
    p.name
HAVING 
    COUNT(DISTINCT m.pizza_name) = 2
ORDER BY 
    p.name;
```

#### Exercise 10 - Find persons from one city

| Exercise 10: Find persons from one city |
```sql
SELECT 
    p1.name AS person_name1,
    p2.name AS person_name2,
    p1.address AS common_address
FROM 
    person p1
JOIN 
    person p2 ON p1.address = p2.address AND p1.id < p2.id
ORDER BY 
    p1.name, p2.name, p1.address;
```
## Day 03

#### Exercise 00 - Let’s find appropriate prices for Kate

| Exercise 00: Let’s find appropriate prices for Kate |    
```sql
SELECT 
    m.pizza_name,
    m.price,
    pz.name AS pizzeria_name,
    pv.visit_date
FROM 
    person p
JOIN 
    person_visits pv ON p.id = pv.person_id
JOIN 
    pizzeria pz ON pv.pizzeria_id = pz.id
JOIN 
    menu m ON pz.id = m.pizzeria_id
WHERE 
    p.name = 'Kate' 
    AND m.price BETWEEN 800 AND 1000
ORDER BY 
    m.pizza_name, m.price, pz.name;
```


#### Exercise 01 - Let’s find forgotten menus

Task: Find all menu identifiers not ordered by anyone (without using JOINs).
```sql
SELECT 
    m.id AS menu_id
FROM 
    menu m
WHERE 
    m.id NOT IN (SELECT DISTINCT menu_id FROM person_order)
ORDER BY 
    menu_id;
```


#### Exercise 02 - Let’s find forgotten pizza and pizzerias

Task: Show pizza names, prices and pizzerias names for menus not ordered by anyone.
```sql
SELECT 
    m.pizza_name,
    m.price,
    pz.name AS pizzeria_name
FROM 
    menu m
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
WHERE 
    m.id NOT IN (SELECT DISTINCT menu_id FROM person_order)
ORDER BY 
    m.pizza_name, m.price;
```


#### Exercise 03 - Let’s compare visits

Task: Find pizzerias visited more often by women or men (with duplicates).

```sql
WITH gender_visits AS (
    SELECT 
        pz.name AS pizzeria_name,
        p.gender,
        COUNT(*) AS visit_count
    FROM 
        person_visits pv
    JOIN 
        pizzeria pz ON pv.pizzeria_id = pz.id
    JOIN 
        person p ON pv.person_id = p.id
    GROUP BY 
        pz.name, p.gender
)
SELECT 
    pizzeria_name
FROM (
    SELECT 
        pizzeria_name,
        gender,
        visit_count,
        RANK() OVER (PARTITION BY pizzeria_name ORDER BY visit_count DESC) AS rank
    FROM 
        gender_visits
) ranked
WHERE 
    rank = 1 AND gender IN ('female', 'male')
ORDER BY 
    pizzeria_name;
```


#### Exercise 04 - Let’s compare orders

Task: Find pizzerias ordered only by women or only by men (without duplicates).

```sql
WITH male_orders AS (
    SELECT DISTINCT pz.name
    FROM person_order po
    JOIN menu m ON po.menu_id = m.id
    JOIN pizzeria pz ON m.pizzeria_id = pz.id
    JOIN person p ON po.person_id = p.id
    WHERE p.gender = 'male'
),
female_orders AS (
    SELECT DISTINCT pz.name
    FROM person_order po
    JOIN menu m ON po.menu_id = m.id
    JOIN pizzeria pz ON m.pizzeria_id = pz.id
    JOIN person p ON po.person_id = p.id
    WHERE p.gender = 'female'
)
SELECT name AS pizzeria_name
FROM (
    (SELECT name FROM male_orders EXCEPT SELECT name FROM female_orders)
    UNION
    (SELECT name FROM female_orders EXCEPT SELECT name FROM male_orders)
) result
ORDER BY pizzeria_name;
```


#### Exercise 05 - Visited but did not make any order

Task: Find pizzerias Andrey visited but didn't order from.

```sql
SELECT DISTINCT
    pz.name AS pizzeria_name
FROM 
    person_visits pv
JOIN 
    pizzeria pz ON pv.pizzeria_id = pz.id
JOIN 
    person p ON pv.person_id = p.id
WHERE 
    p.name = 'Andrey'
EXCEPT
SELECT DISTINCT
    pz.name
FROM 
    person_order po
JOIN 
    menu m ON po.menu_id = m.id
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
JOIN 
    person p ON po.person_id = p.id
WHERE 
    p.name = 'Andrey'
ORDER BY 
    pizzeria_name;
```


#### Exercise 06 - Find price-similarity pizzas

Task: Find same pizza names with same prices from different pizzerias.
```sql
SELECT 
    m1.pizza_name,
    pz1.name AS pizzeria_name_1,
    pz2.name AS pizzeria_name_2,
    m1.price
FROM 
    menu m1
JOIN 
    menu m2 ON m1.pizza_name = m2.pizza_name AND m1.price = m2.price AND m1.pizzeria_id < m2.pizzeria_id
JOIN 
    pizzeria pz1 ON m1.pizzeria_id = pz1.id
JOIN 
    pizzeria pz2 ON m2.pizzeria_id = pz2.id
ORDER BY 
    m1.pizza_name;
```

#### Exercise 07 - Let’s cook a new type of pizza

Task: Register new "greek pizza" in Dominos.
```sql
INSERT INTO menu (id, pizzeria_id, pizza_name, price)
VALUES (19, 2, 'greek pizza', 800);
```

#### Exercise 08 - Let’s cook a new type of pizza with more dynamics

Task: Register new "sicilian pizza" with dynamic ID calculation.
```sql
INSERT INTO menu (id, pizzeria_id, pizza_name, price)
VALUES (
    (SELECT MAX(id) + 1 FROM menu),
    (SELECT id FROM pizzeria WHERE name = 'Dominos'),
    'sicilian pizza',
    900
);
```

#### Exercise 09 - New pizza means new visits

Task: Register visits from Denis and Irina to Dominos on 24/02/2022.
```sql
INSERT INTO person_visits (id, person_id, pizzeria_id, visit_date)
VALUES 
    ((SELECT MAX(id) + 1 FROM person_visits), 
     (SELECT id FROM person WHERE name = 'Denis'), 
     (SELECT id FROM pizzeria WHERE name = 'Dominos'), 
     '2022-02-24'),
    ((SELECT MAX(id) + 2 FROM person_visits), 
     (SELECT id FROM person WHERE name = 'Irina'), 
     (SELECT id FROM pizzeria WHERE name = 'Dominos'), 
     '2022-02-24');
```


#### Exercise 10 - New visits means new orders

Task: Register orders from Denis and Irina for sicilian pizza on 24/02/2022.
```sql
INSERT INTO person_order (id, person_id, menu_id, order_date)
VALUES 
    ((SELECT MAX(id) + 1 FROM person_order),
     (SELECT id FROM person WHERE name = 'Denis'),
     (SELECT id FROM menu WHERE pizza_name = 'sicilian pizza'),
     '2022-02-24'),
    ((SELECT MAX(id) + 2 FROM person_order),
     (SELECT id FROM person WHERE name = 'Irina'),
     (SELECT id FROM menu WHERE pizza_name = 'sicilian pizza'),
     '2022-02-24');
```

#### Exercise 11 - "Improve" a price for clients

Task: Decrease price of greek pizza by 10%.
```sql
UPDATE menu
SET price = price * 0.9
WHERE pizza_name = 'greek pizza';
```


#### Exercise 12 - New orders are coming!

Task: Register orders from all persons for greek pizza on 25/02/2022.
```sql
INSERT INTO person_order (id, person_id, menu_id, order_date)
SELECT 
    (SELECT MAX(id) + GENERATE_SERIES(1, (SELECT COUNT(*) FROM person))),
    p.id,
    (SELECT id FROM menu WHERE pizza_name = 'greek pizza'),
    '2022-02-25'
FROM 
    person p;
```

#### Exercise 13 - Money back to our customers

Task: Delete new orders from exercise 12 and remove greek pizza.
```sql
-- Delete orders from 25/02/2022
DELETE FROM person_order
WHERE order_date = '2022-02-25';

-- Delete greek pizza from menu
DELETE FROM menu
WHERE pizza_name = 'greek pizza';
```
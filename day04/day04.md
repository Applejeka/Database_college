## Day04

#### Exercise 00 - Let's create separated views for persons

Task: Create views for female and male persons.

```sql
CREATE VIEW v_persons_female AS
SELECT * FROM person WHERE gender = 'female';

CREATE VIEW v_persons_male AS
SELECT * FROM person WHERE gender = 'male';
```

#### Exercise 01 - From parts to common view

Task: Combine names from both gender views into one list.
```sql
SELECT name FROM v_persons_female
UNION ALL
SELECT name FROM v_persons_male
ORDER BY name;
```

#### Exercise 02 - "Store" generated dates in one place

Task: Create a view with dates from January 1-31, 2022.
```sql
CREATE VIEW v_generated_dates AS
SELECT generate_series('2022-01-01'::date, '2022-01-31'::date, '1 day')::date AS generated_date
ORDER BY generated_date;
```

#### Exercise 03 - Find missing visit days with Database View

Task: Find dates in January 2022 with no visits.
```sql
SELECT generated_date AS missing_date
FROM v_generated_dates
EXCEPT
SELECT DISTINCT visit_date
FROM person_visits
ORDER BY missing_date;
```

#### Exercise 04 - Let's find something from Set Theory

Task: Create a view showing symmetric union of visits on Jan 2 and Jan 6.
```sql
CREATE VIEW v_symmetric_union AS
(SELECT person_id FROM person_visits WHERE visit_date = '2022-01-02'
 EXCEPT
 SELECT person_id FROM person_visits WHERE visit_date = '2022-01-06')
UNION
(SELECT person_id FROM person_visits WHERE visit_date = '2022-01-06'
 EXCEPT
 SELECT person_id FROM person_visits WHERE visit_date = '2022-01-02')
ORDER BY person_id;
```

#### Exercise 05 - Let's calculate a discount price for each person

Task: Create a view showing orders with 10% discount.
```sql
CREATE VIEW v_price_with_discount AS
SELECT 
    p.name,
    m.pizza_name,
    m.price,
    ROUND(m.price * 0.9) AS discount_price
FROM 
    person_order po
JOIN 
    person p ON po.person_id = p.id
JOIN 
    menu m ON po.menu_id = m.id
ORDER BY 
    p.name, m.pizza_name;
```

#### Exercise 06 - Materialization from virtualization

Task: Create a materialized view of Dmitriy's visits with cheap pizzas.
```sql
CREATE MATERIALIZED VIEW mv_dmitriy_visits_and_eats AS
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

#### Exercise 07 - Refresh our state

Task: Add new visit for Dmitriy and refresh materialized view.
```sql
-- Add new visit
INSERT INTO person_visits (id, person_id, pizzeria_id, visit_date)
VALUES (
    (SELECT MAX(id) + 1 FROM person_visits),
    (SELECT id FROM person WHERE name = 'Dmitriy'),
    (SELECT id FROM pizzeria WHERE name = 'Dominos'),
    '2022-01-08'
);

-- Refresh view
REFRESH MATERIALIZED VIEW mv_dmitriy_visits_and_eats;
```

#### Exercise 08 - Just clear our database

Task: Drop all created views and materialized views.
```sql
DROP VIEW IF EXISTS v_persons_female;
DROP VIEW IF EXISTS v_persons_male;
DROP VIEW IF EXISTS v_generated_dates;
DROP VIEW IF EXISTS v_symmetric_union;
DROP VIEW IF EXISTS v_price_with_discount;
DROP MATERIALIZED VIEW IF EXISTS mv_dmitriy_visits_and_eats;
```

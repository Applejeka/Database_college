## Day05

#### Exercise 00 - Simple aggregated information

Task: Return person IDs and their visit counts, sorted by visit count and person ID.
```sql
SELECT 
    person_id, 
    COUNT(*) AS count_of_visits
FROM 
    person_visits
GROUP BY 
    person_id
ORDER BY 
    count_of_visits DESC, 
    person_id ASC;
```

#### Exercise 01 - Let's see real names

Task: Show top 4 persons by visit count with their names.
```sql
SELECT 
    p.name, 
    COUNT(pv.id) AS count_of_visits
FROM 
    person_visits pv
JOIN 
    person p ON pv.person_id = p.id
GROUP BY 
    p.name
ORDER BY 
    count_of_visits DESC, 
    p.name ASC
LIMIT 4;
```

#### Exercise 02 - Restaurants statistics

Task: Show top 3 pizzerias by visits and orders in one list.
```sql
(SELECT 
    pz.name, 
    COUNT(*) AS count, 
    'visit' AS action_type
FROM 
    person_visits pv
JOIN 
    pizzeria pz ON pv.pizzeria_id = pz.id
GROUP BY 
    pz.name
ORDER BY 
    count DESC
LIMIT 3)
UNION ALL
(SELECT 
    pz.name, 
    COUNT(*) AS count, 
    'order' AS action_type
FROM 
    person_order po
JOIN 
    menu m ON po.menu_id = m.id
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
GROUP BY 
    pz.name
ORDER BY 
    count DESC
LIMIT 3)
ORDER BY 
    action_type ASC, 
    count DESC;
```

#### Exercise 03 - Restaurants statistics #2

Task: Combine visit and order counts for each pizzeria.
```sql
SELECT 
    pz.name, 
    COALESCE(v.visit_count, 0) + COALESCE(o.order_count, 0) AS total_count
FROM 
    pizzeria pz
LEFT JOIN 
    (SELECT pizzeria_id, COUNT(*) AS visit_count FROM person_visits GROUP BY pizzeria_id) v 
    ON pz.id = v.pizzeria_id
LEFT JOIN 
    (SELECT m.pizzeria_id, COUNT(*) AS order_count 
     FROM person_order po JOIN menu m ON po.menu_id = m.id 
     GROUP BY m.pizzeria_id) o 
    ON pz.id = o.pizzeria_id
ORDER BY 
    total_count DESC, 
    pz.name ASC;
```

#### Exercise 04 - Clause for groups

Task: Show persons with more than 3 visits (without WHERE).
```sql
SELECT 
    p.name, 
    COUNT(pv.id) AS count_of_visits
FROM 
    person p
JOIN 
    person_visits pv ON p.id = pv.person_id
GROUP BY 
    p.name
HAVING 
    COUNT(pv.id) > 3;
```

#### Exercise 05 - Person's uniqueness

Task: Find unique persons who made orders (without GROUP BY or UNION).
```sql
SELECT DISTINCT 
    p.name
FROM 
    person p
WHERE 
    EXISTS (SELECT 1 FROM person_order po WHERE po.person_id = p.id)
ORDER BY 
    p.name;
```

#### Exercise 06 - Restaurant metrics

Task: Show order statistics per pizzeria.
```sql
SELECT 
    pz.name,
    COUNT(po.id) AS count_of_orders,
    ROUND(AVG(m.price), 2) AS average_price,
    MAX(m.price) AS max_price,
    MIN(m.price) AS min_price
FROM 
    pizzeria pz
JOIN 
    menu m ON pz.id = m.pizzeria_id
JOIN 
    person_order po ON m.id = po.menu_id
GROUP BY 
    pz.name
ORDER BY 
    pz.name;
```

#### Exercise 07 - Average global rating

Task: Calculate average pizzeria rating.
```sql
SELECT 
    ROUND(AVG(rating), 4) AS global_rating
FROM 
    pizzeria;
```

#### Exercise 08 - Find pizzeria's restaurant locations

Task: Show order counts by address and pizzeria.
```sql
SELECT 
    p.address,
    pz.name,
    COUNT(po.id) AS count_of_orders
FROM 
    person p
JOIN 
    person_order po ON p.id = po.person_id
JOIN 
    menu m ON po.menu_id = m.id
JOIN 
    pizzeria pz ON m.pizzeria_id = pz.id
WHERE 
    p.address = p.address -- Ensures same city (as per task assumption)
GROUP BY 
    p.address, 
    pz.name
ORDER BY 
    p.address, 
    pz.name;
```


#### Exercise 09 - Explicit type transformation

Task: Calculate address-based age metrics with comparison.
```sql
SELECT 
    address,
    ROUND(MAX(age) - (MIN(age) / MAX(age)), 2) AS formula,
    ROUND(AVG(age), 2) AS average,
    (MAX(age) - (MIN(age) / MAX(age))) > AVG(age) AS comparison
FROM 
    person
GROUP BY 
    address
ORDER BY 
    address;
```
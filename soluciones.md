1. **Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.**

```sql
SELECT 
    products.name AS producto,
    companies.name AS empresa,
    citiesormunicipalities.name AS ciudad,
    MIN(companyproduct_prices.price) AS precio_mas_bajo
FROM 
    companyproduct_prices
JOIN companyproducts 
    ON companyproduct_prices.company_id = companyproducts.company_id 
    AND companyproduct_prices.product_id = companyproducts.product_id
JOIN products 
    ON companyproducts.product_id = products.id
JOIN companies 
    ON companyproducts.company_id = companies.id
JOIN citiesormunicipalities 
    ON companies.city_id = citiesormunicipalities.code
GROUP BY 
    products.name, companies.name, citiesormunicipalities.name;

```

2. Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.

   ```sql
   SELECT 
       customers.id,
       customers.name,
       COUNT(*) AS cantidad_calificaciones
   FROM 
       quality_products
   JOIN customers ON quality_products.customer_id = customers.id
   WHERE 
       quality_products.daterating >= NOW() - INTERVAL 6 MONTH
   GROUP BY 
       customers.id, customers.name
   ORDER BY 
       cantidad_calificaciones DESC
   LIMIT 5;
   
   ```

   

3.Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.

```sql
SELECT 
    categories.description AS categoria,
    unitofmeasure.description AS unidad_medida,
    COUNT(*) AS total_productos
FROM 
    companyproducts
JOIN products ON companyproducts.product_id = products.id
JOIN categories ON products.category_id = categories.id
JOIN unitofmeasure ON companyproducts.unitmeasure_id = unitofmeasure.id
GROUP BY 
    categories.description, unitofmeasure.description;

```

4.Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.

```sql
SELECT 
    products.id,
    products.name,
    AVG(quality_products.rating) AS promedio_producto
FROM 
    quality_products
JOIN products ON quality_products.product_id = products.id
GROUP BY 
    products.id, products.name
HAVING 
    AVG(quality_products.rating) > (
        SELECT AVG(rating) FROM quality_products
    );

```

5.Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.

```sql
SELECT 
    companies.id,
    companies.name
FROM 
    companies
LEFT JOIN rates ON companies.id = rates.company_id
WHERE 
    rates.company_id IS NULL;

```

6.Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.

```sql
SELECT 
    details_favorites.product_id,
    products.name,
    COUNT(DISTINCT favorites.customer_id) AS total_clientes
FROM 
    details_favorites
JOIN favorites ON details_favorites.favorite_id = favorites.id
JOIN products ON details_favorites.product_id = products.id
GROUP BY 
    details_favorites.product_id, products.name
HAVING 
    COUNT(DISTINCT favorites.customer_id) > 10;

```

7.Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.

```sql
SELECT 
    citiesormunicipalities.name AS ciudad,
    categories.description AS categoria,
    COUNT(companies.id) AS total_empresas
FROM 
    companies
JOIN citiesormunicipalities ON companies.city_id = citiesormunicipalities.code
JOIN categories ON companies.category_id = categories.id
GROUP BY 
    citiesormunicipalities.name, categories.description;

```

8.Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.

```sql
SELECT 
    ci.name AS ciudad,
    p.name AS producto,
    AVG(qp.rating) AS promedio_calificacion
FROM 
    quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN companies c ON qp.company_id = c.id
JOIN citiesormunicipalities ci ON c.city_id = ci.code
GROUP BY 
    ci.name, p.id, p.name
ORDER BY 
    ci.name, promedio_calificacion DESC;

```

9.Como técnico, quiero identificar productos sin unidad de medida asignada.

```sql
SELECT 
    products.id,
    products.name
FROM 
    products
LEFT JOIN companyproducts 
    ON products.id = companyproducts.product_id
WHERE 
    companyproducts.unitmeasure_id IS NULL;

```

10.Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.

```sql
SELECT 
    memberships.id,
    memberships.name
FROM 
    memberships
LEFT JOIN membershipbenefits 
    ON memberships.id = membershipbenefits.membership_id
WHERE 
    membershipbenefits.membership_id IS NULL;

```

11.Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.

```sql
SELECT 
    p.name AS producto,
    AVG(qp.rating) AS promedio
FROM 
    products p
JOIN quality_products qp ON p.id = qp.product_id
WHERE 
    p.category_id = 1 
GROUP BY 
    p.id, p.name;

```

12.Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.

```sql
SELECT 
    customer_id,
    COUNT(DISTINCT company_id) AS empresas_distintas
FROM 
    quality_products
GROUP BY 
    customer_id
HAVING 
    COUNT(DISTINCT company_id) > 1;

```

13.Como director, quiero identificar las ciudades con más clientes activos.

```sql
SELECT 
    citiesormunicipalities.name AS ciudad,
    COUNT(customers.id) AS total_clientes
FROM 
    customers
JOIN citiesormunicipalities ON customers.city_id = citiesormunicipalities.code
GROUP BY 
    citiesormunicipalities.name
ORDER BY 
    total_clientes DESC;

```

14.Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de `quality_products`.

```sql
SELECT 
    qp.company_id,
    p.name AS producto,
    AVG(qp.rating) AS promedio
FROM 
    quality_products qp
JOIN products p ON qp.product_id = p.id
GROUP BY 
    qp.company_id, p.id, p.name
ORDER BY 
    qp.company_id, promedio DESC;

```

15.Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.

```sql
SELECT 
    company_id,
    COUNT(DISTINCT product_id) AS cantidad_productos
FROM 
    companyproducts
GROUP BY 
    company_id
HAVING 
    COUNT(DISTINCT product_id) > 5;

```

16.Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.

```sql
SELECT 
    df.product_id,
    p.name AS producto
FROM 
    details_favorites df
JOIN products p ON df.product_id = p.id
LEFT JOIN quality_products qp 
    ON df.product_id = qp.product_id 
    AND df.favorite_id IN (
        SELECT id FROM favorites WHERE customer_id = qp.customer_id
    )
WHERE 
    qp.product_id IS NULL;

```

17.Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.

```sql
SELECT 
    audiences.description AS audiencia,
    benefits.description AS beneficio,
    benefits.detail
FROM 
    audiencebenefits
JOIN audiences ON audiencebenefits.audience_id = audiences.id
JOIN benefits ON audiencebenefits.benefit_id = benefits.id;

```

18.Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.

```sql
SELECT 
    citiesormunicipalities.name AS ciudad,
    companies.name AS empresa
FROM 
    companies
JOIN citiesormunicipalities ON companies.city_id = citiesormunicipalities.code
LEFT JOIN companyproducts ON companies.id = companyproducts.company_id
WHERE 
    companyproducts.company_id IS NULL;

```

19.Como técnico, deseo obtener todas las empresas con productos duplicados por nombre.

```sql
SELECT 
    companyproducts.company_id,
    products.name,
    COUNT(*) AS repeticiones
FROM 
    companyproducts
JOIN products ON companyproducts.product_id = products.id
GROUP BY 
    companyproducts.company_id, products.name
HAVING 
    COUNT(*) > 1;

```

20.Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido.

```sql
SELECT 
    customers.id AS cliente_id,
    customers.name AS cliente,
    COUNT(DISTINCT df.product_id) AS total_favoritos,
    AVG(qp.rating) AS promedio_calificacion
FROM 
    customers
LEFT JOIN favorites f ON f.customer_id = customers.id
LEFT JOIN details_favorites df ON f.id = df.favorite_id
LEFT JOIN quality_products qp 
    ON df.product_id = qp.product_id AND customers.id = qp.customer_id
GROUP BY 
    customers.id, customers.name;

```

21.Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.

```sql
SELECT 
    p.id,
    p.name,
    p.price
FROM 
    products p
WHERE 
    p.price > (
        SELECT AVG(p2.price)
        FROM products p2
        WHERE p2.category_id = p.category_id
    );

```

22.Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.

```sql
SELECT 
    c.id,
    c.name,
    COUNT(cp.product_id) AS total_productos
FROM 
    companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY 
    c.id, c.name
HAVING 
    COUNT(cp.product_id) > (
        SELECT AVG(productos_por_empresa)
        FROM (
            SELECT COUNT(product_id) AS productos_por_empresa
            FROM companyproducts
            GROUP BY company_id
        ) AS sub
    );

```

23.Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.

```sql
SELECT 
    p.id,
    p.name
FROM 
    favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
WHERE 
    f.customer_id = 1 -- <- Cambiar por el ID del cliente
    AND EXISTS (
        SELECT 1
        FROM quality_products qp
        WHERE qp.product_id = p.id
        AND qp.customer_id <> f.customer_id
    );

```

24.Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.

```sql
SELECT 
    df.product_id,
    p.name,
    COUNT(*) AS veces_favorito
FROM 
    details_favorites df
JOIN products p ON df.product_id = p.id
GROUP BY 
    df.product_id, p.name
HAVING 
    COUNT(*) = (
        SELECT MAX(total)
        FROM (
            SELECT COUNT(*) AS total
            FROM details_favorites
            GROUP BY product_id
        ) AS sub
    );

```

25.Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla `rates` ni en `quality_products`.

```sql
SELECT 
    c.id,
    c.name
FROM 
    customers c
LEFT JOIN customer_emails ce ON c.id = ce.customer_id
WHERE 
    NOT EXISTS (
        SELECT 1 FROM rates r WHERE r.customer_id = c.id
    )
    AND NOT EXISTS (
        SELECT 1 FROM quality_products q WHERE q.customer_id = c.id
    );

```

26.Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.

```sql
SELECT 
    p.id,
    p.name,
    AVG(qp.rating) AS promedio
FROM 
    products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY 
    p.id, p.name, p.category_id
HAVING 
    AVG(qp.rating) < (
        SELECT MIN(sub_prom)
        FROM (
            SELECT AVG(q.rating) AS sub_prom
            FROM products pr
            JOIN quality_products q ON pr.id = q.product_id
            WHERE pr.category_id = p.category_id
            GROUP BY pr.id
        ) AS sub
    );

```

27.Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.

```sql
SELECT 
    c.code,
    c.name
FROM 
    citiesormunicipalities c
WHERE 
    NOT EXISTS (
        SELECT 1 FROM customers cu WHERE cu.city_id = c.code
    );

```

28.Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.

```sql
SELECT 
    p.id,
    p.name
FROM 
    products p
WHERE 
    NOT EXISTS (
        SELECT 1 FROM quality_products qp WHERE qp.product_id = p.id
    );

```

29.Como auditor, quiero listar los beneficios que no están asignados a ninguna audiencia.

```sql
SELECT 
    b.id,
    b.description
FROM 
    benefits b
WHERE 
    NOT EXISTS (
        SELECT 1 FROM audiencebenefits ab WHERE ab.benefit_id = b.id
    );

```

30.Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.

```sql
SELECT 
    p.id,
    p.name
FROM 
    favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
WHERE 
    f.customer_id = 1 -- <- Cambiar por ID del cliente
    AND NOT EXISTS (
        SELECT 1 FROM companyproducts cp WHERE cp.product_id = p.id
    );

```

31.Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.

```sql
SELECT 
    DISTINCT p.id,
    p.name
FROM 
    companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id
WHERE 
    c.city_id IN (
        SELECT city_id
        FROM companies
        GROUP BY city_id
        HAVING COUNT(*) < 3
    );

```

32.Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.

```sql
SELECT 
    p.id,
    p.name,
    AVG(qp.rating) AS promedio
FROM 
    products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY 
    p.id, p.name
HAVING 
    AVG(qp.rating) > (
        SELECT AVG(rating) FROM quality_products
    );

```

33.Como gestor, quiero ver empresas que sólo venden productos de una única categoría.

```sql
SELECT 
    cp.company_id,
    c.name
FROM 
    companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id
GROUP BY 
    cp.company_id, c.name
HAVING 
    COUNT(DISTINCT p.category_id) = 1;

```

34.Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.

```sql
SELECT 
    p.id,
    p.name,
    MAX(cpp.price) AS max_precio
FROM 
    companyproduct_prices cpp
JOIN products p ON cpp.product_id = p.id
GROUP BY 
    p.id, p.name
HAVING 
    MAX(cpp.price) = (
        SELECT MAX(price) FROM companyproduct_prices
    );

```

35.Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.

```sql
SELECT 
    DISTINCT p.id,
    p.name
FROM 
    favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN quality_products qp ON p.id = qp.product_id
WHERE 
    f.customer_id = 1 -- <- Cambiar por tu ID
    AND qp.customer_id <> f.customer_id
    AND qp.rating > 4;

```

36.Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.

```sql
SELECT 
    p.id,
    p.name
FROM 
    products p
JOIN quality_products qp ON p.id = qp.product_id
WHERE 
    (p.image IS NULL OR p.image = '');

```

37.Como auditor, quiero ver los planes de membresía sin periodo vigente.

```sql
SELECT 
    m.id,
    m.name
FROM 
    memberships m
WHERE 
    NOT EXISTS (
        SELECT 1 FROM membershipperiods mp WHERE mp.membership_id = m.id
    );

```

38.Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.

```sql
SELECT 
    b.id,
    b.description,
    COUNT(ab.audience_id) AS total_audiencias
FROM 
    audiencebenefits ab
JOIN benefits b ON ab.benefit_id = b.id
GROUP BY 
    b.id, b.description
HAVING 
    COUNT(ab.audience_id) > 1;

```

39.Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.

```sql
SELECT 
    DISTINCT cp.company_id,
    c.name
FROM 
    companyproducts cp
JOIN companies c ON cp.company_id = c.id
WHERE 
    cp.unitmeasure_id IS NULL;

```

40.Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.

```sql
SELECT 
    customers.id,
    customers.name
FROM 
    customers
WHERE 
    customers.audience_id IN (
        SELECT DISTINCT audience_id
        FROM membershipbenefits
    )
    AND customers.id NOT IN (
        SELECT customer_id FROM favorites
    );

```

41. Obtener el promedio de calificación por producto

```sql
SELECT 
    product_id,
    AVG(rating) AS promedio
FROM 
    quality_products
GROUP BY 
    product_id;

```

42. Contar cuántos productos ha calificado cada cliente.

    

```sql
SELECT 
    customer_id,
    COUNT(*) AS cantidad_calificaciones
FROM 
    rates
GROUP BY 
    customer_id;

```

43.Sumar el total de beneficios asignados por audiencia.

```sql
SELECT 
    audience_id,
    COUNT(*) AS total_beneficios
FROM 
    audiencebenefits
GROUP BY 
    audience_id;

```

44.Calcular la media de productos por empresa.

```sql
SELECT 
    AVG(cantidad) AS media_productos_por_empresa
FROM (
    SELECT 
        company_id, 
        COUNT(*) AS cantidad
    FROM 
        companyproducts
    GROUP BY 
        company_id
) AS sub;

```

45.Contar el total de empresas por ciudad.

```sql
SELECT 
    city_id,
    COUNT(*) AS total_empresas
FROM 
    companies
GROUP BY 
    city_id;

```

46.Calcular el promedio de precios por unidad de medida.

```sql
SELECT 
    cp.unitmeasure_id,
    AVG(cpp.price) AS promedio_precio
FROM 
    companyproducts cp
JOIN companyproduct_prices cpp 
    ON cp.company_id = cpp.company_id 
    AND cp.product_id = cpp.product_id
GROUP BY 
    cp.unitmeasure_id;

```

47.Contar cuántos clientes hay por ciudad.

```sql
SELECT 
    city_id,
    COUNT(*) AS total_clientes
FROM 
    customers
GROUP BY 
    city_id;

```

48.Calcular planes de membresía por periodo.

```sql
SELECT 
    period_id,
    COUNT(*) AS total_planes
FROM 
    membershipperiods
GROUP BY 
    period_id;

```

49.Ver el promedio de calificaciones dadas por un cliente a sus favoritos.

```sql
SELECT 
    f.customer_id,
    AVG(qp.rating) AS promedio_rating
FROM 
    favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN quality_products qp 
    ON df.product_id = qp.product_id 
    AND f.customer_id = qp.customer_id
GROUP BY 
    f.customer_id;

```

50. Consultar la fecha más reciente en que se calificó un producto.

```sql
SELECT 
    product_id,
    MAX(daterating) AS ultima_calificacion
FROM 
    quality_products
GROUP BY 
    product_id;

```

51.Obtener la desviación estándar de precios por categoría.

```sql
SELECT 
    p.category_id,
    STDDEV(cpp.price) AS desviacion_estandar
FROM 
    companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companyproduct_prices cpp 
    ON cp.company_id = cpp.company_id 
    AND cp.product_id = cpp.product_id
GROUP BY 
    p.category_id;

```

52.Contar cuántas veces un producto fue favorito.

```sql
SELECT 
    product_id,
    COUNT(*) AS veces_favorito
FROM 
    details_favorites
GROUP BY 
    product_id;

```

53.Calcular el porcentaje de productos evaluados.

```sql
SELECT 
    ROUND(
        (SELECT COUNT(DISTINCT product_id) FROM quality_products) * 100.0 / 
        (SELECT COUNT(*) FROM products), 
        2
    ) AS porcentaje_evaluados;

```

54.Ver el promedio de rating por encuesta.

```sql
SELECT 
    poll_id,
    AVG(rating) AS promedio_rating
FROM 
    rates
GROUP BY 
    poll_id;

```

55.Calcular el promedio y total de beneficios por plan.

```sql
SELECT 
    membership_id,
    COUNT(*) AS total_beneficios
FROM 
    membershipbenefits
GROUP BY 
    membership_id;

```

56.Obtener media y varianza de precios por empresa.

```sql
SELECT 
    company_id,
    AVG(price) AS media_precio,
    VARIANCE(price) AS varianza_precio
FROM 
    companyproduct_prices
GROUP BY 
    company_id;

```

57.Ver total de productos disponibles en la ciudad del cliente.

```sql
SELECT 
    c.city_id,
    COUNT(DISTINCT cp.product_id) AS productos_disponibles
FROM 
    companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY 
    c.city_id;

```

58.Contar productos únicos por tipo de empresa.

```sql
SELECT 
    type_id,
    COUNT(DISTINCT cp.product_id) AS productos_unicos
FROM 
    companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY 
    type_id;

```

59.Ver total de clientes sin correo electrónico registrado.

```sql
SELECT 
    COUNT(*) AS clientes_sin_correo
FROM 
    customers
WHERE 
    id NOT IN (
        SELECT DISTINCT customer_id FROM customer_emails
    );

```

60.Empresa con más productos calificados.

```sql
SELECT 
    c.id AS company_id,
    c.name AS empresa,
    COUNT(DISTINCT qp.product_id) AS productos_calificados
FROM 
    companies c
JOIN companyproducts cp ON c.id = cp.company_id
JOIN quality_products qp ON cp.product_id = qp.product_id
GROUP BY 
    c.id, c.name
ORDER BY 
    productos_calificados DESC
LIMIT 1;

```

61. **Registrar una nueva calificación y actualizar el promedio**

```sql
DELIMITER //

CREATE PROCEDURE registrar_calificacion(
  IN p_product_id INT,
  IN p_customer_id INT,
  IN p_poll_id INT,
  IN p_company_id VARCHAR(20),
  IN p_rating DOUBLE
)
BEGIN
  INSERT INTO quality_products(product_id, customer_id, poll_id, company_id, daterating, rating)
  VALUES (p_product_id, p_customer_id, p_poll_id, p_company_id, NOW(), p_rating);

  UPDATE products
  SET detail = CONCAT('Promedio actualizado: ', ROUND(
    (SELECT AVG(rating)
     FROM quality_products
     WHERE product_id = p_product_id), 2))
  WHERE id = p_product_id;
END;
//

DELIMITER ;
CALL registrar_calificacion(1, 1, 1, '6', 4.5);

```

62.Insertar empresa y asociar productos por defecto.

```sql
DELIMITER //

CREATE PROCEDURE insertar_empresa_y_productos(
  IN p_id VARCHAR(20),
  IN p_name VARCHAR(80),
  IN p_type_id INT,
  IN p_category_id INT,
  IN p_city_id VARCHAR(6),
  IN p_audience_id INT
)
BEGIN
  INSERT INTO companies(id, name, type_id, category_id, city_id, audience_id)
  VALUES (p_id, p_name, p_type_id, p_category_id, p_city_id, p_audience_id);

  INSERT INTO companyproducts(company_id, product_id, unitmeasure_id)
  SELECT p_id, id, 1
  FROM products
  WHERE id IN (1, 2, 3);
END;
//

DELIMITER ;
CALL insertar_empresa_y_productos('C010', 'Empresa Default', 1, 1, '05001', 1);

```



63. **Añadir producto favorito validando duplicados**

```sql

DELIMITER //

CREATE PROCEDURE agregar_favorito(
  IN p_customer_id INT,
  IN p_product_id INT
)
BEGIN
  DECLARE fav_id INT;

  SELECT id INTO fav_id FROM favorites
  WHERE customer_id = p_customer_id
  LIMIT 1;

  IF fav_id IS NOT NULL AND NOT EXISTS (
    SELECT 1 FROM details_favorites
    WHERE favorite_id = fav_id AND product_id = p_product_id
  ) THEN
    INSERT INTO details_favorites(favorite_id, product_id, category_id, name, detail, added_date)
    SELECT fav_id, id, category_id, name, detail, NOW()
    FROM products WHERE id = p_product_id;
  END IF;
END;
//

DELIMITER ;
CALL agregar_favorito(1, 1);

```

64.**Generar resumen mensual de calificaciones por empresa**

```sql
CREATE TABLE IF NOT EXISTS resumen_calificaciones (
  id INT AUTO_INCREMENT PRIMARY KEY,
  company_id VARCHAR(20),
  mes VARCHAR(7),
  promedio DOUBLE
);

DELIMITER //
CREATE PROCEDURE generar_resumen_calificaciones()
BEGIN
  INSERT INTO resumen_calificaciones(company_id, mes, promedio)
  SELECT company_id,
         DATE_FORMAT(daterating, '%Y-%m'),
         ROUND(AVG(rating), 2)
  FROM quality_products
  GROUP BY company_id, DATE_FORMAT(daterating, '%Y-%m');
END;
//

DELIMITER ;
CALL generar_resumen_calificaciones();
```



65.**Calcular beneficios activos por membresía**

```sql
DELIMITER //
CREATE PROCEDURE beneficios_activos_membresia()
BEGIN
  SELECT
    mb.membership_id,
    mb.period_id,
    mb.audience_id,
    mb.benefit_id
  FROM membershipbenefits mb
  JOIN membershipperiods mp
    ON mb.membership_id = mp.membership_id
    AND mb.period_id = mp.period_id;
END;
//

DELIMITER ;
CALL beneficios_activos_membresia();

```



66.**Eliminar productos huérfanos**

```sql
DELIMITER //
CREATE PROCEDURE eliminar_productos_huerfanos()
BEGIN
  DELETE FROM products
  WHERE id NOT IN (SELECT product_id FROM quality_products)
    AND id NOT IN (SELECT product_id FROM companyproducts);
END;
//

DELIMITER ;
CALL eliminar_productos_huerfanos();

```



67.**Actualizar precios de productos por categoría**

```sql
DELIMITER //
CREATE PROCEDURE actualizar_precios_categoria(
  IN p_category_id INT,
  IN p_factor DECIMAL(5,2)
)
BEGIN
  UPDATE companyproduct_prices cpp
  JOIN products p ON cpp.product_id = p.id
  SET cpp.price = cpp.price * p_factor
  WHERE p.category_id = p_category_id;
END;
//

DELIMITER ;
CALL actualizar_precios_categoria(1, 1.10);

```



68.**Validar inconsistencia entre `rates` y `quality_products`**

```sql
CREATE TABLE IF NOT EXISTS errores_log (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  company_id VARCHAR(20),
  poll_id INT,
  error_msg TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE PROCEDURE validar_inconsistencias()
BEGIN
  INSERT INTO errores_log(customer_id, company_id, poll_id, error_msg, fecha)
  SELECT r.customer_id, r.company_id, r.poll_id,
         'No coincide con quality_products', NOW()
  FROM rates r
  WHERE NOT EXISTS (
    SELECT 1 FROM quality_products qp
    WHERE qp.customer_id = r.customer_id
      AND qp.company_id = r.company_id
      AND qp.poll_id = r.poll_id
  );
END;
//

DELIMITER ;
CALL validar_inconsistencias();

```



69.**Asignar beneficios a nuevas audiencias**

```sql
DELIMITER //
CREATE PROCEDURE asignar_beneficio_audiencia(
  IN p_benefit_id INT,
  IN p_audience_id INT
)
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM audiencebenefits
    WHERE benefit_id = p_benefit_id AND audience_id = p_audience_id
  ) THEN
    INSERT INTO audiencebenefits(audience_id, benefit_id)
    VALUES (p_audience_id, p_benefit_id);
  END IF;
END;
//

DELIMITER ;
CALL asignar_beneficio_audiencia(1, 2);

```



70.**Activar planes de membresía vencidos con pago confirmado**

```sql
ALTER TABLE membershipperiods
ADD COLUMN status VARCHAR(20) DEFAULT 'INACTIVA',
ADD COLUMN pago_confirmado BOOLEAN DEFAULT FALSE,
ADD COLUMN fecha_fin DATE;

DELIMITER //
CREATE PROCEDURE activar_membresias_vencidas()
BEGIN
  UPDATE membershipperiods
  SET status = 'ACTIVA'
  WHERE fecha_fin < NOW()
    AND pago_confirmado = TRUE
    AND status != 'ACTIVA';
END;
//

DELIMITER ;
CALL activar_membresias_vencidas();

```



71.**Listar productos favoritos del cliente con su calificación**

```sql
DELIMITER //
CREATE PROCEDURE favoritos_con_calificacion(
  IN p_customer_id INT
)
BEGIN
  SELECT
    df.product_id,
    df.name,
    df.detail,
    ROUND(AVG(qp.rating), 2) AS promedio_rating
  FROM favorites f
  JOIN details_favorites df ON f.id = df.favorite_id
  LEFT JOIN quality_products qp ON df.product_id = qp.product_id
  WHERE f.customer_id = p_customer_id
  GROUP BY df.product_id, df.name, df.detail;
END;
//
DELIMITER ;
CALL favoritos_con_calificacion(1);

```



72.**Registrar encuesta y sus preguntas asociadas**

```sql
CREATE TABLE IF NOT EXISTS poll_questions (
  id INT AUTO_INCREMENT PRIMARY KEY,
  poll_id INT,
  question TEXT
);

DELIMITER //
CREATE PROCEDURE registrar_encuesta_completa(
  IN p_name VARCHAR(80),
  IN p_description TEXT,
  IN p_categorypoll_id INT,
  IN p_question1 TEXT,
  IN p_question2 TEXT
)
BEGIN
  DECLARE poll_id INT;
  INSERT INTO polls(name, description, isactive, categorypoll_id)
  VALUES (p_name, p_description, TRUE, p_categorypoll_id);
  SET poll_id = LAST_INSERT_ID();
  INSERT INTO poll_questions(poll_id, question) VALUES (poll_id, p_question1), (poll_id, p_question2);
END;
//
DELIMITER ;
CALL registrar_encuesta_completa('Encuesta Clientes', 'Opinión sobre servicio', 1, '¿Cómo califica el servicio?', '¿Recomendaría este producto?');

```



73.**Eliminar favoritos antiguos sin calificaciones**

```sql
DELIMITER //
CREATE PROCEDURE eliminar_favoritos_antiguos()
BEGIN
  DELETE df
  FROM details_favorites df
  JOIN favorites f ON df.favorite_id = f.id
  LEFT JOIN quality_products qp ON df.product_id = qp.product_id AND f.customer_id = qp.customer_id
  WHERE qp.rating IS NULL AND df.added_date < NOW() - INTERVAL 12 MONTH;
END;
//
DELIMITER ;
CALL eliminar_favoritos_antiguos();

```



74.**Asociar beneficios automáticamente por audiencia**

```sql
DELIMITER //
CREATE PROCEDURE asociar_beneficios_por_audiencia(
  IN p_audience_id INT
)
BEGIN
  INSERT INTO audiencebenefits(audience_id, benefit_id)
  SELECT p_audience_id, b.id
  FROM benefits b
  WHERE NOT EXISTS (
    SELECT 1 FROM audiencebenefits ab
    WHERE ab.audience_id = p_audience_id AND ab.benefit_id = b.id
  );
END;
//
DELIMITER ;
CALL asociar_beneficios_por_audiencia(2);
```



75.**Historial de cambios de precio**

```sql
CREATE TABLE IF NOT EXISTS historial_precios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  company_id VARCHAR(20),
  product_id INT,
  old_price DECIMAL(10,2),
  new_price DECIMAL(10,2),
  change_date DATETIME
);

DELIMITER //
CREATE PROCEDURE registrar_cambio_precio(
  IN p_company_id VARCHAR(20),
  IN p_product_id INT,
  IN p_nuevo_precio DECIMAL(10,2)
)
BEGIN
  DECLARE precio_anterior DECIMAL(10,2);
  SELECT price INTO precio_anterior
  FROM companyproduct_prices
  WHERE company_id = p_company_id AND product_id = p_product_id
  ORDER BY start_date DESC LIMIT 1;

  INSERT INTO historial_precios(company_id, product_id, old_price, new_price, change_date)
  VALUES (p_company_id, p_product_id, precio_anterior, p_nuevo_precio, NOW());

  UPDATE companyproduct_prices
  SET price = p_nuevo_precio
  WHERE company_id = p_company_id AND product_id = p_product_id;
END;
//
DELIMITER ;
CALL registrar_cambio_precio('C001', 1, 45.00);

```



76.**Registrar encuesta activa automáticamente**

```sql
DELIMITER //
CREATE PROCEDURE registrar_encuesta_activa(
  IN p_name VARCHAR(80),
  IN p_description TEXT,
  IN p_categorypoll_id INT
)
BEGIN
  INSERT INTO polls(name, description, isactive, categorypoll_id)
  VALUES (p_name, p_description, TRUE, p_categorypoll_id);
END;
//
DELIMITER ;

CALL registrar_encuesta_activa('Nueva encuesta activa', 'Encuesta generada automáticamente', 2);

```



77.**Actualizar unidad de medida de productos sin afectar ventas**

```sql
DELIMITER //
CREATE PROCEDURE actualizar_unidad_si_no_ha_sido_vendido(
  IN p_product_id INT,
  IN p_nueva_unidad_id INT
)
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM companyproduct_prices WHERE product_id = p_product_id
  ) THEN
    UPDATE products
    SET unit_id = p_nueva_unidad_id
    WHERE id = p_product_id;
  END IF;
END;
//
DELIMITER ;

CALL actualizar_unidad_si_no_ha_sido_vendido(3, 2);

```



78.**Recalcular promedios de calidad semanalmente**

```sql
DELIMITER //
CREATE PROCEDURE promedio_calidad_semanal()
BEGIN
  SELECT
    product_id,
    ROUND(AVG(rating), 2) AS promedio,
    COUNT(*) AS total
  FROM quality_products
  WHERE daterating >= NOW() - INTERVAL 7 DAY
  GROUP BY product_id;
END;
//
DELIMITER ;
CALL promedio_calidad_semanal();

```



79.**Validar claves foráneas entre calificaciones y encuestas**

```sql
DELIMITER //
CREATE PROCEDURE validar_poll_id_en_calificaciones()
BEGIN
  SELECT DISTINCT qp.poll_id
  FROM quality_products qp
  LEFT JOIN polls p ON qp.poll_id = p.id
  WHERE p.id IS NULL;
END;
//
DELIMITER ;
CALL validar_poll_id_en_calificaciones();
```



80.**Generar el top 10 de productos más calificados por ciudad**

```sql
DELIMITER //
CREATE PROCEDURE top_10_productos_por_ciudad()
BEGIN
  SELECT
    cm.name AS ciudad,
    p.name AS producto,
    COUNT(qp.rating) AS total_calificaciones,
    ROUND(AVG(qp.rating), 2) AS promedio
  FROM quality_products qp
  JOIN companies c ON qp.company_id = c.id
  JOIN products p ON qp.product_id = p.id
  JOIN citiesormunicipalities cm ON c.city_id = cm.code
  GROUP BY cm.name, p.name
  ORDER BY cm.name, total_calificaciones DESC
  LIMIT 10;
END;
//
DELIMITER ;
CALL top_10_productos_por_ciudad();

```



81.**Actualizar la fecha de modificación de un producto**

```sql
ALTER TABLE products ADD COLUMN updated_at DATETIME DEFAULT NULL;
DELIMITER //
CREATE TRIGGER trg_update_product_updated_at
BEFORE UPDATE ON products
FOR EACH ROW
BEGIN
  SET NEW.updated_at = NOW();
END;//
DELIMITER ;

```



82.**Registrar log cuando un cliente califica un producto**

```sql
CREATE TABLE IF NOT EXISTS log_acciones (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  product_id INT,
  accion TEXT,
  fecha DATETIME
);
DELIMITER //
CREATE TRIGGER trg_log_calificacion
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
  INSERT INTO log_acciones(customer_id, company_id, poll_id, accion, fecha)
  VALUES (NEW.customer_id, NEW.company_id, NEW.poll_id, 'Calificó un producto', NOW());
END;//
DELIMITER ;

```



83.**Impedir insertar productos sin unidad de medida**

```sql
DELIMITER //
CREATE TRIGGER trg_prevent_null_unit
BEFORE INSERT ON companyproducts
FOR EACH ROW
BEGIN
  IF NEW.unitmeasure_id IS NULL THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'No se puede insertar producto sin unidad de medida.';
  END IF;
END;//
DELIMITER ;
```



84.**Validar calificaciones no mayores a 5**

```sql
DELIMITER //
CREATE TRIGGER trg_validar_rating
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
  IF NEW.rating > 5 THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La calificación no puede ser mayor a 5';
  END IF;
END;//
DELIMITER ;

```



85.**Actualizar estado de membresía cuando vence**

```sql
DELIMITER //
CREATE TRIGGER trg_membership_vencida
BEFORE UPDATE ON membershipperiods
FOR EACH ROW
BEGIN
  IF NEW.fecha_fin < CURDATE() THEN
    SET NEW.status = 'INACTIVA';
  END IF;
END;//
DELIMITER ;
```



86.**Evitar duplicados de productos por empresa**

```sql
DELIMITER //
CREATE TRIGGER trg_no_producto_duplicado
BEFORE INSERT ON companyproducts
FOR EACH ROW
BEGIN
  IF EXISTS (
    SELECT 1 FROM companyproducts
    WHERE company_id = NEW.company_id AND product_id = NEW.product_id
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Este producto ya está asociado a la empresa.';
  END IF;
END;//
DELIMITER 
```



87.**Enviar notificación al añadir un favorito**

```sql
CREATE TABLE IF NOT EXISTS notificaciones (
  id INT AUTO_INCREMENT PRIMARY KEY,
  mensaje TEXT,
  fecha DATETIME
);
DELIMITER //
CREATE TRIGGER trg_notificacion_favorito
AFTER INSERT ON details_favorites
FOR EACH ROW
BEGIN
  INSERT INTO notificaciones(mensaje, fecha)
  VALUES (CONCAT('Se añadió favorito: ', NEW.name), NOW());
END;//
DELIMITER ;

```



88.**Insertar fila en `quality_products` tras calificación**

```sql
DELIMITER //
CREATE TRIGGER trg_rates_to_quality
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
  INSERT INTO quality_products(product_id, customer_id, poll_id, company_id, daterating, rating)
  VALUES (NULL, NEW.customer_id, NEW.poll_id, NEW.company_id, NOW(), NEW.rating);
END;//
DELIMITER ;

```



89.**Eliminar favoritos si se elimina el producto**

```sql
DELIMITER //
CREATE TRIGGER trg_delete_favoritos
AFTER DELETE ON products
FOR EACH ROW
BEGIN
  DELETE FROM details_favorites WHERE product_id = OLD.id;
END;//
DELIMITER ;

```



90.**Bloquear modificación de audiencias activas**

```sql
ALTER TABLE audiences ADD COLUMN estado VARCHAR(20) DEFAULT 'ACTIVA';
DELIMITER //
CREATE TRIGGER trg_bloquear_audiencia_activa
BEFORE UPDATE ON audiences
FOR EACH ROW
BEGIN
  IF OLD.estado = 'ACTIVA' THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'No se puede modificar una audiencia activa.';
  END IF;
END;//
DELIMITER ;

```



91.**Recalcular promedio de calidad del producto tras nueva evaluación**

```sql
DELIMITER //
CREATE TRIGGER trg_actualizar_promedio
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
  UPDATE products p
  JOIN quality_products qp ON p.id = qp.product_id
  SET p.detail = CONCAT('Promedio actualizado: ', ROUND(
    (SELECT AVG(rating) FROM quality_products WHERE product_id = qp.product_id), 2))
  WHERE qp.customer_id = NEW.customer_id AND qp.company_id = NEW.company_id AND qp.poll_id = NEW.poll_id;
END;//
DELIMITER ;
```



92.**Registrar asignación de nuevo beneficio**

```sql
CREATE TABLE IF NOT EXISTS bitacora (
  id INT AUTO_INCREMENT PRIMARY KEY,
  tabla_afectada VARCHAR(50),
  accion TEXT,
  fecha DATETIME
);
DELIMITER //
CREATE TRIGGER trg_log_beneficio_membresia
AFTER INSERT ON membershipbenefits
FOR EACH ROW
BEGIN
  INSERT INTO bitacora(tabla_afectada, accion, fecha)
  VALUES ('membershipbenefits', 'Se asignó nuevo beneficio', NOW());
END;//

CREATE TRIGGER trg_log_beneficio_audiencia
AFTER INSERT ON audiencebenefits
FOR EACH ROW
BEGIN
  INSERT INTO bitacora(tabla_afectada, accion, fecha)
  VALUES ('audiencebenefits', 'Se asignó nuevo beneficio', NOW());
END;//
DELIMITER ;

```



93.**Impedir doble calificación por parte del cliente**

```sql
DELIMITER //
CREATE TRIGGER trg_no_doble_calificacion
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
  IF EXISTS (
    SELECT 1 FROM rates
    WHERE customer_id = NEW.customer_id
      AND poll_id = NEW.poll_id
      AND company_id = NEW.company_id
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Ya existe una calificación para esta combinación.';
  END IF;
END;//
DELIMITER ;

```



94.**Validar correos duplicados en clientes**

```sql
DELIMITER //
CREATE TRIGGER trg_validar_correo_cliente
BEFORE INSERT ON customer_emails
FOR EACH ROW
BEGIN
  IF EXISTS (
    SELECT 1 FROM customer_emails WHERE email = NEW.email
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'El correo ya está registrado.';
  END IF;
END;//
DELIMITER ;

```



95.**Eliminar detalles de favoritos huérfanos**

```sql
DELIMITER //
CREATE TRIGGER trg_eliminar_detalles_huerfanos
AFTER DELETE ON favorites
FOR EACH ROW
BEGIN
  DELETE FROM details_favorites WHERE favorite_id = OLD.id;
END;//
DELIMITER ;

```



96.**Actualizar campo `updated_at` en `companies`**

```sql
ALTER TABLE companies ADD COLUMN updated_at DATETIME DEFAULT NULL;
DELIMITER //
CREATE TRIGGER trg_actualizar_updated_at_companies
BEFORE UPDATE ON companies
FOR EACH ROW
BEGIN
  SET NEW.updated_at = NOW();
END;//
DELIMITER ;

```



97.**Impedir borrar ciudad si hay empresas activas**

```sql
DELIMITER //
CREATE TRIGGER trg_bloquear_delete_ciudad
BEFORE DELETE ON citiesormunicipalities
FOR EACH ROW
BEGIN
  IF EXISTS (
    SELECT 1 FROM companies WHERE city_id = OLD.code
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'No se puede eliminar una ciudad con empresas registradas.';
  END IF;
END;//
DELIMITER ;

```



98.**Registrar cambios de estado en encuestas**

```sql
CREATE TABLE IF NOT EXISTS log_estado_encuesta (
  id INT AUTO_INCREMENT PRIMARY KEY,
  poll_id INT,
  nuevo_estado BOOLEAN,
  fecha DATETIME
);
DELIMITER //
CREATE TRIGGER trg_log_estado_encuesta
AFTER UPDATE ON polls
FOR EACH ROW
BEGIN
  IF OLD.isactive <> NEW.isactive THEN
    INSERT INTO log_estado_encuesta(poll_id, nuevo_estado, fecha)
    VALUES (NEW.id, NEW.isactive, NOW());
  END IF;
END;//
DELIMITER ;

```



99.**Sincronizar `rates` y `quality_products`**

```sql
DELIMITER //
CREATE TRIGGER trg_sync_quality_from_rates
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM quality_products
    WHERE customer_id = NEW.customer_id AND company_id = NEW.company_id AND poll_id = NEW.poll_id
  ) THEN
    INSERT INTO quality_products(product_id, customer_id, poll_id, company_id, daterating, rating)
    VALUES (NULL, NEW.customer_id, NEW.poll_id, NEW.company_id, NOW(), NEW.rating);
  END IF;
END;//
DELIMITER ;

```



100.**Eliminar productos sin relación a empresas**

```sql
DELIMITER //
CREATE TRIGGER trg_eliminar_producto_sin_empresa
AFTER DELETE ON companyproducts
FOR EACH ROW
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM companyproducts WHERE product_id = OLD.product_id
  ) THEN
    DELETE FROM products WHERE id = OLD.product_id;
  END IF;
END;//
DELIMITER ;

```



101.**Borrar productos sin actividad cada 6 meses**

```sql
SET GLOBAL event_scheduler = ON;
```

```sql
CREATE EVENT IF NOT EXISTS evt_borrar_productos_inactivos
ON SCHEDULE EVERY 6 MONTH
DO
  DELETE FROM products
  WHERE id NOT IN (SELECT product_id FROM quality_products)
    AND id NOT IN (SELECT product_id FROM details_favorites)
    AND id NOT IN (SELECT product_id FROM companyproducts);
    
```



102.**Recalcular el promedio de calificaciones semanalmente**

```sql
CREATE TABLE IF NOT EXISTS product_metrics (
  product_id INT PRIMARY KEY,
  promedio DOUBLE,
  total_calificaciones INT,
  actualizado DATETIME
);

CREATE EVENT IF NOT EXISTS evt_actualizar_metricas
ON SCHEDULE EVERY 1 WEEK
DO
  REPLACE INTO product_metrics(product_id, promedio, total_calificaciones, actualizado)
  SELECT product_id, ROUND(AVG(rating), 2), COUNT(*), NOW()
  FROM quality_products
  GROUP BY product_id;
  
```



103.**Actualizar precios según inflación mensual**

```sql
CREATE EVENT IF NOT EXISTS evt_actualizar_precios_inflacion
ON SCHEDULE EVERY 1 MONTH
DO
  UPDATE companyproduct_prices SET price = price * 1.03;

```



104.**Crear backups lógicos diariamente**

```sql
CREATE TABLE IF NOT EXISTS products_backup LIKE products;
CREATE TABLE IF NOT EXISTS rates_backup LIKE rates;

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_backup_diario
ON SCHEDULE EVERY 1 DAY STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
BEGIN
  DELETE FROM products_backup;
  INSERT INTO products_backup SELECT * FROM products;
  DELETE FROM rates_backup;
  INSERT INTO rates_backup SELECT * FROM rates;
END;//
DELIMITER ;

```



105.**Notificar sobre productos favoritos sin calificar**

```sql
CREATE TABLE IF NOT EXISTS recordatorios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  product_id INT,
  mensaje TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_favoritos_no_calificados
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
  INSERT INTO recordatorios(customer_id, product_id, mensaje, fecha)
  SELECT f.customer_id, df.product_id, 'Tienes productos favoritos sin calificar', NOW()
  FROM favorites f
  JOIN details_favorites df ON f.id = df.favorite_id
  LEFT JOIN quality_products qp ON df.product_id = qp.product_id AND f.customer_id = qp.customer_id
  WHERE qp.rating IS NULL;
END;//
DELIMITER ;

```



106.**Revisar inconsistencias entre empresa y productos**

```sql
CREATE TABLE IF NOT EXISTS errores_log (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  company_id VARCHAR(20),
  poll_id INT,
  error_msg TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_inconsistencias_empresas_productos
ON SCHEDULE EVERY 1 WEEK STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
BEGIN
  INSERT INTO errores_log(company_id, error_msg, fecha)
  SELECT p.company_id, 'Producto sin empresa relacionada', NOW()
  FROM quality_products p
  WHERE NOT EXISTS (
    SELECT 1 FROM companyproducts cp WHERE cp.product_id = p.product_id
  );

  INSERT INTO errores_log(company_id, error_msg, fecha)
  SELECT c.id, 'Empresa sin productos relacionados', NOW()
  FROM companies c
  WHERE NOT EXISTS (
    SELECT 1 FROM companyproducts cp WHERE cp.company_id = c.id
  );
END;//
DELIMITER ;

```



107.**Archivar membresías vencidas diariamente**

```sql
DELIMITER //
CREATE EVENT IF NOT EXISTS evt_archivar_membresias
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  UPDATE membershipperiods
  SET status = 'INACTIVA'
  WHERE fecha_fin < CURDATE() AND status != 'INACTIVA';
END;//
DELIMITER ;

```



108.**Notificar beneficios nuevos a usuarios semanalmente**

```sql
CREATE TABLE IF NOT EXISTS notificaciones (
  id INT AUTO_INCREMENT PRIMARY KEY,
  mensaje TEXT,
  fecha DATETIME
);

ALTER TABLE benefits ADD COLUMN created_at DATETIME DEFAULT NOW();

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_notificar_beneficios
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
  INSERT INTO notificaciones(mensaje, fecha)
  SELECT CONCAT('Nuevo beneficio disponible: ID ', id), NOW()
  FROM benefits
  WHERE created_at >= NOW() - INTERVAL 7 DAY;
END;//
DELIMITER ;

```



109.**Calcular cantidad de favoritos por cliente mensualmente**

```sql
CREATE TABLE IF NOT EXISTS favoritos_resumen (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  total_favoritos INT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_resumen_favoritos
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
  INSERT INTO favoritos_resumen(customer_id, total_favoritos, fecha)
  SELECT f.customer_id, COUNT(df.product_id), NOW()
  FROM favorites f
  JOIN details_favorites df ON f.id = df.favorite_id
  GROUP BY f.customer_id;
END;//
DELIMITER ;

```



110.**Validar claves foráneas semanalmente**

```sql
CREATE TABLE IF NOT EXISTS inconsistencias_fk (
  id INT AUTO_INCREMENT PRIMARY KEY,
  tabla_origen VARCHAR(50),
  referencia VARCHAR(100),
  error TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_validar_fk
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
  INSERT INTO inconsistencias_fk(tabla_origen, referencia, error, fecha)
  SELECT 'quality_products', CONCAT('product_id: ', product_id), 'Producto inexistente', NOW()
  FROM quality_products qp
  WHERE NOT EXISTS (SELECT 1 FROM products p WHERE p.id = qp.product_id);

  INSERT INTO inconsistencias_fk(tabla_origen, referencia, error, fecha)
  SELECT 'rates', CONCAT('customer_id: ', customer_id), 'Cliente inexistente', NOW()
  FROM rates r
  WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.id = r.customer_id);
END;//
DELIMITER ;

```



111.**Eliminar calificaciones inválidas antiguas**

```sql
DELIMITER //
CREATE EVENT IF NOT EXISTS evt_eliminar_calificaciones_invalidas
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
  DELETE FROM rates
  WHERE (rating IS NULL OR rating < 0)
    AND daterating < NOW() - INTERVAL 3 MONTH;
END;//
DELIMITER ;

```



112.**Cambiar estado de encuestas inactivas automáticamente**

```sql
DELIMITER //
CREATE EVENT IF NOT EXISTS evt_inactivar_encuestas
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
  UPDATE polls
  SET isactive = FALSE
  WHERE id NOT IN (
    SELECT DISTINCT poll_id FROM quality_products
    WHERE daterating >= NOW() - INTERVAL 6 MONTH
  );
END;//
DELIMITER ;

```



113.**Registrar auditorías de forma periódica**

```sql
CREATE TABLE IF NOT EXISTS auditorias_diarias (
  id INT AUTO_INCREMENT PRIMARY KEY,
  total_productos INT,
  total_clientes INT,
  total_empresas INT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_auditoria_diaria
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  INSERT INTO auditorias_diarias(total_productos, total_clientes, total_empresas, fecha)
  SELECT
    (SELECT COUNT(*) FROM products),
    (SELECT COUNT(*) FROM customers),
    (SELECT COUNT(*) FROM companies),
    NOW();
END;//
DELIMITER ;

```



114.**Notificar métricas de calidad a empresas**

```sql
CREATE TABLE IF NOT EXISTS notificaciones_empresa (
  id INT AUTO_INCREMENT PRIMARY KEY,
  company_id VARCHAR(20),
  mensaje TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_metricas_calidad
ON SCHEDULE EVERY 1 WEEK STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
BEGIN
  INSERT INTO notificaciones_empresa(company_id, mensaje, fecha)
  SELECT qp.company_id,
         CONCAT('Promedio de calidad: ', ROUND(AVG(qp.rating), 2)),
         NOW()
  FROM quality_products qp
  GROUP BY qp.company_id;
END;//
DELIMITER ;

```



115.**Recordar renovación de membresías**

```sql
CREATE TABLE IF NOT EXISTS recordatorios_membresias (
  id INT AUTO_INCREMENT PRIMARY KEY,
  membership_id INT,
  mensaje TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_recordar_membresia
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  INSERT INTO recordatorios_membresias(membership_id, mensaje, fecha)
  SELECT DISTINCT mp.membership_id,
         'Tu membresía está próxima a vencer',
         NOW()
  FROM membershipperiods mp
  WHERE mp.fecha_fin BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY;
END;//
DELIMITER ;

```



116.**Reordenar estadísticas generales cada semana**

```sql
CREATE TABLE IF NOT EXISTS estadisticas (
  id INT AUTO_INCREMENT PRIMARY KEY,
  total_productos INT,
  total_clientes INT,
  total_calificaciones INT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_reordenar_estadisticas
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
  INSERT INTO estadisticas(total_productos, total_clientes, total_calificaciones, fecha)
  SELECT
    (SELECT COUNT(*) FROM products),
    (SELECT COUNT(*) FROM customers),
    (SELECT COUNT(*) FROM quality_products),
    NOW();
END;//
DELIMITER ;

```



117.**Crear resúmenes temporales de uso por categoría**

```sql
CREATE TABLE IF NOT EXISTS resumen_categoria (
  id INT AUTO_INCREMENT PRIMARY KEY,
  category_id INT,
  total_calificados INT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_resumen_categoria
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
  INSERT INTO resumen_categoria(category_id, total_calificados, fecha)
  SELECT p.category_id, COUNT(DISTINCT qp.product_id), NOW()
  FROM quality_products qp
  JOIN products p ON qp.product_id = p.id
  GROUP BY p.category_id;
END;//
DELIMITER ;

```



118.**Actualizar beneficios caducados**

```sql
ALTER TABLE benefits ADD COLUMN expires_at DATETIME DEFAULT NULL;
ALTER TABLE benefits ADD COLUMN activo BOOLEAN DEFAULT TRUE;

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_actualizar_beneficios
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  UPDATE benefits
  SET activo = FALSE
  WHERE expires_at IS NOT NULL AND expires_at < NOW();
END;//
DELIMITER ;

```



119.**Alertar productos sin evaluación anual**

```sql
CREATE TABLE IF NOT EXISTS alertas_productos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  product_id INT,
  mensaje TEXT,
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_alerta_productos_no_calificados
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
  INSERT INTO alertas_productos(product_id, mensaje, fecha)
  SELECT p.id, 'Este producto no ha sido evaluado en el último año', NOW()
  FROM products p
  WHERE NOT EXISTS (
    SELECT 1 FROM quality_products qp
    WHERE qp.product_id = p.id AND qp.daterating >= NOW() - INTERVAL 1 YEAR
  );
END;//
DELIMITER ;

```



120.**Actualizar precios con índice externo**

```sql
CREATE TABLE IF NOT EXISTS inflacion_indice (
  id INT AUTO_INCREMENT PRIMARY KEY,
  valor DECIMAL(5,2),
  fecha DATETIME
);

DELIMITER //
CREATE EVENT IF NOT EXISTS evt_actualizar_precios_por_indice
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
  DECLARE factor DECIMAL(5,2);
  SELECT valor INTO factor FROM inflacion_indice
  ORDER BY fecha DESC LIMIT 1;

  IF factor IS NOT NULL THEN
    UPDATE companyproduct_prices
    SET price = price * factor;
  END IF;
END;//
DELIMITER ;

```



121.**Ver productos con la empresa que los vende**

```sql
SELECT c.name AS empresa, p.name AS producto, cpp.price
FROM companies c
INNER JOIN companyproducts cp ON c.id = cp.company_id
INNER JOIN products p ON cp.product_id = p.id
INNER JOIN companyproduct_prices cpp ON cp.company_id = cpp.company_id AND cp.product_id = cpp.product_id;

```



122.**Mostrar productos favoritos con su empresa y categoría**

```sql
SELECT f.customer_id, p.name AS producto, cat.description AS categoria, c.name AS empresa
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN categories cat ON p.category_id = cat.id
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies c ON cp.company_id = c.id;
```



123.**Ver empresas aunque no tengan productos**

```sql
SELECT c.id, c.name AS empresa, cp.product_id
FROM companies c
LEFT JOIN companyproducts cp ON c.id = cp.company_id;

```



124.**Ver productos que fueron calificados (o no)**

```sql
SELECT p.id, p.name AS producto, qp.rating
FROM products p
LEFT JOIN quality_products qp ON p.id = qp.product_id;

```



125.**Ver productos con promedio de calificación y empresa**

```sql
SELECT c.name AS empresa, p.name AS producto, ROUND(AVG(qp.rating),2) AS promedio
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
JOIN products p ON cp.product_id = p.id
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY c.name, p.name;

```



126.**Ver clientes y sus calificaciones (si las tienen)**

```sql
SELECT cu.id AS cliente_id, cu.name AS nombre_cliente, ra.rating
FROM customers cu
LEFT JOIN rates ra ON cu.id = ra.customer_id;

```



127.**Ver favoritos con la última calificación del cliente**

```sql
SELECT f.customer_id, p.name AS producto, MAX(qp.daterating) AS ultima_calificacion, MAX(qp.rating) AS calificacion
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
LEFT JOIN quality_products qp ON p.id = qp.product_id AND f.customer_id = qp.customer_id
GROUP BY f.customer_id, p.name;

```



128.**Ver beneficios incluidos en cada plan de membresía**

```sql
SELECT m.name AS membresia, bn.description AS beneficio
FROM membershipbenefits mb
JOIN memberships m ON mb.membership_id = m.id
JOIN benefits bn ON mb.benefit_id = bn.id;

```



129.**Ver clientes con membresía activa y sus beneficios**

```sql
SELECT cu.id AS cliente_id, cu.name AS nombre_cliente, m.name AS membresia, bn.description AS beneficio
FROM customers cu
JOIN membershipperiods mp ON cu.audience_id = mp.membership_id
JOIN memberships m ON mp.membership_id = m.id
JOIN membershipbenefits mb ON m.id = mb.membership_id AND mp.period_id = mb.period_id
JOIN benefits bn ON mb.benefit_id = bn.id;

```



130.**Ver ciudades con cantidad de empresas**

```sql
SELECT cm.name AS ciudad, COUNT(c.id) AS total_empresas
FROM citiesormunicipalities cm
LEFT JOIN companies c ON cm.code = c.city_id
GROUP BY cm.name;

```



131.**Ver encuestas con calificaciones**

```sql
SELECT p.name AS encuesta, r.customer_id, r.rating
FROM polls p
JOIN rates r ON p.id = r.poll_id;

```



132.**Ver productos evaluados con datos del cliente**

```sql
SELECT p.name AS producto, cu.name AS cliente, r.daterating, r.rating
FROM rates r
JOIN customers cu ON r.customer_id = cu.id
JOIN quality_products qp ON r.customer_id = qp.customer_id AND r.poll_id = qp.poll_id AND r.company_id = qp.company_id
JOIN products p ON qp.product_id = p.id;

```



133.**Ver productos con audiencia de la empresa**

```sql
SELECT p.name AS producto, a.description AS audiencia
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies c ON cp.company_id = c.id
JOIN audiences a ON c.audience_id = a.id;

```



134.**Ver clientes con sus productos favoritos**

```sql
SELECT cu.name AS cliente, p.name AS producto_favorito
FROM customers cu
JOIN favorites f ON cu.id = f.customer_id
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id;

```



135.**Ver planes, periodos, precios y beneficios**

```sql
SELECT m.name AS plan, mp.period_id, mp.price, b.description AS beneficio
FROM memberships m
JOIN membershipperiods mp ON m.id = mp.membership_id
JOIN membershipbenefits mb ON m.id = mb.membership_id AND mp.period_id = mb.period_id
JOIN benefits b ON mb.benefit_id = b.id;

```



136.**Ver combinaciones empresa-producto-cliente calificados**

```sql
SELECT c.name AS empresa, p.name AS producto, cu.name AS cliente, qp.rating
FROM quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN companies c ON qp.company_id = c.id
JOIN customers cu ON qp.customer_id = cu.id;

```



137.**Comparar favoritos con productos calificados**

```sql
SELECT f.customer_id, p.name AS producto
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN quality_products qp ON p.id = qp.product_id AND f.customer_id = qp.customer_id;

```



138.**Ver productos ordenados por categoría**

```sql
SELECT cat.description AS categoria, p.name AS producto
FROM categories cat
JOIN products p ON p.category_id = cat.id
ORDER BY cat.description, p.name;

```



139.**Ver beneficios por audiencia, incluso vacíos**

```sql
SELECT a.description AS audiencia, b.description AS beneficio
FROM audiences a
LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
LEFT JOIN benefits b ON ab.benefit_id = b.id;

```



140.**Ver datos cruzados entre calificaciones, encuestas, productos y clientes**

```sql
SELECT cu.name AS cliente, p.name AS producto, po.name AS encuesta, r.rating
FROM rates r
JOIN customers cu ON r.customer_id = cu.id
JOIN polls po ON r.poll_id = po.id
JOIN quality_products qp ON r.customer_id = qp.customer_id AND r.poll_id = qp.poll_id AND r.company_id = qp.company_id
JOIN products p ON qp.product_id = p.id;

```



141.Como analista, quiero una función que calcule el **promedio ponderado de calidad** de un producto basado en sus calificaciones y fecha de evaluación.

```sql
DELIMITER //
CREATE FUNCTION calcular_promedio_ponderado(pid INT) RETURNS DOUBLE
DETERMINISTIC
BEGIN
  DECLARE resultado DOUBLE;
  SELECT SUM(rating * (1 / DATEDIFF(NOW(), daterating))) / SUM(1 / DATEDIFF(NOW(), daterating))
  INTO resultado
  FROM quality_products
  WHERE product_id = pid AND DATEDIFF(NOW(), daterating) > 0;
  RETURN IFNULL(resultado, 0);
END;//
DELIMITER ;

SELECT calcular_promedio_ponderado(1);

```

142.Como auditor, deseo una función que determine si un producto ha sido **calificado recientemente** (últimos 30 días).

```sql
DELIMITER //
CREATE FUNCTION es_calificacion_reciente(fecha DATE) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  RETURN fecha >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
END;//
DELIMITER ;

SELECT es_calificacion_reciente('2025-07-01');

```

143.Como desarrollador, quiero una función que reciba un `product_id` y devuelva el **nombre completo de la empresa** que lo vende.

```sql
DELIMITER //
CREATE FUNCTION obtener_empresa_producto(pid INT) RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
  DECLARE nombre_empresa VARCHAR(100);
  SELECT c.name INTO nombre_empresa
  FROM companies c
  JOIN companyproducts cp ON c.id = cp.company_id
  WHERE cp.product_id = pid
  LIMIT 1;
  RETURN nombre_empresa;
END;//
DELIMITER ;

SELECT obtener_empresa_producto(1);

```

144.Como operador, deseo una función que, dado un `customer_id`, me indique si el cliente tiene una **membresía activa**.

```sql
DELIMITER //
CREATE FUNCTION tiene_membresia_actival(cid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM membershipperiods mp
    JOIN customers cu ON cu.audience_id = mp.membership_id
    WHERE cu.id = cid AND CURDATE() BETWEEN mp.start_date AND mp.end_date
  );
END;//
DELIMITER ;

SELECT tiene_membresia_actival(1);

```

145.Como administrador, quiero una función que valide si una ciudad tiene **más de X empresas registradas**, recibiendo la ciudad y el número como

```sql
DELIMITER //
CREATE FUNCTION ciudad_supera_empresas(cid VARCHAR(10), limite INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  DECLARE total INT;
  SELECT COUNT(*) INTO total FROM companies WHERE city_id = cid;
  RETURN total > limite;
END;//
DELIMITER ;

SELECT ciudad_supera_empresas('CO-ANT', 5);

```

146.Como gerente, deseo una función que, dado un `rate_id`, me devuelva una **descripción textual de la calificación** (por ejemplo, “Muy bueno”, “Regular”).

```sql
DELIMITER //
CREATE FUNCTION descripcion_calificacion(valor INT) RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
  RETURN CASE
    WHEN valor = 5 THEN 'Excelente'
    WHEN valor = 4 THEN 'Bueno'
    WHEN valor = 3 THEN 'Regular'
    WHEN valor = 2 THEN 'Malo'
    WHEN valor = 1 THEN 'Muy malo'
    ELSE 'Inválido'
  END;
END;//
DELIMITER ;

SELECT descripcion_calificacion(4);
```

147.Como técnico, quiero una función que devuelva el **estado de un producto** en función de su evaluación (ej. “Aceptable”, “Crítico”).

```sql
DELIMITER //
CREATE FUNCTION estado_producto(pid INT) RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
  DECLARE prom DOUBLE;
  SELECT AVG(rating) INTO prom FROM quality_products WHERE product_id = pid;
  RETURN CASE
    WHEN prom >= 4.5 THEN 'Óptimo'
    WHEN prom >= 3 THEN 'Aceptable'
    WHEN prom IS NULL THEN 'Sin datos'
    ELSE 'Crítico'
  END;
END;//
DELIMITER ;

SELECT estado_producto(1);

```

148.Como cliente, deseo una función que indique si un producto está **entre mis favoritos**, recibiendo el `product_id` y mi `customer_id`.

```sql
DELIMITER //
CREATE FUNCTION es_favorito(cid INT, pid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    WHERE f.customer_id = cid AND df.product_id = pid
  );
END;//
DELIMITER ;

SELECT es_favorito(1, 2);

```

149.Como gestor de beneficios, quiero una función que determine si un beneficio está **asignado a una audiencia específica**, retornando verdadero o falso.

```sql
DELIMITER //
CREATE FUNCTION beneficio_asignado_audiencia(bid INT, aid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM audiencebenefits
    WHERE benefit_id = bid AND audience_id = aid
  );
END;//
DELIMITER ;

SELECT beneficio_asignado_audiencia(3, 2);

```

150.Como auditor, deseo una función que reciba una fecha y determine si se encuentra dentro de un **rango de membresía activa**.

```sql
DELIMITER //
CREATE FUNCTION fecha_en_membresia(f DATE, cid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM membershipperiods mp
    JOIN customers cu ON cu.audience_id = mp.membership_id
    WHERE cu.id = cid AND f BETWEEN mp.start_date AND mp.end_date
  );
END;//
DELIMITER ;

SELECT fecha_en_membresia(CURDATE(), 1);

```

151.Como desarrollador, quiero una función que calcule el **porcentaje de calificaciones positivas** de un producto respecto al total.

```sql
DELIMITER //
CREATE FUNCTION porcentaje_positivas(pid INT) RETURNS DOUBLE
DETERMINISTIC
BEGIN
  DECLARE total INT DEFAULT 0;
  DECLARE positivas INT DEFAULT 0;
  SELECT COUNT(*) INTO total FROM quality_products WHERE product_id = pid;
  SELECT COUNT(*) INTO positivas FROM quality_products WHERE product_id = pid AND rating >= 4;
  RETURN IF(total = 0, 0, (positivas * 100.0 / total));
END;//
DELIMITER ;

SELECT  porcentaje_positivas(1) AS resultado;
```

152.Como supervisor, deseo una función que calcule la **edad de una calificación**, en días, desde la fecha actual.

```sql
DELIMITER //
CREATE FUNCTION edad_calificacion_en_dias(rid INT) RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE fecha DATE;
  SELECT daterating INTO fecha FROM quality_products WHERE id = rid;
  RETURN DATEDIFF(CURDATE(), fecha);
END;//
DELIMITER ;

SELECT edad_calificacion_en_dias(1);

```

153.Como operador, quiero una función que, dado un `company_id`, devuelva la **cantidad de productos únicos** asociados a esa empresa.

```sql
DELIMITER //
CREATE FUNCTION productos_por_empresa(cid VARCHAR(20)) RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE total INT;
  SELECT COUNT(DISTINCT product_id) INTO total FROM companyproducts WHERE company_id = cid;
  RETURN total;
END;//
DELIMITER ;

SELECT productos_por_empresa('C001');

```

154.Como gerente, deseo una función que retorne el **nivel de actividad** de un cliente (frecuente, esporádico, inactivo), según su número de calificaciones.

```sql
DELIMITER //
CREATE FUNCTION nivel_actividad_cliente(cid INT) RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
  DECLARE total INT;
  SELECT COUNT(*) INTO total FROM quality_products WHERE customer_id = cid;
  RETURN CASE
    WHEN total >= 10 THEN 'Frecuente'
    WHEN total >= 3 THEN 'Esporádico'
    WHEN total >= 1 THEN 'Ocasional'
    ELSE 'Inactivo'
  END;
END;//
DELIMITER ;

SELECT nivel_actividad_cliente(1);

```

155.Como administrador, quiero una función que calcule el **precio promedio ponderado** de un producto, tomando en cuenta su uso en favoritos.

```sql
DELIMITER //
CREATE FUNCTION precio_promedio_ponderado(pid INT) RETURNS DOUBLE
DETERMINISTIC
BEGIN
  DECLARE total DOUBLE;
  SELECT AVG(price) INTO total
  FROM companyproduct_prices cpp
  JOIN companyproducts cp ON cpp.company_id = cp.company_id AND cpp.product_id = cp.product_id
  WHERE cpp.product_id = pid;
  RETURN IFNULL(total, 0);
END;//
DELIMITER ;

SELECT  precio_promedio_ponderado(1);

```

156.Como técnico, deseo una función que me indique si un `benefit_id` está asignado a más de una audiencia o membresía (valor booleano).

```sql
DELIMITER //
CREATE FUNCTION benefit_en_multiples_destinos(bid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  DECLARE total INT;
  SELECT (
    (SELECT COUNT(*) FROM audiencebenefits WHERE benefit_id = bid) +
    (SELECT COUNT(*) FROM membershipbenefits WHERE benefit_id = bid)
  ) INTO total;
  RETURN total > 1;
END;//
DELIMITER ;

SELECT benefit_en_multiples_destinos(1);

```

157.Como cliente, quiero una función que, dada mi ciudad, retorne un **índice de variedad** basado en número de empresas y productos.

```sql
DELIMITER //
CREATE FUNCTION indice_variedad_por_ciudad(cid VARCHAR(10)) RETURNS DOUBLE
DETERMINISTIC
BEGIN
  DECLARE empresas INT;
  DECLARE productos INT;
  SELECT COUNT(*) INTO empresas FROM companies WHERE city_id = cid;
  SELECT COUNT(DISTINCT cp.product_id) INTO productos
  FROM companies c
  JOIN companyproducts cp ON c.id = cp.company_id
  WHERE c.city_id = cid;
  RETURN IF(empresas = 0, 0, productos / empresas);
END;//
DELIMITER ;

SELECT indice_variedad_por_ciudad('CO-ANT');

```

158.Como gestor de calidad, deseo una función que evalúe si un producto debe ser **desactivado** por tener baja calificación histórica.

```sql
DELIMITER //
CREATE FUNCTION producto_debe_desactivarse(pid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  DECLARE promedio DOUBLE;
  SELECT AVG(rating) INTO promedio FROM quality_products WHERE product_id = pid;
  RETURN promedio < 2.5;
END;//
DELIMITER ;

SELECT  producto_debe_desactivarse(1);

```

159.Como desarrollador, quiero una función que calcule el **índice de popularidad** de un producto (combinando favoritos y ratings).

```sql
DELIMITER //
CREATE FUNCTION indice_popularidad_producto(pid INT) RETURNS DOUBLE
DETERMINISTIC
BEGIN
  DECLARE favs INT;
  DECLARE califs INT;
  SELECT COUNT(*) INTO favs FROM details_favorites WHERE product_id = pid;
  SELECT COUNT(*) INTO califs FROM quality_products WHERE product_id = pid;
  RETURN favs * 0.6 + califs * 0.4;
END;//
DELIMITER ;

SELECT indice_popularidad_producto(1);

```

160.Como auditor, deseo una función que genere un código único basado en el nombre del producto y su fecha de creación.

```sql
DELIMITER //
CREATE FUNCTION codigo_unico_producto(pid INT) RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
  DECLARE nombre VARCHAR(100);
  DECLARE creado DATE;
  SELECT name INTO nombre FROM products WHERE id = pid;
  SELECT MIN(daterating) INTO creado FROM quality_products WHERE product_id = pid;
  RETURN CONCAT(LEFT(nombre, 3), '-', DATE_FORMAT(creado, '%Y%m%d'));
END;//
DELIMITER ;

SELECT codigo_unico_producto(1);

```


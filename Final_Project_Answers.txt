USE BikeStore;

-- Production data exploration
SELECT * FROM production.brands;
SELECT * FROM production.categories;
SELECT * FROM production.products;
SELECT * FROM production.stocks;

-- Sales data exploration
SELECT * FROM sales.customers;
SELECT * FROM sales.order_items;
SELECT * FROM sales.orders;
SELECT * FROM sales.staffs;
SELECT * FROM sales.stores;



--1
SELECT TOP(1) product_name, list_price
FROM production.products
ORDER BY list_price DESC;


--2
SELECT COUNT(DISTINCT(customer_id)) AS Total_Customer
FROM sales.orders
WHERE order_status != 3; --> Because customers with order_status 3 don`t have a shipped date which means that they cancelled the order so they are not count as customers.


--3
SELECT COUNT(store_id) AS #Stores 
FROM sales.stores;


--4
SELECT order_id, ROUND(SUM(list_price * quantity * (1-discount)),2) AS Total_Price
FROM sales.order_items
GROUP BY order_id;


--5
SELECT s.store_id, ROUND(SUM(OI.list_price * OI.quantity * (1-OI.discount)), 2) AS Total_Revenue
FROM sales.order_items OI
JOIN sales.orders O ON O.order_id = OI.order_id
JOIN sales.stores s ON O.store_id = s.store_id
GROUP BY s.store_id
ORDER BY s.store_id;


--6
SELECT TOP(1) p.category_id, c.category_name, SUM(s.quantity) Total_Quantity
FROM production.products p
JOIN production.categories c
ON p.category_id = c.category_id
JOIN production.stocks s
ON s.product_id = p.product_id
GROUP BY p.category_id, c.category_name
ORDER BY SUM(list_price) DESC;


--7
SELECT TOP(1) p.category_id, COUNT(order_status) AS #Rejected
FROM production.products p
JOIN sales.order_items oi 
on p.product_id = oi.product_id
JOIN sales.orders o 
on oi.order_id = o.order_id
WHERE o.order_status = 3
GROUP BY p.category_id
ORDER BY #Rejected DESC;


--8
SELECT TOP(1) p.product_name, SUM(s.quantity) AS Total_Quantity
FROM production.products p
JOIN production.stocks s
ON p.product_id = s.product_id
GROUP BY p.product_name
ORDER BY Total_Quantity;

--9
SELECT CONCAT(first_name,' ',last_name) AS Full_Name
FROM sales.customers
WHERE customer_id = 259;


--10
SELECT CONCAT(first_name,' ',last_name) AS Customer_Name, order_id, order_date, order_status
FROM sales.customers c
JOIN sales.orders o
ON o.customer_id = c.customer_id
WHERE c.customer_id = 259;


--11
SELECT CONCAT(c.first_name,' ',c.last_name) AS Customer_Name, o.staff_id, CONCAT(s.first_name,' ',s.last_name) AS Staff_Name, o.store_id
FROM sales.customers c
JOIN sales.orders o
ON o.customer_id = c.customer_id
JOIN sales.staffs s
ON s.staff_id = o.staff_id
WHERE c.customer_id = 259;


--12
SELECT COUNT(*) AS #Staff
FROM sales.staffs;

SELECT *
FROM sales.staffs
WHERE manager_id IS NULL; --> If manager_id is null then he is the lead manager as the lead manager shouldn`t have managers.


--13
SELECT TOP(1) b.brand_name, ROUND(SUM(oi.quantity * oi.list_price * (1-oi.discount)),2) AS Total_Sale
FROM production.brands b
JOIN production.products p
ON p.brand_id = b.brand_id
JOIN sales.order_items oi
ON oi.product_id = p.product_id
GROUP BY b.brand_name
ORDER BY Total_Sale DESC;


--14
SELECT COUNT(*) AS #Categories
FROM production.categories;

SELECT TOP(1) c.category_name, ROUND(SUM(oi.quantity * oi.list_price * (1-oi.discount)),2) AS Total_Sale
FROM production.categories c
JOIN production.products p
ON p.category_id = c.category_id
JOIN sales.order_items oi
ON oi.product_id = p.product_id
GROUP BY c.category_name
ORDER BY Total_Sale;


--15
SELECT DISTINCT stock.store_id, store.store_name, b.brand_name
FROM production.stocks stock
JOIN sales.stores store
ON stock.store_id = store.store_id
JOIN production.products p
ON p.product_id = stock.product_id
JOIN production.brands b
ON p.brand_id = b.brand_id
WHERE
	stock.quantity > 0
	AND
	b.brand_name = (
		SELECT TOP(1) b.brand_name
		FROM production.brands b
		JOIN production.products p
		ON p.brand_id = b.brand_id
		JOIN sales.order_items oi
		ON oi.product_id = p.product_id
		GROUP BY b.brand_name
		ORDER BY ROUND(SUM(oi.quantity * oi.list_price * (1-oi.discount)),2) DESC
	)
;


--16
SELECT TOP(1) s.state, ROUND(SUM(oi.quantity * oi.list_price * (1-oi.discount)),2) AS Total_Sale
FROM sales.stores s
JOIN sales.orders o
ON o.store_id = s.store_id
JOIN sales.order_items oi
ON oi.order_id = o.order_id
GROUP BY s.state
ORDER BY Total_Sale DESC;


--17
SELECT product_id, ((1-discount) * list_price) AS Discounted_Price, order_id
FROM sales.order_items
WHERE product_id = 259;


--18
SELECT p.product_name, SUM(oi.quantity) AS Total_Quantity, c.category_name, p.model_year, b.brand_name
FROM production.products p
JOIN sales.order_items oi
ON p.product_id = oi.product_id
JOIN production.categories c
ON p.category_id = c.category_id
JOIN production.brands b
ON p.brand_id = b.brand_id
WHERE p.product_id = 44
GROUP BY p.product_name, c.category_name, p.model_year, b.brand_name;


--19
SELECT zip_code
FROM sales.stores
WHERE state = 'CA';


--20
SELECT COUNT(state) AS #States
FROM sales.stores;


--21
SELECT COUNT(oi.product_id) AS #Bikes
FROM sales.order_items oi
JOIN sales.orders o
ON  o.order_id = oi.order_id
JOIN production.products p
ON p.product_id = oi.product_id
JOIN production.categories c
ON c.category_id = p.category_id
WHERE c.category_name = 'Children Bicycles'
  AND DATEDIFF(MONTH, o.order_date, GETDATE()) <= 8
;


--22
SELECT shipped_date
FROM sales.orders
WHERE customer_id = 523;


--23
SELECT COUNT(order_id) AS PendingOrders
FROM sales.orders
WHERE order_status = 3;


--24
SELECT p.product_name, c.category_name, b.brand_name
FROM production.products p
JOIN production.categories c
ON p.category_id = c.category_id
JOIN production.brands b
ON b.brand_id = p.brand_id
WHERE p.product_name = 'Electra white water 3i - 2018';

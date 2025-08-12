# Online_Book_Store_Analysis_SQL**
Schema**
<img width="939" height="616" alt="image" src="https://github.com/user-attachments/assets/9a0b79ea-2155-4dfc-a614-f0ad4aeb28eb" />


#** QUERIES**
create database book_store;
use book_store;

DROP TABLE IF EXISTS Books;

CREATE TABLE Books (
    Book_ID int PRIMARY KEY,
    Title VARCHAR(100),
    Author VARCHAR(100),
    Genre VARCHAR(50),
    Published_Year INT,
    Price NUMERIC(10, 2),
    Stock INT
);

select * from Books;

DROP TABLE IF EXISTS customers;

CREATE TABLE Customers (
    Customer_ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100),
    Phone VARCHAR(15),
    City VARCHAR(50),
    Country VARCHAR(150)
);
SELECT * FROM Customers;

DROP TABLE IF EXISTS orders;
CREATE TABLE Orders (
    Order_ID INT PRIMARY KEY,
    Customer_ID INT ,
    Book_ID INT ,
    Order_Date DATE,
    Quantity INT,
    Total_Amount NUMERIC(10, 2),
    foreign key (Customer_ID) references Customers(Customer_ID),
	foreign key (Book_ID) references Books(Book_ID)
);
SELECT * FROM Orders;
desc orders;


#Basic Queries
#1) Retrieve all books in the "Fiction" genre

select * from Books where genre = "Fiction";

#2) Find books published after the year 1950

select * from books where Published_year > 1950;

#3) List all customers from the Canada

select * from Customers where country like '%Canada%';
select * from Customers where country = 'Canada';

#4) Show orders placed in November 2023

select * from orders where order_date between '2023-11-01' AND '2023-11-30';

#5) Retrieve the total stock of books available

select sum(Stock) as totalstock from books;

#6) Find the details of the most expensive book

select * from books order by price desc limit 1;

#7) Show all customers who ordered more than 1 quantity of a book

select c.name,order_id,o.quantity from customers c 
inner join orders o on c.customer_id = o.customer_id
where o.quantity > 1;

select * from orders where quantity > 1;

#8) Retrieve all orders where the total amount exceeds $20

select * from orders where Total_Amount > 20;

#9) List all genres available in the Books table

select distinct genre from books;

#10) Find the book with the lowest stock

select * from books  order by price asc limit 1 ;

#11) Calculate the total revenue generated from all orders

select sum(total_amount) as revenue from orders;

#-----------------ADVANCED QUERIES------------------

#1) Retrieve the total number of books sold for each genre

select b.genre,sum(o.quantity) as total_book_sold 
from books b inner join orders o 
on b.book_id = o.book_id
group by b.genre;

#2) Find the average price of books in the "Fantasy" genre.

select genre,avg(price) from books where genre = "Fantasy";

#3) List customers who have placed at least 2 orders.

select o.customer_id,c.name,count(o.order_id) as order_placed 
from customers c inner join orders o
on o.customer_id = c.customer_id
group by name,o.customer_id  
having count(o.order_id) >= 2;

#4) Find the most frequently ordered book.

select b.book_id,b.title,count(o.order_id) as order_count
from books b inner join orders o
on b.book_id = o.book_id
group by b.book_id,b.title
order by order_count desc
limit 1;


#5) Show the top 3 most expensive books of 'Fantasy' Genre.

select title,genre,price from books where genre = 'Fantasy' 
order by price desc limit 3;

#6) Retrieve the total quantity of books sold by each author.

select b.author,b.title,sum(o.quantity) as total_quantity 
from books b left join orders o 
on b.book_id = o.book_id
group  by b.author,b.title ;

#7) List the cities where customers who spent over $30 are located.

select c.city,o.total_amount 
from customers c inner join orders o
on c.customer_id = o.customer_id
where o.total_amount > 30;

#8) Find the customer who spent the most on orders.

select c.name,o.Total_amount 
from customers c inner join orders o
on c.customer_id = o.customer_id
order by o.Total_amount desc limit 1;

#9) Calculate the stock remaining after fulfilling all orders.

SELECT b.book_id, b.title, b.stock, COALESCE(SUM(o.quantity),0) AS Order_quantity,  
	b.stock- COALESCE(SUM(o.quantity),0) AS Remaining_Quantity
FROM books b
LEFT JOIN orders o ON b.book_id=o.book_id
GROUP BY b.book_id ORDER BY b.book_id;

#10)Customers who ordered more than 20 books total.

WITH customer_totals AS (
    SELECT 
        Customer_ID,
        SUM(Quantity) AS Total_Quantity
    FROM orders GROUP BY Customer_ID)
SELECT 
    c.Name,
    ct.Total_Quantity
FROM customer_totals ct
JOIN customers c 
ON ct.Customer_ID = c.Customer_ID
WHERE 
    ct.Total_Quantity > 10;


#11)Write an SQL query to find and list all books that have never been ordered.

with ordered_books as(select distinct book_id 
from books ) 
select book_id,title from books
where book_id not in (select  book_id from ordered_books);

#12)Write an SQL query to return the latest order for every customer.

select * from (
select *,
row_number() over (partition by customer_id order by order_date )as rn 
from orders
) as ranked where rn = 1;

#13)List all orders with status based on quantity ordered.

select o.order_id , c.name , o.quantity ,
case when o.quantity >= 8 then 'BULK ORDER'
    when o.quantity >= 5 then 'MEDIUM ORDER'
    ElSE 'SMALL ORDER'
    END AS order_status
from customers c join orders o
on c.customer_id = o.customer_id ;



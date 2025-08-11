# Customer Support Ticket Analytics Project

## 📌 Project Overview
This project analyzes customer support tickets to gain insights into ticket trends, agent performance, resolution efficiency, and patterns of tags and categories.  
It utilizes **SQL** for data modeling, cleaning, and transformation, and **Power BI** for creating interactive dashboards.  
The goal is to demonstrate **end-to-end data analytics skills** — from database design to dashboard storytelling.

---

## 📂 Dataset
- **Source:** https://www.kaggle.com/datasets/tobiasbueck/multilingual-customer-support-tickets?select=aa_dataset-tickets-multi-lang-5-2-50-version.csv
- **Format:** CSV with multiple columns (ticket details, dates, agents, tags, etc.)
- **Tables Created in SQL:**
  - `tickets` (Fact Table)  
  - `dim_date` (Date Dimension)  
  - `languages` (Language Dimension)  
  - `queues` (Queue Dimension)
  - `priorities` (Priority Dimension)
  - `ticket_types` (Ticket Type Dimension)
  - `tags` (Ticket Tags Dimension) 
  - `raw_data_tickets` (Bridge Table for many-to-many tags)
 
---

## 🛠 Tools Used
- **SQL** (MySQL) – Data modeling, transformation, and relationships
- **Power BI** – Dashboard creation and visual analysis
- **Excel** – Data exploration and initial cleanup

---



<details>
<summary> 
  
## 🗄 SQL Implementation
                   
<details>
<summary>1️⃣ Creating a Database</summary>

```sql
CREATE DATABASE Customer_Support;
USE customer_support;
```

 ### 2️⃣ Create Dimension Tables 
```sql

CREATE TABLE ticket_types (
    type_id INT PRIMARY KEY AUTO_INCREMENT,
    type_name VARCHAR(50) UNIQUE
);

CREATE TABLE queues (
    queue_id INT PRIMARY KEY AUTO_INCREMENT,
    queue_name VARCHAR(100) UNIQUE
);

CREATE TABLE priorities (
    priority_id INT PRIMARY KEY AUTO_INCREMENT,
    priority_name VARCHAR(20) UNIQUE
);

CREATE TABLE languages (
    language_id INT PRIMARY KEY AUTO_INCREMENT,
    language_code VARCHAR(10) UNIQUE
);

CREATE TABLE tags (
    tag_id INT PRIMARY KEY AUTO_INCREMENT,
    tag_name VARCHAR(100) UNIQUE
);
``` 
### 3️⃣ Create Fact Tables
```sql 
CREATE TABLE tickets (
created_at date,
    ticket_id INT PRIMARY KEY AUTO_INCREMENT,
    subject TEXT,
    body TEXT,
    answer TEXT,
    type_id INT,
    queue_id INT,
    priority_id INT,
    language_id INT,
    version INT,
    tag_1 VARCHAR(100),
    tag_2 VARCHAR(100),
    tag_3 VARCHAR(100),
    tag_4 VARCHAR(100),
    tag_5 VARCHAR(100),
    tag_6 VARCHAR(100),
    tag_7 VARCHAR(100),
    tag_8 VARCHAR(100) ) ;
```
### 4️⃣ Insert Data into Dimension tables
```sql

INSERT INTO ticket_types (type_id,type_name) VALUES (1,'Change');
INSERT INTO ticket_types (type_id,type_name) VALUES (2,'Incident');
INSERT INTO ticket_types (type_id,type_name) VALUES (3,'Problem');
INSERT INTO ticket_types (type_id,type_name) VALUES (4,'Request');

insert into queues (queue_name) 
values ('Technical Support'),
('Returns and Exchanges'),
('Billing and Payments'),
('Sales and Pre-Sales'),
('Service Outages and Maintenance'),
('Product Support'),
('IT Support'),
('Customer Service'),
('Human Resources'),
('General Inquiry')

INSERT INTO priorities (priority_name) VALUES ('high'),('medium'), ('low');

INSERT INTO languages (language_code) VALUES ('de'),('en');
```
### 5️⃣ Creating a temporary bridge table
```sql

CREATE TABLE raw_data_tickets (
created_at date,
    subject TEXT,
    body TEXT,
    answer TEXT,
    type_name VARCHAR(50),
    queue VARCHAR(100),
    priority VARCHAR(20),
    language VARCHAR(10),
    version INT,
    tag_1 VARCHAR(100),
    tag_2 VARCHAR(100),
    tag_3 VARCHAR(100),
    tag_4 VARCHAR(100),
    tag_5 VARCHAR(100),
    tag_6 VARCHAR(100),
    tag_7 VARCHAR(100),
    tag_8 VARCHAR(100)
);
```
### 6️⃣ Inserting Data into tickets (fact) table
```sql 

INSERT INTO tickets (
   created_at, subject, body, answer, type_id, queue_id, priority_id, language_id,
    version, tag_1, tag_2, tag_3, tag_4, tag_5, tag_6, tag_7, tag_8
)
SELECT
d.created_at,
    s.subject,
    s.body,
    s.answer,
    t.type_id,
    q.queue_id,
    p.priority_id,
    l.language_id,
    s.version,
    s.tag_1, s.tag_2, s.tag_3, s.tag_4, s.tag_5, s.tag_6, s.tag_7, s.tag_8
FROM raw_data_tickets s
JOIN dim_date d  ON d.created_at=s.created_at
JOIN ticket_types t ON s.type_name = t.type_name
JOIN queues q ON s.queue = q.queue_name
JOIN priorities p ON s.priority = p.priority_name
JOIN languages l ON s.language = l.language_code ;
```
### 7️⃣ Dropping the temporary table
```sql

drop table raw_data_tickets
```
### 8️⃣ Creating a date dimension table
```sql

CREATE TABLE dim_date (
    date_id INT PRIMARY KEY AUTO_INCREMENT,
    created_at DATE
);
```

### 9️⃣ Adding date values from 2024-01-01 and 2024-12-31 ( Inserting date data into the "dim_date" table for the year 2024, as the original dataset does not have date data)
 data)

```sql
INSERT INTO dim_date (created_at)
WITH RECURSIVE date_series AS (
  SELECT DATE('2024-01-01') AS created_at
  UNION ALL
  SELECT DATE_ADD(created_at, INTERVAL 1 DAY)
  FROM date_series
  WHERE created_at < '2024-12-31'
)
SELECT created_at FROM date_series;

select * from dim_date;
```
### 1️⃣0️⃣ Adding foreign Key Constraints to establish relationships in the dataset.

```sql

ALTER TABLE tickets
  ADD CONSTRAINT fk_type
    FOREIGN KEY (type_id) REFERENCES ticket_types(type_id),
  ADD CONSTRAINT fk_queue
    FOREIGN KEY (queue_id) REFERENCES queues(queue_id),
  ADD CONSTRAINT fk_priority
    FOREIGN KEY (priority_id) REFERENCES priorities(priority_id),
  ADD CONSTRAINT fk_language
    FOREIGN KEY (language_id) REFERENCES languages(language_id);



ALTER TABLE dim_date ADD UNIQUE (created_at);

ALTER TABLE tickets
ADD CONSTRAINT fk_created_date
FOREIGN KEY (created_at) REFERENCES dim_date(created_at);

SELECT DISTINCT tag_name
FROM (
    SELECT tag_1 AS tag_name FROM tickets
    UNION
    SELECT tag_2 FROM tickets
    UNION
    SELECT tag_3 FROM tickets
    UNION
    SELECT tag_4 FROM tickets
    UNION
    SELECT tag_5 FROM tickets
    UNION
    SELECT tag_6 FROM tickets
    UNION
    SELECT tag_7 FROM tickets
    UNION
    SELECT tag_8 FROM tickets
) AS all_tags
WHERE tag_name IS NOT NULL AND tag_name <> '';

```

# **TABLES**

```sql
CREATE TABLE countries(
  isocode VARCHAR(6) PRIMARY KEY,
  name VARCHAR(50) UNIQUE,
  alfatwocode VARCHAR(2) UNIQUE,
  alfathreecode VARCHAR(4) UNIQUE
);

CREATE TABLE subdivisioncategories(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(40)
);

CREATE TABLE typeidentifications(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE,
  suffix VARCHAR(5)
);

CREATE TABLE audiences(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE categories(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE unitofmeasure(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE categories_polls(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80) UNIQUE
);

CREATE TABLE memberships(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) UNIQUE,
  description TEXT
);

CREATE TABLE periods(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) UNIQUE
);

CREATE TABLE benefits(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(80),
  detail TEXT
);

CREATE TABLE stateregions(
  code VARCHAR(6) PRIMARY KEY,
  name VARCHAR(50),
  country_id VARCHAR(6),
  codestate16 VARCHAR(10),
  subdivision_id INT,
  FOREIGN KEY (country_id) REFERENCES countries(isocode),
  FOREIGN KEY (subdivision_id) REFERENCES subdivisioncategories(id)
);

CREATE TABLE citiesormunicipalities(
  code VARCHAR(6) PRIMARY KEY,
  name VARCHAR(50),
  statereg_id VARCHAR(6),
  FOREIGN KEY (statereg_id) REFERENCES stateregions(code)
);

CREATE TABLE companies(
  id VARCHAR(20) PRIMARY KEY,
  name VARCHAR(80),
  type_id INT,
  category_id INT,
  city_id VARCHAR(6),
  audience_id INT,
  FOREIGN KEY (type_id) REFERENCES typeidentifications(id),
  FOREIGN KEY (category_id) REFERENCES categories(id),
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
  FOREIGN KEY (audience_id) REFERENCES audiences(id)
);

CREATE TABLE company_phones(
  company_id VARCHAR(20),
  phone VARCHAR(15),
  PRIMARY KEY (company_id, phone),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE company_emails(
  company_id VARCHAR(20),
  email VARCHAR(80),
  PRIMARY KEY (company_id, email),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE customers(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(60),
  city_id VARCHAR(6),
  audience_id INT,
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
  FOREIGN KEY (audience_id) REFERENCES audiences(id)
);

CREATE TABLE customer_phones(
  customer_id INT,
  phone VARCHAR(20),
  PRIMARY KEY (customer_id, phone),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE customer_emails(
  customer_id INT,
  email VARCHAR(100),
  PRIMARY KEY (customer_id, email),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE addresses(
  id INT PRIMARY KEY AUTO_INCREMENT,
  street VARCHAR(80),
  city_id VARCHAR(6),
  postal_code VARCHAR(10),
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code)
);

CREATE TABLE customer_addresses(
  customer_id INT,
  address_id INT,
  PRIMARY KEY (customer_id, address_id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (address_id) REFERENCES addresses(id)
);

CREATE TABLE products(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80),
  detail TEXT,
  price DOUBLE,
  category_id INT,
  image VARCHAR(500),
  stock INT,
  FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE companyproducts(
  company_id VARCHAR(20),
  product_id INT,
  unitmeasure_id INT,
  PRIMARY KEY (company_id, product_id),
  FOREIGN KEY (company_id) REFERENCES companies(id),
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (unitmeasure_id) REFERENCES unitofmeasure(id)
);

CREATE TABLE companyproduct_prices(
  company_id VARCHAR(20),
  product_id INT,
  price DOUBLE,
  start_date DATETIME,
  end_date DATETIME,
  PRIMARY KEY (company_id, product_id, start_date),
  FOREIGN KEY (company_id, product_id) REFERENCES companyproducts(company_id, product_id)
);

CREATE TABLE favorites(
  id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT,
  company_id VARCHAR(20),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE details_favorites(
  favorite_id INT,
  product_id INT,
  category_id INT,
  name VARCHAR(80),
  detail TEXT,
  PRIMARY KEY (favorite_id, product_id),
  FOREIGN KEY (favorite_id) REFERENCES favorites(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE polls(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80) UNIQUE,
  description TEXT,
  isactive BOOLEAN,
  categorypoll_id INT,
  FOREIGN KEY (categorypoll_id) REFERENCES categories_polls(id)
);

CREATE TABLE rates(
  customer_id INT,
  company_id VARCHAR(20),
  poll_id INT,
  daterating DATETIME,
  rating DOUBLE,
  PRIMARY KEY (customer_id, company_id, poll_id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (company_id) REFERENCES companies(id),
  FOREIGN KEY (poll_id) REFERENCES polls(id)
);

CREATE TABLE quality_products(
  product_id INT,
  customer_id INT,
  poll_id INT,
  company_id VARCHAR(20),
  daterating DATETIME,
  rating DOUBLE,
  PRIMARY KEY (product_id, customer_id, poll_id, company_id),
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (poll_id) REFERENCES polls(id),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE membershipperiods(
  membership_id INT,
  period_id INT,
  price DOUBLE,
  PRIMARY KEY (membership_id, period_id),
  FOREIGN KEY (membership_id) REFERENCES memberships(id),
  FOREIGN KEY (period_id) REFERENCES periods(id)
);

CREATE TABLE membershipbenefits(
  membership_id INT,
  period_id INT,
  audience_id INT,
  benefit_id INT,
  PRIMARY KEY (membership_id, period_id, audience_id, benefit_id),
  FOREIGN KEY (membership_id) REFERENCES memberships(id),
  FOREIGN KEY (period_id) REFERENCES periods(id),
  FOREIGN KEY (audience_id) REFERENCES audiences(id),
  FOREIGN KEY (benefit_id) REFERENCES benefits(id)
);

CREATE TABLE audiencebenefits(
  audience_id INT,
  benefit_id INT,
  PRIMARY KEY (audience_id, benefit_id),
  FOREIGN KEY (audience_id) REFERENCES audiences(id),
  FOREIGN KEY (benefit_id) REFERENCES benefits(id)
);


```


---
layout: post
category : mysql
tagline: "Importing a CSV file to a MySQL database"
tags : [ csv, mariadb, mysql, database]
---

### Install a database server.
We'll go for MySQL but MariaDB is an alternative with a lot of improvements.

### Stage your CSV.
I'm doing this on a remote server so I need to transfer my CSV from local to remote. My CSV contains information on ATM machines throughout the State, including
geo location and dates when they were last serviced. 

`
scp det_atm.csv root@104.131.99.220:~/
`

Now let's take a look at the character set that the CSV is in.

`
file -bi det_atm.csv
`

The output in my case was:

`
text/plain; charset=us-ascii
`

That'll be important in a moment. 

### Get your database ready.

I do not have an existing database so I will sketch one out. I want to describe my column-data-types in such a way that they can hold the information from my CSV efficiently and accurately.

#### My original spreadsheet looks a bit like this

    +------------+----------+------/   /----- -+---------+----------------+
    | Customer Id| Services | Route/   / zip   | logitude| latitude       |
    +------------+----------+------/   /-------+---------+----------------+
    | Bank1      | clean    | Det  /   / 48212 |-83.21934| 42.49959       |
    | Big Bank   | survey   | Oh   /   / 98202 |-82.74849| 42.49959       |
    | Bank1      | evaluate | Det  /   / 48940 |-82.85849| 42.49959       |
    | Bank2      | clean    | Det  /   / 73822 |-83.20132| 42.49959       |
    | Small Bank | clean    | Det  /   / 67382 |-83.20132| 42.49959       |
    +------------+----------+------/   /-------+---------+----------------+


#### My sketch like this

    customer_name VARCHAR(255)
    services VARCHAR(255)
    route CHAR(20)
    job_id CHAR(10)
    atm_id CHAR(8)
    description VARCHAR(255)
    address VARCHAR(255)
    city CHAR(20)
    state CHAR(2)
    zip CHAR(5)
    latitude FLOAT
    longitude FLOAT
    placement_information VARCHAR(255)
    atm_manufacturer VARCHAR(255)
    atm_model VARCHAR(255)
    last_service DATE
    service_after DATE
    completion_date DATE

Alternatively if I have an existing database called 'autotellermachines' with a table called 'atms' I can use;

`
SHOW COLUMNS FROM atms IN autotellermachines;
`

### Take note of possible issues.
* I need to take note of any column-data-types that need to be input in a specific way. For example dates in MYSQL must be in the format: YYYY-MM-DD and my CSV dates are in the format MM-DD-YY. 

* If I am importing into an existing database, the columns from my CSV and the columns from the table might not line up. i.e.

    * Columns from the CSV might need to be placed *after* existing columns on the DB table.
    * Columns from the CSV might need to be placed *inbetween* exisiting columns on the DB table.


* Also because my original csv does not include a unique identifier for each ATM I will add a column to my new table to uniquely identify each item. This is what the `db_atm_id SMALLINT(5) NOT NULL AUTO_INCREMENT` and `PRIMARY KEY (db_atm_id)` statements are for.

### Create your new table with the desired columns:

First create a new database, we define the character encoding to conform to our CSV:

    CREATE DATABASE autotellermachines
        DEFAULT CHARACTER SET ascii
        DEFAULT COLLATE ascii_general_ci;

Assuming we have successfully created a database we can add a table with the desired columns. You can ignore this step if you already have a table with columns.

    CREATE TABLE atms (
        db_atm_id SMALLINT(5) NOT NULL AUTO_INCREMENT,
        customer_name VARCHAR(255),
        services VARCHAR(255),
        route CHAR(20),
        job_id CHAR(10),
        atm_id CHAR(8),
        description VARCHAR(255),
        address VARCHAR(255),
        city CHAR(20),
        state CHAR(2),
        zip CHAR(5),
        latitude FLOAT,
        longitude FLOAT,
        placement_information VARCHAR(255),
        atm_manufacturer VARCHAR(255),
        atm_model VARCHAR(255),
        last_service DATE,
        service_after DATE,
        completion_date DATE,
        PRIMARY KEY (db_atm_id)
    );

### Let's write the LOAD DATA infile command that we will use

We are using a LOCAL command, so when we initialize our MySQL session be sure to allow LOCAL access:

    mysql --local-infile autotellermachines

Here is the command, with comments.

    LOAD DATA LOCAL INFILE '~/det_atm.csv'
    INTO TABLE atms
        //What is the seperating character? Hint: This is a CSV.
        COLUMNS TERMINATED BY ','
        //What do columns that contain the same character as the COLUMNS TERMINATED character do? 
        OPTIONALLY ENCLOSED BY '"' 
        ESCAPED BY '"' 
        //What defines new rows in our CSV?
        LINES TERMINATED BY '\n' 
        // Do we want all the lines from the CSV? The IGNORE option can only skip lines at the start of the file.
        IGNORE 1 LINES
        // If our CSV columns imported in a linear way from left to right, without gaps,  what is the order of the columns that they map onto. We can use @dummy to dump CSV columns i.e. map them to nowhere...
        // NOTE we start at the second column of our atms table, the db_atm_id is being auto generated incrementally
        (customer_name, services, route, job_id, atm_id, description, address, city, state, zip, latitude, longitude, placement_information, atm_manufacturer, atm_model, @last_service_var, @service_after_var, @completion_date_var)
    SET 
        last_service = STR_TO_DATE(@last_service_var, '%m/%d/%Y')
        service_after = STR_TO_DATE(@service_after_var, '%m/%d/%Y') 
        completion_date = STR_TO_DATE(@completion_date_var, '%m/%d/%Y');




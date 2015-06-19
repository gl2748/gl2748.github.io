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

```
scp det_atm.csv root@104.131.99.220:~/
```

Now let's take a look at the character set that the CSV is in.

```
file -bi det_atm.csv
```

The output in my case was:

```
text/plain; charset=us-ascii
```

That'll be important in a moment. 

### Get your database ready.

I do not have an existing database so I will sketch one out. I want to describe my column-data-types in such a way that they can hold the information from my CSV efficiently and accurately. Alternatively if I have an existing database called 'autotellermachines' with a table called 'atms' I can use;

```
SHOW COLUMNS FROM atms IN autotellermachines;
```

### Take note of possible issues.
I need to take note of any column-data-types that need to be input in a specific way. For example dates in MYSQL must be in the format: YYYY-MM-DD and my CSV dates are in the format MM-DD-YY.

Another issue with using a pre-existing table is that the columns from my CSV and the columns from the table might not line up. 

Also because my original csv does not include a unique identifier for each ATM i will add a column to my new 



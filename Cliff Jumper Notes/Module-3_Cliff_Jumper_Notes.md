# Module 2: Cliff Jumper Notes

*One continuous lesson stitched from the Module 2 cliff notes of Managing,
Querying, and Preserving Data (CMPINF-2110): why nearly every app ends at a
database, what a relational database is and how keys tie its tables together,
the six jobs a relational database does for a data scientist, the four
components of a database system and what each one does, the line between a
personal system like Access and an enterprise-class product, the three ways a
database comes to be designed, and the fifty-year history that put the
relational model on top.*

---

## 1. Every app ends at a database

Almost every website and mobile app has some sort of database, to store
information and make it useful later. The professor and the textbook, Kroenke
and Auer's Database Processing (16th edition, Chapter 1, the source of every
figure and chapter number below), both open with the same point, in Figure
1-2: whether the client is a PC with a Web browser, a smartphone with an app,
or a smart speaker with a virtual assistant, the request travels over the
Internet or the cell phone data network to a Web server or app data server,
and the server ends at a database.

The pattern has a name. In **client-server architecture**, client applications
on user devices obtain services from server computers, and the servers hold
the databases. People are users, their computers and phones are devices. The
user never touches the database, the Web server does.

The chapter's timeline is quiz bait, so here it is in one breath: Apple II
1977, IBM PC 1981, Ethernet a national standard in 1983, ARPANET 1969, the Web
easily accessible in 1993, Amazon.com online since 1995, iPhone 2007, first
Android phone 2008. The point of the list (added): every device generation
added a new client type, and each one still ends at a database.

**Point-of-sale (POS) systems** record every purchase in a database, monitor
inventory, and track card-holder buying for marketing.

---

## 2. What a database is

The purpose of a database is to help people keep track of things. The
professor's phrasing is "track things of interest." Databases are created and
controlled by software called **database management systems (DBMSs)**. All
DBMSs and databases sort into two categories: relational and non-relational.
The first databases were non-relational. The relational database quickly
became the most common type once introduced and remains so.

Relational: multiple tables, all related to each other, with values that link
data between many objects. The course uses MySQL. Non-relational: no such
relationships. Example: MongoDB, which stores mostly JSON. **JSON** stands for
JavaScript Object Notation.

**Big Data** means enormous datasets. The textbook says Big Data comes from
search tools, Web 2.0 social networks, and scientific data collection and is
often stored in new non-relational databases, which is why the non-relational
database is making a resurgence. The professor says big data is typically
stored in flat files on a file system or in simple storage on a cloud
solution.

A relational database stores data in **tables**. A table has **rows** and
**columns** like a spreadsheet. Rows are also called **records**, and the
professor adds a third name, **tuple**. Columns are also called **fields**. A
database usually has multiple tables, each about a different type of thing.
Each row holds data about one occurrence, or **instance**, of the thing of
interest. Each column stores one characteristic common to all rows. Tables
differ from spreadsheets in two ways: tables have column names instead of
letters, and rows are not necessarily numbered. Instances go in rows and
characteristics in columns in every database in the book and, the authors say,
"99.999999 percent" of databases worldwide.

The textbook's naming convention: table names in all capitals, column names
start with a capital, and compound column names capitalize each word
(**CamelCase**, added name). The reading's examples are STUDENT, CLASS, GRADE,
COURSE_INFO and Term, Section, ClassNumber, StudentName. These are conventions
only, DBMS products do not require them. The payoff is that an all-caps name
is always a table and an initial-cap name is always a column.

The professor's rule of thumb: in relational databases the key is to store the
least amount of data possible, so you use multiple tables.

Two definitions close the section. **Data**: recorded facts and figures.
**Information** has three definitions: knowledge derived from data, data
presented in a meaningful context, or data processed by summing, ordering,
averaging, grouping, comparing, or other similar operations. Databases record
data in a way that lets information be produced from it. The STUDENT, CLASS,
and GRADE data could yield each student's GPA.

---

## 3. Data plus relationships: the keys

A relational database is not complete unless it also shows the relationships
among the rows. A table of values with no link to what they describe is
useless. So a database contains both data and the relationships among the
data. The textbook builds that in four figures.

Figures 1-3 to 1-6 build three tables: STUDENT (StudentNumber, LastName,
FirstName, EmailAddress), CLASS (ClassNumber, ClassName, Term, Section), and
GRADE. With Grade as its only column (Figure 1-4), GRADE cannot say whose
grades they are. With StudentNumber, ClassNumber, and Grade (Figure 1-5), each
grade is linked back to a student and a class.

**Primary key**: uniquely identifies each row of a table. The values of
primary keys are used to create the relationships between tables. **Surrogate
key**: a primary key whose values are automatically generated and assigned
inside the database itself. In MySQL that is the auto-increment option on an
integer column, so the user never worries about assigning the value.
**Composite key**: a primary key formed when more than one column must be
combined to identify a row. **Foreign key**: a column that provides the link
between two tables. Adding a foreign key creates a relationship between the
two tables.

In the textbook figures, GRADE's primary key is the composite (StudentNumber,
ClassNumber), and each of those two columns is also a foreign key back to
STUDENT or CLASS.

Example: TEAM (TeamNumber, TeamName, City). PLAYER (PlayerNumber, LastName,
FirstName). CONTRACT (TeamNumber, PlayerNumber, Salary). A CONTRACT table
holding only salaries is a column of money that belongs to nobody. Add
TeamNumber and PlayerNumber as foreign keys and every salary points at one team
and one player. One contract per team and player pair, so the composite key is
(TeamNumber, PlayerNumber). One team to many contracts, one player to many
contracts.

Figure 1-6 is the same design in the Microsoft Access 2019 relationships
view. Its lesson: GRADE's two foreign keys relate it to STUDENT and to CLASS,
and each relationship is one-to-many, so one student may be linked to many
grades.

The professor's MySQL Workbench demo makes the surrogate key concrete.
STUDENT's primary key is an auto-incrementing number, so it is a surrogate
key. Inserting a new student without supplying the number, the row received
the next number automatically. CLASS has a plain primary key, so inserting a
row without the class number failed with an error that the class number column
has no default value. Nothing assigns that key, so the user must supply it.

---

## 4. The professor's data-science angle

The professor's data-science lecture names the jobs a relational database does
for a data scientist. Several run through SQL, the query language every
relational DBMS understands, defined in Section 6. Six roles: data storage,
data retrieval, data cleaning and pre-processing, integration with analytical
tools, data governance and security, and exploratory data analysis (EDA).

**Storage.** Relational databases store structured data in organized tables
with predefined schemas, which makes it easy to maintain data integrity,
consistency, and accessibility for analysis. A **schema** (added definition)
is the predefined set of tables, columns, and types a relational database
enforces.

**Retrieval.** Using SQL, data scientists extract relevant data by writing
queries that filter, sort, join, and aggregate information from one or more
tables. A **join** (added definition) combines rows from two or more tables on
a matching column. An **aggregate** (added definition) summarizes many rows
into one value, such as a count, sum, or average.

**Cleaning.** Relational databases support cleaning operations in SQL before
the data reaches analytical tools. The operations named: removing duplicates,
handling missing values, transforming columns. Do it in SQL first and Python
or R receives less mess.

**Integration.** Relational databases connect with Python, R, Excel, and BI
platforms. BI stands for business intelligence. Python libraries named:
Pandas, SQLAlchemy, psycopg2. R connects directly as well.

**Governance and security.** **Data governance**: the framework that manages
data availability, usability, integration, and security across an
organization, ensuring data is consistent, trustworthy, and used responsibly.
Governance ensures all modifications can be audited for compliance and
accountability. Security features named: access controls, transaction logs,
schemas, audit logs.

**EDA.** **Exploratory data analysis (EDA)**: initial exploration of data,
such as understanding distributions, frequencies, and trends, done before deep
statistical analysis. EDA can run as SQL queries directly inside the
relational database.

---

## 5. Applications, by size

Before the components, the range of sizes a database comes in (added). Figure
1-7 lists example database applications with their users and sizes.

```
Application            Users                 Size
Sales contact manager  1                     2,000 rows
Patient appointment    15 to 50              100,000 rows
CRM                    500                   10 million rows
ERP                    5,000                 10 million+ rows
E-commerce site        Possibly millions     1 billion+ rows
Digital dashboard      500                   100,000 rows
Data mining            25                    100,000 to millions+ rows
```

A **customer relationship management (CRM) system** manages customer contacts
from initial solicitation through acceptance, purchase, continuing purchase,
support, and so forth. **Enterprise resource planning (ERP)** is an
information system touching every department of a manufacturing company. SAP
is the leading ERP vendor.

The multiuser rows carry a risk: one user's work interferes with another's.
**Concurrency control** mechanisms prevent this (Chapter 9). For instance, two
ticket agents sell the same seat to two buyers at the same moment. And the
largest databases at an e-commerce site are not the order-processing ones. They
are the ones tracking customer browser behavior: pages sent, clicks, cart adds,
abandoned carts.

The last two rows are different in kind. Reporting and data mining
applications do not generate new data. They summarize existing data.
Dashboards and reporting assess past and current performance. Data mining
predicts future performance (Chapter 12).

One more point the book makes: small is not simple. Two companies in the same
line of business at very different sales volumes have similar databases, the
same kinds of data, about the same number of tables, the same complexity of
relationships. Only the amount of data differs.

---

## 6. The four components, and the fifth

A database system consists of four components: users, database application,
database management system (DBMS), database. Figure 1-8 lists them in that
order, and the DBMS's three jobs are to create, process, and administer the
database. Figure 1-9 adds SQL between the application and the DBMS, because
applications send SQL statements to the DBMS, and the chapter summary calls
SQL a fifth component.

**Structured Query Language (SQL)**: an internationally recognized standard
language understood by all commercial relational DBMS products. The professor's
version is "used by all relational databases." SQL is essentially the same in
all products, so it is vendor and product independent, with some syntax
differences. The professor names the typical top four as MySQL, SQL Server,
Postgres, Oracle, and notes that some are stricter than others: Microsoft SQL
Server is not as strict as Oracle or Postgres.

**Database management system (DBMS)**: a computer program used to create,
process, and administer the database. It receives requests encoded in SQL and
translates them into actions on the database. It is large, complicated, and
licensed from a vendor. Companies almost never write their own. The
professor's gloss: "That's your engine."

**Database application**: a set of one or more computer programs that serves
as an intermediary between the user and the DBMS. It reads or modifies data by
sending SQL to the DBMS and presents data as forms and reports. Applications
are bought from vendors or written in house. The **user** employs a database
application to keep track of things, using forms to read, enter, and query
data, and produces reports.

---

## 7. What the application does

Five basic functions of application programs, in the book's order: create and
process forms, process user queries, create and process reports, execute
application logic, control the application itself.

A form hides the structure of the underlying tables. It is built for the
users' needs, not the table structure. The application generates the SQL
insert, update, or delete behind it. Figure 1-11 is the book's example, a form
that reads one class and enters its enrollments.

A **query statement** is an SQL statement that asks the DBMS to obtain
specific data from a database. The application builds the query, sends it to
the DBMS, and formats the result. Reports are similar: query the DBMS, then
format the results, structured to the users' needs, sorted and grouped.
The shape of a basic query:

```sql
SELECT  TeamName, City
FROM    TEAM
WHERE   City = 'Pittsburgh'
```

SELECT names the columns, FROM names the table, WHERE filters the rows
(added).

Application logic. The book's scenario is an order for more units than are in
stock. Refuse and notify, or ship what exists and backorder the rest, or
something else. The application, not the DBMS, decides.

Control the application, two ways: present only logical choices to the user,
such as a menu of valid options, and control data activity with the DBMS, such
as telling it to make a set of changes as a unit, all of them or none of them
(Chapter 9).

---

## 8. What the DBMS does

Five DBMS products hold the lion's share of the market: Microsoft Access,
Oracle Database and MySQL (both Oracle Corporation), SQL Server (Microsoft),
DB2 (IBM). That is the book's list. Two others are in play.

```
Which product list is the question quoting? (added)
Source                             List, in the source's order
Book, five market leaders          Microsoft Access, Oracle Database, MySQL,
                                   SQL Server, DB2
Professor, typical top four        MySQL, SQL Server, Postgres, Oracle
DB-Engines, April 2020, top three  Oracle Database, MySQL, SQL Server
```

Nine functions of a DBMS, in the book's order: create database, create tables,
create supporting structures (e.g., indexes), modify (insert, update, or
delete) database data, read database data, maintain database structures,
enforce rules, control concurrency, perform backup and recovery. Memory hook
(added): three create, two touch data, one maintain, three administer.

An **index** is a supporting structure, akin to the index at the back of a
book, that speeds sorting and searching. Maintaining structures means changing
the format of a table or supporting structure over time.

Enforce rules is where the keys from Section 3 get teeth. A **referential
integrity constraint** is a declared rule that a value in a foreign key column
must already exist in the referenced table's primary key column. The DBMS
disallows an insert or update that breaks it. In Section 3's CONTRACT table, a
row for TeamNumber 9 is rejected if no team 9 exists.

The last three functions are administration, and the book also describes a
security system alongside them. Concurrency control: one user's work must not
inappropriately interfere with another's (Chapter 9). A security system: only
authorized users perform authorized actions, and users can be blocked from
seeing certain data or limited to certain changes. Backup and recovery: the
database is a centralized, valuable asset, protected against errors, hardware
and software failures, and catastrophes.

---

## 9. What the database holds

A **database** is a self-describing collection of integrated tables. An
**integrated table** stores both data and the relationships among the data.
**Self-describing** means the database contains a description of itself,
tables of data that describe the user data. That description is **metadata**:
data about data. Its form and format vary by DBMS, but most products keep it
in tables, so SQL can query it exactly as it queries user tables. Learn SQL
for user tables and you have learned SQL for metadata.

Metadata in action. The SQL Server example queries the compatibility view
SYS.SYSOBJECTS for a user table (Type = 'U') by name. A **view** is a virtual
table defined by a saved SQL query (Chapter 7).

Figure 1-16 lists eight database contents: tables of user data, metadata,
indexes, user-defined functions, stored procedures, triggers, security data,
backup/recovery data. Indexes, user-defined functions, stored procedures, and
triggers are discussed in Chapters 7, 10, 10A, 10B, 10C. Security data and
backup/recovery data are discussed in Chapters 9, 10, 10A, 10B, 10C.
**Triggers** maintain database accuracy and consistency and enforce data
constraints. **Stored procedures** are used for database administration tasks
and are sometimes part of applications.

One action, end to end: a librarian marks a book returned on a form. The
application built the form, turns the click into an UPDATE on the LOAN table
that sets ReturnDate on the matching LoanNumber row, and sends that SQL to the
DBMS. The DBMS checks the change against its declared rules before writing it.
Had the UPDATE set BookNumber to a value no BOOK row carries, the referential
integrity constraint would have rejected it. Metadata told the DBMS that LOAN
is a table, ReturnDate one of its columns, and what type it holds. The
application reads the row back and redraws the form. Four components, one
round trip.

---

## 10. Personal versus enterprise-class

Database systems and DBMS products divide into two classes: personal database
systems and enterprise-class database systems. Microsoft Access is not just a
DBMS. It is a **personal database system**: a DBMS plus an application
generator (form, report, and query components). Those components create SQL
and pass it to the DBMS (Figure 1-17).

The **Access Database Engine (ADE)** is the DBMS engine inside Access, an
Office-specific version of Microsoft's **Joint Engine Technology (JET or
Jet)**. Jet was the engine until Office 2007, when Access switched to ADE. ADE
was originally named the **Office Access Connectivity Engine (ACE)**, the same
technology.

Access is a low-end product for individuals and small workgroups, and
Microsoft hides the technology from the user. Good for beginners, not for
professionals on larger databases. Access ships in the Windows version of
Office, not the macOS version, so it is often a student's first DBMS. Access
2000 and later can replace Jet or ADE with another DBMS, typically SQL Server,
using the **Open Database Connectivity (ODBC)** standard (Chapter 11). ODBC
(added definition): a standard for connecting an application to a DBMS other
than its native engine.

In an **enterprise-class database system**, applications are separate from
each other and from the DBMS. Figure 1-19 shows five application categories,
all sending SQL to the DBMS: client-server applications over a corporate
network (VB.NET, C++, Java), e-commerce and other applications on a Web server
(browsers Edge or the older Internet Explorer, Firefox, Chrome, Web servers
IIS and Apache, languages PHP, Java, C#.NET, VB.NET, Chapter 11), Web portal
reporting applications (IBM Cognos Business Intelligence, MicroStrategy 10,
Chapter 12), XML Web services (Java or .NET, Chapter 12), and mobile apps (not
covered in the book).

Popularity, as of April 2020 (DB-Engines ranking): the three most popular
enterprise-class relational DBMS products, in order, are Oracle Database,
MySQL, Microsoft SQL Server. PostgreSQL ranks fourth, MongoDB fifth, DB2
sixth, and Microsoft Access tenth, probably because it ships with Office. Of
the top 10, seven are relational and three are non-relational.

The book's primary DBMS is Microsoft SQL Server 2019, with support for Oracle
Database and MySQL. Client tools: SQL Server Management Studio, Oracle SQL
Developer, MySQL Workbench. SQL Server 2019 Developer Edition is free and
identical to the full Enterprise edition, but single-user development only.
Express Edition is free and usable for smaller production databases (limited,
for example in maximum storage). Online Chapter 10A.

Oracle Database 19c is the current Enterprise Edition, 20c is in preview, and
Enterprise Edition needs purchased user licenses. The free version is Oracle
Database 18c Express Edition, called Oracle Database XE. Online Chapter 10B.

MySQL Community Server 8.0 is free, full-strength, and production-usable. MySQL
Enterprise Edition 8.0 is purchased for the full support package. MySQL uses
all lowercase object names. Online Chapter 10C.

The book then poses one query to the STUDENT table of its Student_Class_Grade
database. Figures 1-20 to 1-22 run the same query in the three client tools.
The lesson: the same SQL runs unchanged in all three, and only the result
order differs. The dbo in dbo.Student is SQL Server's default schema (added).

A DBMS is an application like any other. It runs on top of an operating system
that handles file reads and writes, printing, and the other basic operations.
The OS products named: Microsoft Windows, Microsoft Windows Server, Apple
macOS, and various versions of Linux. Access and SQL Server run on Microsoft
operating systems, and SQL Server now also runs on Linux. Oracle Database runs
on Windows and Linux, not macOS. MySQL is the only one of the three that runs
on all three operating systems. The three being counted are SQL Server, Oracle
Database, and MySQL, not Access (added).

---

## 11. How databases come to exist

**Database design** (as a process): the creation of the proper structure of
database tables, the proper relationships between tables, appropriate data
constraints, and other structural components of the database. Database design
(as a product): the annotated diagram that results from that process, the
basis for building the actual database in a DBMS. The book uses the term both
ways. Read the context.

Correct design is both important and difficult, so the world is full of poorly
designed databases: poor performance, developers forced to write overly
complex and contrived SQL, difficulty adapting to new and changing
requirements, or failure in some other way. Most of the first half of the text
is about design.

Figure 1-23 gives three types of database design: from existing data (Chapters
3 and 4), new systems development (Chapters 5 and 6), and database redesign
(Chapter 8). Chapter 7 covers database implementation using SQL, and that
knowledge is needed before redesign makes sense.

**From existing data** (Figure 1-24) has two inputs. Either a team is handed
spreadsheets or text files holding tables of data and must design a database
and import them, or the new database is built from extracts of other
databases. The extract route is especially common in **business intelligence
(BI) systems**, which include reporting and data mining. Data from an
operational database (a CRM or ERP database) is copied into a **data
warehouse** or **data mart**, databases that store data organized specifically
for research and reporting (Chapter 12). Named analytical tools: SAS
Enterprise Miner, IBM SPSS Modeler, TIBCO Spotfire.

Even importing a single table raises the design question. Figure 1-25 asks
whether a flat employee list with a department number and department name on
every row should be one table or two. A spreadsheet of shipments carries a
carrier code and carrier name on every row. Option (a), one SHIPMENT table
with the carrier name repeated. Option (b), CARRIER (CarrierCode,
CarrierName) plus SHIPMENT (ShipmentNumber, WeightKg, CarrierCode), linked by
CarrierCode. Repeated descriptive text is the tell. The decision is not
arbitrary: **normalization**, or **normal forms**, is the set of principles
database professionals use to guide and assess designs (Chapter 3).

**New systems development** (Figure 1-26) starts from requirements: desired
forms and reports, user requirement statements, use cases, and other systems
development documents, together called systems requirements. A **use case** is
a scenario of users interacting with an information system to obtain a desired
result. The jump from requirements straight to a design is too big, so the
team takes two steps: create a data model from the requirements, then
transform the data model into a database design. A **data model** is a
blueprint used as a design aid on the way to a database design.
**Entity-relationship (ER) data modeling** is the most popular technique
(Chapter 5), and Chapter 6 teaches the transformation.

The two steps on one requirement: a user statement reads "every loan must
record who borrowed which book and when it is due." Step one, the data model:
entities PATRON, BOOK, and LOAN, with each LOAN tied to one patron and one
book. Step two, the transformation: LOAN becomes a table with LoanNumber as
its primary key, PatronNumber and BookNumber as foreign keys, and DueDate as a
column.

**Database redesign** (Figure 1-27) has two common types. **Database
migration** adapts an existing database to new or changing requirements:
tables created, modified, or removed, relationships altered, constraints
changed. Database integration merges two or more databases into one design,
common when adapting or removing legacy systems and in enterprise application
integration.

---

## 12. Two roles, and what to learn

Two ways to work with database technology: as a user or as a database
administrator. Users come in two flavors. A **knowledge worker** prepares
reports, mines data, and does other data analysis. A **programmer** writes
applications that process the database. A **database administrator (DBA)**
designs, constructs, and manages the database itself. Users are primarily
concerned with constructing SQL to store and retrieve data. DBAs are primarily
concerned with managing the database.

Figure 1-28 maps the two roles onto the database system. Users reach the
database through applications, which send SQL to the DBMS. Knowledge workers
and programmers own the application side, the DBA owns the DBMS and database
side.

Figure 1-29 rates each topic for each role, 1 very important, 2 important, 3
less important.

```
Topic                                Chapter(s)        DBA  KW/Prog
Basic SQL                            2                 1    2
The relational database model        3                 2    2
Design via normalization             4                 2    1
Data models                          5                 2    1
Data model transformation            6                 2    1
SQL DDL and constraint enforcement   7                 3    1
Database redesign                    8                 3    1
Database administration              9                 3    1
Product specifics (SQL Server,       10, 10A to 10D    3    1
  Oracle Database, MySQL, ArangoDB)
Database application technology      11, 12, 13        1    3
```

The authors say opinions vary and to ask your instructor.

---

## 13. Where it came from

The history, first as one table, then era by era (added).
Figure 1-30 is the era table.

```
Era                      Years         Products
Predatabase              Before 1970   File managers
Early database           1970-1980     ADABAS, System2000, Total, IDMS, IMS
Emergence of relational  1978-1985     DB2, Oracle Database, Ingres
Microcomputer DBMS       1982-1992+    dBase-II, R:base, Paradox,
                                       Microsoft Access
Object-oriented DBMS     1985-2000     Oracle ODBMS, Gemstone, O2, Versant
Data warehouses          1998-present  Red Brick Warehouse,
                                       Prism Warehouse Manager, others
Web databases            1995-present  IIS, Apache, PHP, ASP.NET, Java
Open source DBMS         1995-present  MySQL, PostgreSQL, others
XML and Web services     1998-present  XML, SOAP, WSDL, UDDI, other standards
Big Data, NoSQL, cloud   2009-present  Hadoop, Cassandra, Hbase, CouchDB,
                                       ArangoDB, MongoDB, JSON, Azure, AWS,
                                       Google Cloud
```

**The early years.** Before 1970 all data lived in separate files, most on
reels of magnetic tape, and disks and drums were expensive and tiny. The
authors' 1969 payroll machine had 32,000 bytes of memory, against 16 gigabytes
on the machine the history was written on. Integrated processing, relating
records on one tape to records on another, drove the first database
technology. By 1973 several commercial DBMS products had emerged, and they
were in use by the mid-1970s.

The first edition of this text, copyrighted 1977, featured ADABAS, System2000,
Total, IDMS, and IMS. ADABAS and IMS are still in use with no substantial
market share.

Two early ways to structure relationships. **Data Language/I (DL/I)** used
hierarchies or trees. IMS, licensed by IBM, is based on it and is still in
limited use with large manufacturers. The **CODASYL DBTG** model is the
network data model from the Database Task Group of the CODASYL Committee, the
group that developed COBOL. It was unnecessarily complicated, but several
products succeeded. The most successful was IDMS, and its vendor, the
Cullinane Corporation, was the first software company listed on the New York
Stock Exchange.

**The relational model.** In 1970 Edgar Frank (E. F.) Codd, then a
little-known IBM engineer, published a paper in the Communications of the ACM
applying relational algebra to "shared data banks," the term for databases at
the time. All relational DBMS products are built on this model. The first
reaction: too theoretical, too slow, too much storage, never useful
commercially.

Codd reviewed the relational chapter in the 1977 edition of this text, and
Wayne Ratliff got the idea for dBase reading that chapter. Codd convinced IBM
management to build relational products, and the result was DB2. By 1980
several more relational products had shipped. Oracle Database (originally
named just "Oracle") won by running on almost any machine and operating
system, plus an elegant internal design and hard-driving sales.

**The PC era.** dBase was the most successful early PC product, R:base was the
first to implement true relational algebra on the PC, and Paradox came later
and was acquired by Borland. In 1991 Microsoft released Microsoft Access at
$99. No other PC DBMS vendor could survive that price. Access killed R:base and
Paradox. Microsoft then bought the dBase work-alike FoxPro to eliminate dBase,
and later discontinued Visual FoxPro too. Access is the only major survivor.
Today's challenger: OpenOffice.org Base and LibreOffice Base.

**Object-oriented DBMS.** Object-oriented programming (OOP) emerged in the
mid-1980s, and by 1990 some vendors had an **object-oriented DBMS (OODBMS or
ODBMS)**, built to store data encapsulated in OOP objects. Oracle added OOP
constructs to make an **object-relational DBMS** hybrid. OODBMS never caught
on, for two reasons: using one required converting billions of bytes of
relational data, and it offered no substantial advantage for most commercial
processing. SQL is not object-oriented, but it works. The professor's slide
gives the reason as business requirements.

**OLTP, warehouses, and BI.** A purchase is a business transaction that must be
recorded in the company's accounts, and by the late 1980s applications for
**online transaction processing (OLTP)** were well understood. Analysis should
not run on the production database, so a data warehouse is a separate place to
store data for analysis. The first formal presentation of the term was a 1988
article by IBM researchers B. A. Devlin and P. T. Murphy. **Online analytical
processing (OLAP)** is the analysis work done on the data warehouse, an example
of a business intelligence (BI) system, tools used to analyze and report on
company data (Chapters 2 and 12).

**The Web.** By the mid-1990s the Internet had changed how customers and
businesses relate. Early sites were online brochures, then dynamic
database-driven sites appeared. The problem: **Hypertext Transfer Protocol
(HTTP)** is stateless. The server processes a request and forgets the user.
Shopping carts are multistage, so capabilities were built on top to overcome
it.

A **Web database application** is an application with a Web user interface
that depends on a database. The professor's slide describes it as allowing
shopping using an API. The book's running company is Wedgewood Pacific (WP), a
maker of consumer drones (technically unmanned aerial vehicles, UAVs), shown in
Figure 1-31. An **application programming interface (API)** is what
applications and Web pages use, in a language such as PHP or JavaScript, to
connect to a DBMS, send SQL, and receive results (Chapter 11).

**Open source.** MySQL was released in 1995 by the Swedish company MySQL AB.
Sun Microsystems bought MySQL AB in February 2008. Oracle completed its
acquisition of Sun Microsystems in January 2013. Oracle now owns Oracle
Database and Oracle MySQL, and MySQL is popular with Web developers on Linux
servers. Users unhappy with the acquisition forked the last open source MySQL
code into **MariaDB**, largely compatible with MySQL. Both are named for
developer Monty Widenius's daughters, My and Maria.

**XML.** In the late 1990s **eXtensible Markup Language (XML)** was defined to
overcome the problems that occur when HTML is used to exchange business
documents, and it proved superior for exchanging views of database data. Bill
Gates, 2002: "XML is the lingua franca of the Internet Age." Two problems are
still not fully resolved: getting data out of a database into an XML document,
and from an XML document into a database (Appendix H). Web service standards
gave it a further boost: SOAP (once an acronym, now just a name), WSDL (Web
Services Description Language), UDDI (Universal Description, Discovery, and
Integration).

**Big Data, NoSQL, and cloud.** The **NoSQL ("Not only SQL") movement** and
Big Data emerged after a 2009 conference on open source distributed
databases. The authors say it should really be called NoRelational, since
the work is on databases that do not follow the relational model. Big Data
plus NoSQL is the basis of Facebook and Twitter, both of which use Apache
Cassandra, and large datasets run on server clusters for storage and
parallel processing. A **document database** structures data on XML or, more
recently, JSON. MongoDB is the most popular non-relational DBMS and ranks
fifth overall as of April 2020. More document DBMSs use JSON than XML.

The book's non-relational DBMS is ArangoDB, from the German company triAGENS
GmbH. Its **ArangoDB query language (AQL)** is its SQL equivalent, and its
output is JSON with generated columns _key, _id, and _rev (Figure 1-32).
Relational vendors have adjusted: SQL Server 2019 supports graph databases and
MySQL supports document data.

**Cloud computing** means using hardware owned and operated by another company
instead of in-house servers. The authors' test: if you know where your
company's servers are, you are not using cloud computing. If they are at
somebody else's **data center**, a facility that houses many servers and their
infrastructure, you are. Vendors: Microsoft Azure (Azure SQL, Cosmos DB
non-relational), Amazon Web Services (AWS), Google Cloud. Figure 1-33 is the
book's Azure example, with both a relational and a non-relational database
hosted on Azure (Chapters 12 and 13).

---

## 14. The thread

The module is one idea, seen from four heights. First, the ground: a database
keeps track of things, and a relational one does it with tables whose rows are
instances, whose columns are characteristics, and whose keys turn a pile of
tables into data plus relationships. Second, the machine: a user works a form,
the application turns the form into SQL, the DBMS turns the SQL into actions,
and the database holds the tables and the metadata that describes them. Access
packs application and DBMS under one cover, an enterprise-class system keeps
them apart. The data scientist meets that machine through SQL, the knowledge
worker and programmer from the application side, the DBA from the DBMS side.

Third, the craft: a database comes from existing data through
normalization, from new requirements through a data model, or from an old
database through migration or integration. Fourth, the history: tapes, then
trees and networks, then Codd's algebra in 1970, then Access at $99, then
warehouses, the Web, open source, XML, and NoSQL. The first database technology
was built for one problem the tapes could not solve: relating a record on one
tape to a record on another. The relational model's answer is the key, and
everything in this module follows from it.

---

*Authored and directed by **DatJavaClass (Victor S)**, who conceived,
structured, formatted, fact-checked, and edited these notes, with assistance by
Claude. Some material may have been derived from assigned material, but has not
been copied verbatim. For source materials please contact CMPINF-2110 Faculty
and Assistants.*

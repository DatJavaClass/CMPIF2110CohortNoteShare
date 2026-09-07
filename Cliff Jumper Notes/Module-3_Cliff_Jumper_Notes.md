# Module 3: Cliff Jumper Notes

*One continuous lesson stitched from the Module 3 cliff notes of Managing,
Querying, and Preserving Data (CMPINF-2110): what SQL is and where it came
from, the four kinds of SQL statement, why a company copies its sales data
into a second database before asking it questions, the six Cape Codd tables
and the keys that tie them together, the SELECT/FROM/WHERE framework, how a
statement reaches four different DBMS products, DISTINCT and TOP, the WHERE
clause with its comparison and logical operators, ORDER BY, and how to read
an error message when the query breaks.*

---

## 1. The remote control

**SQL (Structured Query Language)** is a data sublanguage for creating and
processing database data and metadata. The professor's analogy is a remote
control: it changes channels and adjusts volume, but you cannot build a TV
with it. SQL manipulates and retrieves data, it does not build the system
that holds it. It is not a full featured programming language, not Python or
Java, and that is by design. It is a sublanguage, used specifically for
managing and manipulating data inside relational databases, which store data
in organized tables. Think of a table as a spreadsheet with columns and rows.

The textbook, Kroenke and Auer's Database Processing (16th edition, Chapter
2, the source of every figure and page number below), calls SQL the universal
query language of relational DBMS products. It sits behind every friendly
query tool. **Query by example (QBE)** is the GUI style Microsoft Access uses
to simplify **ad hoc queries**, and Open Text's Open Text Business
Intelligence product (formerly LiveLink ECM BI Query) does the same job with
its own GUI. Whatever the front end shows, SQL is always behind it. An **ad
hoc SQL query** is how SQL is used to ask questions about the data in the
database, and SQL is an essential asset for a knowledge worker, application
programmer, or database administrator. The enterprise DBMS products the
chapter covers, Oracle Database and Oracle MySQL 8.0 among them, require that
you know SQL, because all data manipulation in those products is expressed
in it. Even Access users have used SQL without knowing it: every time you
process a form, create a report, or run a query, Access generates SQL and
sends it to its internal ACE DBMS engine.

The history, as the professor tells it: SQL was developed by IBM in the
1970s, credited to Raymond Boyce and Donald Chamberlin, and became a standard
endorsed by ANSI, the American National Standards Institute, in 1992. Today
it is a core part of every enterprise level relational system: MySQL,
Postgres, Oracle, Microsoft SQL Server, and others.

The textbook's longer timeline, which is quiz bait: developed by IBM in the
late 1970s; endorsed as a national standard by ANSI in 1986 and by the
International Organization for Standardization (ISO) in 1987; new versions
in 1989 and 1992, the 1992 version being called SQL-92 or ANSI-92 SQL; then
SQL:1999, also called SQL3, which brought in some object oriented concepts;
then SQL:2003, SQL:2006, SQL:2008, SQL:2011, and SQL:2016. Three additions
the book names: the INSTEAD OF trigger standardized in SQL:2008 (triggers
are Chapter 7), support for XML added in SQL:2009 (Chapter 11 and Appendix
H), and support for JSON added in SQL:2016 (Chapter 13). Chapters 2 and 7
mostly use features present since SQL-92, with a few from SQL:2003 and
SQL:2008.

One more thing the professor sets up before any SQL is written: a database
stores data and relationships, not data alone, so data gets split across
tables when one thing can have many of another. A person can have a mobile
phone, a home phone, and a work line, so phone numbers live in their own
table rather than inside the person's row. The Cape Codd tables the module
queries are built that way.

---

## 2. Four kinds of statement

SQL is made up of statement types, or languages, each with its own purpose.
**DDL (data definition language)** creates schema: tables, relationships,
and other structures such as indexes and stored procedures. **DML (data
manipulation language)** inserts, modifies, and deletes data; when you want
to change data, you use a DML statement. **DCL (data control language)**
grants or revokes database permissions, for instance hiding a table that
contains a date of birth column. **DQL (data query language)** queries schema
objects and retrieves data. SELECT is a DQL statement, and SELECT is what
this whole module is about.

A way to hold the four (added): DDL builds the shelves, DML puts things on
them and takes them off, DCL decides who may open the cabinet, DQL reads
what is there without moving anything.

---

## 3. Why the data gets copied first

Before the textbook writes a single SELECT, it explains where the data being
queried came from, and the answer is a second database.

Cape Codd Outdoor Sports is a fictitious company based on a real outdoor
retail equipment vendor. It sells recreational outdoor equipment in 15
retail stores across the United States and Canada, over the Internet from a
Web storefront application, and by mail order through annual catalogs sent
to all recorded customers in early January of each year. All retail sales
are recorded in a sales database managed by the Oracle Database 19c DBMS.
That database is an **online transaction processing (OLTP) system**: the
type of sales system used to record all sales transactions of the company,
whether in a store, on the Web, or from mail order or phone order sales.
OLTP systems are the backbone of businesses as they operate today.

Figure 2-2 draws the extraction path: point of sale applications for Store 1
through Store 15, the Web storefront, and mail order sales all feed the
Oracle Database 19c OLTP sales database, and a retail store sales data
extraction from that database produces the Cape_Codd sales data database.
That second database exists because marketing wants two analyses, in store
sales and catalog content, and marketing analysts ask IT to extract retail
sales data for them.

The general pattern has names. **Business intelligence (BI) systems**
typically store their data in a **data warehouse**: a database system that
has data, programs, and personnel that specialize in the preparation of data
for BI processing (detail in Chapter 12). Warehouses vary from a sole
employee processing a data extract part time to a department with dozens of
employees maintaining libraries of data and programs. **Operational
databases** are the ones that store the company's current day to day
transaction data. An **Extract, Transform, and Load (ETL) system** reads data
from operational databases, from other internal data, or from external data
sources, then cleans and prepares that data for BI processing, and that can
be a complex process. Figure 2-3 chains it: operational databases, other
internal data, and external data feed the ETL system; the ETL system feeds
the data warehouse DBMS; that DBMS holds the data warehouse database and the
data warehouse metadata; the DBMS feeds business intelligence tools, which
serve BI users. A **data mart** is a small, specialized data warehouse.

The warehouse DBMS may or may not be the same product as the operational
DBMS. The book's example runs the operational databases in Oracle Database
19c and the data warehouse in Microsoft SQL Server 2019, which is why the
extract is named Cape_Codd, the SQL Server 2019 name. Other products may use
a variant: MySQL does not use uppercase characters in database names, so
there it would be cape_codd.

Extraction is not a straight copy. The extract includes only retail store
sales; returns and other sales related transactions are not copied. It
selects only a few columns, where point of sale applications process far
more. And it transforms: the Oracle Database 19c order data has an OrderDate
column stored as MM/DD/YYYY (10/22/2020 for October 22, 2020), and the
extraction program converts it into two separate values, OrderMonth and
OrderYear, because that is the format marketing wants. Such filtering and
transformation are typical of a data extraction process, and the book is
firm that this is not an academic exercise: hundreds of businesses worldwide
use BI systems to create extract databases like this one.

---

## 4. The six Cape Codd tables

The Cape_Codd extract is a relational database (Chapter 1, more depth in
Chapter 3), and every SQL example in the module runs against it, so its
shape is worth knowing cold. Two key terms carry over from Module 2. A
**primary key** is one or more columns that uniquely identify each row (or
record) in a table, and every table has one. A **foreign key** is a column
used to create relationships between tables to logically link them, and
some tables have one or more.

Six tables are needed (Figures 2-4 and 2-5). Four are linked and serve the
retail sales analysis: RETAIL_ORDER, ORDER_ITEM, SKU_DATA, and BUYER. Two
are free standing and serve the catalog analysis: CATALOG_SKU_2020 and
CATALOG_SKU_2021, extracted for two years because the analysts want to
compare catalog content year to year. A **stock keeping unit (SKU)** is a
unique identifier for each particular item Cape Codd sells.

The complete schema, in the book's notation (primary keys underlined,
foreign keys italicized on the page):

    RETAIL_ORDER (OrderNumber, StoreNumber, StoreZIP, OrderMonth,
        OrderYear, OrderTotal)
    ORDER_ITEM (OrderNumber, SKU, Quantity, Price, ExtendedPrice)
    SKU_DATA (SKU, SKU_Description, Department, Buyer)
    BUYER (BuyerName, Department, Position, Supervisor)
    CATALOG_SKU_2020 (CatalogID, SKU, SKU_Description, Department,
        CatalogPage, DateOnWebSite)
    CATALOG_SKU_2021 (CatalogID, SKU, SKU_Description, Department,
        CatalogPage, DateOnWebSite)

A **database schema** is that complete logical view: all the tables, all
the columns in each table, the primary key of each table (underlined), and
the foreign keys that link the tables (italicized).

Keys by table. RETAIL_ORDER's primary key is OrderNumber. ORDER_ITEM has a
**composite primary key**, a primary key made of more than one column:
OrderNumber and SKU together, and both columns are also foreign keys,
OrderNumber back to RETAIL_ORDER and SKU back to SKU_DATA. SKU_DATA's primary
key is SKU and its foreign key Buyer points at BuyerName in BUYER. BUYER's
primary key is BuyerName, and its Supervisor column is a foreign key in a
**recursive relationship**, a relationship between columns in the same
table: Supervisor points back at BuyerName. Each CATALOG_SKU table has
CatalogID as its primary key and no foreign keys at all; the SKU column
could link to SKU_DATA, but that link is not created at this time.

Figure 2-4 shows all six tables in the Microsoft Access 2019 Relationships
window, with relationship lines drawn only among the four linked ones. The
key symbol marks the primary key; the number 1 and the infinity symbol mark
one to many, so one retail order may be linked to many order items by
OrderNumber. The cardinalities: RETAIL_ORDER one to ORDER_ITEM many,
SKU_DATA one to ORDER_ITEM many, BUYER one to SKU_DATA many. Access 2019
cannot properly display recursive relationships, so the one inside BUYER is
missing from that window. Figure 2-6 holds the sample
data, part (a) the four linked tables and part (b) the two catalog tables,
and there the recursive relationship and every key are clearly visible. Row
counts in Figure 2-6: RETAIL_ORDER 3, ORDER_ITEM 7, SKU_DATA 13, BUYER 5,
and 9 in each CATALOG_SKU table. The dataset is deliberately small, big
enough to illustrate the chapter while staying manageable; a real extract
would be much larger.

What each table means. RETAIL_ORDER has one row per retail sales order:
OrderNumber, StoreNumber, StoreZIP (the ZIP of the store selling the
order), OrderMonth, OrderYear, OrderTotal. It deliberately lacks the
CustomerLastName, CustomerFirstName, and OrderDay columns an operational
system would carry. ORDER_ITEM has one row per item in an order, the way a
receipt has one line per item: Quantity is how many of that SKU were bought,
Price is the price of each, and ExtendedPrice equals Quantity times Price.
SKU_DATA describes each SKU: SKU 100100 is a yellow, standard size SCUBA
tank and SKU 100200 is the magenta version of the same tank; Department and
Buyer name the department and the person responsible for purchasing it.
BUYER describes the buyers in the Purchasing Department: BuyerName is the
combined first and last name, Department is Purchasing for every buyer,
Position is the buyer's rank, Supervisor is who the buyer reports to. BUYER
is itself a subset of a table named EMPLOYEE that holds every Cape Codd
employee. The CATALOG_SKU tables record the printed catalog and the Web
site: CatalogPage is the page where the item appeared, DateOnWebSite is the
first date the item could be seen on the site, and because some items are
added to the Web site after the catalog is printed, an item on the site may
not be in the corresponding catalog.

Figure 2-5 gives the data types, and the odd ones are the quiz bait:
StoreZIP is Character (9), not a number; OrderMonth is Character (12);
OrderYear is Integer; OrderTotal, Price, and ExtendedPrice are Currency;
SKU_Description, BuyerName, Buyer, and Supervisor are Character (35);
Department is Character (30); Position is Character (10); CatalogPage is
Integer; DateOnWebSite is Date. OrderNumber, StoreNumber, SKU, Quantity,
and CatalogID are Integer.

Three lessons hide in the sample data. First, the ExtendedPrice total of an
order does not equal OrderTotal: order 1000 has SKU 201000 at $300.00 and
SKU 202000 at $130.00, so its lines sum to $430.00, while its OrderTotal is
$445.00, because OrderTotal includes tax, shipping, and other charges that
do not appear in the extract. Second, column names that match across tables
may hold very different data. SKU_DATA.Department holds product departments
(Water Sports, Camping, Climbing); BUYER.Department is Purchasing on every
row. A **domain** is the set of possible data values for a column, and two
columns sharing the same set of values have data from the same domain.
These two do not share a set of values, so they come from different
domains. Better practice would be different names, but that is not always
done. Third, a **null value** is a missing data value. Mary Smith is
Manager of Purchasing and has no supervisor, so her Supervisor is null. In
CATALOG_SKU_2020 the row for SKU 100400 (Std. Scuba Tank, Dark Blue) has no
CatalogPage, and in CATALOG_SKU_2021 the row for SKU 203000 (Half-dome Tent
Vestibule, Wide) has no CatalogPage, though both still carry a
DateOnWebSite. Treat a null like any other value: you can search for it
with the same techniques used for any other value (Chapter 4 has the
detail).

The chapter's Review Questions extend the schema with WAREHOUSE, INVENTORY,
and CATALOG_SKU_2019. Some figures show them; the chapter text does not use
them.

Example, used for the rest of this lesson: a music store's extract with
INSTRUMENT (InstrumentID, InstrumentName, Section, Vendor, ListPrice),
where Section takes values like Strings, Brass, and Percussion, and Vendor
takes values like Harbor Music and Northwind Instruments. InstrumentID is
the primary key. Every query below runs against that one table, because
this module never joins two.

---

## 5. The framework

The **SQL SELECT/FROM/WHERE framework** is the basic form of an SQL query,
and the professor and the textbook describe it the same way. SELECT
specifies which columns are to be listed in the results. FROM specifies
which tables are to be used. WHERE specifies which rows are to be listed.
Columns, tables, rows, in that order as written. The professor adds the
verb: SELECT is the action verb that tells the engine you are retrieving
data, the column list after it is comma delimited, FROM names the table
object where the data lives, WHERE begins the filtering, and the semicolon
terminates the statement.

The simplest query names the columns you want and the table they live in:

    SELECT  InstrumentID, InstrumentName, Section, Vendor, ListPrice
    FROM    INSTRUMENT;

Naming every column returns the entire table. The **SQL asterisk (\*)
wildcard character** is shorthand for all of the columns:

    SELECT  *
    FROM    INSTRUMENT;

With no WHERE clause, that returns every row and every column. The
textbook's own worked queries run against SKU_DATA (SKU, SKU_Description,
Department, Buyer), 13 rows, data in Figure 2-6, and it labels each one in a
comment so the text can point at it: SQL-Query-CH02-01 reads all four
SKU_DATA columns, CH02-02 uses the asterisk, CH02-03 reads Department then
Buyer, CH02-04 reads Buyer then Department.

Three rules ride along with the framework.

**Only SELECT and FROM are required.** The BY THE WAY box on p. 58 says it
plainly: in the SQL SELECT statement, the SELECT clause and the FROM clause
are the only required clauses. You have a complete query by telling SQL
which columns should be read from which table. Everything else, WHERE
included, is optional. The professor's demo makes the same point from the
other side: a query with no WHERE clause is legal, filtering is optional,
sorting is optional.

**Every SQL statement produces one table.** Executing a statement
transforms tables: it starts with a table, processes it in some way, and
places the results in another table structure. Even if the result is a
single number, that number is a table with one row and one column. Some
statements process multiple tables (the end of the chapter), but regardless
of how many go in, one table comes out.

**The SELECT phrase sets the column order.** To get a subset of columns,
name only those columns. The order of the names in the SELECT phrase
determines the order of the columns in the results, so switching two names
switches the two columns in the output while the rows and their values stay
the same. CH02-04 lists Buyer before Department, which is not the order the
columns sit in the table, and the result follows the SELECT phrase. The
professor shows the same thing by swapping two names in a column list. And
selecting a subset of columns does not collapse duplicate rows: Department
with Buyer still returns all 13 SKU_DATA rows, most of them repeats. That
fact sets up DISTINCT in section 8.

Mechanics. Statements terminate with a semicolon, which is required by the
SQL standard; some products let you omit it and some will not, so the habit
is to always write it. The professor says the same thing in the demos:
always end a SQL command with a semicolon. An **SQL comment** is a block of
text used to document a statement but not executed as part of it. The
standard form encloses the comment in /\* and \*/, and anything between is
ignored, so adding a comment does not change the output at all. The book's
label comments look like this:

    /* *** SQL-Query-CH02-01 *** */

The professor's demos add the MySQL forms: two hyphens followed by a space
for a single line comment, and slash star to star slash for a block. The
space after the two hyphens matters in MySQL (added), and # also opens a
line comment there (added). Keywords are case insensitive, so SELECT,
select, and Select all work; the demos type keywords in capitals, and the
conventional style is uppercase keywords with lowercase table and column
names (added).

Last, a coding convention. Standard practice writes the SELECT, FROM, and
WHERE clauses on separate lines. SQL parsers do not require it, and every
DBMS will process the same statement on one line. The multiline form is
kept because it is easier to read, and the alignment in the examples above
is the book's habit, not a rule.

---

## 6. Getting the statement to a DBMS

A SELECT is text. Something has to carry it to the DBMS and bring the
result table back, and the textbook walks through four products so you can
work along with the chapter while reading. The same SELECT text runs
unchanged in all four. What changes is the tool, the button, and where a
query can be kept.

**The lineup.** Microsoft Access 2019, Microsoft SQL Server 2019, Oracle
Database, and Oracle MySQL 8.0. Free options: SQL Server 2019 Developer and
Express editions, Oracle Database XE, and MySQL 8.0 Server Community
Edition; Access 2019 comes with many Microsoft Office suites. Install and
database creation instructions are Chapter 10A (SQL Server), 10B (Oracle),
and 10C (MySQL), with Appendix A for building the Access database by hand.
Scripts for every product are at www.pearsonhighered.com/kroenke, and three
extra tables ship with the downloads for the Review Questions only:
INVENTORY, WAREHOUSE, and CATALOG_SKU_2019.

**Access and the ANSI-89 problem.** The book's SQL is based on features in
the standards since ANSI SQL-92, which Microsoft calls ANSI-92 SQL. Access
2019 still defaults to the earlier SQL-89 version, which Microsoft also
calls **Microsoft Jet SQL** after the Jet DBMS engine, since replaced by the
ACE engine. ANSI-89 differs significantly from SQL-92, so some SQL-92
features will not work in Access. Access 2019, and the 2003 through 2016
versions before it, has a setting to use SQL-92 instead, added so Access
forms and reports could be used in application development for SQL Server.
The chapter stays in default ANSI-89 mode on purpose, to teach the
limitations and how to cope with them, and flags each failure in a box
titled "Does Not Work with Microsoft Access ANSI-89 SQL" with a workaround.
Very few Access users or organizations set the SQL-92 option. Two Access
files ship for the chapter: Cape-Codd.accdb (ANSI-89) and
Cape-Codd-SQL-92.accdb (SQL-92).

The switch itself: File tab, Options, Object Designers button, and on that
page (Figure 2-7, p. 61) the SQL Server Compatible Syntax (ANSI 92) group in
the Query design area, with two check boxes, This database (the open
database only) and Default for new databases. Click OK and the SQL syntax
information dialog (Figure 2-8, p. 61) warns that existing queries may
return different results or not run at all, that the range of data types
and reserved words will change, that different wildcards will be used, and
that you should make a backup copy first, because Access will close the
database, compact it, and re-open it in the new mode.

**Running a query in Access.** Open the database, click the Create tab
(Figure 2-9, p. 62), click Query Design, and the Query1 window opens in
Design view along with the Show Table dialog (Figure 2-10, p. 63); close
that dialog and Query1 shows the Query Tools contextual tab and the Design
tab (Figure 2-11, p. 63). Design view is where Access queries are created
and edited, and it is the home of Access **QBE (Query By Example)**, the
graphical query builder. The Select button is highlighted in the Query Type
group, meaning the query being built is the equivalent of an SQL SELECT
statement, and active buttons on the Ribbon are always shown in gray. The
View gallery in the Results group switches between Design view and SQL
view, and a separate SQL View button sits above it because Access always
presents a most likely needed view as a button. To run: click SQL View, and
Query1 switches to SQL view (Figure 2-12, p. 64) holding only SELECT; which
is an incomplete command that returns nothing; edit the statement, leaving
out the comment line (Figure 2-13, p. 64); click Run on the Design tab and
the results appear (Figure 2-14, p. 65). Access is a personal database with
an application generator, so it can save the query inside the database:
Save on the Quick Access Toolbar opens the Save As dialog (Figure 2-15, p.
65), type the name SQL-Query-CH02-01 and click OK, and the query object
appears in the Queries section of the Navigation Pane (Figure 2-16, p. 66);
close the window and answer Yes if asked to save the design. The assignment
pattern, repeated for every product: work through the other three framework
queries and save each as SQL-Query-CH02-##, ## running 02 to 04 to match
the comment label.

**SQL Server 2019.** The GUI tool is Microsoft SQL Server Management
Studio, which installs with SQL Server setup and manages the DBMS and its
databases. Chapter 10A creates the Cape_Codd database (note the
underscore). To run (Figure 2-17, p. 67): New Query opens a tabbed query
window; pick Cape_Codd in the Available Databases list if it is not shown;
click IntelliSense Enabled to disable IntelliSense; type the SELECT without
the comment line; optionally click Parse to check syntax first, which
returns "Command(s) completed successfully" or an error; click Execute and
the results appear in a results window. The Object Browser on the left
lists database objects, with Cape_Codd expanded to show its tables, and
many functions are reached by right-clicking an object for a shortcut menu.

**SQL script files.** SQL Server 2019 is enterprise class and does not
store queries within the DBMS. It does store SQL Views, which can be
considered a type of query (Chapter 7). Queries are kept instead as an
**SQL script file**: a separately stored plain text file, usually with a
.sql extension, that can be opened and run as an SQL command or set of
commands, often used to create and populate databases and also to store
queries. To save (Figure 2-18, p. 68): Save opens the Save File As dialog;
browse to
\Documents\SQL Server Management Studio\Projects\DBP-e16-Cape-Codd-Database,
where the two creation and population scripts already sit; type
SQL-Query-CH02-01 in File Name; Save. Rerun with Open File, then Execute.
The book recommends a folder per database inside Projects.

**Oracle Database.** Versions named are Oracle Database 18c, 19c, and XE;
Chapter 10B installs XE and creates the database, and because Oracle naming
is complex the text just calls it Cape Codd. Oracle users were long devoted
to the SQL\*Plus command line tool, but professionals are moving to the
Oracle SQL Developer GUI, which the book adopts. SQL Developer ships with
Oracle Database 19c and must be downloaded separately for XE. To run
(Figure 2-19, p. 69): open the Cape-Codd-Database connection, which shows a
tabbed SQL Worksheet; type the SELECT without the comment line; click
Execute, and the results appear in the Query Result tab. The Connections
object browser on the left expands to show tables, right-click for shortcut
menus. Oracle stores SQL Views but not queries, so a query is saved as a
script (Figure 2-20, p. 71): Save, click the Documents button, browse to
DBP-e16-Cape-Codd-Database, type SQL-Query-CH02-01.sql, Save. The path
shown is
Users\\{UserName}\Documents\SQL Developer\DBP-e16-Cape-Codd-Database, a SQL
Developer folder inside Documents with a subfolder per database. Rerun with
Open File, then Execute.

**MySQL 8.0.** Chapter 10C installs MySQL Community Server 8.0 and creates
cape_codd, all lowercase with an underscore. The GUI tool is MySQL
Workbench, installable alongside the DBMS with the MySQL Installer, and
Workbench calls a database a schema. To run (Figure 2-21, p. 72):
right-click the cape_codd schema object and click Set as Default Schema,
which makes it the active database; in the Query 1 tab inside the SQL
Editor, type the SELECT without the comment line; click the "Execute the
statement under the keyboard cursor" button, and the results appear in a
Result Grid. The Navigator on the left expands cape_codd to show its
tables. MySQL stores SQL Views but not queries; to save a script (Figure
2-22, p. 73): click "Save the script to a file", browse to
Documents\MySQL Workbench\Schemas\DBP-e16-Cape-Codd-Database, type
SQL-Query-CH02-01, Save. Rerun with File | Open SQL Script, open the .sql
file, and execute. Workbench stores files in Documents by default, with a
subfolder per MySQL database recommended.

**The patterns worth memorizing.** Only Access stores queries inside the
database, because it is a personal database with an application generator;
the three enterprise class products store SQL Views but not queries, so
queries live in .sql script files. The three enterprise tools all use a
folder named DBP-e16-Cape-Codd-Database, under different parent paths. The
database name changes by product: Cape-Codd.accdb is the Access file name,
Cape_Codd is SQL Server, Cape Codd is the generic Oracle name with a
Cape-Codd-Database connection, cape_codd is MySQL. The run button is Run in
Access, Execute in SQL Server Management Studio and SQL Developer, and
Execute the statement under the keyboard cursor in Workbench. The left pane
is the Navigation Pane in Access, the Object Browser in SQL Server, the
Connections object browser in Oracle, and the Navigator in MySQL. Parse in
SQL Server Management Studio is the only syntax check separate from running
that the section gives; the other three go straight from typing to running
(added). The book's environment for the enterprise walkthroughs is Windows
10, with terminology that may vary on Windows 7, Windows 8.1, Microsoft
Server 2016 or 2019, or Linux.

**The professor's Workbench.** The course runs on MySQL, and the demos show
the same tool from a working seat, with a few habits the book does not
mention. After connecting to the server, click "create a new SQL tab for
executing queries", which opens the SQL query pane. Pick the database one
of two ways: double-click it in the schema list, and the name turns bold to
confirm it is the active schema, or run USE followed by the name and a
semicolon, which turns the name bold too. The demo database follows the
pattern username plus demo. Run a statement with the lightning bolt button
or with Control + Enter, which is what the demos use. The results pane
shows the rows, and the output area under it reports how many came back; in
the demo the customer table returned 91 rows, which is the fast way to learn
a table has 91 records. Naming columns instead of using the asterisk gives a
more focused result and in some cases runs faster than pulling every
column.

---

## 7. A working example, end to end

Here is the whole framework on the music store table, the way the demo
builds it (added, as a walkthrough; every rule in it is from section 5).

Pick the database and confirm it went bold:

    USE harbor_demo;

Look at everything:

    SELECT  *
    FROM    INSTRUMENT;

The output area reports the row count, say 40 rows, so INSTRUMENT holds 40
records. Narrow to the columns that matter, in the order you want them:

    SELECT  InstrumentName, Vendor
    FROM    INSTRUMENT;

Swap the two names and the two output columns swap, with the same 40 rows
underneath. Label it for later, with either comment style:

    /* *** SQL-Query-HM-01 *** */
    -- every instrument with its vendor
    SELECT  InstrumentName, Vendor
    FROM    INSTRUMENT;

Nothing in the comments reaches the server, so the result is the same 40
rows. That is a complete query. Every feature from here on is a way of
trimming, filtering, or ordering those rows.

---

## 8. Trimming the result: DISTINCT and TOP

Selecting a subset of columns can return duplicate rows. The book's example
returns a result whose first two rows are identical, and the professor's
demo shows the same thing by selecting only the order ID column from an
order detail table: each order ID repeats, because in that table one order
ID is spread across differing product ID, unit price, and quantity values.
That makes it a line item table, where the order ID alone does not identify
a row (added). Duplicates appear because the columns that made the
underlying rows unique were left out of the SELECT list (added).

The **SQL DISTINCT keyword** removes duplicate rows from the query result.
It goes immediately after SELECT, before the column list:

    SELECT DISTINCT Vendor, Section
    FROM   INSTRUMENT;

DISTINCT applies to the entire selected column list, not to one column; two
rows count as duplicates only when they match on every named column
(added). It removes duplicates from the view only and deletes nothing from
the table, and the professor's use for it is to see how many unique values
a column holds. Duplicates are not removed by default because doing so is
time consuming: every row must be compared with every other row, and with
100,000 rows in a table that check takes a long time. Removal is always
available on demand.

The **SQL TOP {NumberOfRows} function** displays only the first n rows of
the result, and the **SQL TOP {Percentage} PERCENT function** displays only
the first given percentage of the rows. Both sit in the same slot as
DISTINCT, after SELECT and before the column list:

    SELECT TOP 5 Vendor, Section
    FROM   INSTRUMENT;

    SELECT TOP 75 PERCENT Vendor, Section
    FROM   INSTRUMENT;

TOP is SQL Server only. Oracle Database and MySQL use a LIMIT clause for the
same effect as TOP {NumberOfRows}, and the book names Oracle Database for
the LIMIT equivalent of TOP {Percentage} PERCENT. In the chapter's results
TOP 5 displays the first five of the eight rows, and TOP 75 PERCENT displays
the first ten of the 13 rows. TOP does not remove duplicates: without
DISTINCT the trimmed result can still hold identical rows.

DISTINCT and TOP only trim a result. The real power for controlling which
rows come out of a SELECT is the WHERE clause.

---

## 9. Choosing rows: the WHERE clause

The **WHERE clause** filters the records in a query, retrieving only the
rows that meet the specified conditions. The professor calls it the search
function of the query. A condition is typically a column name, then a
comparison operator, then a value, and the clause goes after FROM:

    SELECT  *
    FROM    INSTRUMENT
    WHERE   Section = 'Brass';

The **SQL comparison operator** is the operator in a WHERE condition that
compares a column against a value; the equal sign is one. SQL does not
require that the column tested in WHERE also appear in the SELECT list, so
filtering on Section while selecting only InstrumentName is legal. Naming
specific columns and filtering with WHERE restricts both dimensions at
once: certain columns, certain rows.

**Quoting.** Text and date comparison values go in single quotation marks;
numeric values take none, and no comma is written inside the number, so
200000, not 200,000. The professor's version: string and VARCHAR values go
in single quotes, numbers do not, where **VARCHAR** is the variable length
character string type (added definition). The quotes must be the plain,
nondirectional single quotes a basic text editor produces. The curly
directional quotes many word processors insert give a syntax error, which
is the usual failure after pasting a query out of one.

**Dates.** Normally a date takes single quotes like any string:

    WHERE   DateOnWebSite = '01-JAN-2020';

Microsoft Access 2019 requires the # symbol instead, as in #01/01/2020#.
The single quoted form works as written in Oracle Database, and in MySQL
the date constant must be written '2020-01-01'. Oracle and MySQL 8.0 have
further date idiosyncrasies, covered in Chapters 10B and 10C.

**Case.** The professor's demo adds a rule the book does not state:
comparisons against the data are case sensitive. Searching the customer
table for the city Berlin returns a matching row; searching for BERLIN in
capitals returns zero rows. The value in the WHERE clause has to be typed
the way it is stored in the table. Keyword case never matters; data case
does.

**The operator table.** Figure 2-23, p. 76, lists the comparison operators
and their meanings, and this is the table to memorize:

| Operator | Meaning |
|---|---|
| = | Is equal to |
| <> | Is NOT Equal to |
| < | Is less than |
| > | Is greater than |
| <= | Is less than OR equal to |
| >= | Is greater than OR equal to |
| IN | Is equal to one of a set of values |
| NOT IN | Is NOT equal to one of a set of values |
| BETWEEN | Is within a range of numbers (includes the end points) |
| NOT BETWEEN | Is NOT within a range of numbers (includes the end points) |
| LIKE | Matches a sequence of characters |
| NOT LIKE | Does NOT match a sequence of characters |
| IS NULL | Is equal to NULL |
| IS NOT NULL | Is NOT equal to NULL |

The professor's lecture names most of them in words: equal, not equal, less
than, greater than, IN and NOT IN for one or more values, BETWEEN and NOT
BETWEEN, LIKE for a sequence of characters, IS NULL and IS NOT NULL. Two
symbols for not equal are in common use, <> and != (added); the book's
figure lists <>.

**Numbers.** Numeric columns take the operators directly with no quotes.
The demo filters the product table on unit price greater than 25 to return
every product above that price:

    SELECT  *
    FROM    INSTRUMENT
    WHERE   ListPrice > 25;

**Ranges.** BETWEEN filters on a range and is inclusive: both endpoint
values are returned, which is the detail worth remembering. For an
exclusive range, use greater than and less than instead; in the demo,
swapping BETWEEN 25 AND 50 for the strict operators drops the rows priced
at exactly 25.

    SELECT  *
    FROM    INSTRUMENT
    WHERE   ListPrice BETWEEN 25 AND 50;

**Patterns.** LIKE replaces the equals sign when you want a pattern match
rather than an exact match. The SQL wildcard character is the percent sign,
which stands for any sequence of characters, and it can go anywhere in the
pattern. On the right it means "starts with"; in front it means "ends
with"; on both ends it matches the term anywhere inside the value (added),
and a pattern with no wildcard matches only an exact value (added).

    SELECT DISTINCT Vendor
    FROM   INSTRUMENT
    WHERE  Vendor LIKE 'H%';

    SELECT  *
    FROM    INSTRUMENT
    WHERE   InstrumentName LIKE '%n';

The demo's two runs: cities ending in B returned no rows, since a trailing
letter in a stored city name is most likely lowercase, and cities ending in
n returned Berlin, London, and others. Adding more literal characters
before the wildcard narrows the match, and DISTINCT on top of LIKE lists
each matching value once. The other SQL wildcard is the underscore, which
matches exactly one character (added).

---

## 10. Ordering rows: ORDER BY

Without ORDER BY, the row order an SQL statement produces is arbitrary,
determined by programs inside the DBMS. The professor's unsorted price
filter came back looking disorganized, which is the setup for the clause.
**ORDER BY** sorts the result set of a query by one or more columns, in
either ascending or descending order. It is added to the SELECT/FROM/WHERE
framework and comes last:

    SELECT  *
    FROM    INSTRUMENT
    WHERE   ListPrice > 25
    ORDER BY ListPrice;

Ascending is the default sorting order, always. The **SQL DESC keyword**,
placed after a column name in ORDER BY, sorts that column descending. ASC is
the ascending keyword and is optional, since ascending is already the
default. To find the most expensive rows, drop the WHERE clause and sort
descending:

    SELECT  *
    FROM    INSTRUMENT
    ORDER BY ListPrice DESC;

Sort on two columns by naming both. The first column is the outer sort and
the second orders rows within each value of the first, so reversing the two
names reverses which sort is the outer one and gives a different result.
Each column carries its own direction, and DESC on one does not change the
others (added). These two are equivalent:

    ORDER BY Vendor DESC, ListPrice ASC;
    ORDER BY Vendor DESC, ListPrice;

The demo's version sorts supplier ID ascending and, within each supplier,
unit price descending, and it notes that the result grid in Workbench can
sort too, ascending or descending. Do the sorting with SQL
anyway, because the labs ask for the SQL. One display detail: Microsoft
Access shows dollar signs in the output of currency data, and the SQL
Server 2019 output the chapter prints does not. The chapter's results were
all generated with SQL Server 2019, and other products will be similar but
may vary a bit.

Clause order for a single table query, now complete: SELECT, FROM, WHERE,
ORDER BY.

---

## 11. Compound conditions

The chapter names five **WHERE clause options** that expand SQL's power:
compound clauses, sets of values, ranges, wildcards, and NULL values. The
Module 3 reading covers the first, and the demos have already shown ranges
(BETWEEN) and wildcards (LIKE) in action.

The **SQL logical operators** AND, OR, and NOT combine or negate the
conditions in a WHERE clause. Figure 2-24, p. 81: AND means both conditions
are TRUE, OR means one or the other or both are TRUE, NOT negates the
associated condition. The **SQL AND operator** requires that each row in
the result meets both conditions:

    SELECT  *
    FROM    INSTRUMENT
    WHERE   Section = 'Brass'
      AND   Vendor = 'Northwind Instruments';

The **SQL OR operator** requires that each row meets one or the other or
both:

    SELECT  *
    FROM    INSTRUMENT
    WHERE   Section = 'Brass'
      OR    Section = 'Percussion';

The layout convention puts the second and later conditions on their own
lines with the logical operator leading the line. An OR chain across
several values of one column is the long form of IN from Figure 2-23
(added): the OR query above and WHERE Section IN ('Brass', 'Percussion')
ask the same question.

---

## 12. When it breaks

Every lab contains a couple of questions dedicated to troubleshooting, and
the professor treats it as a skill worth drilling, so one demo does nothing
but run queries that fail and read the errors.

**Where the error shows up.** The **action output** is the pane at the
bottom of the Workbench query window that logs every statement submitted.
Its fields are the timestamp, the action (the SQL command submitted), the
message, the error code, and the duration, and each row carries a red X if
the statement failed or a green check if it succeeded. Messages are often
cut off in the pane, so right-click a message and choose Copy Response to
pull the full text out; the demo's habit is to paste that text into the
editor above the query as a comment, so the whole message stays visible
while the statement is fixed. Copy the response only, not the duration.
Right-click inside the output and choose Clear when it gets long.

**The red X in the editor.** A red X appears next to a line with a syntax
error before anything is run, and a command underneath an unresolved red X
will not run. The X has to be resolved first, by fixing or commenting the
flagged line. That is why pasted error text is commented: commenting clears
the X and unblocks the statements under it.

**The four failures, in demo order.** USE simple; fails with the message
unknown database simple, because that database does not exist; USE demo;
returns the green check. A SELECT against categories fails because the
table does not exist; the table is category, singular, and correcting the
name returns rows. A filtered query against category, a LIKE search for
produce with a wildcard on each side, draws the red X before the run and a
long syntax error after it, because the WHERE clause never names a column
to compare against, LIKE has a pattern but no left operand; supplying the
category name column clears the X and returns rows. Adding cat ID to the
select list fails with unknown column, because the real column is category
ID; removing or replacing the bad name resolves it, and the demo ends with
category ID appearing twice, two columns of the same data and no error,
because selecting a column already in the result set is legal.

On the music store, the same four shapes (added): USE harbor_demmo; is an
unknown database; SELECT \* FROM instruments; is a table that does not
exist; WHERE LIKE '%horn%' has no column; SELECT InstrumentID, InstName
FROM INSTRUMENT; is an unknown column.

**The workflow.** Read the action output first, because it names what
failed: unknown database, table does not exist, unknown column, in each
case a name in the statement that resolved to nothing on the server. Copy
the full message out before editing. Clear the red X before rerunning
anything below it. Look the error code up online if the message is not
enough. Clear the action output when it gets noisy so the next failure is
easy to spot. And one explicit instruction from the professor: refrain
from using ChatGPT on these queries. The point is to build a foundation
through repetition and to learn to troubleshoot yourself.

---

## 13. The thread

The module is one statement, seen from five sides. First, the language:
SQL is a sublanguage, a remote control for data, standardized by ANSI and
split into DDL, DML, DCL, and DQL, and SELECT is the DQL statement that
reads without changing. Second, the data: a company records sales in an
OLTP database, an ETL process extracts, cleans, and reshapes a slice of it
into a warehouse database, and that extract, six tables with primary keys,
foreign keys, one composite key, one recursive relationship, and a few
nulls, is what the queries run against. Third, the framework: SELECT picks
columns, FROM picks tables, WHERE picks rows, only the first two are
required, every statement returns one table, and a semicolon ends it.
Fourth, the plumbing: the same text runs in Access, SQL Server, Oracle, and
MySQL, with different buttons and different homes for a saved query, and in
the professor's Workbench it is a bold schema name and Control + Enter.
Fifth, the shaping: DISTINCT and TOP trim, WHERE filters with fourteen
comparison operators and three logical ones, ORDER BY sorts, and when a
name in the statement resolves to nothing on the server, the action output
says so and the red X holds everything below it until you fix it.

Module 2 ended on the key, the thing that turns a pile of tables into data
plus relationships. Module 3 begins the language that walks those keys,
one table at a time. The statements that process multiple tables come at
the end of the chapter.

---

*Authored and directed by **DatJavaClass (Victor S)**, who conceived,
structured, formatted, fact-checked, and edited these notes, with assistance by
Claude. Some material may have been derived from assigned material, but has not
been copied verbatim. For source materials please contact CMPINF-2110 Faculty
and Assistants.*

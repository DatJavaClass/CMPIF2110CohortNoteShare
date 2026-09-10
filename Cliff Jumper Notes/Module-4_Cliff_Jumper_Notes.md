# Module 4: Cliff Jumper Notes

*One continuous lesson stitched from the six Module 4 Part 1 cliff notes of
Managing, Querying, and Preserving Data (CMPINF-2110): how a SELECT statement
stops returning rows and starts returning answers. The five built in
aggregate functions and what each one counts, how to name the result, what
COUNT does with duplicates and NULLs, the two things an aggregate will not
do, arithmetic and string expressions in the SELECT list and the WHERE clause,
then GROUP BY, HAVING, and the order the clauses are applied in, closing with
the professor's own aggregation demo in MySQL Workbench. Every worked query
here is the notes' own (a bike shop's six tables) and every result was run;
the book's and the professor's own queries appear only as what the source
did. Definitions and additions beyond the sources are marked "(added ...)".
Part 2 of the module will extend this lesson when its notes land.*

---

## 1. Rows in, one number out

Module 3 ended with a SELECT statement that could pick columns, pick rows,
and sort them (added bridge from the Module 3 notes). Every result was still a
set of rows copied out of a table. Module 4 Part 1 is about the other thing a
query can hand back: a number computed from the rows. How many parts do we
stock? What did the sales add up to? What is the cheapest line on any sale?
The professor's overview calls these **aggregations**, and says they create
**data enrichments**, his term for a number computed from the rows that the
rows themselves do not state (added definition): they are how you tell a
story with the data instead of reading it row by row.

The book's name for the same thing is the **SQL built in aggregate functions**,
and it lists five standard ones in Figure 2-25 on page 88:

    COUNT(*)        count the number of rows in the table
    COUNT({Name})   count the rows where column {Name} IS NOT NULL
    SUM             the sum of all values (numeric columns only)
    AVG             the average of all values (numeric columns only)
    MIN             the minimum of all values
    MAX             the maximum of all values

Some DBMS products add more; the book stays with these five. COUNT, MIN and
MAX work on any data type. SUM and AVG want numbers.

An **aggregate function** takes a whole column of values and returns one
value from them (added definition). That single sentence explains most of
what goes wrong later in this lesson: a function that returns one value per
column cannot sit beside a column that still has one value per row, unless
something tells the DBMS how to fold those rows into groups. Hold that
thought until section 9.

## 2. The bike shop

Here is the running case for the whole lesson, six small tables belonging to
a bike shop. The book works on Cape Codd's tables and the professor's demo
works on a dataset he built for the course; the bike shop plays the same
role here, and every result below was run against it.

PART, ten parts in three categories, each with the buyer who stocks it. Buyer
is a fixed width character column, padded with blanks to fourteen characters,
which matters in section 8.

    PartNo  PartName              Category  Buyer
    1010    Road Frame, 54 cm     Frames    Mia Chen
    1020    Road Frame, 56 cm     Frames    Mia Chen
    1030    Gravel Frame, 54 cm   Frames    Mia Chen
    2010    Front Wheel, 700c     Wheels    Omar Reyes
    2020    Rear Wheel, 700c      Wheels    Omar Reyes
    2030    Inner Tube, 700c      Wheels    Priya Nair
    2040    Wheel Skewer Set      Wheels    Omar Reyes
    3010    Headlight, USB        Lights    Priya Nair
    3020    Tail Light, USB       Lights    Priya Nair
    3030    Spoke Reflector       Lights    Dale Whitfield

SALE, one row per sale with its total, and SALE_LINE, one row per part on a
sale, with the quantity, the unit price, and a stored line total.

    SaleNo  SaleTotal          SaleNo  PartNo  Qty  Price   LineTotal
    5001    860.00             5001    1020    1    700.00  700.00
    5002    140.00             5001    2030    4    8.00    32.00
    5003    215.00             5001    3020    2    64.00   128.00
    5004    620.00             5002    3010    2    45.00   90.00
                               5002    3030    10   5.00    50.00
                               5003    2010    1    95.00   95.00
                               5003    2020    1    120.00  120.00
                               5004    1030    1    540.00  540.00
                               5004    2030    2    8.00    16.00
                               5004    3020    1    64.00   60.00

Look at the last line. One tail light at 64.00 is stored with a line total of
60.00. Nobody at the shop knows that yet. Section 7 finds it.

FLYER_2026, the parts that went into the spring flyer. The gravel frame was
put on the website but never made the printed pages, so its FlyerPage is
NULL.

    FlyerID  PartNo  PartName              Category  FlyerPage  DateOnWebsite
    26001    1010    Road Frame, 54 cm     Frames    2          2026-03-01
    26002    1020    Road Frame, 56 cm     Frames    2          2026-03-01
    26003    1030    Gravel Frame, 54 cm   Frames    NULL       2026-05-15
    26004    2010    Front Wheel, 700c     Wheels    5          2026-03-01
    26005    2020    Rear Wheel, 700c      Wheels    5          2026-03-01
    26006    2030    Inner Tube, 700c      Wheels    6          2026-03-01
    26007    3010    Headlight, USB        Lights    9          2026-03-01
    26008    3020    Tail Light, USB       Lights    9          2026-03-01

DELIVERY, six deliveries by three couriers, and STAFF, the four buyers by
name.

    DeliveryNo  SaleNo  Courier      Fee        StaffID  FirstName  LastName
    7001        5001    PedalPost    12.50      1        Mia        Chen
    7002        5002    CityCourier  6.00       2        Omar       Reyes
    7003        5003    PedalPost    9.50       3        Priya      Nair
    7004        5004    CityCourier  8.00       4        Dale       Whitfield
    7005        5001    VanLine      25.00
    7006        5004    PedalPost    14.00

The queries below are written the way the book writes them, in SQL Server
form. Where MySQL or Oracle spell something differently the lesson says so,
because the professor's demo runs in MySQL Workbench and the quiz can ask
about either.

## 3. SUM, and naming what comes back

The simplest aggregate question: what do the four sales add up to?

    SELECT  SUM(SaleTotal)
    FROM    SALE;

The answer is 1835.00, and it comes back as a table with one cell, because
the result of an SQL statement is always a table. What the cell is called is
the first small lesson. The sum is not a column of any table, so the DBMS has
no name to give it. SQL Server 2019 labels it "(No column name)". Other
products do something equivalent. The book's word for this is ugly.

The **SQL AS keyword** assigns a name to a result column. The name is
arbitrary, anything meaningful to whoever reads the result:

    SELECT  SUM(SaleTotal) AS SaleSum
    FROM    SALE;

Now the cell sits under SaleSum. The book runs the same pair of queries on
its RETAIL_ORDER table (CH02-32 and CH02-33) and gets 1235.00 both ways; only
the label changes.

## 4. Narrowing the rows, mixing the functions

An aggregate gets more useful with a WHERE clause in front of it, because
the WHERE decides which rows the function sees. The lines of one sale:

    SELECT  SUM(LineTotal) AS Sale5001Sum
    FROM    SALE_LINE
    WHERE   SaleNo = 5001;

Sale5001Sum is 860.00, which matches the SALE row for 5001, as it should.
The book's version (CH02-34) sums ExtendedPrice for order 3000 and gets
450.00.

The five functions can be mixed and matched in one statement, each with its
own alias:

    SELECT  SUM(LineTotal) AS LineSum,
            AVG(LineTotal) AS LineAvg,
            MIN(LineTotal) AS LineMin,
            MAX(LineTotal) AS LineMax
    FROM    SALE_LINE;

    LineSum   LineAvg   LineMin   LineMax
    1831.00   183.10    16.00     700.00

One row, four numbers, each computed over all ten lines. Now compare LineSum
with section 3: the sale totals add to 1835.00, the stored line totals add to
1831.00. The four dollars are the bad tail light line, and this is the first
place the data shows it. An aggregate will tell you something is wrong; it
takes an expression (section 7) to tell you where. (added note)

The book's version of this query (CH02-35, on ORDER_ITEM) returns 1180.00,
168.571428, 50.00 and 300.00. Those four numbers are course facts; the
bike shop's are the worked ones.

## 5. COUNT, and its surprises

COUNT sounds like SUM and behaves nothing like it. SUM adds the values in a
column; COUNT counts rows. It is also the only built in function whose
argument can be an asterisk:

    SELECT  COUNT(*) AS NumberOfRows
    FROM    SALE_LINE;

NumberOfRows is 10. With a column name instead of the asterisk, COUNT counts
the rows where that column holds valid data, meaning anything other than
NULL. That produces the first surprise. How many categories does the shop
carry?

    SELECT  COUNT(Category) AS CategoryCount
    FROM    PART;

CategoryCount is 10, not 3. Every one of the ten rows has a Category, so all
ten are counted; the function never asked whether the values were different.
To count the distinct values, say so with the **DISTINCT** keyword, which
tells the function to look at each different value once:

    SELECT  COUNT(DISTINCT Category) AS CategoryCount
    FROM    PART;

CategoryCount is 3. The book reaches the same lesson on SKU_DATA: CH02-37,
COUNT(Department), returns 13, the row count; CH02-38, COUNT(DISTINCT
Department), returns 3. DISTINCT can go inside any of the aggregate
functions, but COUNT is where it is most often used.

The book adds a box: this does not work with Microsoft Access, whose
ANSI-89 SQL will not take DISTINCT inside COUNT. The fix is a subquery in the
FROM clause that builds a temporary table of distinct values, and then the
outer query counts its rows:

    SELECT  COUNT(*) AS CategoryCount
    FROM    (SELECT DISTINCT Category
             FROM   PART) AS CAT;

Same answer, 3. The book points out that this subquery sits in the FROM
clause, unlike the subqueries it shows later, which sit in WHERE.

The second surprise is NULL. COUNT(column) skips NULLs, so counting a column
that has gaps counts the gaps out:

    SELECT  COUNT(FlyerPage) AS Flyer2026NumberOfParts
    FROM    FLYER_2026;

Flyer2026NumberOfParts is 7. The table has eight rows; the gravel frame's
page is NULL and is not counted. COUNT(*) on the same table gives 8, because
the asterisk counts rows, not values. The book's CH02-39 does the same on
CATALOG_SKU_2020 and gets 8 of 9 rows, the missing one being SKU 100400.

**NULL** is the marker for a value that is absent, not a zero and not an
empty string, and the aggregate functions leave it out of their arithmetic
(added definition, the book's chapter uses IS NOT NULL to test for it).

## 6. Two things an aggregate will not do

The book states two limitations, and both come straight from the definition
in section 1: an aggregate makes one value out of a column, so it has nothing
to say about individual rows.

First, except for grouping, you cannot put a plain column name beside an
aggregate function:

    SELECT  Category, COUNT(*)
    FROM    PART;

SQL Server refuses this. Its message (Msg 8120) says that the column is
invalid in the select list because it is not contained in either an
aggregate function or the GROUP BY clause. Access and Oracle Database give
equivalent messages. MySQL 8.0, the book says, will unfortunately process the
query and give a meaningless result: one arbitrary category paired with the
count of all the rows. On the bike shop that would be something like Frames
beside 10, which answers nothing. The professor's overview puts the same rule
in one line: to display a column with an aggregation you must use a GROUP
BY, and without it the statement would not work.

Second, you cannot use an aggregate function in a WHERE clause:

    SELECT  *
    FROM    SALE
    WHERE   SaleTotal > AVG(SaleTotal);

The WHERE clause operates on rows, choosing which will be displayed; the
aggregate operates on a column, computing one value from all of it. The two
do not meet. SQL Server's message (Msg 147) says an aggregate may not appear
in the WHERE clause unless it is in a subquery contained in a HAVING clause
or a select list, and the column being aggregated is an outer reference. The
book defers the fix to subqueries later in the chapter and to views in
Chapter 7; the professor's overview gives the practical rule, which is
section 10 of this lesson: to filter on an aggregation, use HAVING.

## 7. Expressions: doing arithmetic in the query

An **SQL expression** is a formula or set of values that determines the exact
results of an SQL query. The book's way of spotting one: anything that
follows an actual or implied comparison operator, equals or greater than or
less than, or that follows a comparison keyword such as LIKE or BETWEEN. The
list of three names after IN in a WHERE clause is an expression. So is a
piece of arithmetic in the SELECT list, where the equals sign is implied:
LT = Qty * Price.

Suppose the shop wants to check its stored line totals. Compute them:

    SELECT    SaleNo, PartNo, (Qty * Price) AS LT
    FROM      SALE_LINE
    ORDER BY  SaleNo, PartNo;

    SaleNo  PartNo  LT
    5001    1020    700.00
    5001    2030    32.00
    5001    3020    128.00
    5002    3010    90.00
    5002    3030    50.00
    5003    2010    95.00
    5003    2020    120.00
    5004    1030    540.00
    5004    2030    16.00
    5004    3020    64.00

The parentheses around Qty * Price are not required and do not change the
calculation; the book keeps them because they make the expression easy to
see in the query. Now put the stored value beside the computed one by adding
LineTotal to the SELECT list. Nine pairs match. The tenth does not: sale
5004, part 3020, computed 64.00, stored 60.00.

Reading ten rows by eye is fine for a bike shop and hopeless for Cape Codd,
so move the comparison into the WHERE clause. Expressions are allowed there,
as long as they contain no aggregate function:

    SELECT    SaleNo, PartNo
    FROM      SALE_LINE
    WHERE     (Qty * Price) <> LineTotal
    ORDER BY  SaleNo, PartNo;

    SaleNo  PartNo
    5004    3020

One row: the bad line, found without reading the others. The book runs the
same three queries on ORDER_ITEM (CH02-42, 43, 44). Its computed EP column
matches ExtendedPrice on every row, so its CH02-44 returns the **empty set**,
a result with no rows at all, and the book reads that as proof that all the
stored values are correct. The bike shop's version returns one row and proves
the opposite about one line. Same query, opposite news, which is the whole
point of running it.

## 8. Strings: three spellings of one join

Expressions are not only arithmetic. The other use the book shows is
character string manipulation, and its example is to combine two text
columns into one. The bike shop wants a Sponsor column that reads "buyer in
category". In SQL Server 2019 the **concatenation operator** is the plus
sign:

    SELECT    PartNo, PartName, (Buyer + ' in ' + Category) AS Sponsor
    FROM      PART
    ORDER BY  PartNo;

The book's BY THE WAY box gives the other two spellings, and the professor's
overview gives them a third time because the course runs on MySQL. Oracle
Database uses a double vertical bar:

    (Buyer || ' in ' || Category) AS Sponsor

MySQL uses the CONCAT() function, with the pieces separated by commas inside
the parentheses:

    CONCAT(Buyer, ' in ', Category) AS Sponsor

The professor adds that SQL Server takes the plus sign and Oracle, he
believes, two bars. All three produce the same result on the bike shop, and
it is not pretty:

    PartNo  PartName              Sponsor
    1010    Road Frame, 54 cm     Mia Chen       in Frames
    1020    Road Frame, 56 cm     Mia Chen       in Frames
    1030    Gravel Frame, 54 cm   Mia Chen       in Frames
    2010    Front Wheel, 700c     Omar Reyes     in Wheels
    2020    Rear Wheel, 700c      Omar Reyes     in Wheels
    2030    Inner Tube, 700c      Priya Nair     in Wheels
    2040    Wheel Skewer Set      Omar Reyes     in Wheels
    3010    Headlight, USB        Priya Nair     in Lights
    3020    Tail Light, USB       Priya Nair     in Lights
    3030    Spoke Reflector       Dale Whitfield in Lights

The gap after every name except Dale Whitfield's is the padding from section
2: Buyer is stored fourteen characters wide, and the blanks came along for
the ride. The book's result (CH02-45) has the same gaps for the same reason.
Its answer is the **RTRIM** function, which strips trailing blanks off the
right hand side of a string and works in SQL Server, Access, Oracle Database
and MySQL:

    SELECT    PartNo, PartName,
              RTRIM(Buyer) + ' in ' + RTRIM(Category) AS Sponsor
    FROM      PART
    ORDER BY  PartNo;

Now every row reads cleanly: Mia Chen in Frames, Omar Reyes in Wheels, Dale
Whitfield in Lights. In MySQL the same statement is CONCAT(RTRIM(Buyer),
' in ', RTRIM(Category)). The book stops there on purpose: string functions
vary from one DBMS to another, and it sends the reader to the product's own
documentation for the rest.

The professor's overview example is the everyday one, a first name and a
last name from a users table joined with a space between them:

    SELECT  CONCAT(FirstName, ' ', LastName) AS FullName
    FROM    STAFF;

    FullName
    Mia Chen
    Omar Reyes
    Priya Nair
    Dale Whitfield

## 9. Groups: one number per something

Back to section 6's first limitation, and to the exception it named. A plain
column can sit beside an aggregate when the query says how to fold the rows:
the **SQL GROUP BY clause** groups rows together according to common values
in the named column, and the aggregate then runs once per group. The book
calls it powerful and admits it can be difficult to understand.

The book's way in is a question from the boss: how many products from each
department were in the printed catalog? Its Figure 2-26 on page 96 shows the
CATALOG_SKU_2020 rows falling into three department groups, with one SKU
whose CatalogPage is NULL, and Figure 2-27 shows the answer laid out in a
spreadsheet. The bike shop's boss asks the same thing about the spring flyer:
how many parts from each category were printed?

    SELECT    Category, COUNT(PartNo) AS NumberOfFlyerItems
    FROM      FLYER_2026
    GROUP BY  Category;

    Category  NumberOfFlyerItems
    Frames    3
    Lights    2
    Wheels    3

Read that against the FLYER_2026 table in section 2. Frames is wrong. The
gravel frame is in the table with a NULL page, so it was grouped and counted
even though it never printed. The book's CH02-47 has exactly this error:
Water Sports shows 5 when 4 items were in the catalog, because SKU 100400 is
counted. The fix in both cases is a WHERE clause that drops the unprinted
rows before the groups are formed:

    SELECT    Category, COUNT(PartNo) AS NumberOfFlyerItems
    FROM      FLYER_2026
    WHERE     FlyerPage IS NOT NULL
    GROUP BY  Category;

    Category  NumberOfFlyerItems
    Frames    2
    Lights    2
    Wheels    3

That is the book's CH02-48, which corrects Water Sports to 4. Two things to
notice. The groups came back in alphabetical order of Category here, the way
the book's spreadsheet lists Camping, Climbing, Water Sports, though nothing
in the statement asked for that order (section 12). And the WHERE clause
sits between FROM and GROUP BY. The book says to put it there to be safe:
some DBMS products do not require that placement, but others do.

## 10. HAVING: filtering the groups

The boss's second question: which categories had three or more printed
parts? The count already exists, one per group, and section 6 said a WHERE
clause cannot test it. The **SQL HAVING clause** is the clause that can. It
applies a condition to each group after the grouping is done, and it may
contain aggregate functions, because inside a group the function is working
on the set of column values for that group:

    SELECT    Category, COUNT(PartNo) AS NumberOfFlyerItems
    FROM      FLYER_2026
    WHERE     FlyerPage IS NOT NULL
    GROUP BY  Category
    HAVING    COUNT(PartNo) > 2;

    Category  NumberOfFlyerItems
    Wheels    3

Two details from the book (CH02-49). "Three or more" is written as greater
than 2. And the HAVING clause repeats COUNT(PartNo); it does not use the
alias NumberOfFlyerItems. The professor's overview has the same rule from
the MySQL side: an aggregation in a WHERE clause is a syntax error, so to
filter on an aggregation you use HAVING, and HAVING goes at the bottom, after
the GROUP BY. His example was students with an average score greater than
90; the bike shop's is couriers whose average fee is above ten:

    SELECT    Courier, AVG(Fee) AS AvgFee
    FROM      DELIVERY
    GROUP BY  Courier
    HAVING    AVG(Fee) > 10;

    Courier    AvgFee
    PedalPost  12.00
    VanLine    25.00

CityCourier averages 7.00 and drops out. The book's rule of thumb for telling
the two clauses apart is worth memorizing in its own words: the WHERE clause
specifies which rows will be used to determine the groups; the HAVING clause
specifies which groups will be used in the final result. And because a
statement with both could in principle be read two ways, SQL removes the
ambiguity by always applying WHERE before HAVING. The order of the work is
the order of the clauses: FROM picks the table, WHERE picks the rows, GROUP
BY folds them, HAVING picks the groups, and the SELECT list is what you see
(added explanation).

## 11. More than one grouping column, and the column that has to be there

A GROUP BY can name more than one column. The book's question: how many
SKUs is each buyer in each department responsible for? Group by the
department first, then by the buyer within it, and count each combination:

    SELECT    Category, Buyer, COUNT(PartNo) AS Cat_Buyer_PartCount
    FROM      PART
    GROUP BY  Category, Buyer;

    Category  Buyer           Cat_Buyer_PartCount
    Frames    Mia Chen        3
    Lights    Dale Whitfield  1
    Lights    Priya Nair      2
    Wheels    Omar Reyes      3
    Wheels    Priya Nair      1

Priya Nair appears twice because she buys in two categories, and each
category and buyer pair gets its own row. The book's CH02-50 returns four
such pairs for Cape Codd (Cindy Lo 3, Jerry Martin 2, Nancy Meyers 2, Pete
Hansen 6).

Now the rule that trips people up, in the book's words: when using the
GROUP BY clause, any and all column names in the SELECT clause that are not
used by or associated with an SQL built in function must appear in the GROUP
BY clause. Put PartNo in the SELECT list without putting it in the GROUP BY:

    SELECT    Category, PartNo, COUNT(PartNo) AS Cat_PartCount
    FROM      PART
    GROUP BY  Category;

SQL Server refuses it with the same Msg 8120 as section 6, and the book's
explanation is the one to remember: there are many PartNo values in each
Category group, and the DBMS has no place to put them in a one row per
group result. Try to process the statement by hand and it cannot be done.
MySQL 8.0 will, unfortunately, run it and pick an arbitrary PartNo for each
group, something like Frames 1010 3, which looks like an answer and is not
one.

The professor's fix is a habit rather than a rule, and he repeats it in the
overview and in the demo: keep the plain columns on the left of the SELECT
list and the aggregations on the right, always last. Then, when you write
the GROUP BY, start from the rightmost aggregation, copy every column to its
left, and paste them into the GROUP BY. In his words, you will always be
right, you will never miss it. On the bike shop, that habit turns the demo's
sum of prices per sale into a two column version without thinking:

    SELECT    SaleNo, PartNo, SUM(Price) AS PriceSum
    FROM      SALE_LINE
    GROUP BY  SaleNo, PartNo;

Two plain columns on the left, one aggregation on the right, both columns
copied into the GROUP BY. Ten rows come back, one per sale line, each
PriceSum equal to that line's price, because each pair occurs once. Drop
PartNo from both places and the same statement sums the prices per sale:
5001 gives 772.00, 5002 50.00, 5003 215.00, 5004 612.00.

## 12. Ordering the groups

ORDER BY works on grouped results the same way it works on rows, and the
book's last query of the reading (CH02-52) uses every clause of the lesson at
once. The bike shop version excludes the spoke reflector, groups by
category, keeps only the big groups, and sorts by the count:

    SELECT    Category, COUNT(PartNo) AS Cat_PartCount
    FROM      PART
    WHERE     PartNo <> 3030
    GROUP BY  Category
    HAVING    COUNT(PartNo) > 2
    ORDER BY  Cat_PartCount;

    Category  Cat_PartCount
    Frames    3
    Wheels    4

Trace it clause by clause, because the quiz will. Without the WHERE and the
HAVING, GROUP BY Category gives Frames 3, Lights 3, Wheels 4. The WHERE
removes one Lights row, the reflector, before the groups form, so Lights
drops to 2. The HAVING then removes the Lights group itself, because 2 is not
greater than 2. What is left is sorted by the count, smallest first. The
book's CH02-52 does the same on SKU_DATA with WHERE SKU <> 302000 and
HAVING COUNT(SKU) > 1: one Climbing row is removed by the WHERE, the
Climbing department is removed by the HAVING, and Camping 3 and Water
Sports 8 come back in that order. Without the ORDER BY, the book says, the
rows would be presented in arbitrary order of Department.

One more Access box closes the reading. Microsoft Access does not properly
recognize the alias in the ORDER BY clause; it creates a parameter query and
asks for a value of the as yet nonexistent Dept_SKU_Count. The query still
runs, but it will not be sorted correctly. The book's solution is to edit the
query in the Access QBE GUI (Figure 2-28 on page 99), which produces ORDER BY
COUNT(SKU) instead of the alias. On the bike shop, ORDER BY COUNT(PartNo)
returns the same two rows in the same order.

## 13. The demo, end to end

The professor's demo runs in MySQL Workbench on the demo database he built
for the course, and it walks the same ground in his order. Here is that walk
on the bike shop, in MySQL form.

Open a new tab, look at the tables, and start with a row count. COUNT is a
function, so it takes an argument in parentheses; the asterisk means every
row:

    SELECT COUNT(*) FROM PART;          -> 10

The demo's first count was 91 rows in its customer table. Next, count a
column that repeats. SaleNo appears once per line, so counting it counts the
lines; DISTINCT inside the function counts the different values:

    SELECT COUNT(SaleNo) FROM SALE_LINE;            -> 10
    SELECT COUNT(DISTINCT SaleNo) FROM SALE_LINE;   -> 4

The demo did this on order ID in its order detail table, knowing from an
earlier demo that order ID has duplicated values. Then a sum per sale, which
needs the GROUP BY the moment SaleNo joins the SELECT list:

    SELECT    SaleNo, SUM(Price) AS PriceSum
    FROM      SALE_LINE
    GROUP BY  SaleNo;

    SaleNo  PriceSum
    5001    772.00
    5002    50.00
    5003    215.00
    5004    612.00

The demo comments the GROUP BY out and runs it again to show the error
Workbench reports. His rule follows: it works without a column, but if
another column is there that is not part of an aggregation, you need to
group by that column. Adding a second column means adding it to the GROUP
BY too, which is the two column statement from section 11.

Then averages. The demo takes the average freight per shipper on its sales
order table; the bike shop takes the average fee per courier:

    SELECT    Courier, AVG(Fee) AS AvgFee
    FROM      DELIVERY
    GROUP BY  Courier;

    Courier      AvgFee
    CityCourier  7.00
    PedalPost    12.00
    VanLine      25.00

And the last step of the demo, the minimum and the maximum in the same
statement:

    SELECT    Courier, AVG(Fee) AS AvgFee, MIN(Fee) AS MinFee,
              MAX(Fee) AS MaxFee
    FROM      DELIVERY
    GROUP BY  Courier;

    Courier      AvgFee  MinFee  MaxFee
    CityCourier  7.00    6.00    8.00
    PedalPost    12.00   9.50    14.00
    VanLine      25.00   25.00   25.00

VanLine's three numbers agree because it made one delivery: the average, the
minimum and the maximum of a single value are that value. That is the demo,
and it is section 4 through section 11 in five statements.

## 14. The thread

A SELECT statement that returns rows became a statement that returns
answers. Five functions fold a column into one value, AS names the value,
WHERE narrows the rows the function sees, and COUNT has two habits to
remember: it counts rows rather than values unless you say DISTINCT, and it
counts out the NULLs when you give it a column. An aggregate cannot sit
beside a plain column and cannot sit in a WHERE clause, and both limits have
the same cause, one value per column against one value per row. Expressions
put arithmetic and string joins into the SELECT list and the WHERE clause,
and the concatenation operator has three spellings across the products the
course uses. GROUP BY is the exception that lets a column and an aggregate
share a SELECT list, HAVING is the filter that can see the aggregate, WHERE
is always applied before HAVING, and every plain column in the SELECT list
has to be in the GROUP BY. The bike shop's bad tail light line, the frame
that never printed, and the courier with one delivery each showed one of
those rules doing its work. Part 2 of the module picks the thread up from
here.

---

*Authored and directed by **DatJavaClass (Victor S)**, who conceived,
structured, formatted, fact-checked, and edited these notes, with assistance by
Claude. Some material may have been derived from assigned material, but has not
been copied verbatim. For source materials please contact CMPINF-2110 Faculty
and Assistants.*

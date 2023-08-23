---
layout: post
title: Azure SQL Best Practices
date: 2023-02-01 10:25
category: Best practices
tags: ["best practices"]
---

# Azure SQL Best Practices

## Where should business rules go ?
Business rules should go into the application, not in the database.
All data manipulation should be pushed to the database.

Moving data is a heavy, complex, resource-consuming task. 
Moving computing logic is much easier, as it is way more compact and lightweight.

## Naming
Objects, like tables, have their own names like "Invoices" for instance.
Since the same name could be used in a different context, say a different schema, it is a good practice to prefix the object name with the schema it belongs to.

## Unicode strings
The capital N prefix tells Azure that a string is a Unicode string.

e.g.: N'%red shirt%'

## Merging data
Merge is a command that allows you to execute inserts, updates, and deletes at the same time, so that one set of data can be merged into an existing one. 

The table that contains the data you want to merge into another table is your source table, while the other one is the target table. 

By merging a source table into a target table, this is what may happen, depending on what you specify in the MERGE command: 
- All rows that exist in the source but not in the target will be inserted in the target table. 
- All rows that exist in both tables will be updated in the source table. 
- All rows that exist in the target table and not in the source will be deleted from the target table. I said “may” as you are in total control of if and how the insert, update, and delete operations will happen and if they will happen at all:

```SQL
MERGE INTO
 [Warehouse].[Colors] AS [target]
USING
 (VALUES
 (50, 'Deep Sea Blue'),
 (51, 'Deep Sea Light Blue'),
 (52, 'Deep Sea Dark Blue')
 ) [source](Id, [Name])
ON
 [target].[ColorID] = [source].[Id]
WHEN MATCHED THEN
 UPDATE SET [target].[ColorName] = [source].[Name]
WHEN NOT MATCHED THEN
 INSERT ([ColorID], [ColorName], [LastEditedBy]) VALUES ([source].Id,
[source].[Name], 1)
WHEN NOT MATCHED BY SOURCE AND [target].[ColorID] BETWEEN 50 AND 100 THEN
 DELETE
;
```

- **WHEN MATCHED** – If there is a match between rows in source and target table, the code will update the ColorName column in the target table using the related value from the Color column in the source table. 
- **WHEN NOT MATCHED** – If the target table doesn’t have any rows with a ColorID that exists also in the source table, those rows will be taken from the source and inserted into the target. 
- **WHEN NOT MATCHED BY SOURCE** – If the target table contains some rows with a ColorID that doesn’t exist in the source table, those will be deleted from the target. The sample is adding an additional predicate to better define the scope. Not only must there be ColorID in the target that doesn’t exist in the source, but also the ColorID values must be between 50 and 100. This means that all rows in the source table that have ColorID values from 0 to 49, for example, will not be deleted as outside the scope of the defined rule.

## Concurrent updates output
Usually when you update some table, then you want to display the data right after.
What happens in the meantime if another connection alters the data you want to display?
→ you think you broke something or that your update is flawed.
One solution to this is to take advantage of the `OUTPUT` clause.

e.g.:
```SQL
UPDATE 
	[Warehouse].[Colors] 
SET 
	[ColorName] = 'Unknown' 
OUTPUT 
	[Inserted].[ColorID],
	[Inserted].[ColorName],
	[Inserted].[LastEditedBy],
	[Inserted].[ValidFrom],
	[Inserted].[ValidTo], 
WHERE
	[ColorID] = 99;
```


## Identity VS Sequence
Everyone knows about primary keys. The most simple one is an auto-incremented INT that can be created using identity OR sequence. But what's the difference?

- Identity **belongs to the table**.
- Sequence **belongs to the database**.

This means that if you use identities across your tables, they will be unaware of the other identities in use.

This means you could have duplicate PKs, which is most of the time fine.

Here's a scenario where sequence can be useful:

> I have a form to create a person, this person can be either a customer or a standard user.
> I want them in separate tables and to be unique so that I can join them to the Order table.

This could be achieved using a column specifying the "type" of person. But doesn't respect the original need of having these in separate columns.

Here's the final output:

```SQL
SELECT * FROM Customers;
```

| CustomerId | CustomerName | Gender |
| ---------- | ------------ | ------ |
| 1          | Toto         | Male   |
| 2          | Lea          | Female |


```SQL
SELECT * FROM Users;
```

| UserId | UserName | Gender |
| ------ | -------- | ------ |
| 3      | Bob    | Male   |
| 4      | Chanel | Female |
| 5      | John   | Male   |

To achieve this we can use a sequence instead:

```SQL
CREATE SEQUENCE [dbo].[SequenceObject]
AS INT
START WITH 1
INCREMENT BY 1
```

You can then manually control the sequence.

```SQL
INSERT INTO Customers VALUES
   (NEXT VALUE for [dbo].[SequenceObject], 'Toto', 'Male')
INSERT INTO Customers VALUES
   (NEXT VALUE for [dbo].[SequenceObject], 'Lea', 'Female')

INSERT INTO Users VALUES
   (NEXT VALUE for [dbo].[SequenceObject], 'Bob', 'Male')
INSERT INTO Users VALUES
   (NEXT VALUE for [dbo].[SequenceObject], 'Chanel', 'Female')
INSERT INTO Users VALUES
   (NEXT VALUE for [dbo].[SequenceObject], 'John', 'Male')
```

To achieve the same autoincrement behavior that **IDENTITY** has, you can specify a default value at table creation like so:

```SQL
CREATE TABLE [MyTable]
(
    [ID] [bigint] PRIMARY KEY NOT NULL DEFAULT (NEXT VALUE FOR [dbo].[SequenceObject]),
    [Title] [nvarchar](64) NOT NULL
);
```

## Multiple grouping

Azure SQL provides a way to calculate directly this type of matrix:

![](static/posts/grouped-set-matrix.png)

Here's our sample data:

**Supplier table**
| ID  | Name      |
| --- | --------- |
| 1   | Schneider |
| 2   | Legrand   |
| 3   | Rexel          |

**Color table**

| ID  | Name |
| --- | ---- |
| 1   | Blue |
| 2   | Red  |
| 3   | Yellow     |

| ID  | Name | Quantity | SupplierID | ColorID |
| --- | ---- | -------- | ---------- | ------- |
| 1   | Bolt | 5        | 1          | 1       |
| 2   | Bolt | 10       | 1          | 2       |
| 3   | Bolt | 15       | 1          | NULL    |
| 4   | Bolt | 50       | 2          | 1       |
| 5   | Bolt | 100      | 2          | 2       |
| 6   | Bolt | 150      | 1          | NULL    |
| 7   | Bolt | 500      | NULL       | 1       |
| 8   | Bolt | 1000     | NULL       | 2       |
| 9   | Bolt | 1500     | NULL       | NULL    |
| 10  | Bolt | 500      | NULL       | 1       |
| 11  | Bolt | 1000     | NULL       | 2       |
| 12  | Bolt | 1500     | NULL       | NULL    |
| 13  | Rope | 7        | 3          | 3       |
| 14  | Rope | 30       | 3          | 2       |
| 15  | Rope | 37       | 3          | NULL    |
| 16  | Rope | 70       | 2          | 3       |
| 17  | Rope | 300      | 2          | 2       |
| 18  | Rope | 370      | 3          | NULL    |
| 19  | Rope | 700      | NULL       | 3       |
| 20  | Rope | 3000     | NULL       | 2       |
| 21  | Rope | 3700     | NULL       | NULL    |
| 22  | Rope | 700      | NULL       | 3       |
| 23  | Rope | 3000     | NULL       | 2       |
| 24  | Rope | 3700     | NULL       | NULL    |

```SQL
SELECT
[SupplierID],
[ColorID],
COUNT(*) AS NumberOfRows,
SUM(Quantity) as ProductsQuantity,
GROUPING(ColorID) as IsAllColors,
GROUPING(SupplierID) as IsAllSuppliers
FROM
[test].StockItems
GROUP BY
GROUPING SETS
(
	([SupplierID], [ColorID]),
	([SupplierID]),
	([ColorID]),
	()
)
ORDER BY IsAllColors, IsAllSuppliers;
```

And the result, with rows highlighted according to the aforementioned matrix:
![](static/posts/grouped-sets-explained.png)

**IsAllColors = 1** represents the total products available for a specific supplier, no matter the color.
**IsAllSuppliers = 1** represents the total products available for a specific color, no matter the supplier.

## Window functions

Window functions are basically functions that calculate a result between the previous rows and your current row. Here's an example of a running total:

```SQL
SELECT
 [OrderLineID],
 [Description],
 [Quantity],
 SUM(Quantity) OVER (ORDER BY [OrderLineID] ROWS BETWEEN UNBOUNDED
PRECEDING AND CURRENT ROW) AS RunningTotal
FROM [Sales].[OrderLines] 
WHERE [OrderID] = 37
```
![](static/posts/running-total.png)

But what if we want to compute this for all the orders? Of course we don't want to mix lines from different orders. To do so you need to use the `PARTITION` instruction, like so:

```SQL
SELECT
 [OrderID],
 [OrderLineID],
 [Description],
 [Quantity],
 SUM(Quantity) OVER (
 PARTITION BY [OrderID]
 ORDER BY [OrderLineID] ROWS BETWEEN
 UNBOUNDED PRECEDING AND
 CURRENT ROW
 ) AS RunningTotal
FROM
 [Sales].[OrderLines]
WHERE
 [OrderID] in (37, 39);
```

![](static/posts/window-function-partition.png)

## Bulk copy
Calling the BCP utility is much faster, even on python. Exploit it whenever you can.

Here's some Microsoft benchmark:
-   We have 5 CSV files with  111.100.000 and around 22 columns (20 varchar(6) and 2 int data type columns).

1. Using executemany
	1. 40 mins of execution
	2. CPU DB usage = 70/80%
	3. LOG IO usage = 60/70%
	4. 3.4M records inserted every minute.
2. Calling BCP
	1. 20 mins of execution
	2. CPU DB usage = 5/10%
	3. LOG IO usage = 80/90%
	4. 7M records inserted every minute.

## Temporary tables
Temporary tables can be used when you need to store temporary results. They are destroyed with a DROP command **or until it goes out of scope**.
For a **Stored Procedure** or a **Trigger**, the scoped is the lifetime of the function that created the temporary table.

To create a temporary table, just prefix the name with a `#`.

e.g.:
```SQL
CREATE TABLE #temptable
(
	[ID] INT NOT NULL,
	[Name] NVARCHAR(20) NOT NULL
)
```

## Views
Also known as virtual tables, it's nothing more than a query definition, labeled with a name and usable as a table.
Their purpose is quite obvious. Instead of writing a long query each time, you can just save it and query the underlying table each time like so:

```SQL
CREATE OR ALTER VIEW [Sales].[OrderLinesRuninngTotal]
AS
SELECT
 [OrderID],
 [OrderLineID],
 [Description],
 [Quantity],
 SUM(Quantity) OVER (
 PARTITION BY [OrderID]
 ORDER BY [OrderLineID] ROWS BETWEEN
 UNBOUNDED PRECEDING AND
 CURRENT ROW
 ) AS RunningTotal
FROM
 [Sales].[OrderLines];
```
And then to query it:

```SQL
SELECT
 OrderID,
 [OrderLineID],
 [Description],
 [Quantity],
 [RunningTotal]
FROM
 [Sales].[OrderLinesRuninngTotal];
```

Azure SQL will expand the body of the query at runtime.

## Functions and Procedures
Procedures just like functions are used to pass parameters to your queries. Procedures have more benefits and no limitations compared to functions so they should be your favored choice.


Here's a simple example:

```SQL
CREATE PROCEDURE [test].testProcedure
@myParameter INT
AS
SELECT @myParameter;
```

To use it just call it with EXEC like so:

```SQL
EXEC [test].testProcedure 10;
```

But the interesting thing is that procedures also accept **entire TABLES**. Yes you can pass tables as parameters.

Here's how to do it:

```SQL
/* First declare the table type */
CREATE TYPE [test].MyTableType AS TABLE
(
	Name NVARCHAR(100) NOT NULL UNIQUE
);

/* Then use it in your procedure */
CREATE PROCEDURE [test].AddNamesWithStaticId
@someID INT
@Names [test].MyTableType READONLY
AS
INSERT INTO [test].Names SELECT @someID, Name FROM @Names
```

## Test some user privileges
You can use the following statement in T-SQL:

```SQL
EXECUTE AS USER = 'MyUser';
```

From now on, all the command will be executed as `MyUser`.
To revert back to your administrator account:

```SQL
REVERT;
```

## Computed columns
Same as views (or virtual tables) you can use computed columns that will be recalculated every time a query uses it.

e.g.:

```SQL
ALTER TABLE Sales.OrderLines
ADD Profit AS (Quantity*UnitPrice)*(1-TaxRate);
```

If some heavy calculation requires this column, you might want to permanently store the calculated values until the base column changes. To do so, add the PERSISTED instruction:

```SQL
ALTER TABLE Sales.OrderLines
ADD Profit AS (Quantity*UnitPrice)*(1-TaxRate) PERSISTED;
```

The values in the persisted column **are automatically recalculated** whenever any of the base values (Quantity, UnitPrice or TaxRate) is changed!

## Complex type columns

Azure SQL allows you to store the following types in the table columns:

- XML
- Spatial columns (points, lines, polygons...)
- CLR (.NET) columns

Here's an example that uses GeoSpatial data:

```SQL
DECLARE @g geography = 'POINT(-95 45.5)';
SELECT TOP(5) Location.ToString(), CityName
FROM Application.Cities
ORDER BY Location.STDistance(@g) ASC;
```

STDistance is a Nearest Neighbour function.

## Best practices on insertions
- For < 100 rows, use a single-parameterized INSERT command.
- For < 1000 rows, use table-valued parameters
- For >= 1000 rows use BCP.
## Data types
General recommendation is to try to make your columns as small as possible and find the minimal SQL types for a given type.
Even if cloud resources are much more important now to what we had back in the days, types affect performance of queries, and as such shouldn't be disregarded.

## Multi-model capabilities…

Azure SQL can handle NoSQL formats and others directly, this kind of model is called multi-model capabilities:

Azure SQL enables you to represent your models using the following non-relational concepts: 

- JSON that enables you to integrate your databases with a broad range of web/mobile applications and log file formats or even to denormalize and simplify your relational schema. JSON functionality also simplifies a lot of the work needed to be done by a developer to communicate with Azure SQL. You may ask Azure SQL, in fact, to return the result as JSON documents instead of a table with columns and rows, if doing so can simplify your code. 
- Graph capabilities that enable you to represent your data model as a set of nodes and edges. This structure is an ideal choice in the domains where the domain entities are organized in network structure and where you can take advantage of a specialized query language to query graph data. 
- Spatial support that enables you to store geometrical and geographical information in databases, index them using specialized spatial indexes, and use advanced spatial queries to retrieve the data. 
- XML support that enables you to store XML documents in the tables, index XML information using specialized XML indexes, query XML data using T-SQL or XQuery languages, and transform your relational data to or from XML format.

### JSON

![](static/posts/openjson.png)

```SQL
CREATE PROCEDURE dbo.AddTagsToPost
@PostId INT,
@Tags NVARCHAR(MAX)
AS
INSERT INTO dbo.PostTags
SELECT @PostId, T.[value] FROM OPENJSON(@Tags, '$.tags') T
GO

EXEC dbo.AddTagsToPost 1, '{"tags": ["azure-sql", "string_split", "csv"],
"categories": {}}';

```

`$.` corresponds to the current JSON document that is provided to the function.
So you can leverage your usual parameters and array accesses:

e.g.:
- $.info.name.firstName
- $.children[2]

You can also query directly data using some useful functions (in blue on the previous schema):

- **JSON_VALUE(json, path_to_value)**: will return a scalar value from JSON document on the specified path.
- **JSON_QUERY(json, path_to_value)**: will return a complex object or array from JSON document on the specified path
- **JSON_MODIFY(json, path_to_object, object)**: will take the JSON document provided as the first argument and locate value or object on the path specified with the second parameter, and instead of this value, it will inject the value provided as third parameter. The first argument will not be modified, and the modified JSON text will be returned. If you provide NULL value as a third parameter, the value on the path will be deleted.
- **ISJSON(json_string)**: returns 1 if the string provided as argument is valid JSON and 0 otherwise.

For XML, Azure SQL supports XPATH also.

### Graphs
Graphs are complex structures with objects (nodes) that are connected with relationships (edges). Scenarios where you would use graphs might be: 
- Social networks where you have people connected with relationships like friends, family, partners, or co-workers 
- Transportation maps where you have towns and places connected with roads, rivers, and flight lines 
- Bill-Of-Materials solutions where you have parts connected to other parts that are themselves connected to other parts and so on

Nodes and edges are represented as tables. Here's an example:

```SQL
CREATE TABLE Airport (
 AirportID int PRIMARY KEY,
 Name NVARCHAR(100),
 CityID int FOREIGN KEY REFERENCES Application.Cities(CityID)
) AS NODE
GO
CREATE TABLE Flightline (
 Name NVARCHAR(10)
) AS EDGE;
```

In order to explicitly specify that Flightline connects Airport, we need to introduce the following edge constraint:

```SQL
ALTER TABLE Flightline
ADD CONSTRAINT [Connecting airports]
 CONNECTION (Airport TO Airport)
 ON DELETE CASCADE;
```

Every Node table has a hidden column called `$NODE_ID`.
Every Edge table has a hidden relationship consisting of a pair of `$NODE_ID`.

Here's an example to load data from the Blobstorage:

```SQL
INSERT INTO Flightline ($from_id, $to_id, Name)
SELECT f.$NODE_ID, t.$NODE_ID, a.Name
FROM OPENROWSET(
 BULK 'data/flightlines.csv',
 DATA_SOURCE = 'MyAzureBlobStorage',
 FORMATFILE='data/flightlines.fmt',
 FORMATFILE_DATA_SOURCE = 'MyAzureBlobStorage') as a
 JOIN Airport f ON f.Name = a.FromAirport
 JOIN Airport t ON t.Name = a.ToAirport;

```

#### Querying graph data
To do so, we can use Cypher expressions directly into Azure SQL:

```SQL
SELECT
 src.Name, line.Name, dest.Name
FROM
 Airport src, Flightline line, Airport dest
WHERE
 MATCH(src-(line)->dest)
AND
 src.Name='Belgrade';

```

More complex results can be achieved:

```SQL
SELECT p1.name AS Person,
STRING_AGG(p2.name, '->') WITHIN GROUP (GRAPH PATH) AS FriendPath,
LAST_VALUE(p2.name)  WITHIN GROUP (GRAPH PATH) AS Friend,
COUNT(p2.Name) WITHIN GROUP(GRAPH PATH) AS TraversedLevel
FROM Person AS p1, friendOf FOR PATH,Person FOR PATH AS p2
WHERE MATCH(SHORTEST_PATH(p1(-(friendOf)->p2){1,3}))
AND p1.name = 'John
```

| Person | FriendPath         | Friend | TraversedLevel |
| ------ | ------------------ | ------ | -------------- |
| John   | Mary               | Mary   | 1              |
| John   | Mary->Alice        | Alice  | 2              |
| John   | Mary->Alice->John  | John   | 3              |
| John   | Mary->Alice->Leone | Leone  | 3              |


Here's an excellent article on this:

https://www.dataplatformcentral.com/2020/09/04/t-sql-graph-table-tips-demystifying-shortest_path-function/


### Geographical data

Azure SQL has two main base types that can be used to represent geometrical and geographical figures: 
- **GEOMETRY** type represents data in a Euclidean (flat) coordinate system. 
- **GEOGRAPHY** type represents data in a round-earth coordinate system.

Within these two types, as specified by Open Geospatial Consortium, you can create more specific types: 
- **Point** used to represent 2D places like towns 
- **LineString** and **CircularString** that can represent open or closed lines like roads or borders 
- **Polygon** and **CurvePolygon** used to represent areas like countries 
- **MultiPoint**, **MultiLine**, and **MultiPolygon** representing a set of disconnected geographical objects that logically belong together (an archipelago with a set of islands might be represented with MultiPolygon) 

In Azure SQL, once you have created a Geometry or Geography column in a table, you can use any of these types to build the shape you need.

Here's a full example:

```SQL
CREATE TABLE Airport (
 AirportID int PRIMARY KEY,
 Name NVARCHAR(100),
 Location GEOGRAPHY,
 CityID int FOREIGN KEY REFERENCES Application.City(CityID)
) AS NODE
GO
CREATE TABLE FlightLine (
 Name NVARCHAR(10),
 Route GEOGRAPHY
) AS EDGE;
```

Query:

```SQL
DECLARE @nebraska GEOGRAPHY = (
 SELECT TOP (1) Border FROM Application.StateProvinces
 WHERE StateProvinceName = 'Nebraska'
);
SELECT *
FROM FlightLine
WHERE Route.STIntersects(@nebraska) = 1;
```

**STIntersects**: determines if two shapes intersect at some place.
**STDistance**: measures the distance between 2 objects.

e.g. *find airports closest to current location*:

```SQL
DECLARE @currentLocation GEOGRAPHY = 'POINT(-121.626 47.8315)';
SELECT TOP(5) *
FROM Airports
ORDER BY Location.STDistance(@currentLocation) ASC;
```

Spatial objects also have corresponding indexes to improve perforrmance:

![Indexing spatial objects in 4x4 grid](static/posts/spatial-index.png)

You can specify the characteristics of the grid that will be created to index the spatial values.
```SQL
CREATE SPATIAL INDEX SI_Flightline_Routes
 ON Flightline(Route)
 USING GEOGRAPHY_GRID
 WITH (
 GRIDS = ( MEDIUM, LOW, MEDIUM, HIGH ),
 CELLS_PER_OBJECT = 64 );
```

As mentioned before, Azure SQL has two classes of spatial data types that are used in
different scenarios:
- Geometry data types are used to represent planar mathematical
shapes in classic 2D coordinate system.
- Geography data types are used to represent spherical objects and
shapes projected into 2D plane.

The difference between geometry and geography types is one of the most important
things that you need to understand to develop spatial applications.

**Geography types** are used to represent the objects placed in a classic **2D coordinate
system**, and you can imagine them as the objects that you could draw on the plain piece
of paper. Distances and sizes of the objects are measured the same way you would
measure distance of the objects drawn on a paper or a board. If you take a map and
want to find the shortest flight trajectory between Belgrade, Serbia, Europe, and Seattle,
Washington, US, you would probably use the straight horizontal line going via France
and the US east coast. This is geometrically the shortest line between them. However,
due to the rounded shape of the earth, the shortest trajectory (called geodesic) is going
via Iceland, Greenland, and Canada. Geography data model is considering Earth’s actual
shape and is able to find the real-world shortest distance and path.

Mapping the Earth surface to a 2D plane is the most difficult spatial problem.
Famous mathematician Carl Gauss proved in his Theorema Egregium (Latin for
"Remarkable Theorem") that spherical surfaces cannot be mapped to 2D planes without
distortion. You might notice on some maps that the territories closer to the poles such as
Greenland, Antarctica, north of Canada, and Russia might look stretched or sometime
bigger than actual. This happens due to the fact that dense coordinates closer to poles
must be “stretched” to project them in 2D coordinates. To make things even harder, the
Earth surface is not spherical nor even ellipsoid. Irregular shape of Earth and proximity
to poles force people to use different strategies of mapping to 2D plane. There are
mapping rules that preserve correct distance shortest paths, shape, and vice versa, but
in every strategy, something will be distorted. That’s why every geography object has an
associated Spatial Reference Identifier (SRID) that describes what spatial transformation
strategy is used to translate the object from the earth globe into the 2D plane. SRID
describes what coordinates are used (latitude/longitude, easting/northing), unit of
measure, where is coordinate root, and so on. Azure SQL will derive information from
SRID to compare positions of the objects.

In the following example, you can see how to transform coordinates to the
Geography line by specifying that the World Geodetic System 1984 (WGS84) with SRID
4326 is used for transformation:

```SQL
DECLARE @g GEOGRAPHY;
SET @g = GEOGRAPHY::STGeomFromText('LINESTRING(-122.360 47.656, -122.343
47.656)', 4326);
SELECT @g.STSrid;
```

Some countries need to use multiple SRID in their territory, especially if their north
and south borders are far like in Chile. In these scenarios, you would need to align SRID
before comparing positions of the geography objects. Specifying different SRID would
result in the different positions or shapes. Measuring the differences between spatial
objects with coordinates determined using different SRID would lead to wrong results.
Therefore, in Azure SQL spatial operations cannot be performed between spatial objects
with different SRIDs.
WGS84 is the most commonly used standard and is the one also used on the GPS
system in our phone or car. If you are unsure of which SRID to use, very good chances
are that WGS84 will work perfectly for you.
The good news for you is that all these complex transformations are built in into
Azure SQL. The only thing you need to do is to leverage functionalities and learn the
basic principles that will help you to understand how to use Spatial features.

## Rowstore VS ColumnStore format

![](static/posts/rowstore-columnstore.png)
*Rowstore format is optimal when a query needs to access all values from single or few selected rows (left), columnstore is optimal when the query accesses all values from single or few columns (right)*

Azure SQL is smart enough to determine column segments not necessary for your query, using the following rules:

- Column segments that belong to the columns that are not used in the query are ignored.
- Column segments that don't have the values required in the query are ignored.

![](static/posts/column-store-indexing.png)

### Clustered columnstore index
To create a Clustered Columnstore Index (CCI):

```SQL

/* At table creation */
CREATE TABLE Sales.Orders (
 <column definitions>
 INDEX cci CLUSTERED COLUMNSTORE
)

/* On existing table */
CREATE CLUSTERED COLUMNSTORE INDEX cci ON Sales.Orders
```

Creating a clustered **ColumnStore index** will change the underlying table structure. 

### Nonclustered columnstore index

**BUT Azure SQL is pretty unique in the market** as it allows also to create Nonclustered Columnstore indexes (NCCI).
It allows the co-existence of both Columnstore and Rowstore indexes on the same table.

With NCCI on top of classic tables, you are enabling Azure SQL to choose the optimal format for either transactional or analytical queries: 
- If you run some query that selects a single row or small set of rows, Azure SQL will use the underlying rowstore table. Database will leverage index seek or range scan operations that are the perfect fit for this scenario. 
- If you run some report that scans the entire table and aggregates data, Azure SQL will read information from NCCI index, leveraging column segment elimination and batch mode execution to quickly return process all table.

![](static/posts/NCCI.png)
*Classic rowstore table with additional NCCI index built on top of three columns*

```SQL
CREATE TABLE dbo.Sales (
 <column definitions>
 INDEX ncci NONCLUSTERED COLUMNSTORE (Price, State, Date)
)
```


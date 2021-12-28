# Chapter 1 - Data Sources

## Terms

* API = Application Programming Interfaces 
* SQL = Structured Query Language 
* RDBMSs = Relational Database Management Systems 
* ODBC = Open Database Connectivity 
    * uses drivers to standardize interfaces between software applications and 
    databases 
* ERD = Entity Relationship Diagram 
* Foreign key 
    * when a primary key is referenced in another table 

## Structured v. Unstructured Data 

* Unstructured = text documents, images, etc
* Structued = typically tabular (e.g., spreadsheets, database)

## Database v. Database Schema 

* Database = Collection of tables 
* Database schema = Stores table info and relationships (defines structure)

## One-to-many relationships 

* Where a unique entity only occurs in one table *once* but can have multiple entries in another 
    * e.g., patient tbl and appointments tbl 
    
## Many-to-many relationships 

* Connection between entites where records on each side of the relationship can connect 
to multiple records on the other side 
    * Junction of associated table needs to capture the paris of related rows 
    * Allows the ability to reduce the amount of redundant data stored in the database 
    
## Database normalization 

* Idea of not storing redundant data in a database 

## Dimensional Data Warehouses 

* Often contain data from multiple underlying sources 
    * may contain row and summary data 
    * Can contain historical data logs, etc
 
* Star scheme design (pg 7)
    * Divides data into facts/dimensions 
        * Facts tbl = metadata of an entity and measures 
        * Dimension tbl = property of entity you can group or "slice and dice" the fact 
        records by, get further info, etc

* Table *grain*
    * level of detail; what set of columns makes a row unique 
    
* Database roles 
    * SME's = subject matter experts 
    * DBA's = Database administrators 
    * ETL engineers = PEople who extract, transform, and load data from a source system 
    into a data warehouse

# Considerations<a name="cross-database_usage"></a>

When you work with the cross\-database query feature in Amazon Redshift, observe the considerations following:
+ Amazon Redshift supports cross\-database query on the ra3\.4xlarge and ra3\.16xlarge node types\.
+ Amazon Redshift supports joining data from tables or views across one or more databases in the same Amazon Redshift cluster\.
+ For cross\-database query transactional consistencies, all queries within a transaction on the connected database read data in the same state of the other database as it was at the beginning of the transaction\. Therefore, when a transaction has begun on the connected database, cross\-database queries follow snapshot isolation\. Doing this helps ensure that all reads made in a transaction see a consistent snapshot of the database\. Before querying a remote object, a committed snapshot of the unconnected database is taken\. Amazon Redshift uses this committed snapshot throughout the transaction to access shared objects from the unconnected database\.
+ Amazon Redshift cross\-database query with the three\-part notation doesn't support metadata table under schema information\_schema or pg\_catalog as these metadata views are specific to a database\. For applications and tools that integrate with cross\-database queries, to get metadata across databases, either use SVV\_ALL\*, SVV\_REDSHIFT\* metadata views or integrate with JDBC/ODBC drivers\.
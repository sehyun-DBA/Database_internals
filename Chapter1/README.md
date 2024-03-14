Database management systems can serve different purposes: some are used primarily for temporary hot data, some serve as a long-lived cold
storage, some allow complex analytical queries, some only allow accessing values by the key, some are optimized to store time-series data,
and some store large blobs efficiently. To understand differences and draw distinctions, we start with a short classification and overview, as this helps
us to understand the scope of further discussions. Terminology can sometimes be ambiguous and hard to understand without
a complete context. For example, distinctions between column and wide column stores that have little or nothing to do with each other, or how
clustered and nonclustered indexes relate to index-organized tables. This chapter aims to disambiguate these terms and find their precise definitions.
We start with an overview of database management system architecture (see “DBMS Architecture”), and discuss system components and their
responsibilities. After that, we discuss the distinctions among the database management systems in terms of a storage medium (see “Memory- Versus
Disk-Based DBMS”), and layout (see “Column- Versus Row-Oriented DBMS”).

These two groups do not present a full taxonomy of database management systems and there are many other ways they’re classified. For example,
some sources group DBMSs into three major categories: Online transaction processing (OLTP) databases
These handle a large number of user-facing requests and transactions. Queries are often predefined and short-lived.
Online analytical processing (OLAP) databases These handle complex aggregations. OLAP databases are often used
for analytics and data warehousing, and are capable of handling complex, long-running ad hoc queries.

Hybrid transactional and analytical processing (HTAP) 
These databases combine properties of both OLTP and OLAP stores. There are many other terms and classifications: key-value stores, relational
databases, document-oriented stores, and graph databases. These concepts are not defined here, since the reader is assumed to have a high-level
knowledge and understanding of their functionality. Because the concepts we discuss here are widely applicable and are used in most of the
mentioned types of stores in some capacity, complete taxonomy is not necessary or important for further discussion.

Since Part I of this book focuses on the storage and indexing structures, we need to understand the high-level data organization approaches, and the
relationship between the data and index files (see “Data Files and Index Files”).

Finally, in “Buffering, Immutability, and Ordering”, we discuss three techniques widely used to develop efficient storage structures and how
applying these techniques influences their design and implementation.

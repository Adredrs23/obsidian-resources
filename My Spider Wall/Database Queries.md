# NoSQL and Mongo
 
#### Modeling data in a way that data that is accessed together is stored together. When data is modeled this way, you will have better performance and transactions will not be required.

## Queries: 
1. How does NoSQL fetch data faster than SQL? Why NoSQL is faster than SQL?
2. How does NoSQL update data and what is mongoose plugin/Library?
3. What is the CAP theorem?
4. What is indexing, How improper index impacts performance ? 
5. How rebuilding indexes helps performance? How often should we rebuild indexes? How long it takes for rebuilding indexes?
6. What is statistics? How can we rebuild the index manual or automatically?
7. How does DB actually store data. Layer on existing fs or something else?
8. Mongoose is ACID compliant?
    Not Mongoose but primarily MongoDB is ACID Compliant database.
    
    As per their excerpt:  
    "Can NoSQL databases be ACID-compliant?” Well, the answer is simple: Absolutely! Not every single NoSQL database is ACID-compliant, but many are. In fact, [MongoDB is an ACID-compliant database](https://www.mongodb.com/basics/acid-transactions).
    
    However, MongoDB’s data modeling best practice suggests storing related data together in a single document using a variety of data types, including arrays and embedded documents. So, a lot of the time, ACID is not required as it is a single-document transaction.
    
    **B**asically **A**vailable, **S**oft State, **E**ventually Consistent. With BASE, availability is prioritized over consistency. This was originally developed for NoSQL databases that weren’t able to be ACID-compliant. This is no longer the case, though, and ACID is definitely the most desired. MongoDB, for example, cannot be BASE-compliant because transactions are consistent and not eventually consistent, as per the rules of BAS
 9. Do NoSQL follow strict schema? Do we have a schema?  
10. Normalization in NoSQL?
    REFERENCE: https://blog.usu.com/en-us/schema-design-and-relationship-in-nosql-document-based-databases
    Normalization (Referencing) - referencing the document and document in other collection using object ID
    Denormalization (Embedding)
    
    Embedding 
    -  Good:  relevant info in single query - no joins - atomic operations on a single document are ACID compliant
    - Bad: 16 MB doc size, 100 level of Depth
    
    Referencing 
    - Good: smaller docs, selective retrieval, no duplication
    - Bad: at least 2 queries required or populate/lookup functions required
      
     #### General recommendation for schema modelling and relationships:
	- 1-1: embedding model preferred
	- 1-Few: embedding model preferred
	- 1-Many: referencing model preferred
	- Many-Many: referencing model preferred
	  
	1. Favor embedding unless there is a compelling reason not to.
	2. Needing to access an object on its own is a compelling reason not to embed it.
	3. Avoid joins and populate (lookups) if possible, but don't be afraid if they can provide a better schema design.
	4. Arrays should not grow without bound. If there are more than a couple of hundred documents on the many sides, don't embed them; if there are more than a few thousand documents on the many sides, don't use an array of Object ID references.
    
11. How does indexing actually works?
12.  Do we need to rebuild index, on what factors and how often?
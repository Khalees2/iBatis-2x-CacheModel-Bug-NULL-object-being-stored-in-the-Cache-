# iBatis-2x-CacheModel-Bug-NULL-object-being-stored-in-the-Cache
iBatis 2x Cachemodel bug when custom type handler is used. stored object 'SERIALIZABLE_NULL_OBJECT' , retrieved object 'SERIALIZABLE_NULL_OBJECT'

iBATIS is a persistence framework which automates the mapping between SQL databases and objects in Java, .NET, and Ruby on Rails. In Java, the objects are POJOs. The mappings are decoupled from the application logic by packaging the SQL statements in XML configuration files. More information can be found on http://ibatis.apache.org/ and https://github.com/mybatis/ibatis-2 

This repository demonstrates the issue of NULL value getting stored in the cache when we try to cache the results returned by the database procedure calls.

Below are the code snippets from which issue can be reporoduced:

1. DAO layer java class which calls database procedure to fetch student details attached: JavaDaoCallSnippet.txt
2. Java SQLMapConfig class attached: JavaSQLMapConfigSnippet.txt
3. XML Mapping to which calls Database procedure attached: SQLMapConfigXMLSnippet.txt
4. Java Custom ResultHandler class: JavaStudentResultHandlerSnippet.txt

## ERROR
When DB procedure is called:


2020-07-20 16:06:05,028 DEBUG [engine.cache.CacheModel] Cache 'StudentCache': stored object 'SERIALIZABLE_NULL_OBJECT'

When DB procedure is called again:


2020-07-20 16:07:06,760 DEBUG [engine.cache.CacheModel] Cache 'StudentCache': retrieved object 'SERIALIZABLE_NULL_OBJECT'

## Framework's function returning NULL:
com.ibatis.sqlmap.engine.mapping.statement.MappedStatement#executeQueryForObject(StatementScope statementScope, Transaction trans, Object parameterObject,
    Object resultObject) throws SQLException{}

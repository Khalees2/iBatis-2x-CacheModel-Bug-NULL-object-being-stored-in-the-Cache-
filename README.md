# iBatis-2x-CacheModel-Bug-NULL-object-being-stored-in-the-Cache
iBatis 2x Cachemodel bug when custom type handler is used. stored object 'SERIALIZABLE_NULL_OBJECT' , retrieved object 'SERIALIZABLE_NULL_OBJECT'

iBATIS is a persistence framework which automates the mapping between SQL databases and objects in Java, .NET, and Ruby on Rails. In Java, the objects are POJOs. The mappings are decoupled from the application logic by packaging the SQL statements in XML configuration files. More information can be found on http://ibatis.apache.org/ and https://github.com/mybatis/ibatis-2 

This repository demonstrates the issue of NULL value getting stored in the cache when we try to cache the results returned by the database procedure calls.

Below are the code snippets from which issue can be reporoduced:

1. DAO layer java class which calls database procedure to fetch student details
 ```public class StudentDaoImpl {

	private SqlMapClient sqlMap;
	
	public StudentDaoImpl() {
		super();
		sqlMap = SQLMapConfig.getSqlMapInstance(); //this class is shown below
	}
	
	public StudentDTO getStudentDetails(String studentId,Date applicationDate) throws DataAccessException {

		StudentDTO student = null;
		Map params = new HashMap(3);
		
		params.put("studentId", studentId);
		params.put("applicationDate", applicationDate);
		params.put("result", student);

		try {
			sqlMap.queryForObject("getStudentDetails", params);
		} catch (SQLException sqle) {
			String message = "Could not retrieve Student details using id: "
				+ studentId.toString();
			log.error(message, sqle);
			throw new DataAccessException(message, sqle);
		}
		student = (StudentDTO) params.get("result");

		return student;
	}
	
}```

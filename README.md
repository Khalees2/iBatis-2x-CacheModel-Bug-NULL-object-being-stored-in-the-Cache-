# iBatis-2x-CacheModel-Bug-NULL-object-being-stored-in-the-Cache
iBatis 2x Cachemodel bug when custom type handler is used. stored object 'SERIALIZABLE_NULL_OBJECT' , retrieved object 'SERIALIZABLE_NULL_OBJECT'

iBATIS is a persistence framework which automates the mapping between SQL databases and objects in Java, .NET, and Ruby on Rails. In Java, the objects are POJOs. The mappings are decoupled from the application logic by packaging the SQL statements in XML configuration files. More information can be found on http://ibatis.apache.org/ and https://github.com/mybatis/ibatis-2 

This repository demonstrates the issue of NULL value getting stored in the cache when we try to cache the results returned by the database procedure calls.

Below are the code snippets from which issue can be reporoduced:

1. DAO layer java class which calls database procedure to fetch student details
public class StudentDaoImpl {

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
	
}

2. Java config class

public class SQLMapConfig {

	private static final SqlMapClient sqlMap;
	static {
		try {
			String resource = "test/dao/SQLMapConfig.xml";
			Reader reader = Resources.getResourceAsReader (resource);
			sqlMap = SqlMapClientBuilder.buildSqlMapClient(reader);
		} catch (Exception e) {

			e.printStackTrace();
			throw new RuntimeException ("Error initializing MyAppSqlConfig class. Cause: " + e);
		}
	}
	public static SqlMapClient getSqlMapInstance () {
		return sqlMap;
	}
}

3. Snippet from SQLMAPConfig.xml

<cacheModel id="IMAPLandParcelCache" type="MEMORY">
		<flushInterval seconds="600"/>
		<property value="10" name="size"/>
		<property name="reference-type" value="WEAK"/>
</cacheModel>


<parameterMap id="getParameters" class="map">

		<parameter property="result" javaType="test.dto.StudentDTO" jdbcType="ORACLECURSOR" mode="OUT"
			typeHandler="test.dao.StudentResultHandler"/>
		<parameter property="studentId" jdbcType="NUMERIC" mode="IN" />
		<parameter property = "applicationDate" jdbcType="DATE" mode="IN"/>
		
</parameterMap>

<procedure id="getStudentDetails" parameterMap="getParameters" cacheModel="StudentCache">
	{call pkcp_BPS.GET_STUDENT_DETAILS (?,?,?)}
</procedure>

4. ResultHandler class mentioned in the above XML

public class StudentResultHandler implements TypeHandlerCallback{

	public StudentResultHandler() {
		super();
	}
	
	
	public Object getResult(ResultGetter getter) throws SQLException {
	
	StudentDTO student = null;
	ResultSet rs = (ResultSet) getter.getObject();
	
	try {
	
	while(rs.next()) {
	//fetch rows from the resultset returned from the procedure call
	student.setName(rs.getString("name");
	}
	
	}
		finally {
			JDBCUtil.closeResultSet(rs);
		}

		return student;
	
	}
}

## ERRORS
When you call the db procedure:
2020-07-20 16:06:05,028 DEBUG [engine.cache.CacheModel] Cache 'StudentCache': stored object 'SERIALIZABLE_NULL_OBJECT'



When you call the db procedure again.
2020-07-20 16:07:06,760 DEBUG [engine.cache.CacheModel] Cache 'StudentCache': retrieved object 'SERIALIZABLE_NULL_OBJECT'

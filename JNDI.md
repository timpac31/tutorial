# 환경
> Oracle 9i, tomcat6.0

1. server.xml (tomcat)
~~~
<GlobalNamingResources>
    <Resource auth="Container" description="User database that can be updated and saved" 
       factory="org.apache.catalina.users.MemoryUserDatabaseFactory" name="UserDatabase" 
       pathname="conf/tomcat-users.xml" type="org.apache.catalina.UserDatabase"/>
    <Resource auth="Container" driverClassName="oracle.jdbc.OracleDriver" maxActive="20" maxIdle="10" maxWait="-1" 
       name="jdbc/mobile" username="user" password="pass" type="javax.sql.DataSource" 
       url="jdbc:oracle:thin:@220.11.111.11:1521:ORA9" />
    <Resource auth="Container" driverClassName="oracle.jdbc.OracleDriver" maxActive="20" maxIdle="10" maxWait="-1" 
       name="jdbc/home" username="user" password="pass" type="javax.sql.DataSource" 
       url="jdbc:oracle:thin:@220.22.222.22:1521:ORA9"/>
  </GlobalNamingResources>
~~~

2. context.xml
~~~
<!-- Default set of monitored resources -->
  <WatchedResource>WEB-INF/web.xml</WatchedResource>
  <ResourceLink name="jdbc/mobile" global="jdbc/mobile" type="javax.sql.DataSource" /> 
	<ResourceLink name="jdbc/home" global="jdbc/home" type="javax.sql.DataSource" />
~~~

3. web.xml
~~~
<resource-ref>
    <description>mobile DB</description>
    <res-ref-name>jdbc/mobile</res-ref-name>
    <res-type>javax.sql.DataSource</res-type>
    <res-auth>Container</res-auth>
</resource-ref>
<resource-ref>
    <description>home DB</description>
    <res-ref-name>jdbc/home</res-ref-name>
    <res-type>javax.sql.DataSource</res-type>
    <res-auth>Container</res-auth>
</resource-ref>
~~~

4. DBConnectionMgr.java
~~~
package jdbc;

import java.sql.*;
import javax.naming.*;
import javax.sql.*;

public class DBConnectionMgr {

    static private DBConnectionMgr instance;
    
    private DBConnectionMgr() {
        init();
    }

    private void init() {
    }
    public void destroy() {
    }

    static  public DBConnectionMgr getInstance()     {
        if (instance == null) {
            instance = new DBConnectionMgr();
        }
        return instance;
    }
    
    public Connection getConnection() {
        Connection con = null;
        try{
        	InitialContext ctx = new InitialContext();
            Context envContext = (Context) ctx.lookup("java:comp/env");
            DataSource ds = (DataSource)envContext.lookup("jdbc/home");
     	    con = ds.getConnection();
        }catch(Exception e) {
            e.printStackTrace();
        }
        return con;
    }
    
    public Connection getConnection(String jndi) {
        Connection con = null;
        try{
        	InitialContext ctx = new InitialContext();
            Context envContext = (Context) ctx.lookup("java:comp/env");
            DataSource ds = (DataSource)envContext.lookup(jndi);
     	    con = ds.getConnection();
        }catch(Exception e) {
            e.printStackTrace();
        }
        return con;
    }

}
~~~

5. connection.jsp
~~~
<%@ page import="jdbc.DBConnectionMgr %>
<%
DBConnectionMgr pool = DBConnectionMgr.getInstance();
Connection conn = null;
try{
  con = pool.getConnection();
  //con = pool.getConnection("jdbc/mobile");
  out.println("connection success");
}catch(Exception e) {
  out.println("connection fail : " + e);
}
%>
~~~

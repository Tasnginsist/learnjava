# JAVAEE



## JAVA基础语法

## JAVA面向对象

## JAVA高级

### 常用API

#### String类







### 异常

### File类与IO流





### 网络编程

####  网络通信概述



#### TCP协议

### JDK1.8新特性

# JDBC

## 步骤：

1.导入jar包

2.注册驱动

3.获取数据库连接对象

4.定义sql

5.获取执行sql语句的对象statement

6.执行sql，接收返回结果

7.处理结果

8.释放资源

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

import static java.lang.Class.*;

public class Demo {
    public static void main(String[] args) throws Exception {
        //注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        //获取连接
        Connection localConn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","12345678");
//        定义sql
        String sql = "update student set sex = \"女\" where id = 901";
        //获取statement独享
        Statement statement = localConn.createStatement();
        //执行sql
        int cout = statement.executeUpdate(sql);
        System.out.println(cout);
        //关闭资源
        statement.close();
        localConn.close();
    }
}
```



## DriverManager：驱动管理对象

功能：1.注册驱动

​            2.获取数据库连接

## Connection：数据库连接对象

功能：1.获取执行sql的对象

​            2.管理事务：开启事务：setAutoCommit(boolean b)

​                                  提交事务：commit()

​                                  回滚事务:rollback()

## Statement：执行sql对象

1.boolean execute():可以执行任意sql

2.int executeUpdate(String sql):执行DML(增删改)、DDL(create\alter\drop)语句，返回值是影响的行数，可以通过返回值判断语句是否执行成功，大于0执行成功，反之失败

3.ResultSet executeQuery(String sql):执行查询语句

## ResultSet：结果集对象,封装查询结果

1.next():游标向下移动一行，判断当前行是否是最后一行末尾（是否有数据），如果是返回false，如果不是返回true

2.getXxx(参数):获取数据

​            *xxx：为数据类型

​            *参数：int：代表列的编号，从1开始

​                         String：代表列的名称

3.使用步骤：

​      1）游标向下移动一行

​      2）判断是否有数据

​     3）获取数据

```java
import java.sql.*;

public class Demo3 {
    public static void main(String[] args) {
        Connection connection;
        Statement statement;
        try {
            String sql = "select * from user ";
            Class.forName("com.jdbc.cj.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "12345678");
            statement=connection.createStatement();
            ResultSet  resultSet = statement.executeQuery(sql);
            while (resultSet.next()){
                int id= resultSet.getInt(1);
                String uName = resultSet.getString("uname");
                int pwd = resultSet.getInt("3");
                System.out.println(uName + " " + pwd);
            }
        } catch (ClassNotFoundException e) {

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## PreparedStatement：

用来解决sql注入问题

参数使用？作为占位符

步骤：

1.导入jar包

2.注册驱动

3.获取数据库连接对象

4.定义sql，参数使用？作为占位符

5.获取执行sql语句的对象PreparedStatement：Connection.prepareStatement(String sql)

6.给？赋值：setxxx(参数1，参数2)：参数1表示？的位置编号，从1开始，参数2表示问号的值

6.执行sql，接收返回结果，不需要传递参数

7.处理结果

8.释放资源

```java
Connection connection=null;
PreparedStatement preparedStatement =null;
ResultSet resultSet = null;

try {
    connection = JdbcUtils.getConnection();
    String sql ="select * from user where uname = ? and pwd = ?";
    preparedStatement= connection.prepareStatement(sql);
    preparedStatement.setNString(1,"uName");
    System.out.println(uName);
    preparedStatement.setNString(2,"pwd");
    System.out.println(pwd);
    resultSet=preparedStatement.executeQuery();
    return resultSet.next();
} catch (SQLException e) {
    e.printStackTrace();
}finally {
   JdbcUtils.close(resultSet,preparedStatement,connection);

}
```

## JDBC工具类

```java
import java.io.FileReader;
import java.io.IOException;
import java.net.URL;
import java.sql.*;
import java.util.Properties;

public class JdbcUtils {
    private static String url;
    private static String user;
    private static String password;
    private static String driver;

    static {
        Properties properties = new Properties();
        try {
            properties.load(new FileReader("database.properties"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        url = properties.getProperty("url");
        user = properties.getProperty("user");
        password = properties.getProperty("password");
        driver = properties.getProperty("driver");

        //读取资源文件
        try {
            Class.forName(driver);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }


    }    //获取连接对象
public static Connection getConnection() throws  SQLException{


        return DriverManager.getConnection(url, user, password);


}
 //释放资源
    public static void close(Statement statement,Connection connection){
        if (statement != null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

    }
    public static void close(ResultSet resultSet,Statement statement,Connection connection){
        try {
            if (resultSet != null){
                resultSet.close();
            }
        }catch (SQLException e){
            e.printStackTrace();
        }
        if (statement != null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 数据库连接池

概念：存放数据连接的容器

好处：节约资源，用户访问高效

### C3P0数据库连接池

步骤：

1.导入jar包

2.定义配置文件：

​      *名称为c3p0。properties 或c3p0-config.xml

​      *直接将文件放在src目录下

3.创建数据库连接池对象：ComoPooledDataSource()

4.获取连接：getConnection（）

### Druid数据库连接池

步骤：

1.导入jar包

2.定义配置文件，可以为任意名称，可以放在任意目录下

3.加载配置文件

3.获取数据连接池对象：通过工厂类DruidDataSourceFactory()

4.获取连接：getConnection()

```java
import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.util.Properties;

public class DruidDemo {
    public static void main(String[] args) throws  Exception{
        Properties p =new Properties();
        InputStream resourceAsStream = DruidDemo.class.getClassLoader().getResourceAsStream("druid.properties");
        p.load(resourceAsStream);
        DataSource dataSource=DruidDataSourceFactory.createDataSource(p);
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }
}
```

#### druid工具类

```java
import com.alibaba.druid.pool.DruidDataSourceFactory;
import com.mysql.cj.protocol.Resultset;

import javax.sql.DataSource;
import java.io.IOException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JdbcUtil {
    //定义成员变量
    private  static DataSource dataSource;

    static {
        //加载配置文件
        Properties properties = new Properties();
        try {
            properties.load(JdbcUtil.class.getClassLoader().getResourceAsStream("druid.properties"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            dataSource= DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
//获取连接
    public Connection getMyConnection() throws  Exception{
        return  dataSource.getConnection();
    }
    //关闭资源
    public  void  myClose(Statement statement,Connection connection){
        if (statement != null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public void myClose(ResultSet res,Statement statement, Connection connection){
        if (res != null){
            try {
                res.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
      
        if (statement != null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    //获取连接池对象
    public DataSource getDataSource(){
        return dataSource;
    }
}
```

##  JDBCTemplate

1.update():执行增删改语句

2.queryForMap():查询结果，将结果集封装为map集合

3.queryForList():

4.query()

5.queryForObject():

# JAVAWEB

## Web相关概念

软件架构

1.C/S：客户端/服务器端

2.B/S：浏览器/服务器端

资源分类

1.静态资源：所有用户访问后，得到的结果都是一样的称为静态资源

2.动态资源：每个用户访问相同资源后，得到的结果可能不一样，称为动态资源 

网络通信三要素

1.IP

2.端口

3.传输协议

## Tomcat

## Servlet

 Servlet概念：运行在服务器端的小程序，用于接收和响应客户端的请求，本质上就是一个接口，定义了java类被浏览器访问到（tomcat识别）的规则，

servlet工作原理：

### 快速入门

1.创建JAVAEE项目

2.定义一个类，实现servlet接口

3.实现接口中的方法

4.配置servlet

```javascript
<servlet>
    <servlet-name>helloservlet</servlet-name>
    <servlet-class>com.kkb.ServletDemo</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>helloservlet</servlet-name>
    <url-pattern>/hs</url-pattern>
</servlet-mapping>
```

### Servlet执行原理

1.当服务器接收到客户端浏览器的请求后，会解析请求url路径，获取访问的servlet的资源路径

2.查找web.xml文件，是否有对应的<url-pattern>标签体内筒

3.如果有，则再找到对应的 <servlet-class>全类名

4.tomcat将字节码文件加载进内存并且创建其对象

5.调用其方法

### Servlet中的方法

1.init()：初始化方法，在servlet被创建时执行，只会执行一次

2.service()：提供服务的方法，每一次servlet被访问时执行，执行多次

3.destory方()：销毁方法，在服务器被关闭时执行，只执行一次

4.getServletConfig()：

5.getServletInfo()：获取servlet的信息

### Servlet的生命周期

1.被创建：执行init()，只执行一次

<       *Servlet什么时候被创建？

​        *默认情况下，第一次访问时被创建

​      *指定创建时间，在<servlet>标签下配置，<load-on-startup>值为负数，在第一次访问时被创建；<load-on-startup>值为0或正数，在服务器启动时创建

​    *servlet的init方法只执行一次，说明sevlet在内存中只有一个对象，servlet是单例，多个用户同时访问，可能存在线程安全问题

​     *解决：尽量不要在servlet中定义成员变量，即使定义了成员变量，也不要修改值

2.提供服务：执行service方法，执行多次

  <    * 每次访问servlet时，service方法都会被调用一次

3.被销毁：执行detory()，只执行一次

<     *servlet被销毁时执行，服务器关闭时，servlet被销毁

​       *只有服务器正常关闭时，才会执行destory方法

​       *一般用于释放资源

### Servlet3.0注解配置

步骤：1.创建JAVAEE项目，选择servlet3.0以上，可以不创建web.xml

​            2.定义类实现servlet接口

​           3.复写方法

​          4.在类上使用@WebServlet注解进行配置

​               @WebServlet("/hs2")

### Servlet体系结构

![image-20200321213316204](/Users/tang/Library/Application Support/typora-user-images/image-20200321213316204.png)

### Servlet相关配置

1.urlpartten：

​     *一个servlet可以定义多个访问路径@WebServlet({"/demo3","/demo4","/demo5"})

​     *路径定义规则

​          1./xxx:

​          2./xx/xxx:多层路径，目录结构

​          3.*.do:

## Http协议

### Http请求协议

概念： HYPER TEXT TRANSFER PROTOCO 超文本传输协议，定义了客户端和服务器通信时传输数据的格式

特点：1.基于TCP/IP的高级协议

​            2.默认端口号：80

​            3.基于请求/响应模型，一次请求对应一次响应

​            4.无状态的，每次请求之间相互独立，不能交互数据

历史版本：1.0：每一次请求响应都会建立新的连接

​                    1.1：复用连接

数据格式：请求行、请求头、请求空行、请求体

### Http响应协议

数据格式：响应行（协议/版本、响应状态码、状态码描述）

​                   响应头（头名称、值）

​                  响应空行、

​                  响应体

响应状态码：

1）状态码都是3位数字

2）1xx：服务器接收客户端消息，但没有接收完成，等待一段时间后，发送1xx状态码

​      2xx：请求成功

​      3xx：重定向，302（重定向）、304（访问缓存）

​      4xx：客户端错误，404（请求路径没有对应的资源）、405（请求方式没有对应的doxx方法）

​      5xx：服务器端错误，500（服务器内部异常）

响应头：

content-type：响应体数据格式以及编码格式

content-disposition：打开响应体数据的格式，默认值in-line（在当前页面内打开）、attachment（以附件形式打开，用于文件下载）

## Request和Response

### request对象和response对象的原理：

​    1）request对象和response对象是由服务器创建供我们使用

​    2）request对象获取请求消息，response对象用来设置响应消息

### Request继承体系

<img src="/Users/tang/Library/Application Support/typora-user-images/image-20200321223426918.png" alt="image-20200321223426918" style="zoom:50%;" />

### request功能

#### 获取请求消息

##### 获取请求行数据

1.获取请求方式：String getMethod():

2.(*)获取虚拟目录：String getCintextPath():

3.获取servlet路径：String getServletPath():

4.获取get方法请求参数：getQueryString():

5.(*)获取请求URI:String getRequestURI():

​         获取请求URL：StringBuffer getRequestURL():

URI:统一资源标识符

URL：统一资源定位符

6.获取协议及版本：String getProtocol():

7.获取客户机IP地址：String getRemoteAddr():

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/RequestDemo")
public class RequestDemo extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        1.获取请求方式：String getMethod():
        String method = request.getMethod();
        System.out.println(method);
//        2.(*)获取虚拟目录：String getCintextPath():
        String contextPath = request.getContextPath();
        System.out.println(contextPath);
//        3.获取servlet路径：String getServletPath():
        String servicePath = request.getServletPath();
        System.out.println(servicePath);
//        4.获取get方法请求参数：getQueryString():
        String queryString = request.getQueryString();
        System.out.println(queryString);
//        5.(*)获取请求URI:String getRequestURI()\StringBuffer getRequestURL()::
        String URI = request.getRequestURI();
        System.out.println(URI);
        StringBuffer URL = request.getRequestURL();
        System.out.println(URL);
//        6.获取协议及版本：String getProtocol():
        String protocol = request.getProtocol();
        System.out.println(protocol);
//        7.获取客户机IP地址：String getRemoteAddr():
        String IP = request.getRemoteAddr();
        System.out.println(IP);
    }
}
```

##### 获取请求头数据

1.通过请求头名称获取请求头的值：String getHeader(String name)

2.获取所有请求头名称：Enumeration<String> getHeaderNames()

```java
@WebServlet("/requestdemo2")
public class RequestDemo2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取所有请求头名称
        Enumeration<String> headers = request.getHeaderNames();
        while (headers.hasMoreElements()) {
            String name = headers.nextElement();
            String value = request.getHeader(name);
            System.out.println(value);
        }
        //获取请求头数据  user-agent
        String agent = request.getHeader("user-agent");
        if (agent.contains("Chrome")) {
            System.out.println("guge");
        } else if (agent.contains("Firefox")) {
            System.out.println("huohu");

        }
    }
}
```

##### 获取请求体数据

请求体：只有post请求方式有，在请求体中封装了post请求参数

步骤：1.获取流对象

​                *BufferedReader getReader():获取字符流，只能操作字符数据

​                *ServletInputStream getInputStream():获取字节流，可以操作所有类型数据

​            2.再从流对象中拿数据

```java
@WebServlet("/RequestDemo3")
public class RequestDemo3 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        BufferedReader bufferedReader = request.getReader();
        String line = null;
        while ((line=bufferedReader.readLine()) != null){
            System.out.println(line);
        }
    }
```

#### 其他功能

1.获取请求参数通用方式：

​     1）String getParameter(String name)：根据参数名称获取参数值

​     2)String[] getParameterValues():根据参数名称获取参数值的数组

​     3)Enumeration<String> getParameterNames():获取所有请求的参数名称

​     4)Map<String,String[]> getParameterMap():获取所有参数的Map集合

```java
@WebServlet("/RequestDemo3")
public class RequestDemo3 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       request.setCharacterEncoding("utf-8");
        //根据参数名称获取参数值
        String name = request.getParameter("username");
        System.out.println("post" + name);
        //根据参数名称获取参数值数组
        String[] hobbies = request.getParameterValues("hobby");
        for (String hobby : hobbies) {
            System.out.println(hobby);
        }
        //获取所有请求参数的名称
        Enumeration<String> names = request.getParameterNames();
        while (names.hasMoreElements()){
            String name2 = names.nextElement();
            String value = request.getParameter(name2);
            System.out.println(value);
        }
        //获取所有参数的map集合
        Map<String,String[]>  parameterMap = request.getParameterMap();
        Set<String> keyset = parameterMap.keySet();
        for (String name3: keyset) {
            String[] values = parameterMap.get(name3);
            for (String value:values) {
                System.out.println(value);
            }
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      this.doPost(request,response);
    }
}
```

*中文乱码问题：

​      get方式：tomcat8已解决

​      post方式：request.setCharacterEncoding("utf-8");

2.请求转发：一种在服务器内部的资源跳转方式

​      步骤：1.通过request对象获取请求转发器对象:request.getRequestDispatcher

​                  2.使用RequestDispatcher对象转发:forward(request,response)

```java
@WebServlet("/RequestDemo4")
public class RequestDemo4 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo4.......");
        request.getRequestDispatcher("/RequestDemo9").forward(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

​     转发的特点（forward）：浏览器地址栏路径不发生变化

​                只能转发到当前服务器的内部资源中

​                转发是一次请求

3.共享数据

   域对象：一个有作用范围的对象，可以在范围内共享数据

   request域：一次请求的范围，一般用于请求转发的多个资源中共享数据

​     方法：

​           1）void serAttribute(String name,Object obj):存储数据

​           2）Object getAttitude(String name):通过键获取值

​           3）void removeAttribute(String name):通过键移除值

```java
@WebServlet("/RequestDemo4")
public class RequestDemo4 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo4.......");
        request.setAttribute("msg","hello");
        request.getRequestDispatcher("/RequestDemo9").forward(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

```java
@WebServlet("/RequestDemo9")
public class RequestDemo9 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo9.....");
        Object msg = request.getAttribute("msg");
        System.out.println(msg);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

4.获取ServletContext：ServletContext getServletContext()

### Response对象

功能：设置响应行、响应头、响应体

设置状态码：setStatus(int sc)

设置响应头：setHeader(String name,String value)

设置响应体：

​     步骤：1.获取输出流

​                         *获取字符输出流：PrintWriter  getWriter()

​                        *获取字节输出流：ServletOutputStream getOutputStream()

​                 2.使用输出流将数据输出到客户端

案例：1.重定向

​            2.输出字符数据到浏览器

```java
@WebServlet("/ResponseDemo3")
public class ResponseDemo3 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      //设置编码格式
        response.setContentType("text/html;charset=ytf-8");
        PrintWriter printWriter = response.getWriter();
        printWriter.write("hello response");
    }
```

​            3.输出字节数据到浏览器

```java
@WebServlet("/ResponseDemo3")
public class ResponseDemo3 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        ServletOutputStream servletOutputStream = response.getOutputStream();
        servletOutputStream.write("hello response啊啊啊啊".getBytes());
    }
```

​            4.验证码

9

### Response的重定向

```java
@WebServlet("/ResponseDemo")
public class ResponseDemo extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo1");
        response.setStatus(302);
      //response.setHeader("location","/javaaweb/ResponseDemo2");
        response.sendRedirect("/javaaweb/ResponseDemo2");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

```java
@WebServlet("/ResponseDemo2")
public class ResponseDemo2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo2");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

重定向的特点（redirect）：地址栏发生变化

​                           重定向可以访问其他站点（服务器）的资源

​                           重定向是两次请求，不能使用request对象共享数据

绝对路径：通过绝对路径可以确定唯一资源，以/开头

相对路径：通过相对路径不可以确定唯一资源，不以/开头，以点开头

### ServletContext对象

概念：代表整个web应用，可以和服务器通信

获取：1.通过request对象获取：request.getServlerContext()

​            2.通过HttpServlet获取：this.getServletContext()

功能：1.获取MIME类型：getMimeType(String fileName)

​            <font color= 'red'>*MIME类型：在互联网通信过程中定义的一种文件数据类型</font>


​            *格式：大类型/小类型      text/html   image/jpeg

​            2.域对象共享数据

​            3.获取文件的真实（服务器）路径:String getRealPath()

## Cookie

Cookie使用步骤：

1.创建Cookie对象，绑定数据：new Cookie(String name,String value)

2.发送Cookie对象:response.addCookie(Cookie cookie)

3.获取Cookie数据:Cookie[] request.getCookies()

实现原理：

通过响应头set-cookie和请求头cookie实现



cookie细节：

1.一次可以发送多个cookie，创建多个cookie对象即可

2.默认情况下，当浏览器被关闭时，cookie被销毁；

3.cookie的持久化存储：setMaxAge(int seconds),

​        seconds=负数：默认

​        seconds=0:删除cookie

​        seconds=整数：将cookie写到硬盘文件中持久化存储

4.在tomcat8之前，cookie不能直接存储中文数据，tomcat8之后可以

5.默认情况下，cookie不能共享



cookie的特点：

1.cookie存储数据在客户端浏览器

2.浏览器对单个cookie的大小有限制（4kb），对同一个域名下的总cookie数量也有限制（20个）



cookie的作用：

1.用于存储少量的不太敏感的数据

2.在不登录的情况下，完成服务器对客户端的身份识别

## Session

概念：服务器端会话技术，在一次会话的多次请求间共享数据，将数据存在服务器端的对象中 HttpSession

方法：Object getAttribute(String name)

​           Void setAttribute(String name , Object value)

​          void removeAttribute(String name)

原理：session的实现是依赖于cookie的

注意：1.客户端关闭后，服务器不关闭，两次获得的session是同一个吗

​                    *默认情况下不是

​                   *如果需要相同，可以创建cookie，键为SESSION-ID,设置最大存活时间让cookie持久化保存

​           2.客户端不关闭后，服务器关闭，两次获得的session是同一个吗：

​                    *不是同一个，但要确保数据不丢失

​                        *session的钝化：在服务器正常关闭之前，将session对象序列化到硬盘上

​                       *session的活化：在服务器启动后，将session文件转化为内存中的session对象

​        3.session什么时候被销毁

​               *服务器关闭

​               *session对象调用invalidate()

​               *默认失效时间30分钟

session的特点：

1.session用于存储一次会话的多次请求的数据。存在服务器端

2.session可以存储任意类型、任意大小的数据



session和cookie的区别：

1.session存储数据在服务器端，cookie在客户端

2.session没有数据大小限制，cookie有

3.session数据安全，cookie相对于session不安全

## Filter-过滤器

步骤：

1.定义一个类实现Filter

2.复写方法

3.配置拦截路径:web.xml或注解



过滤器生命周期方法：

1.init():在服务器启动后，创建filter对象，然后调用init方法，只执行一次，用于加载资源

2.doFilter：每次请求被拦截资源时执行，执行多次

3.destroy：服务器正常关闭，执行，只执行一次，用于释放资源



拦截路径配置：

1./*：拦截所有资源

2.具体资源路径：

3.拦截目录：/user/*

4.后缀名拦截：*.jsp



拦截方式配置（资源被访问的方式）：

1.注解配置：设置dispatcherTypes属性：REQUEST:浏览器直接请求资源

​                                                                       FOREARD:转发访问资源

​                                                                        INCLUD：包含访问资源

​                                                                       ERROR:错误跳转资源

​                                                                       ASYNC：异步访问资源

2.web.xml配置：<dispatcher>REQUEST</dispatcher>

## Listener-监听器



# JSP

## JSP基础语法

### JSP的脚本

1.<% 代码%>:定义的java代码在service方法中，service方法中可以定义什么，该脚本就可以定义什么

2.<%! 代码%>：定义的java代码在jsp转换后的java类的成员变量位置

3.<%= 代码%>：定义的java代码输出到页面，输出语句可以定义什么，该脚本就可以定义什么

###  JSP的内置对象

在jsp页面中不需要创建，可以直接使用的对象

1.request（HttpServletRequest）:一次请求访问的多个资源（转发）

2.response(HttpServletResponse)：响应

3.out(JspWriter)：字符输出流对象，将数据输出到页面上，和response.getWriter()类似

​           *out和response.getWriter()的区别：response.getWriter()的输出永远在out之前

4.pageContext(PageContext)：:当前页面共享数据，可以获取其他八个对象

5.session(HttpSession)：一次会话对个请求间共享数据

6.application(servletContext)：所有用户间共享数据

7.page(Object)：当前页面(servlet)的对象

8.config(ServletConfig)：servlet的配置对象

9.exception：异常对象

###  JSP的指令

作用：用于配置JSP的页面，导入资源文件

格式：<%@指令名称 属性名1=属性值1 属性名2=属性值2%>

 分类：page:用于配置jsp页面

​                  *contentType：设置响应消息体的mime类型以及字符集，设置当前jsp页面的编码

​                  *language：

​                  *buffer：

​                  *import：导包

​                  *errorPage：当前页面发生异常跳转到指定错误页面

​                  *isErrorPage：表示当前页面是否是错误页面，只有为true时才可以使用内置对象exception

​             include:导入页面的资源文件 <%include file=""%>

​              taglib:导入资源      <%taglib prefix="" uri=""%>      

​                          *prefix:前缀

###  JSP的注释

1.html注释:<!-- -->只能注释html代码片段

2.jsp注释：<%-- --%> 可以注释所有

## MVC开发模式

1.M：Model，模型：业务逻辑操作

2.V：View，视图：展示数据

3.C:Controller，控制器：1.获取客户端的输入 

​                                            2.调用模型

​                                            3.将数据交给视图展示



优点：1.耦合性低，方便维护，利于分工协作

​            2.重用性高

缺点：使项目架构变复杂，对开发人员要求高

## EL表达式

作用：替换和简化jsp页面中java代码的编写

语法：${表达式}

注意：

jsp默认支持EL表达式，如果要忽略el表达式：

​                1）isELIgnored="true"

​               2)    \\\${表达式}:忽略当前这个el表达式



EL的使用：1.运算：算数运算符： + - * /(div) %(mod) 

​                                  比较运算符：>  < >= <= ==  !=

​                                  逻辑运算符：&&(and)      ||(or)    !(not) 

​                                 空运算符：empty  用于判断字符串、集合、数组对象是否为null并且长度是否为0

​                    2.获取值：只能从域对象中获取值、

​                       语法：

​                        1.${域名称.键名称}:从指定域获取指定键的值

​                                  *域名称：pageScope——>pageContext

​                                                   requestScope——>request

​                                                   sessionScope——>session

​                                                  applicationScope——>application（servlet）

​                        2.${键名称}：表示依次从最小的域中查找是否有该键对应的值，知道找到为止

​                        3.${域名称.键名.属性名}：获取对象的值，本质上去调用对象的getter方法

​                        4.${域名称.键名[索引]}：获取list集合

​                       5.${域名称.键名.key名称}或${域名称.键名.["key名称"]}：获取map集合



隐式对象：el表达式有11个隐式对象

pageContext：获取其他八个内置对象

​     *  ${pageContext.request.contextPath}:动态获取虚拟目录

## JSTL标签

# Mybatis




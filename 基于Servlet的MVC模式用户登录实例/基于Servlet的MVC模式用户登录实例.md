#一、基于Servlet的MVC模式用户登录实例
##关于MVC模式的简单解释
M	Model，模型层，例如登录实例中，用于处理登录操作的类；  
V	View，视图层，用于展示以及与用户交互。使用html、js、css、jsp、jQuery等前端技术实现；  
C	Controller，控制器，接受视图层的请求，将请求跳转到对应的模型进行处理，模型层处理完毕后，再将结果返回给请求处。这里用Servlet实现控制器。

##实现过程分析
用户再视图层输入用户名以及密码点击提交，向控制器发出请求  
控制器（Servlet）接受请求，将接受到的用户名以及密码转给模型层
模型层依据用户名和密码在数据库中进行查询，将操作结果返回给控制器  
控制器经过判断返回给用户登录结果。
##代码实现
####项目结构图

![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/MVD_Project_Structer.png)  
####视图层实现(index.jsp)
```

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	    <head>
	        <title>Login</title>
	    </head>
	    <body>
	        <form action="LoginServlet" method="post">
	            Name:<input type="text" name="uname"><br>
	            Pass:<input type="password" name="upwd"><br>
	            <input type="submit" value="Login"><br>
	        </form>
	    </body>
	</html>
```
####控制器层实现(LoginServlet)
```

	package com.ajy.Servlet;
	
	import com.ajy.Model.LoginDao;
	import com.ajy.Entity.User;
	
	import javax.servlet.ServletException;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	import java.io.PrintWriter;
	
	//控制器层
	public class LoginServlet extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        //处理post方式登录请求
	        request.setCharacterEncoding("utf-8");
	        String uname = request.getParameter("uname");
	        String upwd = request.getParameter("upwd");
	        User user = new User(uname,upwd);
	
	        response.setContentType("text/html;charset=utf-8");
	        response.setCharacterEncoding("utf-8");
	        PrintWriter out = response.getWriter();
	        int rs = LoginDao.Login(user);
	        if(rs==-1){
	            out.println("系统错误");
	        }else if(rs==0){
	            out.println("用户名或密码错误");
	        }else{
	            out.println("登录成功");
	        }
	    }
	}
```
####模型层实现(LoginDao)
```

	package com.ajy.Model;
	
	import com.ajy.Entity.User;
	
	import java.sql.*;
	
	//模型层，用于处理登录操作
	public class LoginDao {
	    private static String DBUname = "root";
	    private static String DBUpwd = "121181";
	    private static String URL="jdbc:mysql://localhost:3306/anjiyubase?&serverTimezone=UTC&useSSL=false";
	
	    public static int Login(User user){
	
	        Connection con = null;
	        PreparedStatement pstmt = null;
	        ResultSet rs = null;
	        int count = 0;
	        try {
	            Class.forName("com.mysql.cj.jdbc.Driver");
	            con = DriverManager.getConnection(URL,DBUname,DBUpwd);
	            pstmt = con.prepareStatement("select count(*) from users where uanme = ? and upwd = ?");
	            pstmt.setString(1,user.getUserName());
	            pstmt.setString(2,user.getUserPassword());
	            rs = pstmt.executeQuery();
	            if(rs.next()){
	                count = rs.getInt(1);
	            }
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return -1;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return -1;
	        }finally {
	            try {
	                if (rs!=null)rs.close();
	                if (pstmt!=null)pstmt.close();
	                if (con!=null)con.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return -1;
	            }
	        }
	
	        if (count==0){
	            return 0;
	        }else{
	            return 1;
	        }
	    }
	}
```
####用户实例(User)
```

	package com.ajy.Entity;
	
	public class User {
	    private int id;
	    private String UserName;
	    private String UserPassword;
	
	    public User(String UserName, String UserPassword){
	        this.UserName=UserName;
	        this.UserPassword=UserPassword;
	    }
	
	    public User(int id, String UserName, String UserPassword){
	        this.id=id;
	        this.UserName=UserName;
	        this.UserPassword=UserPassword;
	    }
	    public String getUserName() {
	        return UserName;
	    }
	
	    public void setUserName(String userName) {
	        UserName = userName;
	    }
	
	    public String getUserPassword() {
	        return UserPassword;
	    }
	
	    public void setUserPassword(String userPassword) {
	        UserPassword = userPassword;
	    }
	}

```

####web.xml中的内容
```

	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
	         version="4.0">
	    <welcome-file-list>
	        <welcome-file>login.jsp</welcome-file>
	    </welcome-file-list>
	    
	    <servlet>
	        <servlet-name>LoginServlet</servlet-name>
	        <servlet-class>com.ajy.Servlet.LoginServlet</servlet-class>
	    </servlet>
	    <servlet-mapping>
	        <servlet-name>LoginServlet</servlet-name>
	        <url-pattern>/LoginServlet</url-pattern>
	    </servlet-mapping>
	</web-app>
```
####数据库

![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/MVC_Database.png)


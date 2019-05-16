## 引言
这个实例是上一个实例<a>JavaWeb学习 三层架构实例（一）</a>的加强版，实现的是在前端对数据库中student表的 *增*、*删*、*改*、*查* 操作。关于三层组成云云，这里就不再叙述。
## 实例
#### 效果图
先来看一下主页，将student表中的数据列出来，这里隐藏了地址信息（saddress）。  
![Avartar](https://coding-net-production-file-ci.codehub.cn/007eb7f0-68fd-11e9-9be2-bdcf61b10384.jpg?sign=3B1BXlK2ceSFdlxLBDuqh5WhEz5hPTEyNTcyNDI1OTkmaz1BS0lEYXk4M2xGbWFTNlk0TFRkek1WTzFTZFpPeUpTTk9ZcHImZT0xNTU2NTkzMjM1JnQ9MTU1NjM3NzIzNSZyPTkzNjAyNzEzJmY9LzAwN2ViN2YwLTY4ZmQtMTFlOS05YmUyLWJkY2Y2MWIxMDM4NC5qcGcmYj1jb2RpbmctbmV0LXByb2R1Y3Rpb24tZmlsZQ==)  
这是添加学生信息的页面  
![Avartar](https://coding-net-production-file-ci.codehub.cn/ca351ff0-68fb-11e9-9be2-bdcf61b10384.jpg?sign=Fumb+2glrafr3t4W5VbTMfWfaI5hPTEyNTcyNDI1OTkmaz1BS0lEYXk4M2xGbWFTNlk0TFRkek1WTzFTZFpPeUpTTk9ZcHImZT0xNTU2NTkzMDg1JnQ9MTU1NjM3NzA4NSZyPTIzMjY3ODU1JmY9L2NhMzUxZmYwLTY4ZmItMTFlOS05YmUyLWJkY2Y2MWIxMDM4NC5qcGcmYj1jb2RpbmctbmV0LXByb2R1Y3Rpb24tZmlsZQ==)  
这是修改学生信息的页面（学号不可修改）  
![Avartar](https://coding-net-production-file-ci.codehub.cn/008d5df0-68fd-11e9-9be2-bdcf61b10384.jpg?sign=hfZjsLzamh4YoDxcEBbFu8BYYblhPTEyNTcyNDI1OTkmaz1BS0lEYXk4M2xGbWFTNlk0TFRkek1WTzFTZFpPeUpTTk9ZcHImZT0xNTU2NTkzMjE1JnQ9MTU1NjM3NzIxNSZyPTc2ODkwODE4JmY9LzAwOGQ1ZGYwLTY4ZmQtMTFlOS05YmUyLWJkY2Y2MWIxMDM4NC5qcGcmYj1jb2RpbmctbmV0LXByb2R1Y3Rpb24tZmlsZQ==)  
由于删除学生信息不需要跳转，所以没有删除学生信息的页面。这几个功能的操作结果都会在主页的左上角显示。
#### 项目结构
![Avartar](https://coding-net-production-file-ci.codehub.cn/00860af0-68fd-11e9-9be2-bdcf61b10384.jpg?sign=Su1Vkh09deucay0dUYoTxxxMaYNhPTEyNTcyNDI1OTkmaz1BS0lEYXk4M2xGbWFTNlk0TFRkek1WTzFTZFpPeUpTTk9ZcHImZT0xNTU2NTkzMTk1JnQ9MTU1NjM3NzE5NSZyPTQ4NTAwOTI3JmY9LzAwODYwYWYwLTY4ZmQtMTFlOS05YmUyLWJkY2Y2MWIxMDM4NC5qcGcmYj1jb2RpbmctbmV0LXByb2R1Y3Rpb24tZmlsZQ==)  
如图,自上到下,  
**StudentDao.java** 是直接对数据库进行增删改查操作的,属于原子性的操作,没有逻辑性,只是简单的增删改查.比如,它并不会在删除某条信息之前先判断这条信息是否存在.  
**Student.java** 是"学生"类,此类拥有诸如学号、姓名、年龄、地址等信息以及对应的getter和setter方法。  
**StudentService.java** 这个类，名子含义有点模糊，属于service层，同样是对数据库进行增删改查操作，与上面的Dao类不同的是，service层的操作具有逻辑性，就拿添加学生信息来说，service会先调用Dao类的***查询***方法，先判断这个学生是否存在，根据结果进行信息插入操作。  
**Servlet包** 易发现,这个包中存放的都是Servlet类,属于视图层的后端,每一个类每一个类对应一个视图层前端的功能(增删改查);
#### 代码
**Student.java**  

```java

	package com.ajy.entity;
	public class Student {
	    private int stuNo;
	    private String stuName;
	    private int stuAge;
	    private String stuAddress;
	    public Student(int stuNo,String stuName,int stuAge,String stuAddress){
	        this.stuNo = stuNo;
	        this.stuName = stuName;
	        this.stuAge = stuAge;
	        this.stuAddress = stuAddress;
	    }
	    public Student(String stuName,int stuAge,String stuAddress){
	        this.stuNo = stuNo;
	        this.stuName = stuName;
	        this.stuAge = stuAge;
	        this.stuAddress = stuAddress;
	    }
	
	    public int getStuNo() {
	        return stuNo;
	    }
	
	    public void setStuNo(int stuNo) {
	        this.stuNo = stuNo;
	    }
	
	    public String getStuName() {
	        return stuName;
	    }
	
	    public void setStuName(String stuName) {
	        this.stuName = stuName;
	    }
	
	    public int getStuAge() {
	        return stuAge;
	    }
	
	    public void setStuAge(int stuAge) {
	        this.stuAge = stuAge;
	    }
	
	    public String getStuAddress() {
	        return stuAddress;
	    }
	
	    public void setStuAddress(String stuAddress) {
	        this.stuAddress = stuAddress;
	    }
	
	    @Override
	    public String toString() {
	        return getStuNo()+"--"+getStuName()+"--"+getStuAge()+"--"+getStuAddress();
	    }
	}
```

**StudentDao.java**  
  
```java

	package com.ajy.dao;
	
	import com.ajy.entity.Student;
	
	import java.sql.*;
	import java.util.ArrayList;
	import java.util.List;
	
	public class StudentDao {
	    private final String URL="jdbc:mysql://localhost:3306/anjiyubase?&serverTimezone=UTC&useSSL=false&allowPublicKeyRetrieval=true";
	    private final String DRIVER="com.mysql.cj.jdbc.Driver";
	    private final String NAME="root";
	    private final String PWD="121181";
	
	    //增加学生信息
	    public boolean addStudent(Student stu){
	        Connection con = null;
	        PreparedStatement pstmt = null;
	        int count = 0;
	        try {
	            Class.forName(DRIVER);
	            con = DriverManager.getConnection(URL,NAME,PWD);
	            String sql = "insert into student values(?,?,?,?)";
	            pstmt = con.prepareStatement(sql);
	            pstmt.setInt(1,stu.getStuNo());
	            pstmt.setString(2,stu.getStuName());
	            pstmt.setInt(3,stu.getStuAge());
	            pstmt.setString(4,stu.getStuAddress());
	            count = pstmt.executeUpdate();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return false;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return false;
	        }finally {
	            try {
	                if (pstmt!=null)pstmt.close();
	                if (con!=null)con.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return false;
	            }
	        }
	        if(count==0){
	            return false;
	        }else{
	            return true;
	        }
	    }
	
	    //查询学生是否存在
	    public boolean isExits(int stuNo){
	        return queryStudent(stuNo)==null?false:true;
	    }
	
	    //查询学生
	    public Student queryStudent(int stuNo){
	        Connection con = null;
	        PreparedStatement pstmt = null;
	        ResultSet rs = null;
	        Student stu = null;
	        int count = 0;
	        try {
	            Class.forName(DRIVER);
	            con = DriverManager.getConnection(URL,NAME,PWD);
	            String sql = "select * from student where sno=?";
	            pstmt = con.prepareStatement(sql);
	            pstmt.setInt(1,stuNo);
	            rs = pstmt.executeQuery();
	            if (rs.next()){
	                stu = new Student(rs.getInt("sno"),
	                        rs.getString("sname"),
	                        rs.getInt("sage"),
	                        rs.getString("saddress"));
	            }
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return null;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return null;
	        }finally {
	            try {
	                if (rs!=null)rs.close();
	                if (pstmt!=null)pstmt.close();
	                if (con!=null)con.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return null;
	            }
	        }
	        return stu;
	    }
	
	    //查询全部学生
	    public List<Student> queryAll(){
	        Connection con = null;
	        Statement stmt = null;
	        ResultSet rs = null;
	        List<Student> list = new ArrayList<>();
	        try {
	            Class.forName(DRIVER);
	            con = DriverManager.getConnection(URL,NAME,PWD);
	            String sql = "select *from student";
	            stmt = con.createStatement();
	            rs = stmt.executeQuery(sql);
	            while (rs.next()){
	                int sno = rs.getInt("sno");
	                String sname = rs.getString("sname");
	                int sage = rs.getInt("sage");
	                String saddress = rs.getString("saddress");
	                list.add(new Student(sno,sname,sage,saddress));
	            }
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return null;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return null;
	        }finally {
	            try {
	                if (rs!=null)rs.close();
	                if (stmt!=null)stmt.close();
	                if (con!=null)con.close();
	                return list;
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return null;
	            }
	        }
	    }
	
	    //根据学号删除学生
	    public boolean deleteStudentBySno(int Sno){
	        Connection con = null;
	        PreparedStatement pstmt = null;
	        int count = 0;
	        try {
	            Class.forName(DRIVER);
	            con = DriverManager.getConnection(URL,NAME,PWD);
	            String sql = "delete from student where sno=?";
	            pstmt = con.prepareStatement(sql);
	            pstmt.setInt(1,Sno);
	            count = pstmt.executeUpdate();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return false;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return false;
	        }finally {
	            try {
	                if (pstmt!=null)pstmt.close();
	                if (con!=null)con.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return false;
	            }
	        }
	        if(count==0){
	            return false;
	        }else{
	            return true;
	        }
	    }
	
	    //根据学号修改学号对应的学生信息
	    public boolean updateStudentBySno(int sno,Student stu){
	        Connection con = null;
	        PreparedStatement pstmt = null;
	        int count = 0;
	        try {
	            Class.forName(DRIVER);
	            con = DriverManager.getConnection(URL,NAME,PWD);
	            String sql = "update student set sname=?,sage=?,saddress=? where sno=?";
	            pstmt = con.prepareStatement(sql);
	            //要修改的人
	            pstmt.setInt(4,sno);
	            //修改该后的内容
	            pstmt.setString(1,stu.getStuName());
	            pstmt.setInt(2,stu.getStuAge());
	            pstmt.setString(3,stu.getStuAddress());
	            count = pstmt.executeUpdate();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	            return false;
	        } catch (SQLException e) {
	            e.printStackTrace();
	            return false;
	        }finally {
	            try {
	                if (pstmt!=null)pstmt.close();
	                if (con!=null)con.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	                return false;
	            }
	        }
	        if(count==0){
	            return false;
	        }else{
	            return true;
	        }
	    }
	}
```

**StudentService**  

```java

	package com.ajy.service;
	
	import com.ajy.dao.StudentDao;
	import com.ajy.entity.Student;
	
	import java.util.List;
	
	public class StudentService {
	    StudentDao studentDao = new StudentDao();
	
	    //增加学生
	    public boolean addStudent(Student stu){
	        if (!studentDao.isExits(stu.getStuNo())){
	            return studentDao.addStudent(stu);
	        }else{
	            return false;
	        }
	    }
	
	    //根据学号删除学生
	    public boolean deleteStudentBySno(int sno){
	        //先判断学生是否存在
	        if(!studentDao.isExits(sno)){
	            return false;
	        }else{
	            return studentDao.deleteStudentBySno(sno);
	        }
	    }
	
	    //根据学号查询学生
	    public Student queryStudentBySno(int sno){
	        return studentDao.queryStudent(sno);
	    }
	
	
	    //根据学号，更新对应的学生
	    public boolean updateStudentBySno(int sno,Student stu){
	        //先判断此学号对应的学生是否存在
	        if(!studentDao.isExits(sno)){
	            return false;
	        }else{
	            return studentDao.updateStudentBySno(sno,stu);
	        }
	    }
	
	
	    //查询全部学生
	    public List<Student> queryStudentAll(){
	        return studentDao.queryAll();
	    }
	}
```

**QueryAllStudents.java**  
  
```java

	package com.ajy.servlet;
	
	import com.ajy.entity.Student;
	import com.ajy.service.StudentService;
	
	import javax.servlet.ServletException;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	import java.io.PrintWriter;
	import java.util.List;
	
	@WebServlet(name = "QueryAllStudents",value = "/QueryAllStudents")
	public class QueryAllStudents extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        request.setCharacterEncoding("utf-8");
	        StudentService studentService = new StudentService();
	        List<Student> studentList = studentService.queryStudentAll();
	
	        request.setAttribute("students",studentList);
	
	        request.getRequestDispatcher("studentlist.jsp").forward(request,response);
	    }
	}
```

**AddStudentServlet.java**  
```java  

	package com.ajy.servlet;
	
	import com.ajy.entity.Student;
	import com.ajy.service.StudentService;
	
	import javax.servlet.ServletException;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	
	@WebServlet(name = "AddStudentServlet",value = "/AddStudentServlet")
	public class AddStudentServlet extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	
	        request.setCharacterEncoding("utf-8");
	        int stuNo = Integer.parseInt(request.getParameter("stuNo"));
	        String stuName = request.getParameter("stuName");
	        int stuAge = Integer.parseInt(request.getParameter("stuAge"));
	        String stuAddress = request.getParameter("stuAddress");
	
	        Student stu = new Student(stuNo,stuName,stuAge,stuAddress);
	        StudentService addStudentService = new StudentService();
	        boolean res = addStudentService.addStudent(stu);
	
	        if (res){
	            request.setAttribute("res","添加成功");
	        }else{
	            request.setAttribute("res","添加失败");
	        }
	        request.getRequestDispatcher("QueryAllStudents").forward(request,response);
	    }
	}
```

**DeleteStudentServlet.java**  
```java

	package com.ajy.servlet;
	import com.ajy.service.StudentService;
	import javax.servlet.ServletException;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	import java.io.PrintWriter;
	
	@WebServlet(name = "DeleteStudentServlet",value = "/DeleteStudentServlet")
	public class DeleteStudentServlet extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        request.setCharacterEncoding("utf-8");
	        //根据学号删除学生
	        int sno = Integer.parseInt(request.getParameter("sno"));
	        StudentService studentService = new StudentService();
	        boolean res = studentService.deleteStudentBySno(sno);
	
	        response.setContentType("text/html;charset=utf-8");
	        response.setCharacterEncoding("utf-8");
	
	        PrintWriter out = response.getWriter();
	        if (res){
	            request.setAttribute("res","删除成功");
	        }else{
	            request.setAttribute("res","删除失败");
	        }
	        request.getRequestDispatcher("QueryAllStudents").forward(request,response);
	    }
	}
```

**UpdateStudentBySnoServlet.java**  
```java

	package com.ajy.servlet;
	
	import com.ajy.entity.Student;
	import com.ajy.service.StudentService;
	import javax.servlet.ServletException;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	
	@WebServlet(name = "UpdateStudentBySnoServlet",value = "/UpdateStudentBySnoServlet")
	public class UpdateStudentBySnoServlet extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        request.setCharacterEncoding("utf-8");
	        StudentService studentService = new StudentService();
	
	        int sno = Integer.parseInt(request.getParameter("sno"));
	        String sname = request.getParameter("sname");
	        int sage = Integer.parseInt(request.getParameter("sage"));
	        String saddress = request.getParameter("saddress");
	        Student student = new Student(sname,sage,saddress);
	
	        boolean res = studentService.updateStudentBySno(sno,student);
	        if (res){
	            request.setAttribute("res","修改成功");
	        }else{
	            request.setAttribute("res","修改失败");
	        }
	        request.getRequestDispatcher("QueryAllStudents").forward(request,response);
	    }
	}
```

**QueryStudentBySno.java**  
```java

	package com.ajy.servlet;
	
	import com.ajy.entity.Student;
	import com.ajy.service.StudentService;
	import javax.servlet.ServletException;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	
	@WebServlet(name = "QueryStudentBySno",value = "/QueryStudentBySno")
	public class QueryStudentBySno extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        doGet(request,response);
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        request.setCharacterEncoding("utf-8");
	        StudentService studentService = new StudentService();
	        int sno = Integer.parseInt(request.getParameter("sno"));
	        Student stu = studentService.queryStudentBySno(sno);
	
	        request.setAttribute("student",stu);
	        request.getRequestDispatcher("updateinfo.jsp").forward(request,response);
	    }
	}
```

**addstudent.jsp**  
```jsp

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
	    <title>添加学生</title>
	</head>
	<body>
	<form action="AddStudentServlet" method="post">
	    学号：<input type="number" name="stuNo"><br>
	    姓名：<input type="text" name="stuName"><br>
	    年龄：<input type="number" name="stuAge"><br>
	    地址：<input type="text" name="stuAddress"><br>
	    <input type="submit" value="提交"><br>
	</form>
	<a href="QueryAllStudents">返回首页</a>
	</body>
	</html>
```

**studentlist.jsp** 
```jsp

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	  <head>
	    <title>学生信息列表</title>
	  </head>
	  <body>
	  <%
	    String operateResult = (String) request.getAttribute("res");
	    if (operateResult!=null){
	      out.print(operateResult);
	    }else{
	      out.print("<br>");
	    }
	  %>
	  <table border="1px">
	    <tr>
	      <th>学号</th>
	      <th>姓名</th>
	      <th>年龄</th>
	      <th>操作</th>
	    </tr>
	    <%
	      List<Student> studentList = (List<Student>) request.getAttribute("students");
	      for (Student student:studentList){
	    %>
	        <tr>
	          <td><%=student.getStuNo()%></td>
	          <td><%=student.getStuName()%></td>
	          <td><%=student.getStuAge()%></td>
	          <td><a href="QueryStudentBySno?sno=<%=student.getStuNo()%>">修改</a></td>
	          <td><a href="DeleteStudentServlet?sno=<%=student.getStuNo()%>">删除</a></td>
	        </tr>
	    <%
	      }
	    %>
	  </table>
	  <a href="addstudent.jsp">增加</a>
	  </body>
	</html>
```

**updateinfo.jsp**  
```jsp

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
	    <title>学生个人信息</title>
	</head>
	<body>
	<%
	    Student stu = (Student) request.getAttribute("student");
	%>
	    <form action="UpdateStudentBySnoServlet" method="post">
	        学号：<input type="number" name="sno" value="<%=stu.getStuNo()%>"><br>
	        姓名：<input type="text" name="sname" value="<%=stu.getStuName()%>"><br>
	        年龄：<input type="number" name="sage" value="<%=stu.getStuAge()%>"><br>
	        地址：<input type="text" name="saddress" value="<%=stu.getStuAddress()%>"><br>
	        <input type="submit" value="提交"><br>
	    </form>
	    <a href="QueryAllStudents">返回首页</a>
	</body>
	</html>
```
## 引言
通过上一篇博客<a href="https://henuajy.top/2019/04/27/JavaWeb%E5%AD%A6%E4%B9%A0%E4%B9%8B%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84%E5%AE%9E%E4%BE%8B%EF%BC%88%E4%BA%8C%EF%BC%89/">JavaWeb学习之三层架构实例（二）</a>我们基本上已经实现了对学生信息列表的增删改查操作（UI除外），但是不难看出，代码冗余度太高了，尤其是**StudentDao**这个类，其中的增删改查四个方法，同样都要连接数据库、获取statement等等。为此，我又对这个项目进行了有点点优化。
## 优化日志
1、增加了两个接口 **IStudentDao.java** 、 **IStudentService.java** ；  
2、 **StudentDao.java** 和 **StudentService.java** 分别实现以上两个接口，并加入Impl后缀；  
3、 增加一个数据库帮助类 **DBUtil.java**，目的是简化Dao层代码量；
## 代码
代码结构  
![Avartar](https://coding-net-production-file-ci.codehub.cn/d5698520-6985-11e9-8e4d-7108e91dc1ee.jpg?sign=X8E5FJhwYS4vGybgbqecmNKAhrRhPTEyNTcyNDI1OTkmaz1BS0lEYXk4M2xGbWFTNlk0TFRkek1WTzFTZFpPeUpTTk9ZcHImZT0xNTU2NjUxOTI2JnQ9MTU1NjQzNTkyNiZyPTQ1NjcyNjU5JmY9L2Q1Njk4NTIwLTY5ODUtMTFlOS04ZTRkLTcxMDhlOTFkYzFlZS5qcGcmYj1jb2RpbmctbmV0LXByb2R1Y3Rpb24tZmlsZQ==)  

**数据库帮助类DBUtil.java**  
这是一个通用的类，任何的数据库操作（增删改查），都可以通过这各类来实现（不仅仅是对student表的增删改查）；  
```java

	package com.ajy.util;
	
	import java.sql.*;
	
	//通用的数据库操作方法
	public class DBUtil {
	
	    private static String URL="jdbc:mysql://localhost:3306/anjiyubase?&serverTimezone=UTC&useSSL=false&allowPublicKeyRetrieval=false";
	    private static String DRIVER="com.mysql.cj.jdbc.Driver";
	    private static String NAME="root";
	    private static String PWD="121181";
	    public static Connection con = null;
	    public static PreparedStatement pstmt = null;
	    public static ResultSet rs = null;
	    /***
	     *增删改查四个功能，分两大块
	     *  第一块：增删改
	     *  第二块：查询
	     */
	
	    //实现增删改功能
	    public static boolean executeUpdate(String sql,Object[] objects){
	        try {
	            createPreparedStatement(sql,objects).executeUpdate();
	            shutDownDB(null,pstmt,con);
	            return true;
	        } catch (SQLException | ClassNotFoundException e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	
	    //实现查询功能
	    public static ResultSet executeQuery(String sql,Object[] objects){
	        try {
	            rs = createPreparedStatement(sql,objects).executeQuery();
	            return rs;
	        } catch (SQLException | ClassNotFoundException e) {
	            e.printStackTrace();
	            return null;
	        }
	    }
	
	    public static void shutDownDB(ResultSet rs,Statement stmt,Connection con) throws SQLException {
	        if (rs!=null)rs.close();
	        if (stmt!=null)stmt.close();
	        if (con!=null)con.close();
	    }
	
	    public static Connection getConnection() throws SQLException, ClassNotFoundException {
	        Class.forName(DRIVER);
	        return DriverManager.getConnection(URL,NAME,PWD);
	    }
	
	    public static PreparedStatement createPreparedStatement(String sql,Object[] objects) throws SQLException, ClassNotFoundException {
	        pstmt = getConnection().prepareStatement(sql);
	        if (objects != null) {
	            for (int i = 0; i < objects.length; i++) {
	                pstmt.setObject(i + 1, objects[i]);
	            }
	        }
	        return pstmt;
	    }
	}
```

**学生类Student.java**  
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
#### 数据访问层

**接口IStudentDao.java**

```java

package com.ajy.dao;

import com.ajy.entity.Student;
import java.util.List;

public interface IStudentDao {
    public boolean addStudent(Student stu);

    //查询学生是否存在
    public boolean isExits(int stuNo);

    //查询学生
    public Student queryStudent(int stuNo);

    //查询全部学生
    public List<Student> queryAll();

    //根据学号删除学生
    public boolean deleteStudentBySno(int Sno);
    //根据学号修改学号对应的学生信息
    public boolean updateStudentBySno(int sno,Student stu);
}
```

**实现类StudentDaoImpl.java**

```java

package com.ajy.dao.impl;

import com.ajy.dao.IStudentDao;
import com.ajy.entity.Student;
import com.ajy.util.DBUtil;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class StudentDaoImpl implements IStudentDao {

    //增加学生信息
    public boolean addStudent(Student stu){
        String sql = "insert into student values(?,?,?,?)";
        Object[] objects = {stu.getStuNo(),stu.getStuName(),stu.getStuAge(),stu.getStuAddress()};
        return DBUtil.executeUpdate(sql,objects);
    }

    //查询学生是否存在
    public boolean isExits(int stuNo){
        return queryStudent(stuNo)==null?false:true;
    }

    //查询学生
    public Student queryStudent(int stuNo){
        String sql = "select *from student where sno=?";
        Object[] objects = {stuNo};
        ResultSet rs = DBUtil.executeQuery(sql,objects);
        try {
            if (rs.next()){
                int sno = rs.getInt("sno");
                String sname = rs.getString("sname");
                int sage = rs.getInt("sage");
                String saddress= rs.getString("saddress");
                return new Student(sno,sname,sage,saddress);
            }else{
                return null;
            }
        } catch (SQLException e) {
            e.printStackTrace();
            return null;
        }
    }

    //查询全部学生
    public List<Student> queryAll(){
        String sql = "select*from student";
        ResultSet rs = DBUtil.executeQuery(sql,null);
        try{
            List<Student> list = new ArrayList<>();
            while (rs.next()){
                int sno = rs.getInt("sno");
                String sname = rs.getString("sname");
                int sage = rs.getInt("sage");
                String saddress= rs.getString("saddress");
                list.add(new Student(sno,sname,sage,saddress));
            }
            return list;
        } catch (SQLException e) {
            e.printStackTrace();
            return null;
        }finally {
            try {
                DBUtil.shutDownDB(DBUtil.rs,DBUtil.pstmt,DBUtil.con);
            } catch (SQLException e) {
                e.printStackTrace();
                return null;
            }
        }
    }

    //根据学号删除学生
    public boolean deleteStudentBySno(int Sno){
        String sql = "delete from student where sno=?";
        Object[] objects = {Sno};
        return DBUtil.executeUpdate(sql,objects);
    }

    //根据学号修改学号对应的学生信息
    public boolean updateStudentBySno(int sno,Student stu){
        String sql = "update student set sname=?,sage=?,saddress=? where sno=?";
        Object[] objects = {stu.getStuName(),stu.getStuAge(),stu.getStuAddress(),sno};
        return DBUtil.executeUpdate(sql,objects);
    }
}
```

#### 业务逻辑层

**接口IStudentService.java**

```java

package com.ajy.service;
import com.ajy.entity.Student;
import java.util.List;

public interface IStudentService {
    //增加学生
    public boolean addStudent(Student stu);

    //根据学号删除学生
    public boolean deleteStudentBySno(int sno);

    //根据学号查询学生
    public Student queryStudentBySno(int sno);

    //根据学号，更新对应的学生
    public boolean updateStudentBySno(int sno,Student stu);

    //查询全部学生
    public List<Student> queryStudentAll();
}
```

**实现类StudentServiceImpl.java**

```java

package com.ajy.service.impl;

import com.ajy.dao.IStudentDao;
import com.ajy.dao.impl.StudentDaoImpl;
import com.ajy.entity.Student;
import com.ajy.service.IStudentService;
import java.util.List;

public class StudentServiceImpl implements IStudentService {

    IStudentDao studentDao = new StudentDaoImpl();
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

#### 视图层

###### 后端代码

**AddStudentServlet.java**  
```java

package com.ajy.servlet;

import com.ajy.entity.Student;
import com.ajy.service.IStudentService;
import com.ajy.service.impl.StudentServiceImpl;
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
        IStudentService addStudentService = new StudentServiceImpl();
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

import com.ajy.service.IStudentService;
import com.ajy.service.impl.StudentServiceImpl;
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
        IStudentService studentService = new StudentServiceImpl();
        boolean res = studentService.deleteStudentBySno(sno);

        if (res){
            request.setAttribute("res","删除成功");
        }else{
            request.setAttribute("res","删除失败");
        }
        request.getRequestDispatcher("QueryAllStudents").forward(request,response);
    }
}

```

**QueryAllStudents.java**  
```java

package com.ajy.servlet;

import com.ajy.entity.Student;
import com.ajy.service.IStudentService;
import com.ajy.service.impl.StudentServiceImpl;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet(name = "QueryAllStudents",value = "/QueryAllStudents")
public class QueryAllStudents extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        IStudentService studentService = new StudentServiceImpl();
        List<Student> studentList = studentService.queryStudentAll();
        request.setAttribute("students",studentList);
        request.getRequestDispatcher("studentlist.jsp").forward(request,response);
    }
}

```

**QueryStudentBySno.java**
```java

package com.ajy.servlet;

import com.ajy.entity.Student;
import com.ajy.service.IStudentService;
import com.ajy.service.impl.StudentServiceImpl;
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
        IStudentService studentService = new StudentServiceImpl();
        int sno = Integer.parseInt(request.getParameter("sno"));
        Student stu = studentService.queryStudentBySno(sno);
        request.setAttribute("student",stu);
        request.getRequestDispatcher("updateinfo.jsp").forward(request,response);
    }
}

```

**UpdateStudentBySnoServlet.java**
```java

package com.ajy.servlet;

import com.ajy.entity.Student;
import com.ajy.service.IStudentService;
import com.ajy.service.impl.StudentServiceImpl;
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
        IStudentService studentService = new StudentServiceImpl();
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

###### 前端代码
 
前端代码没有改动,这里就不贴了.
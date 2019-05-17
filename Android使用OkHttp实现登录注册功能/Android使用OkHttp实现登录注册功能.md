### 客户端
在客户端，这里将登录和注册放在了同一个界面，在账号和密码两个EditText中输入内容后，按下LOGIN按钮，进行登录；按下REGISTER按钮，进行注册。  
在写代码之前，先添加OkHttp的依赖:  
```xml
implementation 'com.squareup.okhttp3:okhttp:3.4.1'//这里3.4.1是笔者使用的okhttp版本号，可以自行更改至最新版本
```

，注册网络权限:  
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
<!--more-->
#### activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical"
    android:layout_margin="10dp">
        <EditText
            android:id="@+id/loginAccount_etext"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:hint="请输入账号"/>
        <EditText
            android:id="@+id/loginPassword_etext"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:hint="请输入密码"/>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">
            <Button
                android:id="@+id/loginBtn"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="Login"/>
            <Button
                android:id="@+id/registerBtn"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="Register"/>
        </LinearLayout>
</LinearLayout>
```
#### MainActivity.java
```java
package com.henuajy.logindemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import java.io.IOException;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.Response;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private EditText loginAccount_etext;
    private EditText loginPassword_etext;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        loginAccount_etext = findViewById(R.id.loginAccount_etext);
        loginPassword_etext = findViewById(R.id.loginPassword_etext);
        Button loginBtn = findViewById(R.id.loginBtn);
        Button registerBtn = findViewById(R.id.registerBtn);
        registerBtn.setOnClickListener(this);
        loginBtn.setOnClickListener(this);

    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.loginBtn:
                String loginAddress="http://henuajy.zicp.vip/LoLBoxServer_war_exploded/LoginServlet";
                String loginAccount = loginAccount_etext.getText().toString();
                String loginPassword = loginPassword_etext.getText().toString();
                loginWithOkHttp(loginAddress,loginAccount,loginPassword);
                break;
            case R.id.registerBtn:
                String registerAddress="http://henuajy.zicp.vip/LoLBoxServer_war_exploded/RegisterServlet";
                String registerAccount = loginAccount_etext.getText().toString();
                String registerPassword = loginPassword_etext.getText().toString();
                registerWithOkHttp(registerAddress,registerAccount,registerPassword);
                break;
            default:
                break;
        }
    }

    //实现登录
    public void loginWithOkHttp(String address,String account,String password){
        HttpUtil.loginWithOkHttp(address,account,password, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                //在这里对异常情况进行处理
            }
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                //得到服务器返回的具体内容
                final String responseData = response.body().string();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        if (responseData.equals("true")){
                            Toast.makeText(MainActivity.this,"登录成功",Toast.LENGTH_SHORT).show();
                        }else{
                            Toast.makeText(MainActivity.this,"登录失败",Toast.LENGTH_SHORT).show();
                        }
                    }
                });
            }
        });
    }

    //实现注册
    public void registerWithOkHttp(String address,String account,String password){
        HttpUtil.registerWithOkHttp(address, account, password, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                //在这里对异常情况进行处理
            }
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String responseData = response.body().string();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        if (responseData.equals("true")){
                            Toast.makeText(MainActivity.this,"注册成功",Toast.LENGTH_SHORT).show();
                        }else{
                            Toast.makeText(MainActivity.this,"注册失败",Toast.LENGTH_SHORT).show();
                        }
                    }
                });
            }
        });
    }
}
```
#### HttpUtil.java
```java
package com.henuajy.logindemo;

import okhttp3.FormBody;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;

class HttpUtil {
    //登录
    static void loginWithOkHttp(String address,String account,String password,okhttp3.Callback callback){
        OkHttpClient client = new OkHttpClient();
        RequestBody body = new FormBody.Builder()
                .add("loginAccount",account)
                .add("loginPassword",password)
                .build();
        Request request = new Request.Builder()
                .url(address)
                .post(body)
                .build();
        client.newCall(request).enqueue(callback);
    }
    //注册
    static void registerWithOkHttp(String address,String account,String password,okhttp3.Callback callback){
        OkHttpClient client = new OkHttpClient();
        RequestBody body = new FormBody.Builder()
                .add("registerAccount",account)
                .add("registerPassword",password)
                .build();
        Request request = new Request.Builder()
                .url(address)
                .post(body)
                .build();
        client.newCall(request).enqueue(callback);
    }
}
```

到这里，客户端的工作就结束了。
### 服务器端
首先，建立数据库，加入几条数据。  
<fancybox>
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/database_just_insert.jpg)
</fancybox>

#### User.java
```java
package com.henuajy.Entity;

public class User {
    private String loginAccount;
    private String loginPassword;

    public User(String loginAccount,String loginPassword){
        this.loginAccount = loginAccount;
        this.loginPassword = loginPassword;
    }

    public String getLoginAccount() {
        return loginAccount;
    }

    public void setLoginAccount(String loginAccount) {
        this.loginAccount = loginAccount;
    }

    public String getLoginPassword() {
        return loginPassword;
    }

    public void setLoginPassword(String loginPassword) {
        this.loginPassword = loginPassword;
    }
}
```
#### LoginModel.java
```java
package com.henuajy.Model;

import com.henuajy.Entity.User;
import java.sql.*;

public class LoginModel {
    private static String DBUNAME = "root";
    private static String DBUPWD = "121181";
    private static String DRIVER = "com.mysql.cj.jdbc.Driver";
    private static String URL = "jdbc:mysql://localhost:3306/lolbox?&serverTimezone=UTC&useSSL=false";

    public static boolean login(User user){
        String loginAccount = user.getLoginAccount();
        String loginPassword = user.getLoginPassword();
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        int count = 0;
        try{
            Class.forName(DRIVER);
            con = DriverManager.getConnection(URL,DBUNAME,DBUPWD);
            pstmt = con.prepareStatement("select count(*)from userinfo where account=? and password=?");
            pstmt.setString(1,loginAccount);
            pstmt.setString(2,loginPassword);
            rs = pstmt.executeQuery();
            if (rs.next()){
                count = rs.getInt(1);
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            try{
                if (rs!=null){
                    rs.close();
                }
                if (pstmt!=null){
                    pstmt.close();
                }
                if (con!=null){
                    con.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            if (count==1){
                return true;
            }else{
                return false;
            }
        }
    }

    public static boolean register(User user){
        String loginAccount = user.getLoginAccount();
        String loginPassword = user.getLoginPassword();
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        int count = 0;
        try{
            Class.forName(DRIVER);
            con = DriverManager.getConnection(URL,DBUNAME,DBUPWD);
            pstmt = con.prepareStatement("insert into userinfo values (?,?)");
            pstmt.setString(1,loginAccount);
            pstmt.setString(2,loginPassword);
            count = pstmt.executeUpdate();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            try{
                if (rs!=null){
                    rs.close();
                }
                if (pstmt!=null){
                    pstmt.close();
                }
                if (con!=null){
                    con.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            if (count==1){
                return true;
            }else{
                return false;
            }
        }
    }
}
```
#### LoginServlet.java
```java
package com.henuajy.Servlet;

import com.henuajy.Entity.User;
import com.henuajy.Model.LoginModel;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "LoginServlet",value = "/LoginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        String loginAccount = request.getParameter("loginAccount");
        String loginPassword = request.getParameter("loginPassword");
        User user = new User(loginAccount,loginPassword);
        boolean result = LoginModel.login(user);
        System.out.println("登录账号："+loginAccount+",登陆密码："+loginPassword+",登录结果"+result);
        response.setCharacterEncoding("UTF-8");
        //通过PrintWriter返回给客户端操作结果
        PrintWriter writer = response.getWriter();
        writer.print(result);
    }
}

```
#### RegisterServlet.java
```java
package com.henuajy.Servlet;

import com.henuajy.Entity.User;
import com.henuajy.Model.LoginModel;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "RegisterServlet",value = "/RegisterServlet")
public class RegisterServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        String registerAccount = request.getParameter("registerAccount");
        String registerPassword = request.getParameter("registerPassword");
        User registerUser = new User(registerAccount,registerPassword);
        boolean rs = LoginModel.register(registerUser);
		System.out.println("注册账号："+registerAccount+",注册密码："+registerPassword+",注册结果"+rs);
        //通过PrintWriter返回给客户端操作结果
        PrintWriter writer = response.getWriter();
        writer.print(rs);
    }
}

```
下面是进行UI界面、登录结果、注册结果以及注册后服务器端数据库的信息  
<fancybox>
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/ui.jpg)
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/login.jpg)
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/register.jpg)
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/database_after_register.jpg)
</fancybox>
下面是服务器端的日志
<fancybox>
![Avartar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android%E4%BD%BF%E7%94%A8OkHttp%E5%AE%9E%E7%8E%B0%E7%99%BB%E5%BD%95%E6%B3%A8%E5%86%8C%E5%8A%9F%E8%83%BD/Servlet_log.jpg)
</fancybox>
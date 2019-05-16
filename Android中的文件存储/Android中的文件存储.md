### 一、前言
文件存储时Android中最基本的一种数据存储方式，它不对存储的内容进行任何的格式化处理，所有数据都是原封不动地保存到文件当中的，因此它比较适合用于存储一些简单的文本数据或二进制数据。
### 二、将数据存储到文件中
Context类中提供了一个openFileOutput()方法，可以用于将数据存储到指定的文件中。
<fancybox>
![Avartar]()
</fancybox>  
可以看到，此方法接收两个参数，第一个即是文件的名字，这里不可以包含文件路径，所有的文件都是默认存储到/data/data/<packagename>/files/目录下的。第二个参数是文件的操作模式，主要有两种可以选择，**MODE\_PRIVATE**和**MODE\_APPEND**,其中MODE\_PRIVATE 是默认的操作模式，表示当指定同样文件名的时候，所写入的内容将会覆盖原文件中的内容，而MODE\_APPEND则表示如果该文件存在，就往文件里面追加内容，不存在就创建新文件。  
下面，做一个简单的记事本Demo应用一下。  
**activity_main.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical"
    android:layout_margin="5dp">
    <EditText
        android:id="@+id/beingSavedText"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:gravity="top"
        android:background="#87CEEB" />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <Button
            android:id="@+id/save_text_btn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="保存文本"/>
        <Button
            android:id="@+id/cancle_btn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="取消保存"/>
    </LinearLayout>
</LinearLayout>
```
**MainActivity.java**  
```java
package com.henuajy.savetextdemo;

import android.content.Context;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class MainActivity extends AppCompatActivity {

    private String fileName = "my_text";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        text = findViewById(R.id.beingSavedText);
        Button save_text_btn = findViewById(R.id.save_text_btn);
        save_text_btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = text.getText().toString();
                saveText(data);
            }
        });
    }

    public void saveText(String data){
        FileOutputStream os = null;
        OutputStreamWriter oswriter = null;
        BufferedWriter writer = null;
        try {
            os = openFileOutput(fileName,Context.MODE_APPEND);
            oswriter = new OutputStreamWriter(os);
            writer = new BufferedWriter(oswriter);
            writer.write(data);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if (writer!=null){
                    writer.close();
                }
                if(oswriter!=null){
                    oswriter.close();
                }
                if (os!=null){
                    os.close();
                }
                Toast.makeText(MainActivity.this,"保存成功",Toast.LENGTH_SHORT).show();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
程序运行如图：  
<fancybox>
![Avatrar]()
</fancybox>  
点击保存之前，可以看到，应用数据中并没有files目录，这是因为还未曾保存过文件，到保存的时候会自动创建。  
<fancybox>
![Avatrar]()
</fancybox>  
现在，随便输入一些东西，点击“保存文本”，然后再次查看应用程序目录。  
<fancybox>
![Avatrar]()
</fancybox>  
哇！多了一个files目录，并且目录下多了一个my_text文件，没错，这就是我们的记事本被保存的地方。现在查看一下：  
<fancybox>
![Avatrar]()
</fancybox>  
没错，刚刚我输入了“I'm Prodigal”。下面我们来看看如何查看文件的内容吧！
### 三、从文本中读取数据
类似于将数据存储到文件中，Context类中还提供了一个openFileInput()方法，用于从文件中读取数据。  
<fancybox>
![Avatrar]()
</fancybox>  
可以看到，这个方法只有需要传入一个参数，即要读取的文件的名字，然后系统会自动到/data/data/<package name>/files/目录下去加载这个文件，并返回一个FileInputStream对象。  
对app进行改进，使其在加载时自动载入已存在的文件。  
**MainActivity.java**  
```java
package com.henuajy.savetextdemo;

import android.content.Context;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class MainActivity extends AppCompatActivity {

    private EditText text;
    private String fileName = "my_text";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        text = findViewById(R.id.beingSavedText);
        String beingLoadContent = loadText();
        if (beingLoadContent!=null){
            text.setText(beingLoadContent);
        }
        Button save_text_btn = findViewById(R.id.save_text_btn);
        save_text_btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = text.getText().toString();
                saveText(data);
            }
        });
    }
    
    public void saveText(String data){
        FileOutputStream os = null;
        OutputStreamWriter oswriter = null;
        BufferedWriter writer = null;
        try {
            os = openFileOutput(fileName,Context.MODE_APPEND);
            oswriter = new OutputStreamWriter(os);
            writer = new BufferedWriter(oswriter);
            writer.write(data);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if (writer!=null){
                    writer.close();
                }
                if(oswriter!=null){
                    oswriter.close();
                }
                if (os!=null){
                    os.close();
                }
                Toast.makeText(MainActivity.this,"保存成功",Toast.LENGTH_SHORT).show();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public String loadText(){
        FileInputStream is = null;
        InputStreamReader isreader = null;
        BufferedReader reader = null;
        StringBuilder textContent = new StringBuilder();
        try{
            is = openFileInput(fileName);
            isreader = new InputStreamReader(is);
            reader = new BufferedReader(isreader);
            String line = "";
            while((line=reader.readLine())!=null){
                textContent.append(line);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if (reader!=null){
                    reader.close();
                }
                if (isreader!=null){
                    isreader.close();
                }
                if (is!=null){
                    is.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            return textContent.toString();
        }
    }
}

```



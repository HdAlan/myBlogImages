##问题：
如图，“发现”页即为主页，然后我们切换到“我”页，一切正常。
  
![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/App_Home_Page.png)
![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/App_Me_Page.png)  

那么问题来了，如果切换到“我”页后把手机横屏，则会出现下面的情况。  

![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/App_HengPing_Page.png)  

嗯？怎么又回到“发现”页了？？
##解决办法：
####思考
据自己了解，Android应用程序刷新页面有两种情况，第一种是用户操作；
第二种非用户操作，即系统触发的。很明显这是系统触发的咯。
然后，搬来Android应用程序生命周期图： 

![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/Android_Life_Circle.png)

看到，在整个生命周期中，APP会调用onCreate()、onStart()、onResume()、onPause()、onStop()、onRestart()、onDestroy()这几个函数。所以，我在MainActivity.java中重构这几个函数，使用LogCat来验证在横屏的过程中，APP就调用了哪些函数。  

```

	public class MainActivity extends AppCompatActivity {

	    private BottomNavigationView bottomNavigationView;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        //隐藏默认的顶部导航
	        getSupportActionBar().hide();
	
	        //获取底部导航视图实例
	        bottomNavigationView = findViewById(R.id.bottomNavi);
	
	        //把“发现”页作为主页
	        getSupportFragmentManager().beginTransaction().replace(R.id.contentFrame,new DiscoverFragment()).commit();
	        //注册底部导航按钮点击事件
	        bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
	            @Override
	            public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {
	                FragmentTransaction transition = getSupportFragmentManager().beginTransaction();
	
	                switch (menuItem.getItemId()){
	                    case R.id.bottomNavi_discover:
	                        transition.replace(R.id.contentFrame,new DiscoverFragment()).commit();
	                        break;
	                    case R.id.bottomNavi_friends:
	                        transition.replace(R.id.contentFrame,new FriendFragment()).commit();
	                        break;
	                    case R.id.bottomNavi_communicate:
	                        transition.replace(R.id.contentFrame,new CommunicateFragment()).commit();
	                        break;
	                    case R.id.bottomNavi_myself:
	                        transition.replace(R.id.contentFrame,new MeFragment()).commit();
	                        break;
	                }
	                return true;
	            }
	        });
	        Log.i("MainActivity","onCreate()");
	
	    }
	
	    @Override
	    protected void onStart() {
	        super.onStart();
	        Log.i("MainActivity","onStart()");
	    }
	
	    @Override
	    protected void onResume() {
	        super.onResume();
	        Log.i("MainActivity","onResume()");
	    }
	
	    @Override
	    protected void onPause() {
	        super.onPause();
	        Log.i("MainActivity","onPause()");
	    }
	
	    @Override
	    protected void onStop() {
	        super.onStop();
	        Log.i("MainActivity","onStop()");
	    }
	
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        Log.i("MainActivity","onDestroy()");
	    }
	
	    @Override
	    protected void onRestart() {
	        super.onRestart();
	        Log.i("MainActivity","onRestart()");
	    }
	}
```
  
下面运行程序，横屏后，LogCat输出如下：  

![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/LogCatLog.png)

仔细观察发现，横屏后，程序再次调用了onCreate()函数，页面不刷新才怪勒！

####解决
######思路
在横屏前，先保存当前浏览的数据，然后在横屏后，恢复这个数据就可以了。
所以，添加一个信号量(全局变量)，用来保存当前浏览的页面位置（1，2，3，4）  

```
	private int PageFlag = 1;
```

然后再加入如下代码，目的是在程序调用onDestroy()之前，通过onSaveInstanceState()函数保存当前的PageFlag值，在横屏后调用onRestoreInstanceState()时，恢复PageFlag的值，通过此方法恢复横屏前访问的页面。

```


	@Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt("KEY_PAGE_INDEX",PageFlag);

    }
	@Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        if (savedInstanceState!=null){
            PageFlag = savedInstanceState.getInt("KEY_PAGE_INDEX");
            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
            switch (PageFlag){
                case 1:
                    transaction.replace(R.id.contentFrame,new DiscoverFragment()).commit();
                    PageFlag = 1;
                    break;
                case 2:
                    transaction.replace(R.id.contentFrame,new FriendFragment()).commit();
                    PageFlag = 2;
                    break;
                case 3:
                    transaction.replace(R.id.contentFrame,new CommunicateFragment()).commit();
                    PageFlag = 3;
                    break;
                case 4:
                    transaction.replace(R.id.contentFrame,new MeFragment()).commit();
                    PageFlag = 4;
                    break;
            }
        }
    }
```

再次横屏， 
 
![Avatar](https://raw.githubusercontent.com/HdAlan/myBlogImages/master/App_HengPing_Page2.png)  

OK，问题解决！

###注意
**此APP仅用于学习，其中使用的是***《小黑盒APP》***中的资源**
# Data Binding Guide
这篇文章讲解如何使用Data Binding Library来定义layouts，减少用于绑定app逻辑和界面的控制语句。
Data Binding Library提供了既灵活又强大的通用性，它是一个支持库，所以你可以在所有的android平台（包括Android 2.1，7+）上使用。要使用DataBinding，需要**Gradle 1.5.0-alpha 1 或者更高版本**。

## 编译环境
* 从Android SDK manager下载 Support Repository。
* 在App模块的Build.gradle文件中加入以下代码：
```Java
android{   
    ... 
    dataBinding {
        enabled = true
    }
}
```
* 如果其他app模块使用到的library也用到了DataBinding，那个app模块的build.gradle文件中也要加入如上代码。
* 确定你使用了合适的Android Studio版本，只有**1.3及之后的版本**支持。[Android Studio Support for Data Binding](http://developer.android.com/intl/zh-cn/tools/data-binding/)

## Data Binding Layout文件
* 第一个Data Binding表达式
    * Data-binding layout文件使用**layout**作为根节点，使用**data**和**view**作为子节点，其中，view节点是非DataBinding的layout文件中的根节点。举个例子：
    ```Xml
    <?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```
    * **data**中的**variable**描述layout文件中需要使用的属性。
```Xml
<variable name="user" type="com.example.User"/>
```
    * 表达式使用 **@{}** 。TextView中text的内容使用了user中firstName这个属性。
    ```Xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>
```
* Data对象
    * 假设你有一个User的简单Java对象（POJO）:
```Java
public class User {
` public final String firstName;`
` public final String lastName;`
` public User(String firstName, String lastName) {`
`  this.firstName = firstName;`
`  this.lastName = lastName;`
` }`
}```
    * 这种对象中的数据固定不变。某个数据只使用一次，并且之后再也不会改变，这种情况十分常见。我们也可以使用JavaBean的对象：
```Java
public class User {
` private final String firstName;`
` private final String lastName;`
` public User(String firstName, String lastName) {`
`  this.firstName = firstName;`
`  this.lastName = lastName;`
` }`
` public String getFirstName() {`
`  return this.firstName;`
` }`
` public String getLastName() {`
`  return this.lastName;`
` }`
}
```
    * 从DataBinding的角度看，这两个类是等价的。TextView的android:text属性使用了**@{user.firstName}**，它将会访问第一个类中的**firstName**属性，和后一个类中的**getFirstName()**属性。或者，如果firstName()如果存在的话，也会使用这个方法。
* Binding Data
    * 一般来说，Binding类名会根据layout文件的名字生成，把layout文件名转换成Pascal并且加入Binding后缀。上面的layout文件是**main_activity.xml**，所以，生成的类名是**MainActivityBinding**。这个类持有所有layout属性到layout中View的绑定（例如，**user**变量），并且知道如何为绑定表达式赋值。创建绑定很简单，只要在inflate的时候：
    ```Java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
       User user = new User("Test", "User");
       binding.setUser(user);
    }
    ```
    这样就可以了！运行应用你会在界面上看到Test User。或者，你也可以通过以下方式获取View：
    `MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());`
    如果你在ListView或者RecyclerView适配器中使用Data Binding，最好用：
    ```Java
    ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
    //or
    ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
    ```

* Binding Events
    * Event一般直接跟Handler绑定，和**android:onClick**可以直接使用Activity中的方法类似，Event属性名可以通过listener方法的名字管理。例如，**View.OnLongClickListener**有一个方法叫**onLongClick()**,所以，event的属性叫**android:onLongClick**。
    * 为了把一个Event赋值给Handler，使用绑定表达式，调用的方法名作为值。例如，如果你的数据对象有两个方法：
```Java
public class MyHandlers {
` public void onClickFriend(View view) { ... }`
` public void onClickEnemy(View view) { ... }`
}
```
    绑定表达式可以给View设置点击listener：
```Xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{user.isFriend ? handlers.onClickFriend : handlers.onClickEnemy}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
           android:onClick="@{user.isFriend ? handlers.onClickFriend : handlers.onClickEnemy}"/>
   </LinearLayout>
</layout>
```
    一些专用的点击事件的handler已经存在，需要其他属性而不是**android:onClick**从而避免冲突。如下属性已经创建出来避免冲突：
    | Class | Listener Setter   | Attribute  |
    | ----- | ----------------- | ---------- |



## Layout细节
* Imports
* Variables
* Custom Binding Class Names
* Includes
* Expression Language
    * 普通特性
    * 缺省操作符
    * Null合并操作符
    * 属性引用
    * 避免空指针
    * Collections
    * String常量
    * 资源

## Data对象
* Observable对象
* Observable变量
* Observable Collections

## 生成Binding
* 创建
* View加ID
* Variables
* ViewStubs
* 高级Binding
    * 动态变量
    * 直接绑定
    * 后台线程

## 属性设置器
* 自动设置器
* 重命名设置器
* 自定义设置器

## 转换器
* 自定义转换器
* Android Studio 对Data Binding的支持

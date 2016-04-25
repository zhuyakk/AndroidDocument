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
     public final String firstName;
     public final String lastName;
     public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
     }
    }
    ```
    * 这种对象中的数据固定不变。某个数据只使用一次，并且之后再也不会改变，这种情况十分常见。我们也可以使用JavaBean的对象：
    ```Java
    public class User {
     private final String firstName;
     private final String lastName;
     public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
     }
     public String getFirstName() {
      return this.firstName;
     }
     public String getLastName() {
      return this.lastName;
     }
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
     public void onClickFriend(View view) { ... }
     public void onClickEnemy(View view) { ... }
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
    
    * 一些专用的点击事件的handler已经存在，需要其他属性而不是**android:onClick**从而避免冲突。如下属性已经创建出来避免冲突：
| Class | Listener Setter     | Attribute   |
| :----- | :----------------- | :---------- |
| SearchView|setOnSearchClickListener(View.OnClickListener)|android:onSearchClick|
|ZoomControls|setOnZoomInClickListener(View.OnClickListener)|android:onZoomIn|
|ZoomControls|setOnZoomOutClickListener(View.OnClickListener)|android:onZoomOut|


## Layout细节
* Imports
    * **data**节点中可以不使用import或者多个import节点，好处是，可以在layout文件中方便的引用类，就像Java一样。
    ```Xml
        <data>
        <import type="android.view.View"/>
        </data>
    ```
    现在，绑定表达式中可以使用View了。  
    ```Xml
        <TextView
           android:text="@{user.lastName}"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>  
    ```
    
   * 当类名冲突的时候，其中一个类要重命名为“别名”。
    ```Xml
       <import type="android.view.View"/>
    <import type="com.example.real.estate.View" alias="Vista"/>
    ```
   这样，layout文件中的Vista可以用来引用 **com.example.real.estate.View** ，View可以用来引用**android.view.View**。import type 可以在变量和表达式中作为type使用：  
   ```Xml
       <data>
        <import type="com.example.User"/>
        <import type="java.util.List"/>
        <variable name="user" type="User"/>
        <variable name="userList" type="List&lt;User>"/>
        </data>  
    ```
    [注]Android Studio不能处理imports，所以你的IDE并不支持自动导入变量。您的应用程序仍然会编译好，你可以在变量定义的时候使用完全合格的名称解决此问题。  
    ```Xml
        <TextView
       android:text="@{((User)(user.connection)).lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    ```
   * 引用类型也可以被表达式中引用的静态变量或方法使用。
    ```Xml
       <data>
        <import type="com.example.MyStringUtils"/>
        <variable name="user" type="com.example.User"/>
    </data>
    …
    <TextView
       android:text="@{MyStringUtils.capitalize(user.lastName)}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    ```
   和Java一样，java.lang.* 被自动引用进来。
   
* Variables
    * 任何变量都可以出现在data节点下，每个变量描述了一种属性，layout中设置的这种属性，可以在绑定表达式的时候使用。
    ```Xml
    <data>
        <import type="android.graphics.drawable.Drawable"/>
        <variable name="user"  type="com.example.User"/>
        <variable name="image" type="Drawable"/>
        <variable name="note"  type="String"/>
    </data>
    ```
    * 在编译阶段会检查变量type，所以，如果变量完成了 Observable 接口，或者变量本身是observable collection，应该在type中反映出来。如果变量是基类或者基本接口，不用完成Obervable*接口，变量变化**不会**被观察到。
    * 当配置不同的时候，会有不同的layout文件（例如，横屏或者竖屏），这些layout文件中的变量会被组合使用，所以，不允许文件中有互相冲突的变量定义。
    * 生成的绑定类会为每个变量生成一个setter和getter。变量在setter被调用前都使用默认Java值——引用类型的初始值是null，int初始值是0，boolean型初始值是false，等等。
    * 如果需要的话，会生成一个叫context的变量，这个变量用于绑定表达式。context的值通过根View的getContext()拿到。可以用显示变量声明来重写context变量。

* 自定义绑定类名
    * 一般来说，绑定的类名是基于layout名字命名的，大写开头，去掉下划线（_），下划线以后的第一个字符大写，并以“Binding”结尾。这个类，被放置于模块包名下的databinding包里面。例如，layout文件叫contact_item.xml,生成的绑定类叫ContactItemBinding。如果模块包名叫com.example.my.app,绑定类会被放在com.example.my.app.databinding下面。
    * 绑定类可以被重命名或者放置在不同的包下面，你只需要修改data节点的class属性就可以了：
```Xml
    <data class="ContactItem">
    ...
    </data>
```
    生成的绑定类是ContactItem，放置在模块包的databinding包下。如果绑定类需要生成在模块包的不同包里，需要加上“.”前缀：
```Xml
    <data class=".ContactItem">
    ...
    </data>
```
    这种情况下哎，ContactItem直接生成在模块包下。如果class属性写上了包名全称，那么任何包都可以使用：
```Xml
    <data class="com.example.ContactItem">
    ...
    </data>
```

* Includes
    * 如果变量使用应用程序命名空间和属性变量名，那么它还可以被传递到View节点下的include中。
```Xml
    <?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```
此处，name.xml和contact.xml中必须都有user变量。
Data Binding不支持include直接作为merge节点的子节点，例如，如下的layout是**不支持**的：
```Xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

* Expression Language
    * 普通特性
    Expression Language和Java表达式有些类似，相同点有：
        * 数学运算符 + - / * %
        * 字串连接 +
        * 逻辑运算符 && ||
        * 二进制运算符 & | ^
        * 一元运算符 + - ！ ~
        * 移位运算符 >> >>> <<
        * 比较运算符 == > < >= <=
        * instance of
        * 分组 ()
        * 字串 charactor，String，numeric，null
        * 类型转换
        * 调用方法
        * 成员变量访问
        * 数组访问 []
        * 三元操作符 ?：
        * 例如：
        ```Xml
        android:text="@{String.valueOf(index + 1)}"
        android:visibility="@{age &lt; 13 ? View.GONE : View.VISIBLE}"
        android:transitionName='@{"image_" + id}'
        ```
        
    * 缺少的操作符
    少数操作符你可以在Java中使用，但无法在binding中使用：
        * this
        * super
        * new
        * 显示通用调用(？)
        
    * Null合并操作符
        如果左操作数不为空，空合并操作符（??）选择做操作数，或者右操作数不为空，选择右操作数。  
        `android:text="@{user.displayName ?? user.lastName}"`
        同下面的表达式等价：  
        `android:text="@{user.displayName != null ? user.displayName : user.lastName}"`  
        
    * 属性引用
    已经在上面的[第一个Data Binding表达式](http://developer.android.com/intl/zh-cn/tools/data-binding/guide.html#writing_your_first_data_binding_expressions)中讨论过了，这是简短形式的JavaBean引用。当一个表达式引用类中的某属性的时候，他使用了相同格式的变量，getter和ObervableFilds。  
    `android:text="@{user.lastName}"`
    
    * 避免空指针
    生成的数据绑定代码自动查空并且避免nullpointerExceptions。例如，表达式@{user.name}中，如果user是空的，user.name被赋值为缺省值（null）。如果你引用了user.age，age是int类型的，它缺省被置为0。
    
    * Collections
    一般集合：数组，列表，稀疏列表和map，可以使用[]操作符方便的访问。
```Xml
    <data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&rt;"/>
    <variable name="sparse" type="SparseArray&lt;String&rt;"/>
    <variable name="map" type="Map&lt;String, String&rt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
    </data>
    …
    android:text="@{list[index]}"
    …
    android:text="@{sparse[index]}"
    …
    android:text="@{map[key]}"
```
    
    * String常量
    当属性值使用单引用的时候，表达式使用双引用：  
    `android:text='@{map["firstName"]}'`
    属性值也可以使用双引用，这么做的话，String常量应该用 &quot;或者(`)  
    `android:text="@{map[`firstName`}"`  
    `android:text="@{map[&quot;firstName&quot;]}"`
    
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

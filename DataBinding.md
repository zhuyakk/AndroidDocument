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
    | Class        | Listener Setter                                 | Attribute             |  
    | :----------- | :---------------------------------------------- | :-------------------- |
    | SearchView   | setOnSearchClickListener(View.OnClickListener)  | android:onSearchClick |
    | ZoomControls | setOnZoomInClickListener(View.OnClickListener)  | android:onZoomIn      |
    | ZoomControls | setOnZoomOutClickListener(View.OnClickListener) | android:onZoomOut     |


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
   
* 变量
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
    ```Xml
    android:text="@{map[`firstName`}"
    android:text="@{map[&quot;firstName&quot;]}"
    ```
    
    * 资源
    * 一般的语法允许在表达式中访问资源：  
    `android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"`  
    格式string和plurals可以声明参数：  
    ```Xml
    android:text="@{@string/nameFormat(firstName, lastName)}"
    android:text="@{@plurals/banana(bananaCount)}"
    ```
    * 当plural有多个参数的时候，所有的参数都可以传入：
    ```Xml
    Have an orange
    Have %d oranges

    android:text="@{@plurals/orange(orangeCount, orangeCount)}"
    ```
    * 一些资源需要显示的类型声明：
    | Type	            | Normal Reference | Expression Reference |
    | :---------------- | :--------------- | :------------------- |
    | String[]          | @array           | @stringArray         |
    | int[]	            | @array           | @intArray            |
    | TypedArray    	| @array           | @typedArray          |
    | Animator	        | @animator	       | @animator            |
    | StateListAnimator | @animator	       | @stateListAnimator   |
    | color int     	| @color	       | @color               |
    | ColorStateList	| @color	       | @colorStateList      |

## Data对象
任何普通Java对象（POJO）都能用来做数据绑定，但改变的POJO不会引发UI更新。数据绑定真正的用处在于：当数据发生改变的时候，给Data对象通知的能力。有三种不同的数据改变通知机制，Observable objects, observable fields和observable collections。  
当任意一种observable数据对象和UI绑定，数据对象的属性发生改变，UI也会自动刷新。

* Observable对象
    完成Observable接口的类允许给这个绑定附加一个监听器，这个监听器监听绑定对象的所有改变。  
    observable接口支持增加和移除监听器，但通知是有开发者决定的。为了使开发更容易，创建了基类BaseObservable来完成坚挺着的注册机制。当属性变化的时候，数据类接口负责通知。这是通过给getter分配一个Bindable注释，并在setter中通知来完成的。
    ```Java
    private static class User extends BaseObservable {
       private String firstName;
       private String lastName;
       @Bindable
       public String getFirstName() {
           return this.firstName;
       }
       @Bindable
       public String getLastName() {
           return this.lastName;
       }
       public void setFirstName(String firstName) {
           this.firstName = firstName;
           notifyPropertyChanged(BR.firstName);
       }
       public void setLastName(String lastName) {
           this.lastName = lastName;
           notifyPropertyChanged(BR.lastName);
       }
    }
    ```
    在**编译**期间，Bindable注释在BR类文件中生成一个条目，BR类文件在模块包下。如果数据类的基类不能改变，Observable接口可以用**PropertyChangeRegistry**来有效的存储和通知监听器。
* Observable变量
    创建Observable类的时候，需要做一些事情，如果想要节省时间或者只有很少属性，开发者可以使用**ObservableFieldh**和类似的ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble 和 ObservableParcel。 ObservableFields是独立的Observable对象，它有自己的成员。初始版本在访问操作数的时候尽力避免装箱和拆箱，在数据类中创建一个public final的成员变量：
    ```Java
    private static class User {
       public final ObservableField<String> firstName =
           new ObservableField<>();
       public final ObservableField<String> lastName =
           new ObservableField<>();
       public final ObservableInt age = new ObservableInt();
    }
    ```
    就是这样！要访问这个值，使用set和get存取方法：
    ```Java
    user.firstName.set("Google");
    int age = user.age.get();
    ```
    
* Observable Collections
    一些应用更多使用动态结构来处理数据。Observable collections允许通过key值访问这些数据对象。当key值是引用类型，例如String，可以用**ObservableArrayMap**。
    ```Java
    ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
    user.put("firstName", "Google");
    user.put("lastName", "Inc.");
    user.put("age", 17);
    ```
    在这个layout中，可以使用String的key值来访问map。
    ```Xml
    <data>
        <import type="android.databinding.ObservableMap"/>
        <variable name="user" type="ObservableMap&lt;String, Object>"/>
    </data>
    …
    <TextView
       android:text='@{user["lastName"]}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    <TextView
       android:text='@{String.valueOf(1 + (Integer)user["age"])}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    ```  
    当key值是整数的时候，使用**ObservableArrayList**。
    ```Java
    ObservableArrayList<Object> user = new ObservableArrayList<>();
    user.add("Google");
    user.add("Inc.");
    user.add(17);
    ```  
    layout中，使用索引访问list：
    ```Xml
    <data>
        <import type="android.databinding.ObservableList"/>
        <import type="com.example.my.app.Fields"/>
        <variable name="user" type="ObservableList&lt;Object>"/>
    </data>
    …
    <TextView
       android:text='@{user[Fields.LAST_NAME]}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    <TextView
       android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    ```

## 生成Binding
生成的绑定类把layout变量和layout中的View联系起来。绑定的名字和包可以自定义，生成的绑定类都继承自**ViewDataBinding**。

* 创建
    绑定应该紧跟在inflate之后以保证View层级没有影响到之前的绑定View和layout中表达式的操作。有许多方法绑定layout，最常见的是使用绑定类的静态方法。inflate方法只需要一个步骤就可以创建View层级并且绑定到类上，只使用**LayoutInflater**，传入ViewGroup也是可以的：
    ```Java
    MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater);
    MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater, viewGroup, false);
    ```
    如果layout使用其他方法inflate出来了，绑定也要分开操作：  
    `MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);`  
    有时，开始的时候我们不能预知绑定，在这种情况下，可以使用**DataBindingUtil**绑定。
    ```Java
    ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId, parent, attachToParent);
    ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
    ```
    
* View加ID
    layout会为所有有ID的View生成一个public final的成员变量。绑定View层级中是单向传递的，提取有ID的View。这个机制比为多个View调用findViewById要快得多。例如：
    ```Xml
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
               android:text="@{user.firstName}"
               android:id="@+id/firstName"/>
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"
               android:id="@+id/lastName"/>
       </LinearLayout>
    </layout>
    ```
    将会生成绑定类：
    ```Java
    public final TextView firstName;
    public final TextView lastName;
    ```
    如果没有数据绑定，ID不是必须设置的，但在代码里面访问View的时候，Id仍是必须的。
    
* Variables
    每个变量都会提供存取方法：
    ```Xml
    <data>
        <import type="android.graphics.drawable.Drawable"/>
        <variable name="user"  type="com.example.User"/>
        <variable name="image" type="Drawable"/>
        <variable name="note"  type="String"/>
    </data>
    ```
    将会在绑定中生成setters和getters。
    ```Java
    public abstract com.example.User getUser();
    public abstract void setUser(com.example.User user);
    public abstract Drawable getImage();
    public abstract void setImage(Drawable image);
    public abstract String getNote();
    public abstract void setNote(String note);
    ```
    
* ViewStubs
    ViewStub和一般的View有些不同。他们开始不可见，当被设置成可见或者显示的告知要inflate出来的时候，他们inflate出来其他的layout替换掉自己。  
    由于ViewStub实质上从View层级上消失了，绑定对象的View也必须消失并被回收。由于View是final类型的，我们使用**ViewStubProxy**对象替换ViewStub，这可以允许开发者访问ViewStub或者ViewStub inflate出来的View层级。  
    当inflate其他的layout，必须为新的layout建立bingding。因此，ViewStubProxy必须监听ViewStub的ViewStub.OnInflateListener，并且同时建立绑定。因为只存在一个ViewStubProxy，所以允许开发者设置为它设置一个OnInflateListener并在绑定建立后调用它。

* 高级Binding
    * 动态变量
    有时，某种绑定类是不可知的。例如，一个RecyclerView.Adapter操作任意多个layout不知道绑定类是哪个。那也必须在onBindViewHolder(VH, int)中给绑定赋值。  
    在这个例子里，所有RecyclerView的绑定有一个“item”变量，BindingHolder的getBinding方法可以返回ViewDataBinding基类。
    ```Java
    public void onBindViewHolder(BindingHolder holder, int position) {
       final T item = mItems.get(position);
       holder.getBinding().setVariable(BR.item, item);
       holder.getBinding().executePendingBindings();
    }
    ```
    
    * 直接绑定
    当变量或者Observable改变，下一阵之前绑定也会改变。有时，绑定需要立即执行，使用**excutePendingBindings()**强制执行。
    
    * 后台线程
    当数据不是集合类型的时候，你可以用后台线程改变数据模型。数据绑定在执行的时候，会本地化每个变量/成员防止并发。

## 属性设置器
当绑定值发生改变的时候，生成的绑定类必须调用View的绑定表达式的setter方法。数据绑定框架可以自定义调用哪个setter方法。
* 自动设置器
对一个属性来说，数据绑定想要找到set属性的方法，我们并不关注属性的命名空间，只关心属性名。  
例如，表达式和TextView的android:text属性绑定在一起，将会寻找setText(String)方法。如果表达式返回int值，数据绑定将会寻找一个setText(int)方法。注意一定要保证表达式返回的是正确的类型，如果必要的话，进行数据转换。即使没有指定名字的属性存在，数据绑定也会工作。你可以使用数据绑定为任意setter创建属性。例如，DrawerLayout没有任何属性，但有大量的setters。你可以使用自动setters使用其中的一个：
```Xml
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}"/>
```

* 重命名设置器
一些属性的setters和名字不一致，对于这些方法，通过**BindingMethods**注解将属性和setter联系起来。对每个重命名的方法，必须有个类并且包含BindingMethods注解。例如，android:tint属性和setImageTintList(ColorStateList)关联，而不是setTint。
```Java
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```
由于android 框架属性已经被完成了，开发者不能重命名setters。

* 自定义设置器
    * 一些属性需要自定义绑定逻辑。例如，如果android:paddingLeft属性没有相关的setter，但是setPadding(left,top,right,bottom)存在，当属性被调用的时候，允许开发者自定义一个带有**BindingAdapter**注解的静态的Binding adapter方法，用来决定setter如何使用。
    android属性已经已经有了BindingAdapter创建了。例如：
    ```Java
    @BindingAdapter("android:paddingLeft")
    public static void setPaddingLeft(View view, int padding) {
       view.setPadding(padding,
                       view.getPaddingTop(),
                       view.getPaddingRight(),
                       view.getPaddingBottom());
    }
    ```
    Binding adapter对于其他自定义类型十分重要，例如，一个自定义的加载器可以在线程外被调用来加载图片。  
    当有冲突的时候，开发者自己创建的Binding adapter将会重写缺省的数据绑定adapter。
    * 你也可以使用adapters接收多个参数：
    ```Java
    @BindingAdapter({"bind:imageUrl", "bind:error"})
    public static void loadImage(ImageView view, String url, Drawable error) {
       Picasso.with(view.getContext()).load(url).error(error).into(view);
    }
    ```
    ```Xml
    <ImageView app:imageUrl=“@{venue.imageUrl}”
               app:error=“@{@drawable/venueError}”/>
    ```
    如果imageUrl或者error用于imageView并且imageUrl是个字串，error是一个drawable，那么adapter会被调用。
        * 自定义命名空间在匹配的时候会被忽略
        * 你也可以为android命名空间写adapter
    Binding adapter方法在处理器中也可以使用旧值，使用旧值或者新值的方法应该有旧属性的所有旧值，然后才有新值。
    ```Java
    @BindingAdapter("android:paddingLeft")
    public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
       if (oldPadding != newPadding) {
           view.setPadding(newPadding,
                           view.getPaddingTop(),
                           view.getPaddingRight(),
                           view.getPaddingBottom());
       }
    }
    ```
    事件处理器只能被接口或者只有一个抽象方法的抽象类使用，例如：
    ```Java
    @BindingAdapter("android:onLayoutChange")
    public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
           View.OnLayoutChangeListener newValue) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            if (oldValue != null) {
                view.removeOnLayoutChangeListener(oldValue);
            }
            if (newValue != null) {
                view.addOnLayoutChangeListener(newValue);
            }
        }
    }
    ```
    当监听器有多个方法，将会被分成多个监听器。例如，View.OnAttachStateChangeListener 有两个方法，
    
    
    
## 转换器
* 自定义转换器
* Android Studio 对Data Binding的支持

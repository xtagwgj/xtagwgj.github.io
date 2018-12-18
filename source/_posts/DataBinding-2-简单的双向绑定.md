---
title: DataBinding(2)-简单的双向绑定
tags:
  - DataBinding
  - Android
date: 2017-08-26 10:21:10
categories:
  - 教程
---

## 系统控件

> 项目地址：[https://github.com/xtagwgj/Example_databinding](https://github.com/xtagwgj/Example_databinding)


### EditText

我们平时在开发中，获取输入信息的主要控件，因此比较适合用来演示双向绑定。

1. 动态修改EditText中的内容并保存到实体类中
```xml

<!-- 在data中引入数据 -->
        <variable
            name="student"
            type="com.xtagwgj.dbindingsample.entity.Student" />

<!-- 使用一个TextView来显示学生的姓名，这里是单向绑定 -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="@{student.name.get()}" />

<!-- 使用EditText来进行学生姓名的更改，进行双向绑定比较简单，在'{'前面加一个'='就可以了 -->
	<EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="@={student.name.get()}" />

<!-- 现在我们在EditText中输入信息会发现TextView中的信息也发生相应的改变，这就已经实现了数据的双向绑定 -->

```

2. 修改EditText的提示文字
```xml

<!-- 在data中引入提示文字的变量 -->
        <variable
            name="hintText"
            type="java.lang.String" />

<!-- 添加 android:hint="@{hintText}"，完成提示文字的数据绑定 -->
 	<EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hint="@{hintText}"
            android:text="@={student.name.get()}" />

```

3. 修改EditText的字体大小
```xml

<!-- 在Activity中有两种设置值的方式 ，直接写具体的值，或者是获取dimen里的-->
        binding.textSize = 66F
        binding.textSize=resources.getDimension(R.dimen.textSize)

<!-- 在data中引入提示文字的变量 -->
        <variable
            name="textSize"
            type="java.lang.Float" />

<!-- 添加 android:textSize="@{textSize}" ，完成文字字体大小设置 -->
 	 <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hint="@{hintText}"
            android:text="@={student.name.get()}"
            android:textSize="@{textSize}" />

```

4. 修改EditText的字体颜色
```xml

<!-- 当学生姓名长度小于5，显示为红色否则显示为黑色 -->
<!-- 不能使用小于符号，要讲符号转译为 &lt; -->
<!-- 也可以写为： android:textColor="@{student.name.get().length &lt; 5 ? 0xffff0000:0xff000000}"-->

 	<EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hint="@{hintText}"
            android:text="@={student.name.get()}"
            android:textColor="@{student.name.get().length &lt; 5 ? @color/colorAccent:@android:color/black}"
            android:textSize="@{textSize}" />

```

5. 名字为空时，设置TextView隐藏
```xml

<!-- 在data中引入View  -->
<import type="android.view.View" />

<!-- 设置visibility 使用了三元表达式-->
 <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="16dp"
    android:text="@{student.name.get()}"
    android:visibility="@{student.name.get().length &gt; 0 ? View.VISIBLE:View.GONE}" />


```


### ImageView

- 通过网络地址加载图片信息，加载图片失败时显示本地的drawable图片
- 点击事件监听

1. 新建一个类，用于存放DataBinding适配器
```java

/**
 * DataBinding的适配器
 */

public class BindingUtil {

    /**
     * @param view          imageView
     * @param url           图片地址
     * @param errorDrawable 错误时候的drawable
     */
    @BindingAdapter({"imageUrl", "errorDrawable"})
    public static void loadImage(ImageView view, String url, Drawable errorDrawable) {
        Picasso.with(view.getContext()).load(url).error(errorDrawable).into(view);
    }
}

```

2. 在xml中使用
```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".ImageActivity">

    <data>
        <!-- 图片地址 -->
        <variable
            name="url"
            type="java.lang.String" />

        <!-- activity -->
        <variable
            name="activity"
            type="com.xtagwgj.dbindingsample.ImageActivity" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <!-- 要显示的imageView -->
        <!-- 回调方法没有view时，android:onClick="@{()->activity.imageClick(url)}" -->
        <!-- 回调方法有view时，android:onClick="@{(view)->activity.imageClickWithView(view,url)}" -->
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{(view)->activity.imageClickWithView(view,url)}"
            app:errorDrawable="@{@drawable/ic_launcher_round}"
            app:imageUrl="@{url}" />

    </LinearLayout>

</layout>

```

3. 给图片添加一个点击事件的监听
```java
class ImageActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding = DataBindingUtil.setContentView<ActivityImageBinding>(this, R.layout.activity_image)
        binding.url = "http://img.ivsky.com/img/tupian/pre/201707/27/shenqideyanjingtupian-001.jpg"
        //将activity传递给view
        binding.activity = this
    }

    //定义一个点击回调的方法,点击图片显示图片的地址信息
    //有view返回的    
    fun imageClickWithView(view: View, url: String) {
        toast("$view $url")
    }

    //无view返回的    
    fun imageClick(url: String) {
        toast(url)
    }
    
}
```

### DatePicker

datapicker使用的时候要注意的一点就是`月份是从0开始算的`
```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".DataPickerActivity">

    <data>
        <variable
            name="year"
            type="java.lang.Integer" />

        <variable
            name="month"
            type="java.lang.Integer" />

        <variable
            name="day"
            type="java.lang.Integer" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center_horizontal"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text='@{day+"-"+month+"-"+year}' />


	<!-- 注意month这个属性  -->
	<!-- android:onDateChanged="" 设置这个属性可以获取属性的编发 -->
        <DatePicker
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:day="@={day}"
            android:month="@={month-1}"
            android:year="@={year}" />

    </LinearLayout>

</layout>

```

### TimePicker

```xml

	<!-- TimePicker与DatePicker类似 -->
 	<TimePicker
            android:timePickerMode="spinner"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hour="@={hour}"
            android:minute="@={minute}" />

```

### RatingBar

- 一般只关注一个属性rating，使用的时候设置`android:rating="@={rating}"`，rating是float类型的数据
- 有一个监听方法`android:onRatingChanged`

-----

## 自定义控件

### 可能遇到的问题
1. 属性在代码中没有相应的get、set方法，不能使用双向绑定
2. 属性定义比较混乱
3. 怎么给自定义的控件设置监听事件

### 先写一个控件

1. attr中的属性如下
```xml
<!--自定义的seekbar-->
    <declare-styleable name="CustomSeekBar">
        <attr name="cs_text_size" format="reference|dimension" />
        <attr name="cs_selected_color" format="reference|color" />
        <attr name="cs_normal_textcolor" format="reference|color" />
        <attr name="cs_unselected_color" format="reference|color" />
        <attr name="cs_button_res" format="reference|integer" />
        <attr name="cs_split_color" format="reference|color" />
        <attr name="cs_text_array" format="string|reference" />
        <attr name="cs_leftText_by_pos" format="reference|boolean" />
        <attr name="cs_max_stalls" format="integer" />
        <attr name="cs_selected_index" format="integer" />
    </declare-styleable>
```

2. [控件代码](https://github.com/xtagwgj/Example_databinding/blob/master/dBindingSample/app/src/main/java/com/xtagwgj/dbindingsample/widget/CustomSeekBar.java)

### 单向绑定数据
```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".CustomActivity">

    <data>

        <variable
            name="seekIndex"
            type="java.lang.Integer" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white"
        android:gravity="center"
        android:orientation="vertical">

        <com.xtagwgj.dbindingsample.widget.CustomSeekBar
            android:id="@+id/cs_seat_pos"
            android:layout_width="match_parent"
            android:layout_height="48dp"
            android:layout_gravity="center"
            android:layout_marginEnd="48dp"
            android:layout_marginStart="48dp"
            app:OnSeekChangeListener="@{seekListener}"
            app:cs_max_stalls="3"
            app:cs_selected_index="@{seekIndex}"
            app:cs_text_size="12sp" />

    </LinearLayout>
</layout>

```

在Activity中设置seekIndex为1运行时候出现了错误，没有找到set方法。
```
Error:(44, 38) Cannot find the setter for attribute 'app:cs_selected_index' with parameter type java.lang.Integer on com.xtagwgj.dbindingsample.widget.CustomSeekBar. 
```

我们现在查看源代码
```java
	/**
     * 设置档位
     */
    public void setSelectedIndex(int selectedIndex) {
        setSelectedIndex(null, selectedIndex);
    }

    /**
     * 设置档位
     */
    public void setSelectedIndex(CharSequence[] textArray, int selectedIndex) {
        if (textArray != null && textArray.length > 0) {
            this.textArray = textArray;
            maxStalls = textArray.length;
        }

        if (selectedIndex > maxStalls - 1)
            this.selectedIndex = maxStalls - 1;
        else
            this.selectedIndex = selectedIndex;

        invalidate();
    }
```

发现代码中的set方法和xml中定义的属性相差很大，遇到这种情况，如果要修改源代码的话工程量是很大的。
但是通过注解设置属性的set方法，确是十分的方便
```java

   /**
     * 使用BindingAdapter注解设置cs_selected_index属性的setter函数，注解参数value是在xml中对应的属性名
     *
     * @param seekBar       控件的view
     * @param selectedIndex 需要设置的属性值。
     */
    @BindingAdapter(value = "cs_selected_index")
    public static void setIndex(CustomSeekBar seekBar, int selectedIndex) {
        if (seekBar.getSelectedIndex() != selectedIndex)//为了防止设置数据时的死循环
            seekBar.setSelectedIndex(selectedIndex);
    }

```

在代码中添加以上代码，就可以正常运行，运行可以看到效果

### 双向绑定

要怎么实现双向的绑定呢？毫无疑问的：监听数据的变化
监听到了数据变化怎么获取呢？get方法


```java

<!-- 
InverseBindingMethods：反向绑定方法,用来确定怎么去监听view属性的变化和回调哪一个getter方法
type ：指明了去绑定哪一个控件
attribute：指明了监听哪一个属性的变化
method：指明了获取的get方法
 -->
@InverseBindingMethods({
        @InverseBindingMethod(type = CustomSeekBar.class, attribute = "cs_selected_index", method = "getSelectedIndex")
})
public class CustomSeekBarAdapter {
    /**
     * 使用BindingAdapter注解设置与第三步中event值相同的回调函数的setter函数
     *
     * @param seekBar         控件的view
     * @param inverseListener 双向绑定的回调，告诉视图已经发生变化
     *                        <p>
     *                        <p>
     * NOTE：value的值是通过attribute来设置的,直接在attribute的值后+“AttrChanged”即可
     */
    @BindingAdapter(value =  "cs_selected_indexAttrChanged")
    public static void setOnSeekChangeListener(CustomSeekBar seekBar,
                                               final InverseBindingListener inverseListener) {

        seekBar.setOnSeekChangeListener(
                new CustomSeekBar.OnSeekChangeListener() {
                    @Override
                    public void onChange(CustomSeekBar seekBar1, int pos) 

                        if (inverseListener != null)
                            inverseListener.onChange();

                    }
                });
    }

    /**
     * 使用BindingAdapter注解设置cs_selected_index属性的setter函数，注解参数value是在xml中对应的属性名
     *
     * @param seekBar       控件的view
     * @param selectedIndex 需要设置的属性值。
     */
    @BindingAdapter(value = "cs_selected_index")
    public static void setIndex(CustomSeekBar seekBar, int selectedIndex) {
        if (seekBar.getSelectedIndex() != selectedIndex)//为了防止设置数据时的死循环
            seekBar.setSelectedIndex(selectedIndex);
    }
}

```

### 双监听

虽然DataBinding知道了这个数据发生了变化，但是我们程序员自己还是不知道，所以还要让我们知道，所以要再加上一个监听事件
修改setOnSeekChangeListener方法如下
```java

<!-- 
OnSeekChangeListener：这个方法是我们要在xml中写的自己的监听方法
requireAll：必须为false，除非xml中一定设置了OnSeekChangeListener
 -->
@BindingAdapter(value = {"OnSeekChangeListener", "cs_selected_indexAttrChanged"}, requireAll = false)
    public static void setOnSeekChangeListener(CustomSeekBar seekBar,
                                               final CustomSeekBar.OnSeekChangeListener onSeekChangeListener,
                                               final InverseBindingListener inverseListener) {

        seekBar.setOnSeekChangeListener(
                new CustomSeekBar.OnSeekChangeListener() {
                    @Override
                    public void onChange(CustomSeekBar seekBar1, int pos) {
						
			<!-- 自己的监听 -->
                        if (onSeekChangeListener != null)
                            onSeekChangeListener.onChange(seekBar1, pos);
						
			<!-- DataBinding的监听 -->
                        if (inverseListener != null)
                            inverseListener.onChange();

                    }
                });
    }
```

那么现在xml中的设置为
```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".CustomActivity">

    <data>

        <variable
            name="seekIndex"
            type="java.lang.Integer" />

        <!--seekBar变化的监听-->
        <variable
            name="seekListener"
            type="com.xtagwgj.dbindingsample.widget.CustomSeekBar.OnSeekChangeListener" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white"
        android:gravity="center_horizontal"
        android:orientation="vertical">
        
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text='@{seekIndex+""}' />

        <com.xtagwgj.dbindingsample.widget.CustomSeekBar
            android:layout_width="match_parent"
            android:layout_height="48dp"
            android:layout_gravity="center"
            android:layout_marginEnd="48dp"
            android:layout_marginStart="48dp"
            app:OnSeekChangeListener="@{seekListener}"
            app:cs_max_stalls="3"
            app:cs_selected_index="@={seekIndex}"
            app:cs_text_size="12sp" />

    </LinearLayout>
</layout>

```

activity中的代码为

```java
class CustomActivity : AppCompatActivity() {

    //seekBar的监听器
    private val seekListener = CustomSeekBar.OnSeekChangeListener { seekBar, index ->
        toast("$index")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding = DataBindingUtil.setContentView<ActivityCustomBinding>(this, R.layout.activity_custom)

        var seekIndex = 1

        binding.seekIndex = seekIndex
        binding.seekListener = seekListener
    }

}


### [完整代码查看](https://github.com/xtagwgj/Example_databinding/tree/master/dBindingSample/app/src/main/java/com/xtagwgj/dbindingsample)
```

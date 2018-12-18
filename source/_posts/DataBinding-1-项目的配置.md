---
title: DataBinding(1) 项目的配置
tags:
  - DataBinding
  - Android
date: 2017-08-25 16:53:17
categories:
  - 教程
---

## 项目的配置

> 项目地址：[https://github.com/xtagwgj/Example_databinding](https://github.com/xtagwgj/Example_databinding)
> 官方文档：[https://developer.android.com](https://developer.android.com/topic/libraries/data-binding/index.html#writing_expressions)

### 需求
```
- API 7+
- Android Plugin for Gradle 1.5.0-alpha1
- Android Studio 1.3+

```

### 运行环境(我的测试代码的环境)
```
- IDE：Android Studio 3.0 beta 2
- JDK：Java 1.8.0_122
- Language：Kotlin 1.1.4
```

### 创建工程

* 新建工程，在`Create New Project`的页面选中`Include Kotlin support`

* 在Project级别的 `build.gradle` 文件添加如下

```groovy
buildscript {
	<!-- Kotlin的版本  -->
    ext.kotlin_version = '1.1.4-2'
    <!-- Anko的版本  -->
    ext.anko_version = '0.10.1'
    <!-- DataBinding的版本 -->
    ext.android_plugin_version = '2.3.0'

<!-- 省略其他配置 -->
<!-- ..... -->
}

```

* 在Module级别的 `build.gradle` 做如下配置

```groovy

android {

    ...

    <!-- 开启dataBinding  使用java开发的时候只需要开启这里就可以了-->
    dataBinding {
        enabled = true
    }
}

<!-- 开启kapt，kotlin使用 -->
kapt {
    generateStubs = true
}

dependencies {
    ...
<!-- 依赖dataBinding  -->
    kapt "com.android.databinding:compiler:$android_plugin_version"
    compile "org.jetbrains.anko:anko-commons:$anko_version"
}

```

* 做完以上的配置，同步项目，更新完依赖后，项目即构建完成

-----

## 简单的数据绑定


### 目标

直接将实体类的数据显示在界面上


### 实体类

> 实体类必须继承类`BaseObservable`

下面是我们要使用的实体类(有两种方式)

 1. 直接使用ObservableField
```java
//使用ObservableFiled定义了学生的相关信息，其中id设置一次不能再次改变
data class Student(
        val id: Long,
        var name: ObservableField<String> = ObservableField("Mike"),
        var age: ObservableInt = ObservableInt(18),
        var sex: ObservableBoolean = ObservableBoolean(true),
        var money: ObservableDouble = ObservableDouble(1000.0)
) : BaseObservable() {

    override fun toString(): String {
        return "Student(id=$id,name=${name.get()},age=${age.get()},sex=${sex.get()},money=${money.get()})"
    }
}
```

 2. 使用`Bindable`和`notifyPropertyChanged`，两者必不可少。写好类文件后Rebuild一次工程，BR.**才会生效
```java
data class Teacher(
        val id: Long
) : BaseObservable() {

    var name: String = "Lucy"
            //绑定数据
        @Bindable
        get() = field
        set(value) {
            field = value
            //告知数据被更新了，及时修改显示数据
            notifyPropertyChanged(BR.name)
        }
    var age: Int = 38
        @Bindable
        get() = field
        set(value) {
            field = value
            notifyPropertyChanged(BR.age)
        }
    var sex: Boolean = false
        @Bindable
        get() = field
        set(value) {
            field = value
            notifyPropertyChanged(BR.sex)
        }
    var money: Double = 30000.0
        @Bindable
        get() = field
        set(value) {
            field = value
            notifyPropertyChanged(BR.money)
        }

    override fun toString(): String {
        return "Student(id=$id,name=$name,age=$age,sex=$sex,money=$money)"
    }
}
```

### UI

只需要演示数据的绑定，所以直接使用TextView

- 使用DataBinding之前的xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

</LinearLayout>

```

- 适用于DataBind的xml文件

布局文件将更改为layout节点，同时引入data节点，实体必须继承BaseObservable方法

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".SimpleActivity">

    <!--定义页面要使用到的数据 两种不同的实体类型-->
    <!-- 其中data有name属性，可以设置binding的名称和保存的路径-->
    <data>

        <!--可以使用import的方式导入工具类-->
        <!--使用import的方式导入类，在变量的定义中可以不用写全路径-->
        <!--如果有多个同名的类可以使用alias进行区分，如果只有单个同名的类就可以省略alias不写-->
        <import
            alias="Student"
            type="com.xtagwgj.dbindingsample.entity.Student" />

        <!-- 在data内描述了一个名为student的变量属性，使其可以在这个layout中使用 -->
        <!-- variable表示一个变量。name是自己定义的可以更改；type必须是类的全路径，不然会报错 -->
        <variable
            name="student"
            type="Student" />

        <variable
            name="teacher"
            type="com.xtagwgj.dbindingsample.entity.Teacher" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <!--
            android:text="@{student}" 要显示的数据信息
            -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(student.money.get())}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(teacher.money)}" />

        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:onClick="changeMoney"
            android:text="改变金额" />

    </LinearLayout>

</layout>
```

### Activity

绑定数据有两种方式：
1. binding.student = ...  【其实就相当于java中的binding.setStudent(...)】
2. binding.setVariable(BR.student , ...)


```java
/**
 * 最简单的DataBinding的使用
 * 添加按钮点击事件是为了验证数据的绑定是否成功
 */
class SimpleActivity : AppCompatActivity() {

    val student = Student(2)
    val teacher = Teacher(111)

    <!--
        如果你是在RecyclerView中使用DataBinding，你可以下下面这样写
        ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
        //or
        ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
     -->

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        //使用DataBindingUtil.setContentView方法替代setContentView
        //binding的类型为布局文件的名称去掉下划线，首字母大写+Binding，是有系统自动生成的。所以写完布局文件最好Rebuild一次项目
        //NOTE：我们也可以在data的节点设置name属性，生成该类对应的binding，而不是系统默认的
        val binding = DataBindingUtil.setContentView<ActivitySimpleBinding>(this, R.layout.activity_simple)

        //初始化数据并绑定【相当于Java中的binding.setStudent(...)】
        binding.student = student
        //绑定数据的另外一种方式
        //binding.setVariable(BR.student,Student(3))

        binding.teacher = teacher

    }

    fun changeMoney(view: View) {

        student.money.set(Math.random() * 1000)
        teacher.money = Math.random() * 10000

    }
}

```

### 预览效果

点击按钮，两者的金额发生变化；
截图就不放了

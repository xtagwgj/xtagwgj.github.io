---
title: DataBinding(3) 其他用法
tags:
  - DataBinding
  - Android
date: 2017-08-28 09:45:41
categories:
  - 教程
---

# 其他的用法

## include
> 通过使用application namespace以及在属性中的Variable名字从容器layout中传递Variables到一个被包含的layout

Activity的XML
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".IncludeActivity">

	<!-- 该页面需要用到的数据  -->
    <data>
        <variable
            name="student"
            type="com.xtagwgj.dbindingsample.entity.Student" />

        <variable
            name="listener"
            type="android.view.View.OnClickListener" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

		<!-- include 页面 ，从这里传递数据到include的布局中  -->
		<!-- app:listener 中的 listener 与 include页面中的变量名称相同  -->
        <include
            layout="@layout/include_view"
            app:listener="@{listener}"
            app:student="@{student}" />

    </LinearLayout>
</layout>

```

Include的 `XML`
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

		<!-- 接收传过来的变量数据  -->
        <variable
            name="student"
            type="com.xtagwgj.dbindingsample.entity.Student" />

        <variable
            name="listener"
            type="android.view.View.OnClickListener" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{listener}"
            android:padding="16dp"
            android:text='@{student.money+""}' />
    </LinearLayout>

</layout>
```



## ViewStubs

> ViewStubs跟正常的Views略有不同。他们开始时是不可见的，当他们要么设置为可见或被明确告知要载入时，它们通过载入另外一个layout取代了自己

> 由于 `ViewStub` 基本上从View的层次结构上消失，在Binding对象的View也必须消失来允许被收集。因为Views是最后的，一个 `ViewStubProxy` 对象取带ViewStub，给开发者获得了ViewStub，当它存在以及还可以访问载入的View层次结构时当ViewStub已被载入时

> 当载入另一个layout，为新的布局必需创建一个Binding。因此，`ViewStubProxy `必需监听ViewStub的 `OnInflateListener` 监听器并在那个时候建立Binding。因为只有一个可以存在， `ViewStubProxy` 允许开发者在其上设置一个 `OnInflateListener` 它会在建立Binding后调用

先在xml中定义 `ViewStub` 如下

```xml
 	<!-- 此处没有传入要使用的变量 -->
 	<ViewStub
            android:id="@+id/view_stub"
            android:layout="@layout/stub_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
```

其中 `stub_view` 和普通的布局一样

在获取到binding后，调用ViewStub的监听器监听ViewStub的改变
```java
 //在viewStub初始化的时候调用如下监听器
        binding.viewStub.setOnInflateListener(ViewStub.OnInflateListener { stub, inflated ->
            val viewBinding = DataBindingUtil.bind<StubViewBinding>(inflated)

            viewBinding.stu = Student(222)
        })
```

在需要调用ViewStub的时候如下调用即可
```java
 if (!binding.viewStub.isInflated)
        //显示ViewStub的内容
            binding.viewStub.viewStub.inflate()

```


## Fragment
> 获取DataBinding： `DataBindingUtil.inflate(inflater, R.layout.homepage_fragment, container, false);`

```
 class ExamFragment : Fragment() {

    var binding: FragmentExamBinding? = null

    val teacher = Teacher(3)

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {

        if (binding == null) {
            binding = DataBindingUtil.inflate<FragmentExamBinding>(inflater, R.layout.fragment_exam, container, false)
        }

        binding?.teacher = teacher

        return binding!!.root
    }
}
```

fragment使用的布局与普通的DataBinding布局一样

## RecyclerView

先写一个 `item-list` 的子布局

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="stu"
            type="com.xtagwgj.dbindingsample.entity.Student" />

        <variable
            name="listener"
            type="android.view.View.OnClickListener" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="48dp"
            android:gravity="center"
            android:onClick="@{listener}"
            android:text="@{String.valueOf(stu.id)}"
            tools:text="test" />

    </LinearLayout>

</layout>
```

其他的 `Activity` 和 `Adapter` 的代码（为了方便写在了一个文件内）

```

/**
 * 示范RecyclerView的使用
 * Created by xtagwgj on 2017/8/28.
 */
class ListActivity : AppCompatActivity() {

    private val studentList = mutableListOf<Student>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_list)

        //添加n个数据 只是id不一样
        (0..100).mapTo(studentList) { Student(it.toLong()) }

        //显示数据
        recycler.apply {
            layoutManager = LinearLayoutManager(this@ListActivity)

            //其他代码
        }.adapter = MyAdapter(studentList)
    }

}

/**
 * 适配器
 */
open class MyAdapter(val datas: MutableList<Student>) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

    override fun getItemCount() = datas.size

    /**
     * 实现了数据的绑定
     * 和点击事件的监听
     */
    override fun onBindViewHolder(holder: MyAdapter.ViewHolder, position: Int) {

        //这里只是最简单的使用 ，实际情况可能比较复杂
        holder.binding.stu = datas[position]
        //点击输出log
        holder.binding.listener = View.OnClickListener {
            Log.e("ClickAdapter", datas[position].toString())
        }

        //。。。其他逻辑

        //立即更新数据
        holder.binding.executePendingBindings();
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyAdapter.ViewHolder {
        val binding = DataBindingUtil.inflate<ItemListBinding>(LayoutInflater.from(parent.context),
                R.layout.item_list, parent, false)

        //直接传入ItemListBinding对象
        return MyAdapter.ViewHolder(binding)
    }

    //自定义的viewHolder，超类传view过去
    open class ViewHolder(val binding: ItemListBinding) : RecyclerView.ViewHolder(binding.root)
}
```
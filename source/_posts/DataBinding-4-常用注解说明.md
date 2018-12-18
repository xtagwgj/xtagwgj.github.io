---
title: DataBinding(4) 常用注解说明
tags:
  - Android
  - DataBinding
date: 2017-08-28 16:07:18
categories:
  - 教程
---

# 常用注解说明

## @Bindable

`Observable` 接口提供给开发者添加/移除监听者的机制。为了使开发更便捷，我们创建了 `BaseObservable` 类，它已经实现了 `Observable` 接口中的注册监听者的机制。

继承自 `BaseObservable` 的数据类，仍需手动的通知监听者们数据已发生变更。你可以在 `setter` 方法中发出变更消息，记住同时在 `getter` 方法上标记注解 `@Bindable`。

> `@Bindable` 注解的推荐用法 是修饰继承自 `Observable` 类中的 `getter accessor` 方法，但其实 `getter accessor` 的属性也是可以应用该注解的。
> 注解 `@Bindable` 在编译期间生成一个 `BR` 类，以此持有对应的实例，作用同R类。

```
private static class User extends BaseObservable {
   private String firstName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
}
```

## @BindAdapter

> 应用于用于操作表达式的值如何设置为视图的 `方法`。

`@BindingAdapter` 用于修饰方法。

一些属性需要定制绑定逻辑，一个用 `@BindingAdapter` 修饰的静态方法可以自定义属性的 `setter` 操作。

`android` 自身实现了大量的 `Adapter`，你可以在项目 `module` 的 `android.databinding.adapters` 包下找到这些代码。

```
public class CardViewBindingAdapter {
    @BindingAdapter("contentPadding")
    public static void setContentPadding(CardView view, int padding) {
        view.setContentPadding(padding, padding, padding, padding);
    }
}
```

1. 默认的你的自定义的命名空间，在匹配时会被忽略。
```
@BindingAdapter("contentPadding")
```

2. 允许重写android的命名空间
```
@BindingAdapter("android:contentPadding")
```

`app:contentPadding` 与 `android:contentPadding` 处理行为可以不一样。
`app:contentPadding` 与 `custom:contentPadding` 处理行为是一致的。（仅 `android` 是特殊的命名空间）。

需要注意，当你创建的适配器属性与系统默认的产生冲突时，你的自定义适配器将会覆盖掉系统原先定义的注解，这将会产生一些意外的问题。

假设需要对下面接口，做适配。

```
public interface ILogAction{
      void login();
      void logout();
}
```
则需要 `一个方法一个接口`，这么做的原因是避免 `login()` 的修改影响到 `logout()`。所以根据业务需要，可能需要排列组合适配这两个接口。

> 1、适配 login
> 2、适配 logout
> 3、适配 login + logout

## @BindingConversion

> 注解方法用于将表达式的类型自动得转为 `settter` 方法的值类型

有时候会遇到类型不匹配的问题，比如 `R.color.white` 是 `int`，但是通过 `Data Binding` 赋值给 `android:background` 属性后，需要把 `int` 转换为 `ColorDrawable`。

```
@BindingConversion
public static Drawable convertColorToDrawable(int drawable) {
  return new ColorDrawable(drawable);
}
```

## @BindingMethod && @BindingMethods
> 使用Bindingethods注解描述一个将属性重命名来用于该属性的 `setter` 方法
> 用枚举重命名属性到 `setter` 的方法

`@BindingMethods` 用于修饰类。

一些属性虽然拥有 `setters` 但是并不与名字相匹配，这些方法的属性可以通过 `@BindingMethod && @BindingMethods` 注释 `setters`.

```
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```

开发人员不太可能需要重命名 `setters` ，因为android框架属性已经实现了这一部分.

## @InverseBindingAdapter

`InverseBindingAdapter` 用于关联某个用于接收 `View` 变更的方法，典型的例子 `EditText.TextWatcher` 接收输入字符的变更。这与 `BindingAdapters` 有一定的相似性

```
@InverseBindingAdapter(attribute = "android:text", event = "android:textAttrChanged")
 public static String captureTextValue(TextView view, CharSequence originalValue) {
     CharSequence newValue = view.getText();
     CharSequence oldValue = value.get();
     if (oldValue == null) {
         value.set(newValue);
     } else if (!contentEquals(newValue, oldValue)) {
         value.set(newValue);
     }
 }
```

事件的默认值是带有 `AttrChanged` 的属性名称。在上面的例子中，默认值是 `android:textAttrChanged`，即使它没有提供。

事件属性用于通知数据绑定系统值已更改。开发人员通常会创建一个 `BindingAdapter` 来分配事件。比如：

```
@BindingAdapter(value = {"android:beforeTextChanged", "android:onTextChanged",
                          "android:afterTextChanged", "android:textAttrChanged"},
                          requireAll = false)
 public static void setTextWatcher(TextView view, final BeforeTextChanged before,
                                   final OnTextChanged on, final AfterTextChanged after,
                                   final InverseBindingListener textAttrChanged) {
     TextWatcher newValue = new TextWatcher() {
         ...
         @Override
         public void onTextChanged(CharSequence s, int start, int before, int count) {
             if (on != null) {
                 on.onTextChanged(s, start, before, count);
             }
             if (textAttrChanged != null) {
                 textAttrChanged.onChange();
             }
         }
     }
     TextWatcher oldValue = ListenerUtil.trackListener(view, newValue, R.id.textWatcher);
     if (oldValue != null) {
         view.removeTextChangedListener(oldValue);
     }
     view.addTextChangedListener(newValue);
 }
```

如同 `BindingAdapters` 一样， `InverseBindingAdapter` 方法 也可以将 `DataBindingComponent` 作为第一个参数，可以是具有从 `DataBindingComponent` 检索的实例的实例方法。

`InverseBindingListener` 非常有用。[参考 InverseBindingListener](https://developer.android.com/reference/android/databinding/InverseBindingListener.html)

## @InverseBindingMethod

`InverseBindingMethod` 用于标识如何监听对 `View` 属性的更改以及要调用的 `getter` 方法。 `InverseBindingMethod`  应该与 `InverseBindingMethods` 的部分方法相关联。

```
@InverseBindingMethods({@InverseBindingMethod(
     type = android.widget.TextView.class,
     attribute = "android:text",
     event = "android:textAttrChanged",
     method = "getText")})
 public class MyTextViewBindingAdapters { ... }

```

`@InverseBindingMethods` 中的属性 `method` 是可选的。

> 如果其没有提供， 属性名称会查找如下几种可能性：方法名称，前缀为 `is` 或者 `get` 的方法名称。 如属性 `android:text`, 数据绑定框架会在 `TextView` 中搜索 `public CharSequence getText()` 方法。

`@InverseBindingMethods` 中的属性 `event` 是可选的。

> 如果其没有提供，默认会使用 `属性名+AttrChanged` 后缀。如属性 `android:text`, 默认的事件名称 `android:textAttrChanged`。

这个事件也需要配置相关的 `@BindingAdapter` ，如下：
```
@BindingAdapter(value = {"android:beforeTextChanged", "android:onTextChanged",
                          "android:afterTextChanged", "android:textAttrChanged"},
                          requireAll = false)
 public static void setTextWatcher(TextView view, final BeforeTextChanged before,
                                   final OnTextChanged on, final AfterTextChanged after,
                                   final InverseBindingListener textAttrChanged) {
     TextWatcher newValue = new TextWatcher() {
         ...
         @Override
         public void onTextChanged(CharSequence s, int start, int before, int count) {
             if (on != null) {
                 on.onTextChanged(s, start, before, count);
             }
             if (textAttrChanged != null) {
                 textAttrChanged.onChange();
             }
         }
     }
     TextWatcher oldValue = ListenerUtil.trackListener(view, newValue, R.id.textWatcher);
     if (oldValue != null) {
         view.removeTextChangedListener(oldValue);
     }
     view.addTextChangedListener(newValue);
 }
```

## @InverseBindingMethods

> 用于枚举属性，`getter` 和事件关联。

# 参考

简书文章：[DataBinding·常用注解说明](http://www.jianshu.com/p/a05c9735f595)

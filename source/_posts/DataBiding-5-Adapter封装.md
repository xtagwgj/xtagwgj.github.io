---
title: DataBiding(5) Adapter封装
tags:
  - DataBiding
  - Android
date: 2017-09-01 09:42:05
categories:
  - 教程
---

> 为了使用的方便特意封装了这两个类，后期发现问题会及时更新

## 最初的构想
- ViewHolder类

```java
public class DBViewHolder extends RecyclerView.ViewHolder {
    public ViewDataBinding binding;

    public DBViewHolder(ViewDataBinding binding) {
        super(binding.getRoot());
        this.binding = binding;
    }
}
```

- Adapter类

```java
public abstract class DBAdapter<T> extends RecyclerView.Adapter<DBViewHolder> {
    /**
     * 数据集合
     */
    protected List<T> mDatas;

    /**
     * adapter要使用的布局
     */
    private int mLayoutRes;

    protected Context mContext;

    public DBAdapter(List<T> mDatas, int layoutRes) {
        this.mDatas = mDatas == null ? new ArrayList<T>() : mDatas;
        this.mLayoutRes = layoutRes;

        if (layoutRes == 0)
            throw new NullPointerException("mLayoutRes must not be null");
    }

    /**
     * 添加数据
     *
     * @param addList 待添加的数据
     */
    public void addData(List<T> addList) {
        if (addList == null || addList.size() == 0)
            return;

        mDatas.addAll(addList);
        notifyItemRangeInserted(mDatas.size() - addList.size(), addList.size());
    }

    /**
     * 刷新数据
     *
     * @param refreshList 刷新获取的数据
     */
    public void refreshData(List<T> refreshList) {
        mDatas.clear();

        if (refreshList != null && refreshList.size() > 0)
            mDatas.addAll(refreshList);

        notifyDataSetChanged();
    }

    @Override
    public DBViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        mContext = parent.getContext();

        ViewDataBinding binding = DataBindingUtil.inflate(
                LayoutInflater.from(mContext), mLayoutRes, parent, false);

        return new DBViewHolder(binding);
    }

    @Override
    public void onBindViewHolder(DBViewHolder holder, int position) {
        onBindViewHolder(holder, position, mDatas.get(position));
        //立刻更新视图
        holder.binding.executePendingBindings();
    }

    public abstract void onBindViewHolder(DBViewHolder holder, int position, T item);

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

}
```

但是上面这种封装的方式每次使用的时候,都会去将 `binding` 强制转换成相应的 `ViewDataBinding` ,这不利于代码的扩展，所以有了下面的方法

## 最终版

- ViewHolder类

```java
public class DBViewHolder<VDB extends ViewDataBinding> extends RecyclerView.ViewHolder {
    public VDB binding;

    public DBViewHolder(VDB binding) {
        super(binding.getRoot());
        this.binding = binding;
    }
}
```

- Adapter类

```java
public abstract class DBAdapter<T, VDB extends ViewDataBinding> extends RecyclerView.Adapter<DBViewHolder<VDB>> {
    /**
     * 数据集合
     */
    protected List<T> mDatas;

    /**
     * adapter要使用的布局
     */
    private int mLayoutRes;

    protected Context mContext;

    public DBAdapter(List<T> mDatas, int layoutRes) {
        this.mDatas = mDatas == null ? new ArrayList<T>() : mDatas;
        this.mLayoutRes = layoutRes;

        if (layoutRes == 0)
            throw new NullPointerException("mLayoutRes must not be null");
    }

    /**
     * 添加数据
     *
     * @param addList 待添加的数据
     */
    public void addData(List<T> addList) {
        if (addList == null || addList.size() == 0)
            return;

        mDatas.addAll(addList);
        notifyItemRangeInserted(mDatas.size() - addList.size(), addList.size());
    }

    /**
     * 刷新数据
     *
     * @param refreshList 刷新获取的数据
     */
    public void refreshData(List<T> refreshList) {
        mDatas.clear();

        if (refreshList != null && refreshList.size() > 0)
            mDatas.addAll(refreshList);

        notifyDataSetChanged();
    }

    @Override
    public DBViewHolder<VDB> onCreateViewHolder(ViewGroup parent, int viewType) {
        mContext = parent.getContext();

        VDB binding = DataBindingUtil.inflate(
                LayoutInflater.from(mContext), mLayoutRes, parent, false);

        return new DBViewHolder<>(binding);
    }

    @Override
    public void onBindViewHolder(DBViewHolder<VDB> holder, int position) {
        onBindViewHolder(holder, position, mDatas.get(position));
        //立刻更新视图
        holder.binding.executePendingBindings();
    }

    public abstract void onBindViewHolder(DBViewHolder<VDB> holder, int position, T item);

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

}
```

- 使用方法

```java
//其中ItemFavoritesBinding是根据layout来取的,如果不想制定为ItemFavoritesBinding，可以使用ViewDataBinding
//但是使用的是否最好要强制转换成相应的类
public class FavoritesAdapter extends DBAdapter<SupplierItemBean,ItemFavoritesBinding> {

    public FavoritesAdapter() {
        super(null, R.layout.item_favorites);
    }

    @Override
    public void onBindViewHolder(DBViewHolder<ItemFavoritesBinding> holder, int position, SupplierItemBean item) {
        holder.binding.setItem(item);
        //其他的操作
        ...
    }
}
```
使用的时候如果 `adapter` 有多种布局的时候，`VDB` 就必须是 `ViewDataBinding`，在 `onBindViewHolder` 中通过 `viewType` 来使用相应的 `ViewDataBinding` 更新数据

# Fragment



![img](https://developer.android.google.cn/images/fragment_lifecycle.png)

![img](https://developer.android.google.cn/images/activity_fragment_lifecycle.png)

### 创建和使用

继承Fragment, 实现必要的方法，然后Activity加载Fragment。一般需要实现onCreateView, onViewCreated分别创建View，以及View创建完成之后数据的初始化。

使用方法：

1. 通过Activity.getSupportFragmentManager 加载Fragment
2. 布局中<fragment> 标签直接声明

### 各类型Fragment

```
DialogFragment
```

显示浮动对话框。使用此类创建对话框可有效地替代使用`Activity` 类中的对话框帮助程序方法，因为您可以将片段对话框纳入由 Activity 管理的片段返回栈，从而使用户能够返回清除的片段。

```
ListFragment
```

显示由适配器（如 `SimpleCursorAdapter`）管理的一系列项目，类似于 `ListActivity`。它提供了几种管理列表视图的方法，如用于处理点击事件的 `onListItemClick()` 回调。

```
PreferenceFragment
```

以列表形式显示 `Preference` 对象的层次结构，类似于 `PreferenceActivity`。这在为您的应用创建“设置” Activity 时很有用处。



### FragmentManager和事务

要想管理您的 Activity 中的片段，您需要使用 `FragmentManager`。要想获取它，请从您的 Activity 调用`getFragmentManager()`。

您可以使用 `FragmentManager` 执行的操作包括：

- 通过 `findFragmentById()`（对于在 Activity 布局中提供 UI 的片段）或 `findFragmentByTag()`（对于提供或不提供 UI 的片段）获取 Activity 中存在的片段。
- 通过 `popBackStack()`（模拟用户发出的*返回*命令）将片段从返回栈中弹出。
- 通过 `addOnBackStackChangedListener()` 注册一个侦听返回栈变化的侦听器。

如需了解有关这些方法以及其他方法的详细信息，请参阅 `FragmentManager` 类文档。

如上文所示，您也可以使用 `FragmentManager` 打开一个 `FragmentTransaction`，通过它来执行某些事务，如添加和移除片段。



在 Activity 中使用片段的一大优点是，可以根据用户行为通过它们执行添加、移除、替换以及其他操作。 您提交给 Activity 的每组更改都称为事务，您可以使用 `FragmentTransaction` 中的 API 来执行一项事务。您也可以将每个事务保存到由 Activity 管理的返回栈内，从而让用户能够回退片段更改（类似于回退 Activity）。

您可以像下面这样从 `FragmentManager` 获取一个 `FragmentTransaction` 实例：

```
FragmentManager fragmentManager = getFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```



每个事务都是您想要同时执行的一组更改。您可以使用 `add()`、`remove()` 和 `replace()` 等方法为给定事务设置您想要执行的所有更改。然后，要想将事务应用到 Activity，您必须调用 `commit()`。

不过，在您调用 `commit()` 之前，您可能想调用 `addToBackStack()`，以将事务添加到片段事务返回栈。 该返回栈由 Activity 管理，允许用户通过按*返回*按钮返回上一片段状态。

例如，以下示例说明了如何将一个片段替换成另一个片段，以及如何在返回栈中保留先前状态：

```
// Create new fragment and transaction
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```



在上例中，`newFragment` 会替换目前在 `R.id.fragment_container` ID 所标识的布局容器中的任何片段（如有）。通过调用 `addToBackStack()` 可将替换事务保存到返回栈，以便用户能够通过按*返回*按钮撤消事务并回退到上一片段。

如果您向事务添加了多个更改（如又一个 `add()` 或 `remove()`），并且调用了 `addToBackStack()`，则在调用`commit()` 前应用的所有更改都将作为单一事务添加到返回栈，并且*返回*按钮会将它们一并撤消。

向 `FragmentTransaction` 添加更改的顺序无关紧要，不过：

- 您必须最后调用 `commit()`
- 如果您要向同一容器添加多个片段，则您添加片段的顺序将决定它们在视图层次结构中的出现顺序

如果您没有在执行移除片段的事务时调用 `addToBackStack()`，则事务提交时该片段会被销毁，用户将无法回退到该片段。 不过，如果您在删除片段时调用了 `addToBackStack()`，则系统会*停止*该片段，并在用户回退时将其恢复。
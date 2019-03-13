# Android [平台架构](https://developer.android.google.cn/guide/platform)

## 基本名词

1. 四大组件
2. 五种存储方式
3. 五大布局
4. 生命周期

## Activity

### 生命周期

### ![img](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)创建和使用

继承Activity，或者ListActivity, AppCompatActivity, PreferenceActivity等，在清单文件注册，调用startActivity(new Intent(context, YourActivity.class)) 启动你的Activity, 调用finish()关闭Activity。可使用startActivityForResult 接收Activity结果

[隐式启动Activity，以及获取 Activity 的结果](https://developer.android.google.cn/training/basics/intents/result)

### 启动模式

1. standard

    模式启动模式，每次激活Activity时都会创建Activity，并放入任务栈中。

2. singleTop

    如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否者就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。

3. singleTask

    如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent())。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。 

4. singleInstance

    在一个新栈中创建该Activity实例，并让多个应用共享改栈中的该Activity实例。一旦改模式的Activity的实例存在于某个栈中，任何应用再激活改Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中。





### 处理状态变更

1. 保存Activity状态

   [管理 Activity 生命周期](https://developer.android.google.cn/guide/components/activities.html#Lifecycle)的引言部分简要提及，当 Activity 暂停或停止时，Activity 的状态会得到保留。 确实如此，因为当 Activity 暂停或停止时，`Activity` 对象仍保留在内存中 — 有关其成员和当前状态的所有信息仍处于活动状态。 因此，用户在 Activity 内所做的任何更改都会得到保留，这样一来，当 Activity 返回前台（当它“继续”）时，这些更改仍然存在。

   不过，当系统为了恢复内存而销毁某项 Activity 时，`Activity` 对象也会被销毁，因此系统在继续 Activity 时根本无法让其状态保持完好，而是必须在用户返回 Activity 时重建 `Activity` 对象。但用户并不知道系统销毁 Activity 后又对其进行了重建，因此他们很可能认为 Activity 状态毫无变化。 在这种情况下，您可以实现另一个回调方法对有关 Activity 状态的信息进行保存，以确保有关 Activity 状态的重要信息得到保留：`onSaveInstanceState()`。

   系统会先调用 `onSaveInstanceState()`，然后再使 Activity 变得易于销毁。系统会向该方法传递一个 `Bundle`，您可以在其中使用 `putString()` 和`putInt()` 等方法以名称-值对形式保存有关 Activity 状态的信息。然后，如果系统终止您的应用进程，并且用户返回您的 Activity，则系统会重建该 Activity，并将 `Bundle` 同时传递给 `onCreate()` 和 `onRestoreInstanceState()`。您可以使用上述任一方法从 `Bundle` 提取您保存的状态并恢复该 Activity 状态。如果没有状态信息需要恢复，则传递给您的 `Bundle` 是空值（如果是首次创建该 Activity，就会出现这种情况）。

   ![img](https://developer.android.google.cn/images/fundamentals/restore_instance.png)

**图 2.** 在两种情况下，Activity 重获用户焦点时可保持状态完好：系统在销毁 Activity 后重建 Activity，Activity 必须恢复之前保存的状态；系统停止 Activity 后继续执行 Activity，并且 Activity 状态保持完好。

**注**：无法保证系统会在销毁您的 Activity 前调用 `onSaveInstanceState()`，因为存在不需要保存状态的情况（例如用户使用“返回”按钮离开您的 Activity 时，因为用户的行为是在显式关闭 Activity）。 如果系统调用 `onSaveInstanceState()`，它会在调用 `onStop()` 之前，并且可能会在调用`onPause()` 之前进行调用。

不过，即使您什么都不做，也不实现 `onSaveInstanceState()`，`Activity` 类的 `onSaveInstanceState()` 默认实现也会恢复部分 Activity 状态。具体地讲，默认实现会为布局中的每个 `View` 调用相应的 `onSaveInstanceState()` 方法，让每个视图都能提供有关自身的应保存信息。Android 框架中几乎每个小部件都会根据需要实现此方法，以便在重建 Activity 时自动保存和恢复对 UI 所做的任何可见更改。例如，`EditText` 小部件保存用户输入的任何文本，`CheckBox` 小部件保存复选框的选中或未选中状态。您只需为想要保存其状态的每个小部件提供一个唯一的 ID（通过 [`android:id`](https://developer.android.google.cn/guide/topics/resources/layout-resource.html#idvalue) 属性）。如果小部件没有 ID，则系统无法保存其状态。

您还可以通过将`android:saveEnabled` 属性设置为`"false"` 或通过调用`setSaveEnabled()` 方法显式阻止布局内的视图保存其状态。您通常不应将该属性停用，但如果您想以不同方式恢复 Activity UI 的状态，就可能需要这样做。

尽管 `onSaveInstanceState()` 的默认实现会保存有关您的Activity UI 的有用信息，您可能仍需替换它以保存更多信息。例如，您可能需要保存在 Activity 生命周期内发生了变化的成员值（它们可能与 UI 中恢复的值有关联，但默认情况下系统不会恢复储存这些 UI 值的成员）。

由于 `onSaveInstanceState()` 的默认实现有助于保存 UI 的状态，因此如果您为了保存更多状态信息而替换该方法，应始终先调用 `onSaveInstanceState()` 的超类实现，然后再执行任何操作。 同样，如果您替换`onRestoreInstanceState()` 方法，也应调用它的超类实现，以便默认实现能够恢复视图状态。

**注**：由于无法保证系统会调用 `onSaveInstanceState()`，因此您只应利用它来记录 Activity 的瞬态（UI 的状态）— 切勿使用它来存储持久性数据，而应使用 `onPause()` 在用户离开 Activity 后存储持久性数据（例如应保存到数据库的数据）。

您只需旋转设备，让屏幕方向发生变化，就能有效地测试您的应用的状态恢复能力。 当屏幕方向变化时，系统会销毁并重建 Activity，以便应用可供新屏幕配置使用的备用资源。 单凭这一理由，您的 Activity 在重建时能否完全恢复其状态就显得非常重要，因为用户在使用应用时经常需要旋转屏幕。

### 处理配置变更

有些设备配置可能会在运行时发生变化（例如屏幕方向、键盘可用性及语言）。 发生此类变化时，Android 会重建运行中的 Activity（系统调用`onDestroy()`，然后立即调用 `onCreate()`）。此行为旨在通过利用您提供的备用资源（例如适用于不同屏幕方向和屏幕尺寸的不同布局）自动重新加载您的应用来帮助它适应新配置。

如果您对 Activity 进行了适当设计，让它能够按以上所述处理屏幕方向变化带来的重启并恢复 Activity 状态，那么在遭遇 Activity 生命周期中的其他意外事件时，您的应用将具有更强的适应性。

正如上文所述，处理此类重启的最佳方法是利用`onSaveInstanceState()` 和 `onRestoreInstanceState()`（或 `onCreate()`）保存并恢复 Activity 的状态。

如需了解有关运行时发生的配置变更以及应对方法的详细信息，请阅读[处理运行时变更](https://developer.android.google.cn/guide/topics/resources/runtime-changes.html)指南。

### 协调 Activity

当一个 Activity 启动另一个 Activity 时，它们都会体验到生命周期转变。第一个 Activity 暂停并停止（但如果它在后台仍然可见，则不会停止）时，同时系统会创建另一个 Activity。 如果这些 Activity 共用保存到磁盘或其他地方的数据，必须了解的是，在创建第二个 Activity 前，第一个 Activity 不会完全停止。更确切地说，启动第二个 Activity 的过程与停止第一个 Activity 的过程存在重叠。

生命周期回调的顺序经过明确定义，当两个 Activity 位于同一进程，并且由一个 Activity 启动另一个 Activity 时，其定义尤其明确。 以下是当 Activity A 启动 Activity B 时一系列操作的发生顺序：

1. Activity A 的 `onPause()` 方法执行。
2. Activity B 的 `onCreate()`、`onStart()` 和 `onResume()` 方法依次执行。（Activity B 现在具有用户焦点。）
3. 然后，如果 Activity A 在屏幕上不再可见，则其 `onStop()` 方法执行。

您可以利用这种可预测的生命周期回调顺序管理从一个 Activity 到另一个 Activity 的信息转变。 例如，如果您必须在第一个 Activity 停止时向数据库写入数据，以便下一个 Activity 能够读取该数据，则应在 `onPause()` 而不是 `onStop()` 执行期间向数据库写入数据。



### 常用属性或场景

始终横屏 android:screenOrientation="portrait"

始终横屏 android:screenOrientation="landscape"

android:screenOrientation:

| "`unspecified`" | 默认值 由系统来判断显示方向.判定的策略是和设备相关的，所以不同的设备会有不同的显示方向. |
| --------------- | ------------------------------------------------------------ |
| "`landscape`"   | 横屏显示（宽比高要长）                                       |
| "`portrait`"    | 竖屏显示(高比宽要长)                                         |
| "`user`"        | 用户当前首选的方向                                           |
| "`behind`"      | 和该Activity下面的那个Activity的方向一致(在Activity堆栈中的) |
| "`sensor`"      | 有物理的感应器来决定。如果用户旋转设备这屏幕会横竖屏切换。   |
| "`nosensor`"    | 忽略物理感应器，这样就不会随着用户旋转设备而更改了 （ "`unspecified`"设置除外 ）。 |

旋转屏幕不重新创建Activity android:configChanges="keyboardHidden|orientation|screenSize"

监听配置变更，比如横竖屏切换： Activity.onConfigurationChanged



### IntentFilter

```
//默认启动Activity和Launcher图标
<intent-filter>
    <action android:name="android.intent.action.MAIN"/>

    <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>

```

//data，type等属性可以提供给其他App用隐式启动的方法来打开你的App，开放你的App功能给其他App使用，比如打开网页，打开文件等，然后使用getIntent().getData()，获取意图，判断，解析，打开你支持的文件

### [允许其他应用启动您的 Activity](https://developer.android.google.cn/training/basics/intents/filters)

### 与其他应用交互

```
Uri webpage = Uri.parse("http://www.android.com");
Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
```

### 验证是否存在接收 Intent 的应用

尽管 Android 平台保证某些 Intent 可以分解为内置应用之一（比如，“电话”、“电子邮件”或“日历”应用），您应在调用 Intent 之前始终包含确认步骤。

**注意：**如果您调用了 Intent，但设备上没有可用于处理 Intent 的应用，您的应用将崩溃。

要确认是否存在可响应 Intent 的可用 Activity，请调用 `queryIntentActivities()` 来获取能够处理您的 `Intent` 的 Activity 列表。如果返回的 `List` 不为空，您可以安全地使用该 Intent。例如：

```
PackageManager packageManager = getPackageManager();
List activities = packageManager.queryIntentActivities(intent,
        PackageManager.MATCH_DEFAULT_ONLY);
boolean isIntentSafe = activities.size() > 0;
```



如果 `isIntentSafe` 是 `true`，则至少有一个应用将响应该 Intent。 如果它是 `false`，则没有任何应用处理该 Intent。



### 显示应用选择器

![img](https://developer.android.google.cn/images/training/basics/intent-chooser.png)

**图 2.** 选择器对话框。

注意，当您通过将您的 `Intent` 传递至 `startActivity()` 而启动 Activity 时，有多个应用响应 Intent，用户可以选择默认使用哪个应用（通过选中对话框底部的复选框；见图 1）。当执行用户通常希望每次使用相同应用进行的操作时，比如当打开网页（用户可能只使用一个网络浏览器）或拍照（用户可能习惯使用一个相机）时，这非常有用。

但是，如果要执行的操作可由多个应用处理并且用户可能 习惯于每次选择不同的应用 — 比如“共享”操作， 用户有多个应用分享项目 — 您应明确显示选择器对话框， 如图 2 所示。选择器对话框 强制用户选择用于每次操作的 应用（用户不能对此操作选择默认的应用）。

要显示选择器，请使用 `createChooser()` 创建`Intent` 并将其传递给 `startActivity()`。例如：

```
Intent intent = new Intent(Intent.ACTION_SEND);
...

// Always use string resources for UI text.
// This says something like "Share this photo with"
String title = getResources().getString(R.string.chooser_title);
// Create intent to show chooser
Intent chooser = Intent.createChooser(intent, title);

// Verify the intent will resolve to at least one activity
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```



这将显示一个对话框，其中包含响应传递给 `createChooser()` 方法的 Intent 的应用列表，并且将提供的文本用作对话框标题。



### [多窗口支持](https://developer.android.google.cn/guide/topics/ui/multi-window)



## [Fragment](https://developer.android.google.cn/guide/components/fragments)

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



## Service 

### [关于Android Service真正的完全详解，你需要知道的一切 ](https://blog.csdn.net/javazejian/article/details/52709857)

### [Android Service使用详解](https://www.jianshu.com/p/95ec2a23f300)

Service是Android中实现程序后台运行的解决方案，非常适合用于去执行哪些不需要和用户交互而且还要求长期运行的任务。不能运行在一个独立的进程当中，而是依赖与创建服务时所在的应用程序进程。只能在后台运行，并且可以和其他组件进行交互。

Service可以在很多场合使用，比如播放多媒体的时候用户启动了其他Activity，此时要在后台继续播放；比如检测SD卡上文件的变化；比如在后台记录你的地理信息位置的改变等等，总之服务是藏在后台的。

服务不会自动开启线程，我们需要在服务的内部手动创建子线程，并在这里执行具体的任务。关于多线程的知识：可以参考另外一篇文章：[Android多线程----异步消息处理机制之Handler详解](http://www.cnblogs.com/smyhvae/p/4003922.html)





![img](http://upload-images.jianshu.io/upload_images/1981935-bd709d5989105a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Service简单概述
  Service是Android系统中的四大组件之一，主要有两个应用场景：后台运行和跨进程访问。Service可以在后台执行长时间运行操作而不提供用户界面，除非系统必须回收内存资源，否则系统不会停止或销毁服务。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)
 需要注意的是，Service是在主线程里执行操作的，可能会因为执行耗时操作而导致ANR



Service可以分为以下三种形式：

- 启动
   当应用组件通过调用 **startService()** 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方
- 绑定
   当应用组件通过调用 **bindService()** 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。多个组件可以同时绑定服务，服务只会在组件与其绑定时运行，一旦该服务与所有组件之间的绑定全部取消，系统便会销毁它
- 启动且绑定
   服务既可以是启动服务，也允许绑定。此时需要同时实现以下回调方法：**onStartCommand()**和 **onBind()**。系统不会在所有客户端都取消绑定时销毁服务。为此，必须通过调用 **stopSelf()** 或 **stopService()** 显式停止服务

无论应用是处于启动状态还是绑定状态，或者处于启动且绑定状态，任何应用组件均可像使用 Activity 那样通过调用 Intent 来使用服务（即使此服务来自另一应用），也可以通过清单文件将服务声明为私有服务，阻止其他应用访问

要使用服务，必须继承Service类（或者Service类的现有子类），在子类中重写某些回调方法，以处理服务生命周期的某些关键方面并提供一种机制将组件绑定到服务

- onStartCommand()
   当组件通过调用 **startService()** 请求启动服务时，系统将调用此方法（如果是绑定服务则不会调用此方法）。一旦执行此方法，服务即会启动并可在后台无限期运行。 在指定任务完成后，通过调用 **stopSelf()** 或 **stopService()** 来停止服务
- onBind()
   当一个组件想通过调用 **bindService()** 与服务绑定时，系统将调用此方法（如果是启动服务则不会调用此方法）。在此方法的实现中，必须通过返回 **IBinder** 提供一个接口，供客户端用来与服务进行通信
- onCreate()
   首次创建服务时，系统将调用此方法来执行初始化操作（在调用 **onStartCommand()** 或 **onBind()** 之前）。如果在启动或绑定之前Service已在运行，则不会调用此方法
- onDestroy()
   当服务不再使用且将被销毁时，系统将调用此方法，这是服务接收的最后一个调用，在此方法中应清理占用的资源

仅当内存过低必须回收系统资源以供前台 Activity 使用时，系统才会强制停止服务。如果将服务绑定到前台 Activity，则它不太可能会终止，如果将服务声明为在前台运行，则它几乎永远不会终止。或者，如果服务已启动并要长时间运行，则系统会随着时间的推移降低服务在后台任务列表中的位置，而服务也将随之变得非常容易被终止。如果服务是启动服务，则必须将其设计为能够妥善处理系统对它的重启。 如果系统终止服务，那么一旦资源变得再次可用，系统便会重启服务（这还取决于 onStartCommand() 的返回值）



Service在清单文件中的声明
  前面说过Service分为启动状态和绑定状态两种，但无论哪种具体的Service启动类型，都是通过继承Service基类自定义而来，也都需要在AndroidManifest.xml中声明，那么在分析这两种状态之前，我们先来了解一下Service在AndroidManifest.xml中的声明语法，其格式如下：

<service android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:isolatedProcess=["true" | "false"]
    android:label="string resource"
    android:name="string"
    android:permission="string"
    android:process="string" >
    . . .
</service>

android:exported：代表是否能被其他应用隐式调用，其默认值是由service中有无intent-filter决定的，如果有intent-filter，默认值为true，否则为false。为false的情况下，即使有intent-filter匹配，也无法打开，即无法被其他应用隐式调用。

android:name：对应Service类名

android:permission：是权限声明

android:process：是否需要在单独的进程中运行,当设置为android:process=”:remote”时，代表Service在单独的进程中运行。注意“：”很重要，它的意思是指要在当前进程名称前面附加上当前的包名，所以“remote”和”:remote”不是同一个意思，前者的进程名称为：remote，而后者的进程名称为：App-packageName:remote。

android:isolatedProcess ：设置 true 意味着，服务会在一个特殊的进程下运行，这个进程与系统其他进程分开且没有自己的权限。与其通信的唯一途径是通过服务的API(bind and start)。

android:enabled：是否可以被系统实例化，默认为 true因为父标签 也有 enable 属性，所以必须两个都为默认值 true 的情况下服务才会被激活，否则不会激活。 

首先要创建服务，必须创建 Service 的子类（或使用它的一个现有子类如IntentService）。

### Service绑定服务
  绑定服务是Service的另一种变形，当Service处于绑定状态时，其代表着客户端-服务器接口中的服务器。当其他组件（如 Activity）绑定到服务时（有时我们可能需要从Activity组建中去调用Service中的方法，此时Activity以绑定的方式挂靠到Service后，我们就可以轻松地方法到Service中的指定方法），组件（如Activity）可以向Service（也就是服务端）发送请求，或者调用Service（服务端）的方法，此时被绑定的Service（服务端）会接收信息并响应，甚至可以通过绑定服务进行执行进程间通信 (即IPC，这个后面再单独分析)。与启动服务不同的是绑定服务的生命周期通常只在为其他应用组件(如Activity)服务时处于活动状态，不会无限期在后台运行，也就是说宿主(如Activity)解除绑定后，绑定服务就会被销毁。那么在提供绑定的服务时，该如何实现呢？实际上我们必须提供一个 IBinder接口的实现类，该类用以提供客户端用来与服务进行交互的编程接口，该接口可以通过三种方法定义接口：

扩展 Binder 类 
  如果服务是提供给自有应用专用的，并且Service(服务端)与客户端相同的进程中运行（常见情况），则应通过扩展 Binder 类并从 onBind() 返回它的一个实例来创建接口。客户端收到 Binder 后，可利用它直接访问 Binder 实现中以及Service 中可用的公共方法。如果我们的服务只是自有应用的后台工作线程，则优先采用这种方法。 不采用该方式创建接口的唯一原因是，服务被其他应用或不同的进程调用。

使用 Messenger 
  Messenger可以翻译为信使，通过它可以在不同的进程中共传递Message对象(Handler中的Messager，因此 Handler 是 Messenger 的基础)，在Message中可以存放我们需要传递的数据，然后在进程间传递。如果需要让接口跨不同的进程工作，则可使用 Messenger 为服务创建接口，客户端就可利用 Message 对象向服务发送命令。同时客户端也可定义自有 Messenger，以便服务回传消息。这是执行进程间通信 (IPC) 的最简单方法，因为 Messenger 会在单一线程中创建包含所有请求的队列，也就是说Messenger是以串行的方式处理客户端发来的消息，这样我们就不必对服务进行线程安全设计了。

使用 AIDL 

由于Messenger是以串行的方式处理客户端发来的消息，如果当前有大量消息同时发送到Service(服务端)，Service仍然只能一个个处理，这也就是Messenger跨进程通信的缺点了，因此如果有大量并发请求，Messenger就会显得力不从心了，这时AIDL（Android 接口定义语言）就派上用场了， 但实际上Messenger 的跨进程方式其底层实现 就是AIDL，只不过android系统帮我们封装成透明的Messenger罢了 。因此，如果我们想让服务同时处理多个请求，则应该使用 AIDL。 在此情况下，服务必须具备多线程处理能力，并采用线程安全式设计。使用AIDL必须创建一个定义编程接口的 .aidl 文件。Android SDK 工具利用该文件生成一个实现接口并处理 IPC 的抽象类，随后可在服务内对其进行扩展。



以上3种实现方式，我们可以根据需求自由的选择，但需要注意的是大多数应用“都不会”使用 AIDL 来创建绑定服务，因为它可能要求具备多线程处理能力，并可能导致实现的复杂性增加。



### 管理服务的生命周期（从创建到销毁）有以下两种情况：

启动服务 
该服务在其他组件调用 startService() 时创建，然后无限期运行，且必须通过调用 stopSelf() 来自行停止运行。此外，其他组件也可以通过调用 stopService() 来停止服务。服务停止后，系统会将其销毁。

绑定服务 

该服务在另一个组件（客户端）调用 bindService() 时创建。然后，客户端通过 IBinder 接口与服务进行通信。客户端可以通过调用 unbindService() 关闭连接。多个客户端可以绑定到相同服务，而且当所有绑定全部取消后，系统即会销毁该服务。 （服务不必自行停止运行）



### IntentService

Service服务是Android四大组件之一，在Android中有着举足重轻的作用。Service服务是工作的UI线程中，当你的应用需要下载一个文件或者播放音乐等长期处于后台工作而有没有UI界面的时候，你肯定要用到Service+Thread来实现。因此你需要自己在Service服务里面实现一个Thread工作线程来下载文件或者播放音乐。然而你每次都需要自己去写一个Service+Thread来处理长期处于后台而没有UI界面的任务，这样显得很麻烦，没必要每次都去构建一个Service+Thread框架处理长期处于后台的任务。Google工程师给我们构建了一个方便开发者使用的这么一个框架---IntentService。

IntentService是一个基础类，用于处理Intent类型的异步任务请求。当客户端调用android.content.Context#startService(Intent)发送请求时，Service服务被启动，且在其内部构建一个工作线程来处理Intent请求。当工作线程执行结束，Service服务会自动停止。IntentService是一个抽象类，用户必须实现一个子类去继承它，且必须实现IntentService里面的抽象方法onHandleIntent来处理异步任务请求。

#### IntentService总结   

1. 子类需继承IntentService并且实现里面的onHandlerIntent抽象方法来处理intent类型的任务请求。
2. 子类需要重写默认的构造方法，且在构造方法中调用父类带参数的构造方法。
3. IntentService类内部利用HandlerThread+Handler构建了一个带有消息循环处理机制的后台工作线程，客户端只需调用Content#startService(Intent)将Intent任务请求放入后台工作队列中，且客户端无需关注服务是否结束，非常适合一次性的后台任务。比如浏览器下载文件，退出当前浏览器之后，下载任务依然存在后台，直到下载文件结束，服务自动销毁。
4. 只要当前IntentService服务没有被销毁，客户端就可以同时投放多个Intent异步任务请求，IntentService服务端这边是顺序执行当前后台工作队列中的Intent请求的，也就是每一时刻只能执行一个Intent请求，直到该Intent处理结束才处理下一个Intent。因为IntentService类内部利用HandlerThread+Handler构建的是一个单线程来处理异步任务。



## [BroadcastReceiver](https://www.jianshu.com/p/f348f6d7fe59)

在 Android 系统中，广播（Broadcast）是在组件之间传播数据的一种机制，这些组件可以位于不同的进程中，起到进程间通信的作用

**BroadcastReceiver** 是对发送出来的 **Broadcast** 进行过滤、接受和响应的组件。首先将要发送的消息和用于过滤的信息（Action，Category）装入一个 **Intent** 对象，然后通过调用 **Context.sendBroadcast()** 、 **sendOrderBroadcast()** 方法把 Intent 对象以广播形式发送出去。 广播发送出去后，所以已注册的 BroadcastReceiver 会检查注册时的 **IntentFilter** 是否与发送的 Intent 相匹配，若匹配则会调用 BroadcastReceiver 的 **onReceiver()** 方法

所以当我们定义一个 BroadcastReceiver 的时候，都需要实现 onReceiver() 方法。BroadcastReceiver 的生命周期很短，在执行 onReceiver() 方法时才有效，一旦执行完毕，该Receiver 的生命周期就结束了

Android中的广播分为两种类型，标准广播和有序广播

- 标准广播
   标准广播是一种完全异步执行的广播，在广播发出后所有的广播接收器会在同一时间接收到这条广播，之间没有先后顺序，效率比较高，且无法被截断
- 有序广播
   有序广播是一种同步执行的广播，在广播发出后同一时刻只有一个广播接收器能够接收到， 优先级高的广播接收器会优先接收，当优先级高的广播接收器的 onReceiver() 方法运行结束后，广播才会继续传递，且前面的广播接收器可以选择截断广播，这样后面的广播接收器就无法接收到这条广播了

### 注册方法

#### 静态注册

静态注册即在**清单文件**中为 **BroadcastReceiver** 进行注册，使用**< receiver >**标签声明，并在标签内用 **< intent-filter >** 标签设置过滤器。这种形式的 BroadcastReceiver 的生命周期伴随着整个应用，如果这种方式处理的是系统广播，那么不管应用是否在运行，该广播接收器都能接收到该广播

#### 动态注册

动态注册 BroadcastReceiver 是在代码中定义并设置好一个 **IntentFilter** 对象，然后在需要注册的地方调用 **Context.registerReceiver()** 方法，调用 **Context.unregisterReceiver()** 方法取消注册，此时就不需要在清单文件中注册 Receiver 了

#### 本地广播**LocalBroadcastManager**

本地广播是无法通过静态注册的方式来接收的，因为静态注册广播主要是为了在程序未启动的情况下也能接收广播，而本地广播是应用自己发送的，此时应用肯定是启动的了



#### 使用私有权限

使用动态注册广播接收器存在一个问题，即系统内的任何应用均可监听并触发我们的 Receiver 。通常情况下我们是不希望如此的

解决办法之一是在清单文件中为 **< receiver >** 标签添加一个 **android:exported="false"** 属性，标明该 Receiver 仅限应用内部使用。这样，系统中的其他应用就无法接触到该 Receiver 了



## [ContentProvider](https://www.jianshu.com/p/f5ec75a9cfea)



ContentProvider一般为存储和获取数据提供统一的接口，可以在不同的应用程序之间共享数据。

之所以使用ContentProvider，主要有以下几个理由：
 1，ContentProvider提供了对底层数据存储方式的抽象。比如下图中，底层使用了SQLite数据库，在用了ContentProvider封装后，即使你把数据库换成MongoDB，也不会对上层数据使用层代码产生影响。



![img](https:////upload-images.jianshu.io/upload_images/1362430-1857f913ab2e7a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/515/format/webp)

ContentProvider角色

2，Android框架中的一些类需要ContentProvider类型数据。如果你想让你的数据可以使用在如SyncAdapter, Loader, CursorAdapter等类上，那么你就需要为你的数据做一层ContentProvider封装。

3，第三个原因也是最主要的原因，是ContentProvider为应用间的数据交互提供了一个安全的环境。它准许你把自己的应用数据根据需求开放给其他应用进行增、删、改、查，而不用担心直接开放数据库权限而带来的安全问题。



ContentProvider进行增，删，改，查的操作使用ContentResolver

ContentResolver来统一管理与不同ContentProvider间的操作。



![img](https://upload-images.jianshu.io/upload_images/1362430-a336044d52818448.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/836/format/webp)



ContentResolver是如何来区别不同的ContentProvider的呢？这就涉及到URI（Uniform Resource Identifier）问题，对URI是什么还不明白的童鞋请自行Google。

### ContentProvider中的URI

ContentProvider中的URI有固定格式，如下图：





![img](https:////upload-images.jianshu.io/upload_images/1362430-b39bc91ec8e272af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/304/format/webp)

URI

Authority：

授权信息，用以区别不同的ContentProvider；

Path：

表名，用以区分ContentProvider中不同的数据表；



## [Intent](https://www.cnblogs.com/smyhvae/p/3959204.html)



## 存储

### 使用SharedPreferences存储数据

​    适用范围**：**保存少量的数据，且这些数据的格式非常简单：字符串型、基本类型的值。比如应用程序的各种配置信息（如是否打开音效、是否使用震动效果、小游戏的玩家积分等），解锁口 令密码等

​    核心原理**：**保存基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息。通过DDMS的File Explorer面板，展开文件浏览树,很明显SharedPreferences数据总是存储在/data/data/<package name>/shared_prefs目录下。SharedPreferences对象本身只能获取数据而不支持存储和修改,存储修改是通过SharedPreferences.edit()获取的内部接口Editor对象实现。 SharedPreferences本身是一 个接口，程序无法直接创建SharedPreferences实例，只能通过Context提供的getSharedPreferences(String name, int mode)方法来获取SharedPreferences实例，该方法中name表示要操作的xml文件名，第二个参数具体如下：

​                 **Context.MODE_PRIVATE**: 指定该SharedPreferences数据只能被本应用程序读、写。

​                 **Context.MODE_WORLD_READABLE**:  指定该SharedPreferences数据能被其他应用程序读，但不能写。

​                 ***Context.MODE_WORLD_WRITEABLE***:  指定该SharedPreferences数据能被其他应用程序读，写

Editor有如下主要重要方法：

​                 **SharedPreferences.Editor clear():**清空SharedPreferences里所有数据

​                 **SharedPreferences.Editor putXxx(String key , xxx value):** 向SharedPreferences存入指定key对应的数据，其中xxx 可以是boolean,float,int等各种基本类型据

​                 **SharedPreferences.Editor remove():** 删除SharedPreferences中指定key对应的数据项

​                 **boolean commit():** 当Editor编辑完成后，使用该方法提交修改



### 文件存储数据      

Context提供了两个方法来打开数据文件里的文件IO流 FileInputStream openFileInput(String name); FileOutputStream(String name , int mode),这两个方法第一个参数 用于指定文件名，第二个参数指定打开文件的模式。具体有以下值可选：

​             ***MODE_PRIVATE***：为默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容，如果想把新写入的内容追加到原文件中。可   以使用Context.MODE_APPEND

​             **MODE_APPEND**：模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。

​             ***MODE_WORLD_READABLE***：表示当前文件可以被其他应用读取；

​             **MODE_WORLD_WRITEABLE**：表示当前文件可以被其他应用写入。

 除此之外，Context还提供了如下几个重要的方法：

​             **getDir(String name , int mode)**:在应用程序的数据文件夹下获取或者创建name对应的子目录

​             **File getFilesDir()**:获取该应用程序的数据文件夹得绝对路径

​             **String[] fileList()**:返回该应用数据文件夹的全部文件



### SQLite数据库存储数据

嵌入式数据库，可通过SQLiteOpenHelper.getWriteableDatabase获取实例，然后进行增删改查操作

**SQLiteOpenHelper**

| 方法名                                                       | 方法描述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SQLiteOpenHelper(Context context,String name,SQLiteDatabase.CursorFactory factory,int version)** | 构造方法，其中context 程序上下文环境 即：XXXActivity.this;name :数据库名字;factory:游标工厂，默认为null,即为使用默认工厂;version 数据库版本号 |
| **onCreate(SQLiteDatabase db)**                              | 创建数据库时调用                                             |
| **onUpgrade(SQLiteDatabase db,int oldVersion , int newVersion)** | 版本更新时调用                                               |
| **getReadableDatabase()**                                    | 创建或打开一个只读数据库                                     |
| **getWritableDatabase()**                                    | 创建或打开一个读写数据库                                     |

### 使用ContentProvider存储数据

### 网络存储数据



## View

View是Android中所有控件的基类，不管是简单的TextView，Button还是复杂的LinearLayout和ListView，它们的共同基类都是View；
 View是一种界面层的控件的一种抽象，它代表了一个控件，除了View还有ViewGroup，从名字来看ViewGroup可以翻译为控件组，即一组View；
 在Android中，ViewGroup也继承了View，这就意味着View可以是单个控件，也可以是由多个控件组成的一组控件；

![img](https://img-blog.csdn.net/20160416151944699)

![img](https://img-blog.csdn.net/20160416152117873)

view提供的方法

getTop：获取到的，是view自身的顶边到其父布局顶边的距离 
getLeft：获取到的，是view自身的左边到其父布局左边的距离 
getRight：获取到的，是view自身的右边到其父布局左边的距离 
getBottom：获取到的，是view自身的底边到其父布局顶边的距离

MotionEvent提供的方法

getX()：获取点击事件相对控件左边的x轴坐标，即点击事件距离控件左边的距离 
getY()：获取点击事件相对控件顶边的y轴坐标，即点击事件距离控件顶边的距离 
getRawX()：获取点击事件相对整个屏幕左边的x轴坐标，即点击事件距离整个屏幕左边的距离 

getRawY()：获取点击事件相对整个屏幕顶边的y轴坐标，即点击事件距离整个屏幕顶边的距离

![img](https://img-blog.csdn.net/20160419001204719)

scrollTo是基于所传递参数的绝对移动，而scrollBy是基于当前位置的相对移动；就是To是我就移动到这个位置就不动了，By是基于我当前的位置继续偏移；

ScrollTo和ScrollBy滑动的是view的显示内容，并不改变view的坐标



## 适配

主要是高低版本适配

5.0

6.0 	动态权限

7.0 	多窗口支持

8.0 	通知渠道

​	启动图标



## 国际化

values-zh-rCN

values-en



## 打包

## 安全

### 混淆

## 性能优化

## 其他要点

### 序列化

Parcelable的性能要强于Serializable的原因

  1）. 在内存的使用中,前者在性能方面要强于后者

  2）. 后者在序列化操作的时候会产生大量的临时变量,(原因是使用了反射机制)从而导致GC的频繁调用,因此在性能上会稍微逊色

  3）. Parcelable是以Ibinder作为信息载体的.在内存上的开销比较小,因此在内存之间进行数据传递的时候,Android推荐使用Parcelable,既然是内存方面比价有优势,那么自然就要优先选择.

  4）. 在读写数据的时候,Parcelable是在内存中直接进行读写,而Serializable是通过使用IO流的形式将数据读写入在硬盘上.

### 获取状态栏高度

/** 
* 获取状态栏高度——方法1 
* */  
  int statusBarHeight1 = -1;  
  //获取status_bar_height资源的ID  
  int resourceId = getResources().getIdentifier("status_bar_height", "dimen", "android");  
  if (resourceId > 0) {  
    //根据资源ID获取响应的尺寸值  
    statusBarHeight1 = getResources().getDimensionPixelSize(resourceId);  
  }  
  Log.e("-------", "状态栏-方法1:" + statusBarHeight1)

/** 
 * 获取状态栏高度——方法2 
 * */  
    int statusBarHeight2 = -1;  
     try {  
    Class<?> clazz = Class.forName("com.android.internal.R$dimen");  
    Object object = clazz.newInstance();  
    int height = Integer.parseInt(clazz.getField("status_bar_height")  
              .get(object).toString());  
    statusBarHeight2 = getResources().getDimensionPixelSize(height);  
     } catch (Exception e) {  
    e.printStackTrace();  
     }  
     Log.e("-------", "状态栏-方法2:" + statusBarHeight2);

/** 
* 获取状态栏高度——方法3 
* 应用区的顶端位置即状态栏的高度 
* *注意*该方法不能在初始化的时候用 
* */  
  Rect rectangle= new Rect();  
  getWindow().getDecorView().getWindowVisibleDisplayFrame(rectangle);  
  //高度为rectangle.top-0仍为rectangle.top  
  Log.e("-------", "状态栏-方法3:" + rectangle.top);

/** 
* 获取状态栏高度——方法4 
* 状态栏高度 = 屏幕高度 - 应用区高度 
* *注意*该方法不能在初始化的时候用 
* */  
  //屏幕  
  DisplayMetrics dm = new DisplayMetrics();  
  getWindowManager().getDefaultDisplay().getMetrics(dm);  
  //应用区域  
  Rect outRect1 = new Rect();  
  getWindow().getDecorView().getWindowVisibleDisplayFrame(outRect1);  
  int statusBar = dm.heightPixels - outRect1.height();  //状态栏高度=屏幕高度-应用区域高度  
  Log.e("------------", "状态栏-方法4:" + statusBar);

### App字体是否随系统字体大小缩放

因为APP[界面](https://www.baidu.com/s?wd=%E7%95%8C%E9%9D%A2&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)中文字元素的放大或缩小，会影响APP的呈现效果。有的时候为了界面美观和可用，我们需要做下限制，使用系统默认的字体比例

Resources res = super.getResources();
Configuration config = new Configuration();
config.setToDefaults();

res.updateConfiguration(config, res.getDisplayMetrics());



### Resource

常用的缺省目录和对应资源类型在SDK帮助中有表格列出，简单摘抄如下

| **目录Directory** | **资源类型Resource Types**                                   |
| ----------------- | ------------------------------------------------------------ |
| res/animator      | 存放定义了property animations（android 3.0新定义的动画框架）的XML文件 |
| res/anim/         | 存放定义了补间动画（tweened animation）或逐帧动画（frame by frame animation）的XML文件。（该目录下也可以存放定义property animations的XML文件，但是最好还是分开存放） |
| res/raw/          | 存放直接复制到设备中的任意文件。它们无需编译，添加到你的应用程序编译产生的压缩文件中。要使用这些资源，可以调用Resources.openRawResource()，参数是资源的ID，即R.raw.*somefilename*。 |
| res/drawable/     | 存放能转换为绘制资源（Drawable Resource）的位图文件（后缀为.png, .9.png, .jpg, .gif的图像文件)或者定义了绘制资源的XML文件 |
| res/color/        | 存放定义了颜色状态列表资源(Color State List Resource)的XML文件 |
| res/layout/       | 存放定义了用户界面布局的XML文件                              |
| res/menu/         | 存放定义了应用程序菜单资源的XML文件                          |
| res/values/       | 存放定义了多种类型资源的XML文件这些资源的类型可以是字符串，数据，颜色、尺寸、样式等等，具体在后面详述 |
| res/xml/          | 存放任意的XML文件，在运行时可以通过调用Resources.getXML()读取 |

Resource适配：使用ResourcesCompat获取资源

### Activity转场动画

```
<item name="android:windowAnimationStyle">@style/ActivityAnim</item>
---
<style name="ActivityAnim" parent="@android:style/Animation.Activity">
    <item name="android:activityOpenEnterAnimation">@anim/enter_anim</item>
    <item name="android:activityOpenExitAnimation">@anim/exit_anim</item>
    <item name="android:activityCloseEnterAnimation">@anim/close_enter_anim</item>
    <item name="android:activityCloseExitAnimation">@anim/close_exit_anim</item>
</style>
```

### [ActivityOptionsCompat](https://blog.csdn.net/ss1168805219/article/details/53445063)

### 获取和设置屏幕亮度

一、获取屏幕的亮度



public static int getScreenBrightness(Activity activity) {

int value = 0;

ContentResolver cr = activity.getContentResolver();

try {

value = Settings.System.getInt(cr, Settings.System.SCREEN_BRIGHTNESS);

} catch (SettingNotFoundException e) {

}

return value;

}





二、设置屏幕亮度：



public static void setScreenBrightness(Activity activity, int value) {

​    WindowManager.LayoutParams params = activity.getWindow().getAttributes();

​    params.screenBrightness = value / 255f;

​    activity.getWindow().setAttributes(params);

}

## 

### Drawable

#### StateListDrawable 属性

```
drawable:引用的Drawable位图,我们可以把他放到最前面,就表示组件的正常状态~
state_focused:是否获得焦点
state_window_focused:是否获得窗口焦点
state_enabled:控件是否可用
state_checkable:控件可否被勾选,eg:checkbox
state_checked:控件是否被勾选
state_selected:控件是否被选择,针对有滚轮的情况
state_pressed:控件是否被按下
state_active:控件是否处于活动状态,eg:slidingTab
state_single:控件包含多个子控件时,确定是否只显示一个子控件
state_first:控件包含多个子控件时,确定第一个子控件是否处于显示状态
state_middle:控件包含多个子控件时,确定中间一个子控件是否处于显示状态
state_last:控件包含多个子控件时,确定最后一个子控件是否处于显示状态


```

### [开机启动App](https://blog.csdn.net/a872822645/article/details/74740544)

### AppWidget 小部件

### getDimension，getDimensionPixelOffset和getDimensionPixelSize的一点说明

**getDimension和getDimensionPixelOffset的功能类似，**

**都是获取某个dimen的值，但是如果单位是dp或sp，则需要将其乘以density**

**如果是px，则不乘。并且getDimension返回float，getDimensionPixelOffset返回int.**

**而getDimensionPixelSize则不管写的是dp还是sp还是px,都会乘以denstiy.**

### NetworkInfo

isAvailable 网络是否可用

isConnected 网络是否已连接

### [Handler、Thread、HandlerThread三者的区别](https://blog.csdn.net/weixin_41101173/article/details/79687313)



### [关于getChildFragmentManager()、 getFragmentManager()、getSupportFragmentManager()的使用](https://blog.csdn.net/u013531824/article/details/49333343) 



## [动画](https://www.jianshu.com/p/0eb89d43eea4)

## 调试 logcat adb TraceView等

### [Adb](https://blog.csdn.net/zhonglunshun/article/details/78362439)

#### 常用Adb命令

```
adb devices
adb start-server
adb kill-server
adb version
无线连接，需要借助USB线
adb tcpip 8888
adb connect ip:8888
adb disconnect ip:8888

查看应用列表格式：
adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]
查看所有应用 adb shell pm list packages


```

#### 安装APK
```
adb install [-lrtsdg] <path_to_apk>
adb install 后面可以跟一些可选参数来控制安装 APK 的行为，可用参数及含义如下：

参数	含义
-l	将应用安装到保护目录 /mnt/asec
-r	允许覆盖安装
-t	允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用
-s	将应用安装到 sdcard
-d	允许降级覆盖安装
-g	授予所有运行时权限

```

```
卸载apk adb uninstall [-k] <packagename>
<packagename> 表示应用的包名，-k 参数可选，表示卸载应用但保留数据和缓存目录。 

```



```
查看前台Activity adb shell dumpsys activity activities | grep mFocusedActivity

adb version 查看版本

adb devices 查看连接设备

adb install <apkfile> 安装

adb uninstall <packge> 卸载

adb start-server 启动Server

adb kill-server 停止Server

adb logcat 查看日志

adb get-serialno 获取序列号

adb shell pm list packages 列出所有包名

adb shell pm list packages -s 列出系统包名

adb shell pm list packages -3 列出第三方包名

adb shell pm list packages | grep qq grep过滤

adb shell pm clear <packagename> 清除应用数据

adb shell cat /sys/class/net/wlan0/address 获取MAC地址

adb shell getprop ro.product.model 查看设备型号

adb shell getprop ro.build.version.release 查看Android版本

adb shell wm size 查看分辨率

adb shell wm density 查看屏幕密度

```



### LeakCanary



## 测试

## 传感器

[基本介绍](https://www.xuebuyuan.com/3179424.html)

[传感器类型](https://source.android.google.cn/devices/sensors/sensor-types)



## OpenGL

## 官方库/第三方库

1. OkHttp
2. Retrofit
3. GreenDao
4. RxJava
5. RxAndroid
6. Gson



## 源码分析

1. Handler
2. View
3. ViewGroup
4. LruCache



## 更多问题

### Application

#### 作用

1. onCreate 初始化全局资源
2. registerComponentCallbacks() & unregisterComponentCallbacks() 注册和注销 `ComponentCallbacks2`回调接口

```java
registerComponentCallbacks(new ComponentCallbacks2() {
// 接口里方法下面会继续介绍
            @Override
            public void onTrimMemory(int level) {

            }

            @Override
            public void onLowMemory() {

            }

            @Override
            public void onConfigurationChanged(Configuration newConfig) {

            }
        });
```



3. onTrimMemory 通知 应用程序 当前内存使用情况（以内存级别进行识别）

   ![img](http://upload-images.jianshu.io/upload_images/944365-438db7cf11e0591b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. onLowMemory 监听 `Android`系统整体内存较低时刻

   应用场景：Android 4.0前 检测内存使用情况，从而避免被系统直接杀掉 & 优化应用程序的性能体验

   类似于 OnTrimMemory（）

   特别注意：OnTrimMemory（） & OnLowMemory（） 关系

   OnTrimMemory（）是 OnLowMemory（） Android 4.0后的替代 API
   OnLowMemory（） = OnTrimMemory（）中的TRIM_MEMORY_COMPLETE级别

   若想兼容Android 4.0前，请使用OnLowMemory（）；否则直接使用OnTrimMemory（）即可

5. ### onConfigurationChanged 监听 应用程序 配置信息的改变，如屏幕旋转等

6. ### registerActivityLifecycleCallbacks（） & unregisterActivityLifecycleCallbacks（）注册 / 注销对 应用程序内 所有`Activity`的生命周期监听

7. ### onTerminate 应用程序结束时调用 但该方法只用于Android仿真机测试，在Android产品机是不会调用的

#### 使用场景

- 初始化 应用程序级别 的资源，如全局对象、环境配置变量等
- 数据共享、数据缓存，如设置全局共享变量、方法等
- 获取应用程序当前的内存使用情况，及时释放资源，从而避免被系统杀死
- 监听 应用程序 配置信息的改变，如屏幕旋转等
- 监听应用程序内 所有Activity的生命周期

#### 生命周期

```java
public class App extends Application {

    @Override
    public void onCreate() {
        // 程序创建的时候执行
        Log.d(TAG, "onCreate");
        super.onCreate();
    }
    @Override
    public void onTerminate() {
        // 程序终止的时候执行
        Log.d(TAG, "onTerminate");
        super.onTerminate();
    }
    @Override
    public void onLowMemory() {
        // 低内存的时候执行
        Log.d(TAG, "onLowMemory");
        super.onLowMemory();
    }
    @Override
    public void onTrimMemory(int level) {
        // 程序在内存清理的时候执行
        Log.d(TAG, "onTrimMemory");
        super.onTrimMemory(level);
    }
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        Log.d(TAG, "onConfigurationChanged");
        super.onConfigurationChanged(newConfig);
    }
    
}
```



### 在ActionBar中，即便设置showAsAction="always"，items仍然在overflow中显示的问题

如果你为了兼容 Android 2.1 的版本使用了 Support 库，在 android 命名空间下showAsAction 属性是不可用的。Support 库会提供替代它的属性，你必须声明自己的 XML 命名空间，并且使用该命名空间作为属性前缀。（一个自定义 XML 命名空间需要以你的 app 名称为基础，但是可以取任何你想要的名称，它的作用域仅仅在你声明的文件之内。）



添加此命名空间 xmlns:app="<http://schemas.android.com/apk/res-auto>" ，使用app:showAsAction代替android:showAsAction。



### 设置Dialog背景透明

```html
<style name="TransparentBgDialog" parent="@android:style/Theme.Dialog">
    <item name="android:windowFrame">@null</item><!--边框-->
    <item name="android:windowIsFloating">true</item><!--是否浮现在activity之上-->
    <item name="android:windowIsTranslucent">true</item><!--半透明-->
    <item name="android:windowNoTitle">true</item><!--无标题-->
    <item name="android:windowBackground">@color/transparent</item><!--背景透明-->
    <item name="android:backgroundDimEnabled">true</item><!--模糊-->
</style>
```



### RecyclerView ListView 有Padding时滚动不到padding区的问题

```
clipToPadding=false

```

意思是padding非空的时候，viewgroup会根据padding值来裁剪子view。 
默认情况下，clipToPadding为true，表示会裁剪子view并且resize边界效果。 

实际上是clipToPadding影响的是绘制的时候是否绘制padding里的子view内容。可以这么理解，clipToPadding对measure和layout无影响，只会影响draw阶段。无论clipToPadding是true还是false，滚的时候，都会滚到padding里去，但是clipToPadding为true，就会clip，这样padding里的部分就看不到了。回头看看刚才的问题，滚动前，布局是考虑padding的，所以padding里没有child元素，然后滚动的时候会把child滚动到padding里去，而由于clipToPadding=false，不裁剪，所以我们能看到padding里的child。如果设置clipToPadding=true，那padding里的child就会被裁剪掉，我们就看不到了。



### [NestedScrollView嵌套RecyclerView时滑动不流畅问题的解决办法](https://blog.csdn.net/u010839880/article/details/52672489)

```
 LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        layoutManager.setSmoothScrollbarEnabled(true);
        layoutManager.setAutoMeasureEnabled(true);

        recyclerView.setLayoutManager(layoutManager);
        recyclerView.setHasFixedSize(true);
        recyclerView.setNestedScrollingEnabled(false);
```



### 修改drawableRight图片

```

Drawable nav_up=getResources().getDrawable(R.drawable.button_nav_up);
nav_up.setBounds(0, 0, nav_up.getMinimumWidth(), nav_up.getMinimumHeight());
textview1.setCompoundDrawables(null, null, nav_up, null);
```



### [ScrollView拖动回弹效果(包括横向和竖向)](http://blog.csdn.net/cc20032706/article/details/51917328)

### [Activity全局切换动画](http://blog.csdn.net/gdeer/article/details/51214255)

###  [ViewPager相互嵌套，里层ViewPager无法滑动](http://blog.csdn.net/gaojinshan/article/details/17953895)

### [Android抽象布局——include、merge 、ViewStub](http://blog.csdn.net/xyz_lmn/article/details/14524567)

### [ScrollView(滚动条)](http://www.runoob.com/w3cnote/android-tutorial-scrollview.html)

### [Android Attr、Style和Theme详解](https://www.jianshu.com/p/d448ee26671d)

### [【Android】编写Drawable XML绘制底部带指示条的背景](https://blog.csdn.net/u011494050/article/details/38392091)


  ### 屏幕方向的说明

Activity在屏幕当中显示的方向。属性值可以是下表中列出的一个值：

 

| "**unspecified**"      | 默认值，由系统来选择方向。它的使用策略，以及由于选择时特定的上下文环境，可能会因为设备的差异而不同。 |
| ---------------------- | ------------------------------------------------------------ |
| "**user**"             | 使用用户当前首选的方向。                                     |
| "**behind**"           | 使用Activity堆栈中与该Activity之下的那个Activity的相同的方向。 |
| "**landscape**"        | 横向显示（宽度比高度要大）                                   |
| "**portrait**"         | 纵向显示（高度比宽度要大）                                   |
| "**reverseLandscape**" | 与正常的横向方向相反显示，在API Level 9中被引入。            |
| "**reversePortrait**"  | 与正常的纵向方向相反显示，在API Level 9中被引入。            |
| "**sensorLandscape**"  | 横向显示，但是基于设备传感器，既可以是按正常方向显示，也可以反向显示，在API Level 9中被引入。 |
| "**sensorPortrait**"   | 纵向显示，但是基于设备传感器，既可以是按正常方向显示，也可以反向显示，在API Level 9中被引入。 |
| "**sensor**"           | 显示的方向是由设备的方向传感器来决定的。显示方向依赖与用户怎样持有设备；当用户旋转设备时，显示的方向会改变。但是，默认情况下，有些设备不会在所有的四个方向上都旋转，因此要允许在所有的四个方向上都能旋转，就要使用fullSensor属性值。 |
| "**fullSensor**"       | 显示的方向（4个方向）是由设备的方向传感器来决定的，除了它允许屏幕有4个显示方向之外，其他与设置为“sensor”时情况类似，不管什么样的设备，通常都会这么做。例如，某些设备通常不使用纵向倒转或横向反转，但是使用这个设置，还是会发生这样的反转。这个值在API Level 9中引入。 |
| "**nosensor**"         | 屏幕的显示方向不会参照物理方向传感器。传感器会被忽略，所以显示不会因用户移动设备而旋转。除了这个差别之外，系统会使用与“unspecified”设置相同的策略来旋转屏幕的方向。 |



 

注意：在给这个属性设置的值是“landscape”或portrait的时候，要考虑硬件对Activity运行的方向要求。正因如此，这些声明的值能够被诸如Google Play这样的服务所过滤，以便应用程序只能适用于那些支持Activity所要求的方向的设备。例如，如果声明了“landscape”、“reverseLandscape”、或“sensorLandscape”，那么应用程序就只能适用于那些支持横向显示的设备。但是，还应该使用<uses-feature>元素来明确的声明应用程序所有的屏幕方向是纵向的还是横行的。例如：<uses-feature android:name="android.hardware.screen.portrait"/>。这个设置由Google Play提供的纯粹的过滤行为，并且在设备仅支持某个特定的方向时，平台本身并不控制应用程序是否能够被按照。

### [VideoView播放视频是出现黑边的问题](https://blog.csdn.net/keep_driving_xinyang/article/details/50607601)

### [ViewPager的高度根据item的高度自适应 ](https://blog.csdn.net/mchenys/article/details/54353136)

### WebView

WebSettings webSettings = webview.getSettings();

​        //设置双击变大变小

​        webSettings.setUseWideViewPort(true);

​         要支持缩放，肯定要先支持Javascript，加如下代码：



​        //支持JS

​        WebSettings settings = mWebView.getSettings();

​        settings.setJavaScriptEnabled(true);



​        //支持屏幕缩放,重点来了，要想支持缩放，要加如下代码支持

​        settings.setSupportZoom(true);

​        settings.setBuiltInZoomControls(true);

​        其中settings.setBuiltInZoomControls(true)必须要加，不然缩放不起作用，笔者就曾在这掉过坑

​                //加载的html代码中，必须去掉这一句。这句加上就不支持缩放了。

​            //<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />



​        //不让从当前网页跳转到系统的浏览器中

​        webview.setWebViewClient(new WebViewClient() {

​            //当加载页面完成的时候回调

​            @Override

​            public void onPageFinished(WebView view, String url) {

​                super.onPageFinished(view, url);

​                pbLoading.setVisibility(View.GONE);

​            }

​        });

​        webview.loadUrl(url);



​          //设置字体大小

​          webSettings.setTextZoom(200);







//设置支持js调用java

​        webView.addJavascriptInterface(new AndroidAndJSInterface(),"Android");







​    /**

​     \* js可以调用该类的方法

​     */

​    class AndroidAndJSInterface{

​        @JavascriptInterface

​        public void showToast(){

​            Toast.makeText(JavaAndJSActivity.this, "我被js调用了", Toast.LENGTH_SHORT).show();

​        }

​    }



//js调用java的方法

<input type="button" value="点击Android被调用" onclick="window.android.showToast()" />




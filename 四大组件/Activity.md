# Activity

## 生命周期


![img](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)

## 创建和使用

继承Activity，或者ListActivity, AppCompatActivity, PreferenceActivity等，在清单文件注册，调用startActivity(new Intent(context, YourActivity.class)) 启动你的Activity, 调用finish()关闭Activity。可使用startActivityForResult 接收Activity结果

[隐式启动Activity，以及获取 Activity 的结果](https://developer.android.google.cn/training/basics/intents/result)

## 启动模式

1. standard

   模式启动模式，每次激活Activity时都会创建Activity，并放入任务栈中。

2. singleTop

   如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否者就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。

3. singleTask

   如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent())。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。 

4. singleInstance

   在一个新栈中创建该Activity实例，并让多个应用共享改栈中的该Activity实例。一旦改模式的Activity的实例存在于某个栈中，任何应用再激活改Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中。





## 处理状态变更

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

## 处理配置变更

有些设备配置可能会在运行时发生变化（例如屏幕方向、键盘可用性及语言）。 发生此类变化时，Android 会重建运行中的 Activity（系统调用`onDestroy()`，然后立即调用 `onCreate()`）。此行为旨在通过利用您提供的备用资源（例如适用于不同屏幕方向和屏幕尺寸的不同布局）自动重新加载您的应用来帮助它适应新配置。

如果您对 Activity 进行了适当设计，让它能够按以上所述处理屏幕方向变化带来的重启并恢复 Activity 状态，那么在遭遇 Activity 生命周期中的其他意外事件时，您的应用将具有更强的适应性。

正如上文所述，处理此类重启的最佳方法是利用`onSaveInstanceState()` 和 `onRestoreInstanceState()`（或 `onCreate()`）保存并恢复 Activity 的状态。

如需了解有关运行时发生的配置变更以及应对方法的详细信息，请阅读[处理运行时变更](https://developer.android.google.cn/guide/topics/resources/runtime-changes.html)指南。

## 协调 Activity

当一个 Activity 启动另一个 Activity 时，它们都会体验到生命周期转变。第一个 Activity 暂停并停止（但如果它在后台仍然可见，则不会停止）时，同时系统会创建另一个 Activity。 如果这些 Activity 共用保存到磁盘或其他地方的数据，必须了解的是，在创建第二个 Activity 前，第一个 Activity 不会完全停止。更确切地说，启动第二个 Activity 的过程与停止第一个 Activity 的过程存在重叠。

生命周期回调的顺序经过明确定义，当两个 Activity 位于同一进程，并且由一个 Activity 启动另一个 Activity 时，其定义尤其明确。 以下是当 Activity A 启动 Activity B 时一系列操作的发生顺序：

1. Activity A 的 `onPause()` 方法执行。
2. Activity B 的 `onCreate()`、`onStart()` 和 `onResume()` 方法依次执行。（Activity B 现在具有用户焦点。）
3. 然后，如果 Activity A 在屏幕上不再可见，则其 `onStop()` 方法执行。

您可以利用这种可预测的生命周期回调顺序管理从一个 Activity 到另一个 Activity 的信息转变。 例如，如果您必须在第一个 Activity 停止时向数据库写入数据，以便下一个 Activity 能够读取该数据，则应在 `onPause()` 而不是 `onStop()` 执行期间向数据库写入数据。



## 常用属性或场景

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



## IntentFilter

```
//默认启动Activity和Launcher图标
<intent-filter>
    <action android:name="android.intent.action.MAIN"/>

    <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>

```

//data，type等属性可以提供给其他App用隐式启动的方法来打开你的App，开放你的App功能给其他App使用，比如打开网页，打开文件等，然后使用getIntent().getData()，获取意图，判断，解析，打开你支持的文件

## [允许其他应用启动您的 Activity](https://developer.android.google.cn/training/basics/intents/filters)

## 与其他应用交互

```
Uri webpage = Uri.parse("http://www.android.com");
Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
```

## 验证是否存在接收 Intent 的应用

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



## 显示应用选择器

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


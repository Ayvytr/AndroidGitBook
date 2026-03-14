# Activity

## 生命周期


![img](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)

## 创建和使用

继承Activity，或者ListActivity, AppCompatActivity, PreferenceActivity等，在清单文件注册，调用startActivity(new Intent(context, YourActivity.class)) 启动你的Activity, 调用finish()关闭Activity。可使用startActivityForResult 接收Activity结果

[隐式启动Activity，以及获取 Activity 的结果](https://developer.android.google.cn/training/basics/intents/result)

## 启动模式

1. standard

   默认启动模式，每次激活Activity时都会创建Activity，并放入任务栈中。

2. singleTop

   如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否则就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。

3. singleTask

   如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent()。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。 

4. singleInstance

   在一个新栈中创建该Activity实例，并让多个应用共享该栈中的该Activity实例。一旦该模式的Activity的实例存在于某个栈中，任何应用再激活该Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中。



## 在Android系统中，每个应用程序默认拥有一个独立的任务栈（Task Stack），这是由系统设计决定的，主要出于以下原因：

### 多任务管理需求

Android系统支持多任务并行运行，每个应用程序需要独立的任务栈来管理其内部Activity的启动、运行和销毁。例如，用户可能同时运行多个应用（如浏览器、通讯录等），每个应用需独立维护其界面状态和历史记录。 ‌

### 进程隔离与资源保护

任务栈通过进程隔离机制保障不同应用间的数据安全。若多个应用共享同一任务栈，可能导致数据泄露或异常交互，例如恶意应用可能通过任务栈漏洞访问其他应用的敏感信息。 ‌

### 用户体验优化

独立任务栈可避免界面跳转干扰。例如，当用户从浏览器切换回聊天应用时，若两者共享任务栈，浏览器历史记录可能被误触显示。独立任务栈确保应用间互不干扰，提升操作流畅度。 ‌

### 兼容性与扩展性

Android系统支持跨设备多任务（如平板模式），每个设备或窗口对应独立任务栈。这种设计便于系统扩展至多屏交互场景，同时保持对传统单设备场景的兼容性



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


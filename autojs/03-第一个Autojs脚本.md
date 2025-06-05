# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 第三章：第一个 Auto.js 脚本

---

## 一、理论讲解：Auto.js 脚本基础

Auto.js 脚本基于 JavaScript 语法，结合 Android 系统的无障碍服务，实现对手机界面和应用的自动化操作。初学者只需掌握基本 JS 语法和 Auto.js 常用 API，即可快速上手。

### 1.1 脚本结构
- 脚本入口：从上到下顺序执行
- 常用 API：toast、auto、app、text、click、sleep 等
- 支持模块化、异步、UI 交互等高级特性

### 1.2 运行方式
- 在 Auto.js App 内新建脚本，粘贴代码后点击"运行"
- 支持定时、快捷方式、悬浮窗等多种触发方式

---

## 二、代码示例：Hello World 与基础操作

### 2.1 Hello World

```javascript
// 最简单的弹窗
toast("Hello, Auto.js!");
```

### 2.2 自动点击与查找控件

```javascript
// 自动查找并点击"设置"按钮
auto.waitFor(); // 等待无障碍服务
if (text("设置").exists()) {
    text("设置").findOne().click();
    toast("已点击设置");
} else {
    toast("未找到设置按钮");
}
```

### 2.3 自动启动 App

```javascript
// 启动微信
app.launchApp("微信");
toast("已启动微信");
```

### 2.4 组合操作：自动打开设置并切换 WiFi

```javascript
// 自动打开设置并切换 WiFi
app.launchApp("设置");
sleep(2000);
if (text("WLAN").exists()) {
    text("WLAN").findOne().click();
    toast("已进入 WLAN 设置");
}
```

### 2.5 自动输入文本

```javascript
// 自动输入内容到输入框
app.launchApp("备忘录");
sleep(2000);
if (className("android.widget.EditText").exists()) {
    className("android.widget.EditText").findOne().setText("自动输入内容示例");
    toast("已输入内容");
}
```

### 2.6 滑动操作

```javascript
// 向上滑动屏幕
swipe(device.width/2, device.height*0.8, device.width/2, device.height*0.2, 500);
toast("已滑动屏幕");
```

### 2.7 截图并保存

```javascript
// 需要在设置中开启截图权限
if (requestScreenCapture()) {
    var img = captureScreen();
    images.save(img, "/sdcard/autojs_screenshot.png");
    toast("截图已保存");
}
```

### 2.8 循环查找与条件判断

```javascript
// 循环查找某个按钮并点击，最多尝试5次
for (let i = 0; i < 5; i++) {
    let btn = text("同意").findOne(1000);
    if (btn) {
        btn.click();
        toast("已点击同意");
        break;
    } else {
        sleep(1000);
    }
}
```

### 2.9 处理弹窗

```javascript
// 检查并自动关闭弹窗
if (text("关闭").exists()) {
    text("关闭").findOne().click();
    toast("已关闭弹窗");
}
```

### 2.10 定时任务

```javascript
// 每隔10秒执行一次任务
setInterval(function() {
    toast("定时任务执行中");
}, 10000);
```

### 2.11 模块化调用

```javascript
// modules/utils.js
function sayHello(name) {
    toast("你好，" + name + "！");
}
module.exports = { sayHello };

// main.js
let utils = require("./modules/utils.js");
utils.sayHello("鲫小鱼");
```

### 2.12 长按操作

```javascript
// 长按某个控件（如按钮）
var btn = text("删除").findOne(3000);
if (btn) {
    longClick(btn.bounds().centerX(), btn.bounds().centerY());
    toast("已长按删除按钮");
}
```

### 2.13 下拉刷新

```javascript
// 下拉刷新操作
swipe(device.width/2, device.height*0.3, device.width/2, device.height*0.7, 500);
toast("已下拉刷新");
```

### 2.14 获取剪贴板内容

```javascript
// 获取剪贴板内容
var clip = getClip();
toast("剪贴板内容：" + clip);
```

### 2.15 设置剪贴板内容

```javascript
// 设置剪贴板内容
setClip("Auto.js 赋值测试");
toast("已设置剪贴板内容");
```

### 2.16 发送通知

```javascript
// 发送系统通知
importClass(android.app.NotificationManager);
importClass(android.app.Notification);
var manager = context.getSystemService(context.NOTIFICATION_SERVICE);
var notification = new Notification.Builder(context)
    .setContentTitle("Auto.js 通知")
    .setContentText("这是一个自动化脚本通知")
    .setSmallIcon(android.R.drawable.ic_dialog_info)
    .build();
manager.notify(1, notification);
```

### 2.17 监听按键

```javascript
// 监听音量下键退出脚本
events.observeKey();
events.onKeyDown("volume_down", function(event) {
    toast("检测到音量下键，脚本退出");
    exit();
});
```

### 2.18 弹出输入框

```javascript
// 弹出输入框获取用户输入
var input = rawInput("请输入要搜索的内容：");
toast("你输入的是：" + input);
```

### 2.19 判断网络状态

```javascript
// 判断当前是否有网络连接
if (device.isWifiConnected() || device.isDataEnabled()) {
    toast("网络已连接");
} else {
    toast("无网络连接");
}
```

### 2.20 打开网页

```javascript
// 调用系统浏览器打开网页
app.openUrl("https://www.autojs.org/");
```

### 2.21 调用系统分享

```javascript
// 调用系统分享面板
app.sendBroadcast({
    action: "android.intent.action.SEND",
    type: "text/plain",
    packageName: "com.android.chrome",
    extras: {
        "android.intent.extra.TEXT": "Auto.js 分享测试"
    }
});
```

### 2.22 录音与播放音频

```javascript
// 录音和播放音频（需相关权限）
media.playMusic("/sdcard/Music/test.mp3");
sleep(5000);
media.stopMusic();
```

### 2.23 震动反馈

```javascript
// 震动 300 毫秒
device.vibrate(300);
```

### 2.24 获取当前应用包名

```javascript
// 获取当前前台应用包名
toast(currentPackage());
```

### 2.25 获取屏幕分辨率

```javascript
// 获取屏幕宽高
toast("分辨率：" + device.width + "x" + device.height);
```

---

## 三、实战项目：自动亮屏并解锁手机

### 3.1 需求分析
- 自动点亮屏幕
- 自动滑动解锁
- 兼容不同机型的解锁方式

### 3.2 代码实现

```javascript
// 自动亮屏并解锁
if (!device.isScreenOn()) {
    device.wakeUp();
    sleep(1000);
}
swipe(device.width/2, device.height*0.8, device.width/2, device.height*0.2, 500); // 向上滑动解锁
```

---

## 四、常见问题与解决方案

| 问题                   | 解决方案                                   |
|----------------------|------------------------------------------|
| 脚本无反应              | 检查无障碍服务、悬浮窗权限是否开启               |
| 控件找不到/点击无效        | 尝试 text/desc/id 多种查找方式，或增加等待时间      |
| 脚本报错                | 查看日志输出，定位具体报错行                   |
| 运行后界面无变化           | 检查目标 App 是否已启动，控件是否在当前页面         |

---

## 五、性能优化建议
- 控件查找建议加超时，避免死循环
- 合理使用 sleep，避免无效等待
- 复杂操作建议拆分为函数，提升可维护性
- 多用日志输出，便于调试和排查

---

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！
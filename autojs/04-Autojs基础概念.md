# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 第四章：Auto.js 基础概念

---

## 一、理论讲解：Auto.js 核心基础

Auto.js 作为一款强大的 Android 自动化工具，核心基础包括控件操作、界面布局、事件处理和权限管理。理解这些基础概念，是实现高效自动化脚本开发的前提。

### 1.1 控件操作
- 通过 text、desc、id、className 等多种方式查找控件
- 支持 findOne、find、exists、click、setText 等常用方法
- 可结合 bounds、parent、child 等属性实现复杂控件定位
- 支持多层级遍历、批量操作、属性获取与设置
- 支持正则匹配、模糊查找、动态控件适配

#### 1.1.1 常用控件查找方式
```javascript
// 通过文本查找
var btn = text("登录").findOne();
// 通过描述查找
var img = desc("头像").findOne();
// 通过 id 查找
var input = id("com.xx.app:id/input").findOne();
// 通过 className 查找
var list = className("android.widget.ListView").findOne();
```

#### 1.1.2 层级遍历与属性获取
```javascript
// 遍历所有按钮并输出文本
var btns = className("android.widget.Button").find();
btns.forEach(btn => log(btn.text()));

// 获取控件的父节点、子节点、坐标
var node = text("设置").findOne();
log(node.parent().className());
log(node.childCount());
log(node.bounds());
```

#### 1.1.3 复杂定位与批量操作
```javascript
// 查找所有"删除"按钮并依次点击
var dels = text("删除").find();
dels.forEach(d => d.click());

// 正则模糊查找
var regBtn = textMatches(/(确定|OK|Yes)/).findOne();
regBtn && regBtn.click();
```

#### 1.1.4 动态控件适配
```javascript
// 兼容不同版本控件 id
var btn = id("btn_ok").exists() ? id("btn_ok").findOne() : text("确定").findOne();
btn && btn.click();
```

---

### 1.2 界面布局
- 支持 UI 脚本（"ui" 语法），可自定义界面
- 常用布局：vertical、horizontal、frame、scroll、drawer、card 等
- 支持 text、button、input、list、webview、switch、checkbox、progress 等控件
- 支持嵌套布局、动态添加控件、主题切换、数据绑定

#### 1.2.1 多种布局示例
```javascript
"ui";
ui.layout(
    <drawer id="drawer">
        <vertical>
            <appbar>
                <toolbar id="toolbar" title="多布局示例" />
            </appbar>
            <scroll>
                <vertical>
                    <text text="欢迎使用 Auto.js" textSize="20sp" />
                    <button id="btn1" text="按钮1" />
                    <button id="btn2" text="按钮2" />
                    <input id="input" hint="请输入内容" />
                    <list id="list">
                        <vertical>
                            <text text="{{this}}" />
                        </vertical>
                    </list>
                </vertical>
            </scroll>
        </vertical>
        <vertical layout_gravity="left">
            <text text="侧边栏内容" />
        </vertical>
    </drawer>
);
ui.list.setData(["A", "B", "C"]);
```

#### 1.2.2 动态添加控件与数据绑定
```javascript
// 动态添加按钮
def addBtn(txt) {
    let btn = ui.inflate(<button text={txt} />, ui.layout, false);
    ui.layout.addView(btn);
}
addBtn("新按钮");

// 数据绑定
ui.input.on("change", () => {
    ui.btn1.setText(ui.input.text());
});
```

#### 1.2.3 弹窗与对话框
```javascript
// 简单弹窗
dialog("这是一个弹窗");
// 输入对话框
ui.inputDialog("请输入内容", "默认值", (input) => toast("你输入了：" + input));
```

---

### 1.3 事件处理
- 支持点击、长按、滑动、输入、切换、选中等事件
- 可监听按键、悬浮窗、定时任务、全局广播、手势等
- 支持 events 模块实现全局事件监听、回调、解绑

#### 1.3.1 常用事件绑定
```javascript
ui.btn1.on("click", () => toast("点击了按钮1"));
ui.btn2.on("long_click", () => toast("长按按钮2"));
ui.list.on("item_click", (item, i) => toast("点击了第" + i + "项"));
```

#### 1.3.2 定时器与异步事件
```javascript
// 定时执行任务
setInterval(() => log("定时任务执行中"), 5000);
// 延时执行
timers.setTimeout(() => toast("延时3秒"), 3000);
```

#### 1.3.3 全局事件监听
```javascript
// 监听返回键退出脚本
events.observeKey();
events.on("key_down", function(event) {
    if (event.keyCode === 4) {
        toast("返回键退出");
        exit();
    }
});
```

#### 1.3.4 悬浮窗拖动与手势
```javascript
// 悬浮窗拖动
floaty.window.setOnTouchListener(function(view, event) {
    // 处理拖动逻辑
    return true;
});
// 手势监听
setGesture(1000, [
    [300, 1000], [300, 500]
]);
```

---

### 1.4 权限管理
- 主要权限：无障碍服务、悬浮窗、存储、录音、截图、定位、网络等
- 可通过代码检测和请求权限
- 动态适配不同 Android 版本权限
- 权限不足会导致脚本无法正常运行，需友好提示

#### 1.4.1 权限检测与动态申请
```javascript
// 检查无障碍服务
if (!auto.service) {
    toast("请先开启无障碍服务");
    exit();
}
// 检查悬浮窗权限
if (!floaty.checkPermission()) {
    floaty.requestPermission();
    toast("请授予悬浮窗权限");
    exit();
}
// 检查存储权限
if (!files.isWritable("/sdcard/")) {
    toast("请授予存储权限");
}
```

#### 1.4.2 异常处理与适配
```javascript
try {
    // 可能抛出权限异常的操作
    var img = captureScreen();
} catch (e) {
    toast("截图失败，请检查权限");
}
```

---

## 二、代码示例：基础用法

### 2.1 控件查找与操作

```javascript
// 查找并点击"登录"按钮
auto.waitFor();
var btn = text("登录").findOne(5000);
if (btn) {
    btn.click();
    toast("已点击登录");
}
```

### 2.2 UI 脚本自定义界面

```javascript
"ui";
ui.layout(
    <vertical padding="16">
        <text textSize="24sp" textColor="#222222">Auto.js 自定义界面示例</text>
        <input id="input" hint="请输入内容" />
        <button id="btn" text="提交" />
        <text id="result" textColor="#666666" />
    </vertical>
);
ui.btn.on("click", () => {
    ui.result.setText("你输入的是：" + ui.input.text());
});
```

### 2.3 事件监听

```javascript
// 监听音量上键退出脚本
events.observeKey();
events.onKeyDown("volume_up", function(event) {
    toast("检测到音量上键，脚本退出");
    exit();
});
```

### 2.4 权限检测与请求

```javascript
// 检查无障碍服务
if (!auto.service) {
    toast("请先开启无障碍服务");
    exit();
}
// 检查悬浮窗权限
if (!floaty.checkPermission()) {
    floaty.requestPermission();
    toast("请授予悬浮窗权限");
    exit();
}
```

### 2.5 批量操作与复杂控件
```javascript
// 批量点击所有"同意"按钮
text("同意").find().forEach(btn => btn.click());

// 获取所有输入框并依次输入内容
className("android.widget.EditText").find().forEach((input, i) => input.setText("内容" + i));
```

### 2.6 获取控件属性与状态
```javascript
var btn = text("提交").findOne();
if (btn) {
    log("是否可见：" + btn.visibleToUser());
    log("是否可点击：" + btn.clickable());
    log("控件文本：" + btn.text());
}
```

### 2.7 复杂 UI 交互
```javascript
// 多步操作：查找、滑动、点击、输入
var list = className("android.widget.ListView").findOne();
if (list) {
    list.scrollForward();
    sleep(500);
    var item = text("目标项").findOne();
    item && item.click();
}
```

### 2.8 动态 UI 与主题切换
```javascript
// 动态切换主题
ui.theme.setTheme("dark");
```

---

## 三、实战项目：多功能悬浮窗工具箱

### 3.1 需求分析
- 实现一个可拖动的悬浮窗，包含快捷按钮、日志输出、主题切换、数据存储
- 支持一键启动常用功能（如截图、自动点击、定时任务、录音、分享等）
- 支持自定义快捷方式、配置保存、日志查看

### 3.2 代码实现（简化版）
```javascript
"ui";
ui.layout(
    <vertical>
        <button id="btnScreenshot" text="截图" />
        <button id="btnClick" text="自动点击" />
        <button id="btnTheme" text="切换主题" />
        <button id="btnLog" text="查看日志" />
        <button id="btnSave" text="保存配置" />
        <text id="log" textColor="#888888" />
    </vertical>
);
ui.btnScreenshot.on("click", () => {
    toast("截图功能待实现");
    // 可调用 requestScreenCapture + captureScreen
});
ui.btnClick.on("click", () => {
    toast("自动点击功能待实现");
});
ui.btnTheme.on("click", () => {
    // 切换主题
    toast("切换主题");
});
ui.btnLog.on("click", () => {
    ui.log.setText("日志内容示例");
});
ui.btnSave.on("click", () => {
    files.write("/sdcard/autojs_config.json", JSON.stringify({ time: Date.now() }));
    toast("配置已保存");
});
```

---

## 四、常见问题与解决方案

| 问题                   | 解决方案                                   |
|----------------------|------------------------------------------|
| 控件找不到/点击无效        | 检查控件属性、增加等待时间、尝试多种查找方式         |
| UI 脚本报错                | 检查布局语法、控件 id 是否重复                   |
| 事件监听无效               | 检查 events 是否正确引入、权限是否足够             |
| 权限请求无效               | 检查系统设置、重启 App 或手机                    |
| 脚本卡顿/闪退              | 检查死循环、内存泄漏、日志输出过多                 |
| 兼容性问题                 | 适配不同 Android 版本、不同品牌手机                |
| UI 适配问题                | 合理使用 dp/sp 单位，避免硬编码像素                |
| 录音/截图/存储失败         | 检查相关权限、路径、设备支持情况                   |
| 悬浮窗无法拖动/消失         | 检查悬浮窗权限、脚本逻辑、系统设置                 |

---

## 五、性能优化建议
- 控件查找建议加超时，避免死循环
- UI 脚本布局尽量简洁，减少嵌套层级
- 事件监听及时移除，避免内存泄漏
- 权限检测放在脚本入口，提升用户体验
- 合理拆分模块，复用通用逻辑，减少重复代码
- 日志分级输出，开发阶段多输出，正式运行时关闭或降级
- 异步处理耗时操作，避免阻塞主线程
- 定期清理无用变量、释放资源，降低内存占用
- 合理使用 sleep、timers，避免无效等待
- 适配不同分辨率和系统，提升兼容性

---

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！
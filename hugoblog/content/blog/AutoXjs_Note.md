+++
date = "2026-04-21T10:20:00+05:30"
title = "AutoX.js 實戰:UiSelector、Threads、Floaty 與 webpack 打包"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "AutoXjs,Android,JS"
+++

## 引擎與生態

- **AutoX.js**:基於 Rhino(Java 寫的 JS 引擎),**不相容 Node.js**,npm 套件只能用純 JS 且不吃 Node API。只支援 ES5 + 部分 ES6。社群活躍、免費。
- **Auto.js Pro 9**:改用 Node.js,支援 npm,效能比 Rhino 快約 100×,ES2021+;付費。
- 兩者 API 類似但不完全相容,新開專案優先確定要用哪個。
- Android 10 以下 `auto.root` 在 AutoX.js 6.6.0 / 6.6.7 都會丟錯([Issue #723](https://github.com/kkevsekk1/AutoX/issues/723));仍可用 `select(...).findOne()` 系列 API 繞過。
- **備用 repo**:原作者的 `kkevsekk1/AutoX` 已停止維護,社群目前主要 release 都集中在 fork `aiselp/AutoX`,新版 APK 請到這邊下載:<https://github.com/aiselp/AutoX/releases?page=1>。

## UiSelector / UiObject / UiCollection

```js
// 單一控件
var btn = id('com.app:id/login').findOne();    // 阻塞到找到
btn.click();

var maybe = id('com.app:id/ok').findOne(2000); // 2 秒超時回 null
if (maybe) maybe.click();

var now = text('確定').findOnce();              // 不阻塞,找不到立刻回 null

// 集合
var items = className('android.widget.TextView').find();
items.forEach(it => log(it.text()));

// 父子關係
var parent = btn.parent();
var firstChild = parent.child(0);
```

## 點擊 / 手勢 / 螢幕

```js
// 控件 clickable=false 時改用座標點擊
var b = btn.bounds();
click(b.centerX(), b.centerY());

// 全局滑動
gesture(500, [100, 1500], [100, 500]);          // 500ms 往上滑
home(); back(); recents();
sleep(1000); toast('done');
```

## Threads & execScript 的雷

- `threads.shutDownAll()` 會殺掉 **main + 所有 threads.start() 啟的 sub-thread**,包括你用 `execScript()` 跑起來的腳本。
- `threads.start(fn)` 啟動的 sub-thread **接收不到 events.emit() 從其他腳本發的事件**——event 通道有 scope 限制。
- 想跨腳本通訊要走 `engines.execScriptFile()` / `engines.execScript()` 啟動另一個 engine,event 才能雙向。

## `const` 在 for loop 裡的 Rhino bug

```js
// ❌ 被這寫法咬過:所有 iteration 讀到的 row 都是「第一個值」
for (const row of rows) { /* ... */ }

// ✅ 改成 let / var
for (let row of rows) { /* ... */ }
```

Rhino 實作 let/const 的 scope 有 bug,for loop + const 會退化成 var 但 closure 綁錯。

## 浮動視窗 (Floaty) 技巧

- `floaty.window(layout)` 建立的懸浮視窗會**遮擋底下控件的 `id()` 查找**;此時改用 `text()` 或 `desc()` 可能有效。
- 懸浮視窗啟動後要用 `threads.start(() => engines.execScriptFile(...))` 執行主邏輯,**不要擋住 UI thread**。
- input 在 floaty 裡常叫不出系統鍵盤;可在 `touch_down` event 裡呼 `requestFocus()` 手動觸發。

## Threads 監聽關廣告 / 同意彈窗

```js
// 啟動前掛一個背景偵測
threads.start(function() {
  while (true) {
    var ad = text('跳過').findOne(300);
    if (ad) ad.click();
  }
});

// 主流程照跑
```

## 列表滑到底

```js
while (scrollForward()) sleep(300);       // scrollForward 回傳 false 就是到底
// 若 UI 是動態 loading,要在 while 內額外判斷 "新 item 數 > 舊 item 數"
```

## Webpack 打包(webpack-autojs)

```
work/
  my-project/
    src/main.js
  scriptConfig.js
```

```js
// scriptConfig.js
module.exports = {
  projects: [
    { name: 'my-project', entry: 'src/main.js', compile: true }
  ],
  config: {
    watch: 'none',         // rerun | deploy | none
    baseDir: 'work',
    target: 'node',        // Rhino 兼容
  }
};
```

`npm run build` → 產 `dist/my-project/`;推到手機:

```bash
adb push dist/my-project /storage/emulated/0/脚本/
```

UI layout 寫在 `ui.layout()` 時 webpack 會把 `<vertical>` 誤當 JSX 解,改法:

```js
ui.layout(`                  // 用模板字串包 XML
  <vertical>
    <text text="hello"/>
  </vertical>
`);
```

## ADB 常用

```bash
# macOS 把 adb 加進 PATH
export PATH=$PATH:$HOME/Library/Android/sdk/platform-tools

# logcat 過濾 AutoX.js
adb logcat | grep org.autojs.autoxjs.v6
adb shell pidof org.autojs.autoxjs.v6       # 拿 PID

# 檔案推送 / 拉回
adb push file.js /storage/emulated/0/Download/
adb pull /storage/emulated/0/Pictures/Screenshots ./screenshots

# 指定多裝置中的一台
adb -s emulator-5554 shell pm list packages
```

## APK 打包(在 AutoX.js App 內)

1. 右上 **New Project** → 輸入 main entry JS。
2. 產生專案資料夾後再 import 其他 JS / assets。
3. 右鍵 `build apk`。安裝失敗時:**關閉 Play Protect 掃描**,或先刪除舊版簽名不同的 APK。

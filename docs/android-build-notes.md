# Android 构建说明

> 本仓库把 iOS 与 Android 放在同一套源码里。差异只在 `Project.xml` 的
> 平台分支，**`source/` 完全共用**（包括那层 hxcodec → hxvlc 兼容层）。

## 1. 视频链路（与 iOS 同一套）

```
模组 / 引擎调用 hxCodec 的旧 API
        ↓                      （new MP4Handler() → playVideo() → finishCallback）
source/hxcodec/VideoHandler.hx  ← 兼容层（#if (VIDEOS_ALLOWED && (ios || android))）
        ↓  翻译成
hxvlc 1.9.3 的 hxvlc.flixel.FlxVideoSprite
        ↓
libVLC
        ├─ iOS     : project/vlc/lib/iOS/libvlc_device.a / libvlc_sim.a
        └─ Android : project/vlc/lib/Android/libvlc-{64,v7,x86,x86_64}.so
                     + libc++_shared-*.so
```

**为什么安卓也能用 hxvlc**：hxvlc 的 `include.xml` 里有

```xml
<section if="android">
    <ndll name="c++_shared" dir="project/vlc/lib" />
    <ndll name="vlc"        dir="project/vlc/lib" />
</section>
```

即 **hxvlc 随库自带 Android 的 libVLC 原生库**，不需要自己编 VLC、
也不需要往 gradle 里加 AAR。

### 版本要求与 iOS 完全一致

| 组件 | 版本 | 不能换的理由 |
|---|---|---|
| Haxe | **4.2.4** | hxvlc 1.9.4+ 用了 4.3 的 `?.` 语法，4.2.4 编译报错 |
| hxvlc | **1.9.3** | 1.7/1.8.x 没有 `forceRendering` + GPU 纹理路径 → 有声音没画面；1.9.4+ 语法不通 |
| lime | `LIME-0.6.3-FIX`（git） | Psych 0.6.3 的定制分支 |
| openfl | 9.1.0 | 与上面配套 |
| flixel | 4.11.0 | 同上 |

## 2. 工具链

| 组件 | 版本 | 说明 |
|---|---|---|
| JDK | **11** | Gradle 7.x 不支持 JDK 17+ |
| Android SDK | **API 34** + **Build-Tools 34.0.0** | `lime setup android` 会读这两个 |
| Android NDK | **r15c** | ⚠ **必须 r15c**。新版 NDK 移除了 gcc、改了 STL 头，本仓库锁的 hxcpp 会编不过 |

### NDK 路径配置

```bash
haxelib run lime setup android
# 依次输入：
#   Android SDK 路径   （如 ~/Android/Sdk）
#   Android NDK 路径   （如 ~/android-ndk/android-ndk-r15c）
#   JDK 路径           （如 /usr/lib/jvm/java-11-openjdk-amd64）
```

## 3. 编译

```bash
haxelib run lime build android -release
# 产物：export/release/android/bin/app/build/outputs/apk/release/*.apk
```

CI（`.github/workflows/main.yml` 的 `build-android` job）会自动做这些事并
把 APK 作为 artifact 上传。

### 打的 ABI

`Project.xml`：

```xml
<section if="android">
    <architecture name="arm64" />
    <architecture name="armv7" />
</section>
```

只打真机用得上的两个 ABI（少打 x86/x86_64 能显著减小包体、缩短链接时间）。
**要跑模拟器就把 `x86` / `x86_64` 加回来。**

## 4. 排障

### APK 装完一放视频就崩

先查 APK 里有没有原生库：

```bash
unzip -l app-release.apk | grep -E 'libvlc|libc\+\+_shared'
# 应该看到 lib/arm64-v8a/libvlc.so、lib/arm64-v8a/libc++_shared.so 等
```

没有 → `hxvlc` 没装上或版本不对，回看 CI 里「自检 hxvlc」那一步的输出。

### 有声音、没画面

与 iOS 侧同一个原因：hxvlc 版本 < 1.9.3（缺 GPU 渲染路径）。
确认 `haxelib list` 里 hxvlc 是 `1.9.3`。

### 视频「偏右 / 右边和下面被裁」

`FlxVideoSprite` 内层那个 `openfl.display.Bitmap` 被打开了。
它挂在 `FlxG.game` 上、尺寸自动跟随 `bitmapData`、不受相机管辖，
一旦可见就会叠出「第二个视频」。
`source/Main.hx` 的 `ensureVideoBitmapVisible()` 每 12 帧强制关掉它。

### 视频只播 1 秒就过 / 播完卡住

hxvlc 过早 / 未派发 `onEndReached`。
兼容层里有 `MIN_PLAY_SECONDS = 2.5` 门槛 + `pendingEnd` 挂起 + 超时兜底，
这套逻辑平台无关，安卓同样生效。

## 5. 诊断文件（游戏目录下）

| 文件 | 内容 |
|---|---|
| `pe_ios_video.txt` | 视频播放全过程：路径 / load 返回值 / 帧尺寸 / 缩放 / 内层 Bitmap 状态 |

> ℹ 文件名里的 `pe_ios_` 是历史遗留（这套诊断最早为 iOS 写的），
> 安卓上会生成同样名字的文件，内容格式一致。

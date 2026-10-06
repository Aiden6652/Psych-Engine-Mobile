# Friday Night Funkin' - Psych Engine
Engine originally used on [Mind Games Mod](https://gamebanana.com/mods/301107), intended to be a fix for the vanilla version's many issues while keeping the casual play aspect of it. Also aiming to be an easier alternative to newbie coders.

## Installation:
You must have [the most up-to-date version of Haxe](https://haxe.org/download/), seriously, stop using 4.1.5, it misses some stuff.

open up a Command Prompt/PowerShell or Terminal, type `haxelib install hmm`

after it finishes, simply type `haxelib run hmm install` in order to install all the needed libraries for *Psych Engine!*

## Customization:

if you wish to disable things like *Lua Scripts* or *Video Cutscenes*, you can read over to `Project.xml`

inside `Project.xml`, you will find several variables to customize Psych Engine to your liking

to start you off, disabling Videos should be simple, simply Delete the line `"VIDEOS_ALLOWED"` or comment it out by wrapping the line in XML-like comments, like this `<!-- YOUR_LINE_HERE -->`

same goes for *Lua Scripts*, comment out or delete the line with `LUA_ALLOWED`, this and other customization options are all available within the `Project.xml` file

# Psych Engine Mobile (iOS + Android)

本仓库把 **iOS** 与 **Android** 放在同一套源码里：
`source/` 完全共用，平台差异只在 `Project.xml` 的分支与构建工具链。

## 视频（过场动画 / 开机片头）

两端**共用同一套链路**：

```
hxCodec 旧 API  →  source/hxcodec/VideoHandler.hx（兼容层）  →  hxvlc 1.9.3  →  libVLC
```

- **hxvlc 1.9.3**：唯一同时满足「有 GPU 渲染路径（否则有声音没画面）」+
  「Haxe 4.2.4 语法通过（1.9.4+ 用了 4.3 的 `?.`）」+「随库带移动端静态/动态库」的版本。
- **不要用 hxCodec**：iOS 上它链的 `libvlc.a` 从未随库发布；安卓上它的原生扩展
  与这条 Lime/OpenFL 链不兼容。兼容层会把它那些调用翻译成 hxvlc 的操作。
- 兼容层对老模组**完全透明**：`new MP4Handler()` / `playVideo()` / `finishCallback`
  照旧可用，模组一行都不用改。

### Haxe 必须 4.2.4

`4.2.5` 与 `4.3+` 都不要用（前者与 iOS 侧不一致，后者与 hxvlc 1.9.3 语法冲突）。
CI 里写死 4.2.4。

## iOS 构建

见 [`docs/ios-build-notes.md`](docs/ios-build-notes.md)。
CI 在 `macos-14` 上跑，产物 `PsychEngine_iOS_unsigned.ipa`。

## Android 构建

见 [`docs/android-build-notes.md`](docs/android-build-notes.md)。

要点：

| 组件 | 版本 | 备注 |
|---|---|---|
| Haxe | **4.2.4** | 见上 |
| hxvlc | **1.9.3** | 随库带 `libvlc-{64,v7,x86,x86_64}.so` |
| JDK | **11** | Gradle 7.x 不支持 17+ |
| Android SDK | **API 34** + Build-Tools 34.0.0 | |
| Android NDK | **r15c** | 不要升级，新版 NDK 与本仓库锁的 hxcpp 不兼容 |

```bash
haxelib run lime setup android     # 填 SDK / NDK / JDK 路径
haxelib run lime build android -release
```

CI 在 `ubuntu-latest` 上跑，产物 `PsychEngine_Android_unsigned.apk`
（默认打 `arm64-v8a` + `armeabi-v7a` 双 ABI）。

## 排障

| 现象 | 看这里 |
|---|---|
| 视频有声音没画面 | hxvlc 版本不是 1.9.3（缺 GPU 渲染路径） |
| 视频偏右 / 右边和下面被裁 | `source/Main.hx` 的 `ensureVideoBitmapVisible()`（强制隐藏 hxvlc 内层 Bitmap） |
| 视频只播 1 秒 / 播完卡住 | `VideoHandler.hx` 里的 `MIN_PLAY_SECONDS` + `pendingEnd` + 超时兜底 |
| APK 一放视频就崩 | 检查 APK 里有没有 `libvlc.so`（CI 有自检步骤） |

诊断文件落在游戏目录：`pe_ios_video.txt`、`pe_ios_viewport.txt`
（文件名里的 `pe_ios_` 是历史遗留，安卓上生成同名文件、格式一致）。

## Credits:
* Shadow Mario - Programmer
* RiverOaken - Artist
* Yoshubs - Assistant Programmer

### Special Thanks
* bbpanzu - Ex-Programmer
* Yoshubs - New Input System
* SqirraRNG - Crash Handler and Base code for Chart Editor's Waveform
* KadeDev - Fixed some cool stuff on Chart Editor and other PRs
* iFlicky - Composer of Psync and Tea Time, also made the Dialogue Sounds
* PolybiusProxy - .MP4 Video Loader Library (hxCodec)
* Keoiki - Note Splash Animations
* Smokey - Sprite Atlas Support
* Nebula the Zorua - LUA JIT Fork and some Lua reworks
_____________________________________

# Features

## Attractive animated dialogue boxes:

![](https://user-images.githubusercontent.com/44785097/127706669-71cd5cdb-5c2a-4ecc-871b-98a276ae8070.gif)


## Mod Support
* Probably one of the main points of this engine, you can code in .lua files outside of the source code, making your own weeks without even messing with the source!
* Comes with a Mod Organizing/Disabling Menu.


## Atleast one change to every week:
### Week 1:
  * New Dad Left sing sprite
  * Unused stage lights are now used
### Week 2:
  * Both BF and Skid & Pump does "Hey!" animations
  * Thunders does a quick light flash and zooms the camera in slightly
  * Added a quick transition/cutscene to Monster
### Week 3:
  * BF does "Hey!" during Philly Nice
  * Blammed has a cool new colors flash during that sick part of the song
### Week 4:
  * Better hair physics for Mom/Boyfriend (Maybe even slightly better than Week 7's :eyes:)
  * Henchmen die during all songs. Yeah :(
### Week 5:
  * Bottom Boppers and GF does "Hey!" animations during Cocoa and Eggnog
  * On Winter Horrorland, GF bops her head slower in some parts of the song.
### Week 6:
  * On Thorns, the HUD is hidden during the cutscene
  * Also there's the Background girls being spooky during the "Hey!" parts of the Instrumental

## Cool new Chart Editor changes and countless bug fixes
![](https://github.com/ShadowMario/FNF-PsychEngine/blob/main/docs/img/chart.png?raw=true)
* You can now chart "Event" notes, which are bookmarks that trigger specific actions that usually were hardcoded on the vanilla version of the game.
* Your song's BPM can now have decimal values
* You can manually adjust a Note's strum time if you're really going for milisecond precision
* You can change a note's type on the Editor, it comes with two example types:
  * Alt Animation: Forces an alt animation to play, useful for songs like Ugh/Stress
  * Hey: Forces a "Hey" animation instead of the base Sing animation, if Boyfriend hits this note, Girlfriend will do a "Hey!" too.

## Multiple editors to assist you in making your own Mod
![Screenshot_3](https://user-images.githubusercontent.com/44785097/144629914-1fe55999-2f18-4cc1-bc70-afe616d74ae5.png)
* Working both for Source code modding and Downloaded builds!

## Story mode menu rework:
![](https://i.imgur.com/UB2EKpV.png)
* Added a different BG to every song (less Tutorial)
* All menu characters are now in individual spritesheets, makes modding it easier.

## Credits menu
![Screenshot_1](https://user-images.githubusercontent.com/44785097/144632635-f263fb22-b879-4d6b-96d6-865e9562b907.png)
* You can add a head icon, name, description and a Redirect link for when the player presses Enter while the item is currently selected.

## Awards/Achievements
* The engine comes with 16 example achievements that you can mess with and learn how it works (Check Achievements.hx and search for "checkForAchievement" on PlayState.hx)

## Options menu:
* You can change Note colors, Delay and Combo Offset, Controls and Preferences there.
 * On Preferences you can toggle Downscroll, Middlescroll, Anti-Aliasing, Framerate, Low Quality, Note Splashes, Flashing Lights, etc.

## Other gameplay features:
* When the enemy hits a note, their strum note also glows.
* Lag doesn't impact the camera movement and player icon scaling anymore.
* Some stuff based on Week 7's changes has been put in (Background colors on Freeplay, Note splashes)
* You can reset your Score on Freeplay/Story Mode by pressing Reset button.
* You can listen to a song or adjust Scroll Speed/Damage taken/etc. on Freeplay by pressing Space.
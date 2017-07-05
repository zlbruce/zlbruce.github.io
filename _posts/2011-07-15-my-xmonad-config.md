---
title: 我的 xmonad 配置
author: zlbruce
layout: post
permalink: /2011/07/15/my-xmonad-config/
categories:
  - Linux
tags:
  - xmobar
  - xmonad
---
本文讲述我的 [xmonad][1] 桌面的配置，它使用 [xmobar][2] 显示各种状态，使用 [taryer][3] 来提供 systray。

xmonad 是一种平铺式窗口管理器，窗口之间不会遮挡，且自动调整大小以填充整个屏幕，这样就节省了使用鼠标调整各个窗口位置的时间，一切的操作都可以通过自定义的快捷键来完成。

xmonad 是由 [Haskell][4] 开发的，其配置文件也是使用 Haskell 来配置的，不过了解一些简单的语法之后，配置起来也是很方便的。当然默认的配置就已经非常好用了，我也只是在默认的配置上进行了一点小的修改而已。  
下面是我的 xmonad 配置与默认配置之间的差异（只是为了表现差异，最后会有整个配置文件的下载）：

{% highlight diff %}
--- /usr/share/xmonad-0.9.2/ghc-7.0.4/man/xmonad.hs 2011-07-05 00:22:51.800397623 +0800
+++ .xmonad/xmonad.hs 2011-07-14 22:39:43.118625125 +0800
@@ -14,10 +14,15 @@
import qualified XMonad.StackSet as W
import qualified Data.Map as M

+import XMonad.Util.Run
+import XMonad.Hooks.DynamicLog
+import XMonad.Hooks.ManageHelpers
+import XMonad.Util.Cursor
+
-- The preferred terminal program, which is used in a binding below and by
-- certain contrib modules.
--
-myTerminal = "xterm"
+myTerminal = "gnome-terminal"

-- Whether focus follows the mouse pointer.
myFocusFollowsMouse :: Bool
@@ -32,7 +37,7 @@
-- ("right alt"), which does not conflict with emacs keybindings. The
-- "windows key" is usually mod4Mask.
--
-myModMask = mod1Mask
+myModMask = mod4Mask

-- The mask for the numlock key. Numlock status is "masked" from the
-- current modifier status, so the keybindings will work with numlock on or
@@ -127,6 +132,12 @@
-- Deincrement the number of windows in the master area
, ((modm , xK_period), sendMessage (IncMasterN (-1)))

+ -- screenshot screen
+ , ((modm , xK_Print), spawn "/usr/bin/screenshot scr")
+
+ -- screenshot window or area
+ , ((modm .|. shiftMask, xK_Print), spawn "/usr/bin/screenshot win")
+
-- Toggle the status bar gap
-- Use this binding with avoidStruts from Hooks.ManageDocks.
-- See also the statusBar function from Hooks.DynamicLog.
@@ -221,6 +232,7 @@
myManageHook = composeAll
[ className =? "MPlayer" --> doFloat
, className =? "Gimp" --> doFloat
+ , isFullscreen --> doFloat
, resource =? "desktop_window" --> doIgnore
, resource =? "kdesktop" --> doIgnore ]

@@ -239,9 +251,13 @@
-- Status bars and logging

-- Perform an arbitrary action on each internal state change or X event.
--- See the 'XMonad.Hooks.DynamicLog' extension for examples.
+-- See the 'DynamicLog' extension for examples.
+--
+-- To emulate dwm's status bar
+--
+-- > logHook = dynamicLogDzen
--
-myLogHook = return ()
+myLogHook = dynamicLog

------------------------------------------------------------------------
-- Startup hook
@@ -251,14 +267,19 @@
-- per-workspace layout choices.
--
-- By default, do nothing.
-myStartupHook = return ()
+myStartupHook = do
+ spawn "fcitx"
+ setDefaultCursor xC_left_ptr
+ spawn "feh --bg-scale /home/zhanglei/Pictures/mlxmonadwall1trans.png"
+ spawn "trayer --edge top --align right --widthtype percent --width 10 --SetDockType true --SetPartialStrut true --transparent true --alpha 0 --tint 0x000000 --expand true --heighttype pixel --height 25"
+ --return ()

------------------------------------------------------------------------
-- Now run xmonad with all the defaults we set up.

-- Run xmonad with the settings you specify. No need to modify this.
--
-main = xmonad defaults
+main = xmonad =<< xmobar defaults

-- A structure containing your configuration settings, overriding
-- fields in the default config. Any you don't override, will
{% endhighlight %}

以上修改的一些简单说明：

  * 使用 gnome-terminal 作为我的终端
  * 使用 Win 健作为 xmonad 的 Mod 健，默认为 Alt，但是会和 bash 的一些快捷键冲突
  * 增加了两个快捷键 Win + Print 与 Win + Shift + Print 用来进行截图
  * 加入 isFullscreen –> doFloat 以解决 flash 不能全屏的问题
  * myLogHook = dynamicLog 可以让 xmobar 显示出当前的状态
  * 在 myStartupHook 函数中加入我的启动项： 
      1. fcitx 输入法
      2. 设置鼠标样式
      3. 使用 feh 设置桌面背景
      4. 启动 trayer 以显示 systary
  * 最后，main 函数中使用 xmobar 函数启动 xmobar

下面是一些经常使用到的快捷键：

  * Win + Shift + Enter: 启动 Terminal
  * Win + P: 启动 dmenu，用于启动各种命令
  * Win + 1 – 9: 在 1 – 9 虚拟桌面中进行切换
  * Win + Shift + 1 – 9: 将当前的窗口移动到指定的虚拟桌面
  * Win + Tab: 在同一虚拟桌面中的窗口之间切换
  * Win + Shift + C: 杀死当前窗口
  * Win + Q: 重启 xmonad，用于改为配置之后使其生效

下面是我的 xmobar 的配置，其配置文件为 .xmobarrc

{% highlight haskell %}
Config { font = "xft:WenQuanYi Zen Hei Mono-12"
, bgColor = "black"
, fgColor = "grey"
, position = TopW L 90
, lowerOnStart = True
, commands = [ Run Weather "ZGSZ" ["-t",": C","-L","18","-H","25","--normal","green","--high","red","--low","lightblue"] 36000
, Run Network "eth0" ["-L","0","-H","32","--normal","green","--high","red"] 10
, Run Network "eth1" ["-L","0","-H","32","--normal","green","--high","red"] 10
, Run Cpu ["-L","3","-H","50","--normal","green","--high","red"] 10
, Run Memory ["-t","Mem: %"] 10
, Run Swap [] 10
, Run StdinReader
, Run Com "uname" ["-s","-r"] "" 36000
, Run Date "%a %b %_d %Y %H:%M:%S" "date" 10
]
, sepChar = "%"
, alignSep = "}{"
, template = "%cpu% | %memory% * %swap% | %eth0% | %StdinReader% }{ <fc=#ee9a00>%date%</fc> | %ZGSZ% | %uname%"
}
{% endhighlight %}

其中有三点值得注意：

  1. position 指定了 xmobar 的位置：左上，长度 90%，而 trayer 的则为右上，长度 10%。刚好能够接上。
  2. 深圳天气的代码为 ZGSZ，其他地方的可以到[这个页面][5]进行查询。
  3. 需要有 StdinReader，否则在 xmonad 中的 dynamicLog 函数就不会起作用。

最后，提供这两个配置文件的下载：  
[xmonad.hs][6]  
[.xmobarrc][7]

 [1]: http://xmonad.org/ "xmonad"
 [2]: http://projects.haskell.org/xmobar/ "xmobar"
 [3]: http://code.google.com/p/trayer/ "trayer"
 [4]: http://www.haskell.org/ "Haskell"
 [5]: http://www.rap.ucar.edu/weather/surface/stations.txt "各个地区的天气代码"
 [6]: https://github.com/zlbruce/dotconfig/blob/master/.xmonad/xmonad.hs "xmonad.hs"
 [7]: https://github.com/zlbruce/dotconfig/blob/master/.xmobarrc ".xmobarrc"

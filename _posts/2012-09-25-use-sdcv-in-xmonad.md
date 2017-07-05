---
title: xmonad下使用sdcv进行翻译
author: zlbruce
layout: post
permalink: /2012/09/25/use-sdcv-in-xmonad/
categories:
  - Linux
tags:
  - dmenu
  - sdcv
  - xmonad
---
主要是看到了[换种方式使用星际译王][1]这篇文章，里面是用的 awesome。这两天在 xmonad 折腾了一下，虽然没有像原文中那样好用，不过也算是可用了。

xmonad 没有自己的通知窗口，这里只能用 notify-send 来发送翻译结果。基本思路很简单，就是按照以下步骤来：

  1. 获取选中的文字，或者是输入的文字，这里使用 [dmenu][2] 获取用户输入
  2. 调用 sdcv 获得翻译后的文字
  3. 调用 notify-send 显示翻译结果

折腾了两天原因主要是卡在第三步上了，一共遇到两个问题。

第一个问题是只显示了一个标题，不显示翻译内容。在我手动将翻译结果一行一行的使用 notify-send 显示后，发现只要带有 < 号的都不可以；后来才知道我用的 notification-daemon 可以使用 html 式标签对显示内容进行格式化。而 sdcv 输出的结果有 html 的一些特殊字符，导致解析失败。

第二个问题是乱码，显示的中文都变成了乱码，但是在终端中手动发送中文字符是没有问题的。后来看了 xmonad-contrib 的源代码才知道我使用的 safeSpawn 函数会使用 encodeString 对输入的参数进行编码。暂时不清楚为什么会对参数编码，简单的在配置文件中定义一个 mySafeSpawn 函数搞定。

下面是配置方法，首先，引入一些模块：

{% highlight haskell %}
import Data.Char
import XMonad.Util.XSelection
import System.Posix.Process (createSession, executeFile, forkProcess)
{% endhighlight %}

定义快捷键，我这里使用 modMask + s 对选中的文件进行翻译，使用 modMask + Shift + s 键弹出 dmenu，对输入的文字进行翻译：

{% highlight haskell %}
...
myKeys conf@(XConfig {XMonad.modMask = modm}) = M.fromList $

    -- launch a terminal
    [ ((modm .|. shiftMask, xK_Return), spawn $ XMonad.terminal conf)
      ...
      , ((modm              , xK_s),      getSelection  >>= sdcv)
      , ((modm .|. shiftMask, xK_s),      getDmenuInput >>= sdcv)
      ...
    ]
...
{% endhighlight %}

其中 getSelection 是获取选中文字的函数，包含在 XMonad.Util.XSelection 模块中；getDmenuInput 函数是用来获取 dmenu 输入的文字的，这里过滤了不可见字符：

{% highlight haskell %}
getDmenuInput = fmap (filter isPrint) $ runProcessWithInput "dmenu" ["-p", "Dict: "] ""
{% endhighlight %}

sdcv 函数完成调用 sdcv 取得翻译结果，并调用 notify-send 进行发送：

{% highlight haskell %}
sdcv word = do
    output <- runProcessWithInput "sdcv" [word] ""
    mySafeSpawn "notify-send" [word, trString output]
{% endhighlight %}

这里使用了 mySafeSpawn 去执行 notify-send 命令，mySafeSpawn 是从 XMonad.Util.Run 里面拷贝过来的，只是去掉了 encodeString 的调用：

{% highlight haskell %}
mySafeSpawn :: MonadIO m => FilePath -> [String] -> m ()
mySafeSpawn prog args = io $ void_ $ forkProcess $ do
    uninstallSignalHandlers
    _ <- createSession
    executeFile prog True args Nothing
        where void_ = (>> return ()) -- TODO: replace with Control.Monad.void / void not in ghc6 apparently
{% endhighlight %}

trString 函数是用于将翻译后结果中的 html 特殊字符进行转义的，我这里只对<>&三个符号进行了转义，如果你使用的是 xfce4-notifyd 这样的服务端，应该就不需要转义了：

{% highlight haskell %}
trString = foldl (\s c -> s ++ (trChar c)) ""

trChar c
    | c == '<' = "&lt;"
    | c == '>' = "&gt;"
    | c == '&' = "&amp;"
    | otherwise = [c]
{% endhighlight %}

修改完后，使用 modm + q 重启 xmonad，在浏览器或者终端中选中一个单词，然后按 modMask + s 应该就能再通知窗口中显示翻译的结果了。不过与我参考的原文相比较，还是有不完美的地方，那就是不能使用快捷键关闭翻译窗口，只能点一下鼠标，或者等待超时自动关闭。不过暂时先这样了。

 [1]: http://linuxtoy.org/archives/awesome-%E7%AA%97%E5%8F%A3%E7%AE%A1%E7%90%86%E5%99%A8%E2%80%94%E2%80%94%E6%8D%A2%E7%A7%8D%E6%96%B9%E5%BC%8F%E4%BD%BF%E7%94%A8%E6%98%9F%E9%99%85%E8%AF%91%E7%8E%8B.html "换种方式使用星际译王"
 [2]: http://tools.suckless.org/dmenu/ "dmenu"
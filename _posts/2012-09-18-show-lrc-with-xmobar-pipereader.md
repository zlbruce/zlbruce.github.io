---
title: 使用PipeReader在xmobar上显示歌词
author: zlbruce
layout: post
permalink: /2012/09/18/show-lrc-with-xmobar-pipereader/
categories:
  - Linux
tags:
  - lrcdis
  - xmobar
  - xmonad
---
公司的电脑装了 Gentoo 了，现在又开始折腾 [xmonad][1] 了，前两天看到 [xmobar][2] 有个 PipeReader 的插件，可以从一个管道中读取数据并显示。而 [lrcdis][3] 则支持将歌词输出到管道中。将这两者结合起来就可以了。

首先修改 xmobar 配置文件，加入 PipeReader，读取 /dev/shm/lrcfifo 这个管道：

{% highlight haskell %}
...
, commands = [
    ...
    , Run PipeReader "/dev/shm/lrcfifo" "lrc"
    ...
             ]

, template = "%cpu% | %memory% * %swap% | %eth0% | %StdinReader% }{ %lrc% | %date% | %ZGSZ% | %default:Master%"
...
{% endhighlight %}

然后运行让 lrcdis 输出到管道

    $ lrcdis -m fifo

重启 xmobar 看看效果，可惜什么都显示不出来。

看了下 lrcdis 的代码，发现写入管道的时候使用了 echo -n，没有写入换行符，导致 PipeReader 读取时没有返回，简单修改下就可以了，主要是不使用 echo -n 以及去掉两处多余的 echo，patch 如下：

{% highlight diff %}
--- lrcdis.org        2009-09-08 15:21:13.000000000 +0800
+++ lrcdis       2012-09-17 12:56:15.000000000 +0800
@@ -605,7 +605,7 @@
        local tm min sec tmptm
        if [ "$Dismode" = "fifo" -a -p "/dev/shm/lrcfifo" ];then
                echo "write to fifo"
-               echo -n "">/dev/shm/lrcfifo #防止读lrcfifo时等待 amoblin 4.14 9:49
+               #echo -n "">/dev/shm/lrcfifo #防止读lrcfifo时等待 amoblin 4.14 9:49
        fi
 
        case "${1:-rhythmbox}" in
@@ -739,7 +739,7 @@
        elif [ "$Dismode" = "fifo" -a -p "/dev/shm/lrcfifo" ];then
                [ "$1" = T ] && line="****** $line ******"
                [ "$1" = E ] && line="错误: $line"
-               echo -ne "$line" > /dev/shm/lrcfifo
+               echo -e "$line" > /dev/shm/lrcfifo
        elif [ "$Dismode" = "notify" -a "`which notify-send 2>/dev/null`" ];then
                [ "$2" = "" ] && return
                [ "$1" = T ] && line="****** $line ******"
@@ -945,7 +945,7 @@
 PlayerStat="Unknow"
 
 GET_STAT |while read line0;do
-       [ "$Dismode" = "fifo" -a -p "/dev/shm/lrcfifo" ] && echo -n "">/dev/shm/lrcfifo #防止读lrcfifo时等待 amoblin 4.14 9:49 #非fifo模式不必要生成该文件 bones7456
+       [ "$Dismode" = "fifo" -a -p "/dev/shm/lrcfifo" ] #&& echo -n "">/dev/shm/lrcfifo #防止读lrcfifo时等待 amoblin 4.14 9:49 #非fifo模式不必要生成该文件 bones7456
        DEBUGECHO "GET_STAT: $line0<<"
        [ "$line0" = "$line1" ] && continue
        line1="$line0"
{% endhighlight %}

再试一下，完美显示。

不过如果你重启机器会发现又不灵了，并且运行 lrcdis 也会提示 /dev/shm/lrcfifo 已存在，这是因为 xmobar 的 PipeReader 插件直接使用 ReadWriteMode 打开管道文件，导致如果文件不存在的时候 PipeReader 会自己创建一个普通的文件。

这里我想了半天也没想到一个好的解决方案，最后只好修改 xmobar 的 PipeReader 插件，在打开管道文件之前使用 checkPipeLoop 检查传入的文件是否为管道文件，如果不是则等待，直到是管道文件才进行后面的打开操作。patch 如下（xmobar 0.18 版本已经合入了该补丁，就不需要再设置了）：

{% highlight diff %}
--- xmobar-0.15-old/src/Plugins/PipeReader.hs   2012-09-17 13:09:07.000000000 +0800
+++ xmobar-0.15/src/Plugins/PipeReader.hs       2012-09-18 13:08:25.000000000 +0800
@@ -16,6 +16,9 @@
 
 import System.IO
 import Plugins
+import System.Posix.Files
+import Control.Concurrent(threadDelay)
+import Control.Exception
 
 data PipeReader = PipeReader String String
     deriving (Read, Show)
@@ -23,6 +25,17 @@
 instance Exec PipeReader where
     alias (PipeReader _ a)    = a
     start (PipeReader p _) cb = do
+        checkPipeLoop p
         h <- openFile p ReadWriteMode
         forever (hGetLineSafe h >>= cb)
         where forever a = a >> forever a
+                
+checkPipeLoop :: FilePath -> IO ()
+checkPipeLoop file = do
+    handle (\(SomeException _) -> waitForPipe) $ do
+    status <- getFileStatus file
+    if isNamedPipe status
+      then return ()
+      else waitForPipe
+    where waitForPipe = threadDelay 100000 >> checkPipeLoop file
+
{% endhighlight %}

最后看一下最终的效果，歌词在右上角时间的左边： 

{% include picture.html img='/images/2012-09/show-lrc-with-xmobar-pipereader.png' alt='使用 xmobar 的 PipeReader 显示歌词' %}


 [1]: http://xmonad.org/ "xmonad"
 [2]: http://projects.haskell.org/xmobar/ "xmobar"
 [3]: http://code.google.com/p/lrcdis/ "lrcdis"

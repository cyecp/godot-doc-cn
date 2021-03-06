.. _doc_data_paths:

数据路径
==========

路径分隔符
---------------

为了尽可能多的支持更多的平台着想, Godot仅接受unix风格路径分融符 (``/``). 
包括windows在引擎任何地方都一样的.

有一个路径: ``C:\Projects`` 要写成这样 ``C:/Projects``.

资源路径
-------------

就像以前说过的. Godot的工程存在于所给的包含"engine.cfg"文本文件的目录,
就算这个目录是空的也一样.

任何可访问的工程文件都可以通过用``res://``这种路径作为根目录. 打个比方, 
一个在根目录下的纹理文件可以用这样的路径名打开: ``res://sometexture.png``.

用户路径 (持久化的数据)
-------------------------------

当一个工程正在运行, 资源路径是只读的, 这是一个很常见的情形, 由于在一个包内, 
它本身就包含可执行或者系统范围的安装路径.

持久化存储的文件通常应该用``user://`` 作为前缀, 举个栗子: ``user://gamesave.txt``.

在许多设备中 (打个比方, 移动广告终端) 对于app这个路径就是惟一的. 
在桌面操作系统中,  在OSX系统和Linux系统中引擎使用经典的 ~/.Name (项目设置检查一下), 
在windows系统中使用 APPDATA/Name.

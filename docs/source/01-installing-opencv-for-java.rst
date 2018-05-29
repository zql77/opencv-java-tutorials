==========================
开始安装OpenCV的Java支持包
==========================

OpenCV的Java支持包简介
--------------------------------
从OpenCV 2.4.4开始，OpenCV支持Java桌面程序的开发。本教程将帮助您在桌面操作系统上安装OpenCV。

安装最新的Java版本
--------------------------------
你可以从 `Oracle <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 网站下载最新的Oracle Java JDK。 下载完成后您应该可以打开刚刚下载的文件来安装最新版的Oracle Java JDK。

安装最新版的Eclipse IDE
-----------------------------------
从 `Eclipse下载页面 <https://www.eclipse.org/downloads/eclipse-packages/>`_ 下载最新版的Eclipse IDE。建议选着 ``Eclipse IDE for Java Developers`` 版本。

将下载的压缩文件解压到你的计算机的任意一个地方。你不用安装任何程序，运行 eclipse.exe 或者eclipse.sh就可以运行Eclipse IDE。或者，你可以在 Windows 系统上尝试eclipse安装程序。

在Windows系统上安装OpenCV 3.x
------------------------------------
首先，我们需要从 `OpenCV 发行 <http://opencv.org/releases.html>`_ OpenCV 3.x 开发库。

然后，运行你下载 ''opencv-3.4.1-vc14_vc15.exe'' 提取OpenCV开发库到你喜欢的目录。 ''build'' 和 ''sources'' 将在 ``opencv`` 文件夹中。

有两个文件是我们需要的。在 ``\opencv\build\java``文件夹中的 ``opencv-3xx.jar`` 和在 ``\opencv\build\java\x64`` (64位动态链接库) 或者 ``\opencv\build\java\x86`` (32 位动态链接库)中的``opencv_java3xx.dll``。 每个文件的`3xx`后缀是当前OpenCV版本数字，例如OpenCV 3.0的是`300`;OpenCV 3.3将是`330`。

在macOS系统中安装OpenCV 3.x
---------------------------------
在macOS系统中安装OpenCV,最快速的方式是使用 `Homebrew <http://brew.sh>`_。 在安装Homebrew后, 你必须检查你的系统上是否已经安装了`XCode Command Line Tools`。

为此，可以打开“终端”并执行：
``xcode-select --install``
如果macOS要求安装一些工具，请继续进行下载和安装。 或者，请继续进行OpenCV安装。

作为先决条件，请检查Apache Ant是否已安装。 否则，用Homebrew运行以下命令安装它：
``brew install ant``.
Ant 将安装到 ``/usr/local/bin/ant`` 中。

要通过Homebrew安装支持Java的OpenCV库,你需要编辑Homebrew中的*opencv*配置, 使其支持Java，具体操作如下：
``brew edit opencv``
在打开的文本编辑器中，更改该行：
``-DBUILD_opencv_java=OFF``
改变为：
``-DBUILD_opencv_java=ON``
那么在保存文件之后，可以有效地安装OpenCV库了：
``brew install --build-from-source opencv``

在安装OpenCV之后， 需要的jar文件和dylib库将位于 ``/usr/local/Cellar/opencv/3.x.x/share/OpenCV/java/`` 中, 例如, ``/usr/local/Cellar/opencv/3.3.1/share/OpenCV/java/``。

请注意，如果你从以前的版本更新OpenCV，则此方法将不起作用：你需要卸载OpenCV并重新安装它。

在Linux下安装OpenCV 3.x
---------------------------------
.. note:: Linux软件包管理系统（`apt-get`，`yum`等）可以提供发行版仓库中的OpenCV库版本。如果需要特定版本的OpenCV就需要自己编译，如果您想在Windows或MacOS下编译OpenCV，以下说明也很有用。

第一步，下载安装 `CMake <http://www.cmake.org/download/>`_ 和 `Apache Ant <http://ant.apache.org/>`_,或者运行 ''sudo apt-get install cmake cmake-gui ant'' 进行安装。成功安装好Cmake 和 Ant后，从 `OpenCV网站 <http://opencv.org/releases.html>`_ 下载OpenCV库源文件。
解压下载的OpenCV源文件到本地目录(例如：~/opencv/)，并用CMake(cmake-gui)打开。
在``Where is the source code field``中填写OpenCV源码目录 (例如: ~/opencv/)_。 在 ``Where to build the binaries`` 中填写构建目录 (例如: ~/opencv/build/).
最后，勾选 ``Grouped`` 和 ``Advanced`` 复选框。

.. image:: _static/01-00.png

点击 ``Configure`` 然后选择 ``Unix Makefiles``作为默认编译器。首先，你的保证你已经安装了C/C++编译器。(可以运行 ''sudo apt-get install build-essential'' 进行安装。)
在``Ungrouped Entries`` 组中, 填入Apache Ant可执行文件路径(例如： ``/apache-ant-1.9.6/bin/ant``)。
在 ``BUILD`` 组中，取消选择：

* ``BUILD_PERF_TESTS``
* ``BUILD_SHARED_LIBRARY`` 使Java绑定动态库就足够了
* ``BUILD_TESTS``
* ``BUILD_opencv_python``

在 ``CMAKE`` 组中, 设置 ``Debug`` (或者 ``Release``) 为 ``CMAKE_BUILD_TYPE``

在 ``JAVA`` 组中:

* 填写 Java AWT include文件夹的路径 (例如：``/usr/lib/jvm/java-1.8.0/include/``)
* 填写 Java AWT 库路径 (例如： ``/usr/lib/jvm/java-1.8.0/include/jawt.h``)
* 填写 Java include文件夹路径 (例如： ``/usr/lib/jvm/java-1.8.0/include/``)
* 填写 alternative Java include文件夹路径 (例如： ``/usr/lib/jvm/java-1.8.0/include/linux``)
* 填写 JVM 库路径 (例如：``/usr/lib/jvm/java-1.8.0/include/jni.h``)

请点击 ``Configure`` 两次。 如果还有红色目录警告存在，请修复并在此按下 ``Configure`` 直到没有红色目录警告。点击 ``Generate`` 生成 Makefiles 后关闭CMake。

.. image:: _static/01 - 01.png

打开终端，切换到 ``build`` 文件夹并使用下面的命令编译OpenCV库: ``make -j4``。请注意：`-j`标志告诉`make`允许编译线程的最大数量进行编译，这使得构建理论上更快，通常使他为你的CPU核心数就可以了。
等待编译完成...
如果一切顺利，你将在 ``/opencv/build/bin`` 文件夹得到 ``opencv-3xx.jar`` 同时在 ``/opencv/build/lib`` 文件夹得到 ``libopencv_java3xx.so``。 每个文件的`3xx`后缀是当前OpenCV版本数字，例如OpenCV 3.0的是`300`;OpenCV 3.3将是`330`。

在Eclipse中设置OpenCV的Java支持包
----------------------------------
打开Eclipse并选择您选择的工作目录。 创建一个用户库，为你接下来的项目使用: 点击 ``Window > Preferences...``.

.. image:: _static/01 - 02.png

依次操作 ``Java> Build Path> User Libraries`` 并选择 ``New ...``。
输入该库的名称（例如：opencv3.x ）并选择新创建的用户库。
选着 ``Add External JARs...``,从你的电脑浏览并选着``opencv-3xx.jar``。
添加 jar 包后, 导入它, 选择 ``Native library location`` 然后点击 ``Edit...``。

.. image:: _static/01 - 03.png

选择 ``External Folder...`` 并浏览以选择包含OpenCV库的文件夹 (例如：``C:\opencv\build\java\x64`` Windows系统中，''~/opencv/build/java/x64'' 在Linux系统上).

在macOS系统中, 如果你不是通过 Homebrew 安装的OpenCV库, 你需要创建一个.so的软连接并连接到.dylib文件 例如，在终端中输入以下命令:
``ln -s libopencv_java300.so libopencv_java300.dylib``

在其他IDEs 中设置OpenCV的Java支持包 (实验)
---------------------------------------------------
如果您使用的是IntelliJ，则可以使用VM参数 ``-Djava.library.path=/opencv/build/lib`` 来指定OpenCV库的位置。

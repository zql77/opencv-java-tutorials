=======================================
你的第一个OpenCV Java 程序
=======================================

.. note:: 我们假设您现在已经阅读了前面的教程。 如果没有，请在 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_查看教程。 你也可以在`<https://github.com/opencv-java/>`_找到源代码和资源。

一个使用 OpenCV 的Java程序
------------------------------
本教程将指导你在Eclipse中的使用OpenCV库创建简单的Java控制台应用程序。

本教程中我们将要进行的操作：
--------------------------------
在本教程中，我们将会:
 * 创建一个新的Java工程
 * 为新创建的工程添加用户库（OpenCV）
 * 编写OpenCV 的Java代码
 * 编译并运行我们的程序

创建一个新的工程
--------------------
打开 Eclipse创建一个新的Java 工程; 打开 ``File`` 菜单, 选择 ``New`` 然后点击 ``Java Project``.

.. image:: _static/02-00.png

在 ``New Java Project`` 对话框中输入你想要的工程名字然后点击 ``Finish``.

添加用户库目录
------------------
如果你按照前面的教程 (``开始安装OpenCV的Java支持包``), 您应该已经在工作区的用户库中设置了OpenCV库; 如果不是，请查看上一个教程。
现在将OpenCV库添加到你的项目中。
在Eclipse的 ``Package Explorer`` 里面，右键点击你的项目文件夹，然后进入 ``Build Path --> Add Libraries...``。

.. image:: _static/02-01.png

选择 ``User Libraries`` 然后点击 ``Next``, 勾选OpenCV库复选框然后点击 ``Finish``。

.. image:: _static/02-02.png

创建一个简单的应用
---------------------------
现在通过右键单击项目文件夹并转到``New --> Class``，为项目添加一个新类。 
为程序包和类取名字，然后点击``Finish``。
现在我们准备编写我们的第一个应用程序的代码。
我们首先需要定义``main``方法:

.. code-block:: java

    public class HelloCV {
	    public static void main(String[] args){
		    System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
		    Mat mat = Mat.eye(3, 3, CvType.CV_8UC1);
		    System.out.println("mat = " + mat.dump());
	    }
    }

第一步，我们需要加载之前在我们项目中设置的OpenCV本机库。

.. code-block:: java

    System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

然后我们定义一个新的Mat类。

.. 注意:: 类**Mat**表示一个n维密集数值单通道或多通道阵列。 它可以用来存储实值或复值向量和矩阵，灰度或彩色图像，体素卷，矢量场，点云，张量，直方图。在 OpenCV `页面 <http://docs.opencv.org/3.0.0/dc/d84/group__core__basic.html>`_有更多详细信息。

.. code-block:: java

    Mat mat = Mat.eye(3, 3, CvType.CV_8UC1);

 ``Mat.eye`` 可以创建一个单位矩阵，我们设置它的尺寸为（3x3）和它的数据类型为CV_8UC1（OpenCV定义的数据类型：8位单通道。`更多信息 <https://docs.opencv.org/3.4.1/d1/d1b/group__core__hal__interface.html>`_）。

你应该会注意到, Eclipse编辑器会给你一系列错误报告。这是由于eclipse无法解析某些变量。你可以将鼠标光标定位在似乎是错误的文字上，然后等待对话框弹出，然后单击 ``Import...``。这样你讲导入所依赖的包:

.. code-block:: java

    import org.opencv.core.Core;
    import org.opencv.core.CvType;
    import org.opencv.core.Mat;

我们现在可以尝试通过单击“运行”按钮来构建和运行我们的应用程序。
我们得到以下的输出信息，我们得到了一个单位矩阵：

.. image:: _static/02-03.png

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/HelloCV/>`_。

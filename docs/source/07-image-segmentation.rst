==================
图像分割
==================

.. note::我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

目标
----
在本教程中，我们将创建一个JavaFX应用程序，我们可以决定应用从我们的网络摄像头捕获的视频流，或者使用* Canny边缘检测器*或使用两种基本形态学操作去除平凡背景：*膨胀*和*侵蚀*。

Canny边缘检测器
-------------------
** Canny边缘检测器**是一种边缘检测算子，它使用多级算法来检测图像中各种各样的边缘。 它由John F. Canny于1986年开发。

Canny边缘检测是一个四步过程：
 1. 应用高斯模糊来清除任何斑点并释放噪声图像。
 2. 应用梯度算子来获得梯度的强度和方向。
 3. 非最大抑制确定像素是否比边缘的边缘更适合边缘。
 4. 滞后阈值查找边缘开始和结束的位置。

Canny算法包含许多可调参数，这些参数会影响算法的计算时间和有效性。
 - 高斯滤波器的大小：第一阶段使用的平滑滤波器直接影响Canny算法的结果。 较小的滤镜可以减少模糊，并且可以检测到细小锐利的线条。 较大的滤镜会导致更多的模糊，并会在图像的较大区域上抹去给定像素的值。
 - 阈值：阈值设置过高可能会错过重要信息。 另一方面，设置的阈值太低会错误地将不相关的信息（如噪声）识别为重要的。 很难给出一个适用于所有图像的通用阈值。 对这个问题还没有经过验证的方法。

为了达到我们的目的，我们将设置过滤器大小为3，并使用Slider可以编辑阈值。

彭咋和侵蚀
----------------------
膨胀和侵蚀是最基本的形态学操作。 膨胀将像素添加到图像中对象的边界，而侵蚀则删除对象边界上的像素。 从图像中的对象添加或移除的像素数量取决于用于处理图像的结构化元素的大小和形状。 在形态膨胀和腐蚀操作中，输出图像中任何给定像素的状态是通过将规则应用于输入图像中的相应像素及其邻居来确定的。 用于处理像素的规则将操作定义为膨胀或侵蚀。

**膨胀**：输出像素的值是输入像素邻域中所有像素的最大值。 在二进制图像中，如果任何像素被设置为值1，则输出像素被设置为1。

**腐蚀**: 输出像素的值是输入像素邻域中所有像素的最小值。 在二进制图像中，如果任何像素被设置为0，则输出像素被设置为0。

我们将在本教程中做什么
--------------------------------
在本教程中，我们将会:
 * 添加一个复选框和一个* Slider *来选择和控制Canny边缘检测器。
 *使用OpenCV提供的Canny边缘功能。
 * 使用OpenCV提供的膨胀和腐蚀功能。
 * 创建一个简单的背景删除功能。

开始
---------------
建立一个新的 JavaFX 工程。在 Scene Builder 中设置窗口元素，获得一个GUI窗口:

- 在顶部有一个包含两个HBox的VBox，还有一个分隔器。

 + 在第一个HBox中，我们需要一个复选框和一个* slider *，第一个是选择Canny e.d. 模式，第二个将用于控制传递给Canny e.d的阈值。 功能。

	.. code-block:: xml

    		<CheckBox fx:id="canny" onAction="#cannySelected" text="Edge detection"/>
    		<Label text="Canny Threshold" />
    		<Slider fx:id="threshold" disable="true" />

 + 在第二个HBox中，我们需要两个复选框，第一个选择背景删除模式，第二个复选框说我们是否想要颠倒算法（一种“前景去除”）。

	.. code-block:: xml

    		<CheckBox fx:id="dilateErode" onAction="#dilateErodeSelected" text="Background removal"/>
    		<CheckBox fx:id="inverse" text="Invert" disable="true"/>

- 在** CENTER **中，我们将为网络摄像头流设置一个ImageView。

.. code-block:: xml

    <ImageView fx:id="originalFrame" />

- 在** BOTTOM **上，我们可以添加通常的按钮来启动/停止流

.. code-block:: xml

    <Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />

GUI看起来像这样：

.. image:: _static/07-00.png

使用Canny边缘检测
------------------------------
如果我们选择了Canny复选框，我们可以执行“doCanny”方法。

.. code-block:: java

    if (this.canny.isSelected()){
	frame = this.doCanny(frame);
    }

``doCanny``是我们定义的执行边缘检测的方法。
首先，我们将图像转换为灰度图像，并使用内核大小为3的滤镜进行模糊处理：

.. code-block:: java

    Imgproc.cvtColor(frame, grayImage, Imgproc.COLOR_BGR2GRAY);
    Imgproc.blur(grayImage, detectedEdges, new Size(3, 3));

其次，我们使用OpenCV中的Canny函数：

.. code-block:: java

    Imgproc.Canny(detectedEdges, detectedEdges, this.threshold.getValue(), this.threshold.getValue() * 3, 3, false);

参数是：
 - ``detectedEdges``: 源图像，灰度
 - ``detectedEdges``: 检测器的输出（可以与输入相同）
 - ``this.threshold.getValue()``: 用户输入的值来改变Slider
 - ``this.threshold.getValue() * 3``: 在程序中设置为下限阈值的三倍（遵循Canny的建议）
 - ``3``: 使用的Sobel内核的大小
 - ``false``: 一个标志，指示是否使用更精确的幅度梯度计算。

然后我们用零填充“dest”图像（意味着图像完全是黑色的）。

.. code-block:: java

    Mat dest = new Mat();
    Core.add(dest, Scalar.all(0), dest);

最后，我们将使用函数copyTo来仅映射被识别为边缘的图像区域（在黑色背景上）。

.. code-block:: java

    frame.copyTo(dest, detectedEdges);

``copyTo`` 将src图像复制到``dest``。但是，它只会复制具有非零值的位置的像素。


Canny边缘检测的结果
------------

.. image:: _static/07-01.png

使用背景删除
----------------------------
如果我们选择了背景删除复选框，我们可以执行“doBackgroundRemoval”方法

.. code-block:: java

    else if (this.dilateErode.isSelected())
    {
	frame = this.doBackgroundRemoval(frame);
    }

``doBackgroundRemoval`` 是我们定义的执行背景删除的方法。

首先我们需要转换HSV中的当前帧：

.. code-block:: java

    hsvImg.create(frame.size(), CvType.CV_8U);
    Imgproc.cvtColor(frame, hsvImg, Imgproc.COLOR_BGR2HSV);
    Now let's split the three channels of the image:
    Core.split(hsvImg, hsvPlanes);

计算色调分量平均值：

.. code-block:: java

    Imgproc.calcHist(hue, new MatOfInt(0), new Mat(), hist_hue, histSize, new MatOfFloat(0, 179));
    for (int h = 0; h < 180; h++)
	average += (hist_hue.get(h, 0)[0] * h);
    average = average / hsvImg.size().height / hsvImg.size().width;

如果背景是统一的并填充大部分框架，其值应该接近刚刚计算的平均值。
然后，我们可以使用均值作为阈值将背景从前景中分离出来，具体取决于我们需要执行后（前）地面去除的反转复选框：

.. code-block:: java

    if (this.inverse.isSelected())
	Imgproc.threshold(hsvPlanes.get(0), thresholdImg, threshValue, 179.0, Imgproc.THRESH_BINARY_INV);
   else
	Imgproc.threshold(hsvPlanes.get(0), thresholdImg, threshValue, 179.0, Imgproc.THRESH_BINARY);

现在我们使用一个5x5内核掩码的低通滤波器（模糊）来增强结果：

.. code-block:: java

    Imgproc.blur(thresholdImg, thresholdImg, new Size(5, 5));

最后在图像上应用*膨胀*然后*侵蚀*（**关操作**）：

.. code-block:: java

    Imgproc.dilate(thresholdImg, thresholdImg, new Mat(), new Point(-1, -1), 1);
    Imgproc.erode(thresholdImg, thresholdImg, new Mat(), new Point(-1, -1), 3);

这些函数采用这些参数：
 - ``thresholdImg`` 输入图像;
 - ``thresholdImg`` 输出与thresholdIng相同大小和类型的图像;
 - ``new Mat()`` 一个内核;
 - ``new Point(-1, -1)`` 元素内锚的位置; 默认值（-1，-1）表示锚点位于元素中心。
 - ``6`` 操作的次数。

关闭后，我们需要做一个新的二进制阈值：

.. code-block:: java

    Imgproc.threshold(thresholdImg, thresholdImg, threshValue, 179.0, Imgproc.THRESH_BINARY);

最后，我们可以将刚刚获得的图像作为掩码应用于原始帧：

.. code-block:: java

    Mat foreground = new Mat(frame.size(), CvType.CV_8UC3, new Scalar(255, 255, 255));
    frame.copyTo(foreground, thresholdImg);

背景去除结果
-------------------------

.. image:: _static/07-02.png

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。

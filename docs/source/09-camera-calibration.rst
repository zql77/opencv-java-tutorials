==================
相机校准
==================

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

.. warning:: 这个教程没有更新到OpenCV 3.x.

目标
----
本教程的目标是学习如何在给定一组棋盘图像的情况下校准摄像机。

什么是相机校准？
-------------------------------
相机校准是我们可以获得相机参数（如内部和外部参数，失真等）的过程。 当镜头和光学传感器芯片之间的对准不正确时，通常需要校准相机; 在低质量摄像机中，由错误对齐产生的效果通常更明显。

校准模式
-------------------
正如我们前面所说的，我们需要某种程序可以识别的模式来进行校准工作。 我们要使用的模式是棋盘图像。

.. image:: _static/05-00.png

我们使用这个图像的原因是因为有一些OpenCV函数可以识别这个模式并绘制一个突出显示每个块之间交集的方案。
要进行校准工作，您需要打印棋盘图像并将其显示给凸轮; 保持纸张仍然很重要，如果粘在表面上会更好。
为了做出好的校准，我们需要从不同的角度和距离获得大约20个样本。

本教程中我们将要进行的操作
--------------------------------
在本教程中，我们将会:
 * 创建一些TextEdit字段来为我们的程序提供一些输入
 * 使用一些OpenCV函数识别模式
 * 校准并显示视频流。

开始
---------------
用通常的OpenCV用户库创建一个新的JavaFX项目（例如 " CameraCalibration " ）。
打开 Scene Builder 并添加一个边框窗格：

- 在顶部，我们需要设置校准样本的数量，测试图像中水平角的数量，测试图像中垂直角的数量以及更新按钮 这个数据。 为了让事情变得更清洁，我们把所有这些元素都放入HBox中。

.. code-block:: xml

    <HBox alignment="CENTER" spacing="10">

让我们在每个文本字段前添加一些标签。
每个文本字段都需要一个id，我们已经为它们设置了一个标准值。

.. code-block:: xml

    <Label text="Boards #" />
    <TextField fx:id="numBoards" text="20" maxWidth="50" />
    <Label text="Horizontal corners #" />
    <TextField fx:id="numHorCorners" text="9" maxWidth="50" />
    <Label text="Vertical corners #" />
    <TextField fx:id="numVertCorners" text="6" maxWidth="50" />

对于该按钮，请为onAction字段设置id和方法：

.. code-block:: xml

    <Button fx:id="applyButton" alignment="center" text="Apply" onAction="#updateSettings" />

- 在** LEFT **上添加一个ImageView在一个VBox内的正常凸轮流; 为它设置一个ID。

.. code-block:: xml

    <ImageView fx:id="originalFrame" />

- 在** RIGHT **上添加一个用于校准凸轮流的VBox内的ImageView; 为它设置一个ID。

.. code-block:: xml

    <ImageView fx:id="originalFrame" />

-在** BOTTOM **中，在HBox内添加一个开始/停止采样视频流按钮和一个快照按钮; 为每一个设置一个id和一个操作方法。

.. code-block:: xml

    <Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />
    <Button fx:id="snapshotButton" alignment="center" text="Take snapshot" onAction="#takeSnapshot" disable="true" />

你的GUI看起来将会这是这样:

.. image:: _static/05-03.png

模式识别
-------------------
校准过程包括从不同角度，深度和视角向凸轮显示棋盘图案。 对于我们需要跟踪的每个识别模式：

 -棋盘所在的一些参考系统的3D点（让我们假设Z轴始终为0）：

	.. code-block:: java

		for (int j = 0; j < numSquares; j++)
		   obj.push_back(new MatOfPoint3f(new Point3(j / this.numCornersHor, j % this.numCornersVer, 0.0f)));

 - 图像的2D点（由OpenCV使用findChessboardCorners进行的操作):

	.. code-block:: java

		boolean found = Calib3d.findChessboardCorners(grayImage, boardSize, imageCorners, Calib3d.CALIB_CB_ADAPTIVE_THRESH + Calib3d.CALIB_CB_NORMALIZE_IMAGE + Calib3d.CALIB_CB_FAST_CHECK);

 " findChessboardCorners " 函数试图确定输入图像是否是棋盘图案的视图，并找到内部棋盘角落。
参数如下:

 - **image** 源棋盘视图。 它必须是8位灰度或彩色图像。
 - **patternSize** 每个棋盘行和列的内角的数量。
 - **corners** 输出检测到的角落数组。
 - **flags** 各种操作标志可以是零或以下值的组合：
	- ``CV_CALIB_CB_ADAPTIVE_THRESH`` 使用自适应阈值将图像转换为黑白，而不是固定的阈值级别（从平均图像亮度计算）。
	- ``CV_CALIB_CB_NORMALIZE_IMAGE`` 在应用固定或自适应阈值之前，使用“equalizeHist”对图像伽玛进行归一化。
	- ``CV_CALIB_CB_FILTER_QUADS`` 使用额外的标准（如轮廓区域，周长，方形形状）来过滤在轮廓检索阶段提取的虚假四边形。
	- ``CALIB_CB_FAST_CHECK`` 对查找棋盘角的图像执行快速检查，如果找不到任何内容，则快速调用该电话。 当没有观察到棋盘时，这可以极大地加速在退化状态下的呼叫。

.. warning:: 在执行``findChessboardCorners``之前，将图像转换为灰度并将纸板大小保存为一个Size变量：

	.. code-block:: java

	    Imgproc.cvtColor(frame, grayImage, Imgproc.COLOR_BGR2GRAY);
	    Size boardSize = new Size(this.numCornersHor, this.numCornersVer);

如果识别进行得很好，" 发现 " 应该是" 正确的 "。

对于方形图像，角落的位置只是近似的。 我们可以通过调用 " cornerSubPix " 函数来改善这一点。 它会产生更好的校准结果。

.. code-block:: java

    TermCriteria term = new TermCriteria(TermCriteria.EPS | TermCriteria.MAX_ITER, 30, 0.1);
    Imgproc.cornerSubPix(grayImage, imageCorners, new Size(11, 11), new Size(-1, -1), term);

我们现在可以突出显示在线发现的点：

.. code-block:: java

    Calib3d.drawChessboardCorners(frame, boardSize, imageCorners, found);

该功能可以将检测到的单个棋盘角落绘制为红色圆圈（如果找不到该板），或者如果找到该板，则将彩色拐角与线连接。

参数如下:

 - **image** 目的地图像。 它必须是8位彩色图像。
 - **patternSize** 每个棋盘行和列的内角的数量。
 - **corners** 检测角落的阵列，输出检测到的角点
 - **patternWasFound** 指示是否找到完整的参数。 " findChessboardCorners " 的返回值应该在这里传递。

现在我们可以激活快照按钮来保存数据。

.. code-block:: java

    this.snapshotButton.setDisable(false);

.. image:: _static/05-01.png

.. image:: _static/05-02.png

我们应该从不同的角度和深度采集设置的“快照”数量，以便进行校准。

.. note:: 我们实际上并没有保存图像，只是保存了我们需要的数据。

保存数据
-----------
通过点击快照按钮，我们称之为 " takeSnapshot " 方法。 如果我们没有做足够的样本，我们需要保存数据（2D和3D点）：

.. code-block:: java

    this.imagePoints.add(imageCorners);
    this.objectPoints.add(obj);
    this.successes++;

否则，我们可以校准相机。

相机校准
------------------
对于相机校准，我们应该创建一些需要的变量，然后调用实际的校准函数：

.. code-block:: java

    List<Mat> rvecs = new ArrayList<>();
    List<Mat> tvecs = new ArrayList<>();
    intrinsic.put(0, 0, 1);
    intrinsic.put(1, 1, 1);

    Calib3d.calibrateCamera(objectPoints, imagePoints, savedImage.size(), intrinsic, distCoeffs, rvecs, tvecs);

calibrateCamera函数估计每个视图的内在摄像机参数和外部参数。 该算法基于[Zhang2000]和[BouguetMCT]。 必须指定3D对象点的坐标及其在每个视图中对应的2D投影。
参数如下:

 - **objectPoints** 在新界面中，它是校准图案坐标空间中的校准图案点矢量的向量。 外部向量包含与模式视图数量一样多的元素。 这些点是3D的，但由于它们处于图案坐标系中，因此如果平台是平面的，则将模型放置到XY坐标平面以使每个输入对象点的Z坐标为0是有意义的。
 - **imagePoints** 它是校准模式点投影向量的向量。
 - **imageSize** 仅用于初始化内置相机矩阵的图像大小。
 - **cameraMatrix** 输出3x3浮点相机矩阵* A = | fx 0 cx | | 0 fy cy | | 0 0 1 | *。 如果指定了 " CV_CALIB_USE_INTRINSIC_GUESS " 或 " CV_CALIB_FIX_ASPECT_RATIO " ，则在调用函数之前，必须初始化* fx *，* fy *，* cx *，* cy *中的部分或全部。
 - **distCoeffs** 4，5或8个元素的失真系数的输出向量。
 - **rvecs** 为每个模式视图估计的旋转矢量的输出矢量。 也就是说，每个第k个旋转矢量与相应的第k个平移矢量一起。
 - **tvecs** 为每个模式视图估算的平移向量的输出向量。

我们进行了校准并获得了具有失真系数的相机矩阵，我们可能需要使用 " undistort " 函数来校正图像：

.. code-block:: java

    if (this.isCalibrated)
    {
	// prepare the undistored image
	Mat undistored = new Mat();
	Imgproc.undistort(frame, undistored, intrinsic, distCoeffs);
	undistoredImage = mat2Image(undistored);
    }

 " undistort " 功能可转换图像以补偿径向和切向镜头失真。

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。

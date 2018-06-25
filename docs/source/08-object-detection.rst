=================
对象检测
=================

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。
Goal
----

在本教程中，我们将识别并追踪一个或多个网球。 它通过使用球的颜色范围，侵蚀和膨胀以及findContours方法在网络摄像头视频流中执行网球检测。

形态学图像处理
------------------------------
是与图像中的特征形态相关的非线性操作的集合。 形态学操作仅依赖于像素值的相对排序而不依赖于其数值。
一些基本的形态学操作是扩张和侵蚀。 膨胀会导致对象扩大或增大，从而将像素添加到图像中对象的边界，因此不同区域内的空洞会变小。 例如，膨胀允许连接看似分离的对象的部分。
通过从图像中的物体边界剥离像素层，侵蚀会导致物体收缩，因此不同区域内的空洞会变大。 由于扫描过程，侵蚀可用于消除图像中的噪音或小错误。
开放是一个复合操作，它包含一个侵蚀和一个扩展，两个操作使用相同的结构元素。 此操作可从图像的前景中移除小对象，并可用于查找特定结构元素可适用的东西。 开口可以打开通过像素的薄桥连接的物体之间的间隙。 任何受侵蚀的地区都可以通过扩张恢复原有的规模。


本教程中我们将要进行的操作
--------------------------------
在本教程中，我们将会:

 * 插入3组滑块以控制图像的HSV（色调，饱和度和值）的大小。
 * 捕获并处理网络摄像头中的图像以消除噪音，以促进对象识别。
 * 最后使用形态学算子如侵蚀和膨胀，我们可以使用图像处理后得到的结果来识别物体。

开始
---------------
我们来创建一个新的JavaFX项目。 在场景生成器中，我们设置了窗口元素，以便我们有一个边框窗格：

-在RIGHT CENTRE上，我们可以添加一个VBox。 在这一个中，我们将需要6个滑块，第一对将控制色调，下一个饱和度和最终亮度，使用这些滑块可以更改HSV图像的值。

	.. code-block:: xml

               	<Label text="Hue Start" />
		<Slider fx:id="hueStart" min="0" max="180" value="20" blockIncrement="1" />
		<Label text="Hue Stop" />
		<Slider fx:id="hueStop" min="0" max="180" value="50" blockIncrement="1" />
		<Label text="Saturation Start" />
		<Slider fx:id="saturationStart" min="0" max="255" value="60" blockIncrement="1" />
		<Label text="Saturation Stop" />
		<Slider fx:id="saturationStop" min="0" max="255" value="200" blockIncrement="1" />
		<Label text="Value Start" />
		<Slider fx:id="valueStart" min="0" max="255" value="50" blockIncrement="1" />
		<Label text="Value Stop" />
		<Slider fx:id="valueStop" min="0" max="255" value="255" blockIncrement="1" />


- 在中心。 我们将放置三个ImageViews，第一个显示来自网络摄像头流的正常图像，第二个显示蒙版图像，最后一个显示变形图像。 HBox用于正常图像，VBox用于放置其他图像。

	.. code-block:: xml

		<HBox alignment="CENTER" spacing="5">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="originalFrame" />
			<VBox alignment="CENTER" spacing="5">
				<ImageView fx:id="maskImage" />
				<ImageView fx:id="morphImage" />
			</VBox>
		</HBox>

- 在BOTTOM上，我们可以添加通常的按钮来启动/停止视频流，并使用滑块选择当前的HSV值。

	.. code-block:: xml

		<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" />
		<Separator />
		<Label fx:id="hsvCurrentValues" />

GUI看起来像这样：

.. image:: _static/09-00.png


图像处理
----------------
为了使用形态学算子并获得良好的结果，我们需要处理图像并消除噪音，将图像更改为HSV可以轻松获得轮廓。

- ``去除噪声``
	我们可以使用Imgproc类的方法blur去除图像的一些噪点，然后将转换应用于
	HSV为了促进对象识别的过程。

	.. code-block:: java

		Mat blurredImage = new Mat();
		Mat hsvImage = new Mat();
		Mat mask = new Mat();
		Mat morphOutput = new Mat();

		// remove some noise
		Imgproc.blur(frame, blurredImage, new Size(7, 7));

		// convert the frame to HSV
		Imgproc.cvtColor(blurredImage, hsvImage, Imgproc.COLOR_BGR2HSV);



- ``HSV图像的大小``
	我们可以使用滑块修改HSV图像的值，图像将实时更新，
	允许增加或减少识别物体进入图像的能力。

	.. code-block:: java


		// get thresholding values from the UI
		// remember: H ranges 0-180, S and V range 0-255
		Scalar minValues = new Scalar(this.hueStart.getValue(), this.saturationStart.getValue(),
		this.valueStart.getValue());
		Scalar maxValues = new Scalar(this.hueStop.getValue(), this.saturationStop.getValue(),
		this.valueStop.getValue());

		// show the current selected HSV range
		String valuesToPrint = "Hue range: " + minValues.val[0] + "-" + maxValues.val[0]
		+ "\tSaturation range: " + minValues.val[1] + "-" + maxValues.val[1] + "\tValue range: "
		+ minValues.val[2] + "-" + maxValues.val[2];
		this.onFXThread(this.hsvValuesProp, valuesToPrint);

		// threshold HSV image to select tennis balls
		Core.inRange(hsvImage, minValues, maxValues, mask);
		// show the partial output
		this.onFXThread(maskProp, this.mat2Image(mask));


形态学操作员
-----------------------
首先，我们需要定义形态算子扩张和侵蚀的两个矩阵，然后使用类Imgproc的侵蚀和扩张方法在每次操作中对图像进行两次处理，结果是矩阵morphOutput将成为局部输出。


	.. code-block:: java

	       // morphological operators
	       // dilate with large element, erode with small ones
	        Mat dilateElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(24, 24));
		Mat erodeElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(12, 12));

		Imgproc.erode(mask, morphOutput, erodeElement);
		Imgproc.erode(mask, morphOutput, erodeElement);

		Imgproc.dilate(mask, morphOutput, dilateElement);
		Imgproc.dilate(mask, morphOutput, dilateElement);

		// show the partial output
		this.onFXThread(this.morphProp, this.mat2Image(morphOutput));



对象跟踪
------------------
通过在我们使用类Imgpoc的findContours方法获得的部分输出获得一个矩阵与识别的对象的映射，然后我们绘制这些对象的轮廓。


	.. code-block:: java

		// init
		List<MatOfPoint> contours = new ArrayList<>();
		Mat hierarchy = new Mat();

		// find contours
		Imgproc.findContours(maskedImage, contours, hierarchy, Imgproc.RETR_CCOMP, Imgproc.CHAIN_APPROX_SIMPLE);

		// if any contour exist...
		if (hierarchy.size().height > 0 && hierarchy.size().width > 0)
		{
			// for each contour, display it in blue
			for (int idx = 0; idx >= 0; idx = (int) hierarchy.get(0, idx)[0])
			{
				Imgproc.drawContours(frame, contours, idx, new Scalar(250, 0, 0));
			}
		}


最后我们将会得到这样的结果：

.. image:: _static/09-01.png

.. image:: _static/09-02.png

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。
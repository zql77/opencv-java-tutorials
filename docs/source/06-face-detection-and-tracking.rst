=============================
人脸检测和跟踪
=============================

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

目标
----
在本教程中，我们将使用已经由OpenCV进行培训和分发的知名分类器，以便检测和跟踪视频流中的移动面。

级联分类器
-------------------
对象识别过程（在我们的例子中为脸部）通常是有效的，如果它基于包含有关要接管的对象类的附加信息的特征接管。 在本教程中，我们将使用* Haar-like特征*和*局部二值模式*（LBP）来编码人脸突出显示的对比度及其与图片中存在的其他对象的空间关系。
通常这些特征是使用*级联分类器*提取的，必须对这些特征进行训练以便精确识别不同的对象：面部的分类将与汽车的分类大不相同。

本教程中我们将要进行的操作
--------------------------------
在本教程中，我们将会:
 * 插入一个复选框以选择Haar分类器，检测并跟踪一张脸，然后在检测到的脸部周围画一个绿色矩形。
 * 插入复选框以选择LBP分类器，检测并跟踪脸部，并在检测到的脸部周围绘制绿色矩形。

开始
---------------
建立一个新的 JavaFX 工程。在 Scene Builder 中设置窗口元素，获得一个GUI窗口:

- 在顶部有一个 VBox ,一个 HBox 和一个分离器 . 在HBox中，我们需要两个复选框，第一个是选择Haar分类器，第二个是选择LBP分类器。

	.. code-block:: xml

		<CheckBox fx:id="haarClassifier" onAction="#haarSelected" text="Haar Classifier"/>
		<CheckBox fx:id="lbpClassifier" onAction="#lbpSelected" text="LBP Classifier"/>

- 在CENTER中，我们将为网络摄像机流添加一个ImageView。

	.. code-block:: xml

		<ImageView fx:id="originalFrame" />

- 在底部，我们可以添加通常的按钮来启动/停止视频采样流。

	.. code-block:: xml

		<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />

GUI将会像以下这种样子:

.. image:: _static/08-00.png

加载分类器
-----------------------
首先，我们需要为我们的项目添加一个文件夹 "resource" ，并将分类器放入其中。
为了使用分类器，我们需要从资源文件夹中加载它们，所以每次我们检查其中一个复选框时，我们将加载正确的分类器。
为此，我们来实现我们之前已经声明的``OnAction``方法：

- ``haarSelected``
	在这个方法中，我们将加载所需的Haar分类器（例如``haarcascade_frontalface_alt.xml``），如下所示：

	.. code-block:: java

		this.checkboxSelection("resources/lbpcascades/haarcascade_frontalface_alt.xml");
		...
		private void checkboxSelection(String... classifierPath)
		{
			// load the classifier(s)
			for (String xmlClassifier : classifierPath)
			{
				this.faceCascade.load(xmlClassifier);
			}

			// now the capture can start
			this.cameraButton.setDisable(false);
		}

- ``lbpSelected``
	对于LPB，我们可以使用相同的方法并更改要加载的分类器的路径：

	.. code-block:: java

		this.checkboxSelection("resources/lbpcascades/lbpcascade_frontalface.xml");

检测和跟踪
----------------------
一旦我们加载了分类器，我们就可以开始检测; 我们将在``detectAndDisplay``方法中实现检测。
首先，我们需要将灰度转换为帧并均衡直方图以改善结果：

.. code-block:: java

    Imgproc.cvtColor(frame, grayFrame, Imgproc.COLOR_BGR2GRAY);
    Imgproc.equalizeHist(grayFrame, grayFrame);

然后，我们必须设置要检测的面部的最小尺寸（这在实际检测功能中是需要的）。 我们将最小尺寸设置为框架高度的20％：

.. code-block:: java

    if (this.absoluteFaceSize == 0)
    {
	int height = grayFrame.rows();
	if (Math.round(height * 0.2f) > 0)
	{
		this.absoluteFaceSize = Math.round(height * 0.2f);
	}
    }

现在我们可以开始检测：

.. code-block:: java

    this.faceCascade.detectMultiScale(grayFrame, faces, 1.1, 2, 0 | Objdetect.CASCADE_SCALE_IMAGE, new Size(this.absoluteFaceSize, this.absoluteFaceSize), new Size());

``detectMultiScale``函数检测输入图像中不同大小的对象。 返回检测到的对象的矩形列表。
参数是：

 - **image** 包含检测对象的图像的类型为CV_8U的矩阵。
 - **objects** 每个矩形包含检测到的对象的矩形矢量。
 - **scaleFactor** 指定每个图像比例缩小图像大小的参数。
 - **minNeighbors** 指定每个候选矩形应该保留多少个邻居的参数。
 - **flags** 参数与旧函数cvHaarDetectObjects中的相同。 它不用于新的级联。
 - **minSize** 最小可能的对象大小。 小于此值的对象将被忽略。
 - **maxSize** 最大可能的对象大小。 大于此值的对象将被忽略。

所以检测结果将会出现在** objects **参数中，或者我们的情况是`faces``。

让我们把这个结果放在一个矩阵数组中，并在框架上绘制它们，这样我们可以显示检测到的面：

.. code-block:: java

    Rect[] facesArray = faces.toArray();
    for (int i = 0; i < facesArray.length; i++)
	Imgproc.rectangle(frame, facesArray[i].tl(), facesArray[i].br(), new Scalar(0, 255, 0, 255), 3);

正如你所看到的，我们用透明背景选择了绿色：``标量（0,255,0,255）``。
``.tl（）``和``.br（）``代表*左上角*和右下角*，它们代表两个相反的顶点。
最后一个参数只是设置矩形边框的厚度。

跟踪部分可以通过调用每个帧的“detectAndDisplay”方法来实现。

.. image:: _static/08-01.png

.. image:: _static/08-02.png

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。

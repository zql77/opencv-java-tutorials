=============
OpenCV基础
=============

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

本教程中我们将要进行的操作
--------------------------------
在本教程中，我们将会:
 * 创建一个基本的*复选框（checkbox）* 交互的改变视频流的颜色。
 * 添加一个基本的 *复选框（checkbox）* 交互的改变视频流的标志的 "阿尔法度（alpha over）"。
 * 显示视频流的 *直方图* (不管是单通道还是三通道)。

开始
---------------
对于本教程，我们可以创建一个新的JavaFX项目，我们像前面实现一个场景一样实现一个场景。我们会得到一个带边框的窗口，其中：

- 在底部，我们在*HBox*中实现一个按钮:

.. code-block:: xml

    <HBox alignment="CENTER" >
       <padding>
          <Insets top="25" right="25" bottom="25" left="25"/>
       </padding>
       <Button fx:id="button" alignment="center" text="Start camera" onAction="#startCamera" />
    </HBox>

- 在 **CENTER** 我们实现一个 ImageView:

.. code-block:: xml

    <ImageView fx:id="currentFrame" />

彩色通道复选框
----------------------
用Scene Builder打开我们的fxml文件，同时将一个垂直布局的 ``VBox`` 添加到我们的BorderPane得 **RIGHT** 文件中。`` VBox `` 中的子元素都是单向垂直排列的。如果 `` VBox `` 设定了 `` border `` 或者 `` padding `` 属性, 则将内容放入这些标签中。 此外，还会调整子控件(如果大小可调)到适应的大小，``fillWidth`` 标签用来设定自己的大小是否是可自适应调整的，或者自己设定的大小（fillWidth默认值为true）。
``HBox`` 和 `` VBox `` 工作原理一样，只是在他里面的子控件的排列方向是水平的。

现在我们在 `` VBox `` 添加一个 `` checkbox ``, 将他的 `` text `` 属性设定为 "Show in gray scale", 同时设定他的ID (例如： "grayscale").

.. code-block:: xml

    <CheckBox fx:id="grayscale" text="Show in gray scale" />

让我们为 `` VBox `` 中的复选框。然后, 设置他的text属性为"Controls" (可以在 ``Shapes`` 菜单下找到text属性)。

.. code-block:: xml

    <Text text="Controls" />

在Scene Builder中我们最终得到:

.. image:: _static/04-00.png

第一项任务的图形界面已完成，现在我们要完善我们的控制器类；在之前的教程中，我们可以使用以下方式控制图像的颜色变换：

.. code-block:: java

    Imgproc.cvtColor(frame, frame, Imgproc.COLOR_BGR2GRAY);

为了使用复选框控制此转换，我们必须将复选框与FXML变量绑定起来：

.. code-block:: java

    @FXML
    private CheckBox grayscale;

现在我们可以通过添加一个简单的“if”条件语句来实现控制，该条件仅在我们的复选框被选中时才会执行转换：

.. code-block:: java

    if (grayscale.isSelected())
    {
       Imgproc.cvtColor(frame, frame, Imgproc.COLOR_BGR2GRAY);
    }

加载图像并将其转换为视频流
--------------------------------------
下一步是添加另一个复选框，当选中该复选框，将触发在相机流上显示图像。
我们首先将图像添加到项目中; 在项目的根目录下创建一个新文件夹，并将图像放在那里。
在我的项目里，我将``Poli.png``图像放到了``resources``文件夹里。
回到Eclipse中刷新工作区(你就会发现你新建立的文件夹)。
然后我们使用Scene Builder打开FXML文件在控制颜色的复选框下面添加一个新的复选框; 我们需要设定``text``和``id``值, 同时还需要在``OnAction``字段设定回调方法的名称。
比如我们这么写:

.. code-block:: xml

    <CheckBox fx:id="logoCheckBox" text="Show logo" onAction="#loadLogo" />

在控制器类中我们需要定义一个与复选框相关联的变量,``OnAction``字段中设定的方法使得复选框被选中时能够在视频上显示`` logo ``。
变量:

.. code-block:: java

    @FXML
    private CheckBox logoCheckBox;


``loadLogo`` 方法:
只要`` logoCheckBox ``被选中(被勾选)，我们就会通过这个方法加载图像。
我们需要使用OpenCV的imread函数来加载图像。
他需要两个参数，一个是输入图像，另一个是图像标志（> 0 RGB图像，= 0灰度，<alpha通道）,并返回Mat数据结。

.. code-block:: java

    @FXML
    protected void loadLogo()
    {
     if (logoCheckBox.isSelected())
        this.logo = Imgcodecs.imread("resources/Poli.png");
    }

修改这代码。

为了在视频流中显示我们的`` logo `` 我们需要对代码做一些修改。因此在捕获每一帧图像后，和将图像转换为1-3通道的图像流前，我们需要设定 **ROI** （感兴趣区域）并将``logo``添加到感兴趣区域。 This means that for each frame capture, before the image could be converted into 1 or 3 channels, we have to set a **ROI** (region of interest) in which we want to place the logo.
通常ROI是图像的一部分，我们可以将ROI定义为Rect对象。
Rect是2D矩形的模板类，以下是它的参数描述：

 * 坐标原点。 这是OpenCV中Rect.x和Rect.y的默认是在左上角。当然，在你的算法中，你可以设定原点在左下角。
 * 感兴趣矩形区域的宽和高。

.. code-block:: java

    Rect roi = new Rect(frame.cols()-logo.cols(), frame.rows()-logo.rows(), logo.cols(), logo.rows());

只有相同尺寸的Mat才可以进行相加，所以我们需要操纵ROI(感兴趣区域)，使得能够把我们添加的`` logo ``添加到每一帧设定的感兴趣区域内。

.. code-block:: java

    Mat imageROI = frame.submat(roi);

 怎样将两个Mat数据结构相加呢? 我们知道我们的`` logo ``有4个通道 (RGB + alpha)。所以我们可以使用这两个函数: ``addWeighted`` 和 ``copyTo``。
``addWeighted``函数计算两个数组的加权和，如下所示：

		*dst(I)= saturate(src1(I)* alpha + src2(I)* beta + gamma)*

其中`` I ``是数组元素的多维索引。 在多通道阵列的情况下，每个通道都是独立处理的。 该函数可以用一个矩阵表达式替换：

		*dst = src1*alpha + src2*beta + gamma*

.. 注意:: 当输出数组的深度是``CV_32S``时不应用饱和度。在溢出的情况下，我们会得不到正确的结果。

参数:
 - **src1** 第一个输入数组。
 - **alpha** 第一个数组元素的权重。
 - **src2** 与 **src1** 具有相同大小和通道编号的第二个输入数组。
 - **beta** 第二个数组元素的权重。
 - **gamma** 添加到每一个和的标量。
 - **dst** 输出数组与输入数组具有相同大小和数量的通道。

具体如下:

.. code-block:: java

    Core.addWeighted(imageROI, 1.0, logo, 0.7, 0.0, imageROI);

第二种``copyTo``方法只是简单的将一个Mat复制到另一个中。代码如下:

.. code-block:: java

    Mat mask = logo.clone();
    logo.copyTo(imageROI, mask);

迄今为止，我们将`` logo ``添加到感兴趣区域的每一步操作都是建立在，复选框被选中，同时图像已经加载完成的情况下。所以我们必须添加一个if语句判断是否满足以上两个条件：

.. code-block:: java

    if (logoCheckBox.isSelected() && this.logo != null)
    {
	Rect roi = new Rect(frame.cols() - logo.cols(), frame.rows() - logo.rows(), logo.cols(),logo.rows());
	Mat imageROI = frame.submat(roi);
	// add the logo: method #1

	Core.addWeighted(imageROI, 1.0, logo, 0.7, 0.0, imageROI);
	// add the logo: method #2
	// Mat mask = logo.clone();
	// logo.copyTo(imageROI, mask);
    }

计算直方图
---------------------
直方图是数值数据分布的精确图形表示，由一个一个的区间表示为直方图。
在我们的例子中，数据表示像素的亮度，所以它的范围四（0,256）。

既然我们知道了每个像素值的范围，我们可以取一个一个像素值的范围（称为箱子），最后统计所有像素在这些范围的分布：
 1. **dims**: 你想要设定的“箱子”的个数。
 2. **bins**: 每个暗箱中的细分数。 在我们的例子中，bin = 256
 3. **range**: 要测量的值的范围。在我们的例子中：范围= [0,255]

我们的最后一个目标是显示RGB或灰度图像视频流的直方图。
为此我们在控制器类中创建一个方法，它接受一个Mat(当前帧)，返回一个布尔值，判断图像是RGB还是灰度图像的。代码如下：

.. code-block: java

    private void showHistogram(Mat frame, boolean gray){ ... }

我们首先需要把当前帧分解为`` n ``个帧，`` n ``为我们图像的通道数，这需要利用 ``Core.split`` 函数完成。它需要一个Mat（输入图像）和一个List<Mat>（储存每个通道的图像副本，如果图像是灰度图片，那么List<Mat>将只有一个）。

.. code-block: java

    List<Mat> images = new ArrayList<Mat>();
    Core.split(frame, images);


在我们计算每个通道的直方图之前，我们必须了解`` calcHist ``函数所需的所有参数。
calcHist函数计算一个或多个数组的直方图。每一个归类都是输入数组中满足一定范围值的统计。
参数:

 - **images** 源数组。 源数组都应具有相同的深度（如CV_8U或CV_32F）以及相同的尺寸。源数组可以有任意通道。
 - **channels** 用于计算直方图的调光通道列表。第一个通道数组从 0 到 images[0].channels()-1, 第二个通道数组从 images[0].channels() 到 images[0].channels() + images[1].channels()-1, 如此下去。
 - **mask**掩码数组。 如果矩阵不为空，则它必须是与images[i]大小相同的8位数组。 非零掩码元素标记在直方图中计数的数组元素。
 - **hist** 输出直方图，它是一个密集或稀疏的dims -dimensional数组。
 - **histSize** 每个维度中的直方图大小。
 - **ranges** Array of the dims arrays of the histogram bin boundaries in each dimension. When the histogram is uniform (uniform =true), then for each dimension i it is enough to specify the lower (inclusive) boundary L_0 of the 0-th histogram bin and the upper (exclusive) boundary U_(histSize[i]-1) for the last histogram bin histSize[i]-1. That is, in case of a uniform histogram each of ranges[i] is an array of 2 elements. When the histogram is not uniform (uniform=false), then each of ranges[i] contains histSize[i]+1 elements: L_0, U_0=L_1, U_1=L_2,..., U_(histSize[i]-2)=L_(histSize[i]-1), U_(histSize[i]-1). The array elements, that are not between L_0 and U_(histSize[i]-1), are not counted in the histogram.
 - **accumulate**积累标志。 如果已ture，则直方图在分配时不会在开始时清除。 此功能使您能够从多组数组中计算单个直方图，或者及时更新直方图。

输入数组就是当前帧，不需要掩码数组，最后一个标志位设定为false;因此我们只需要定义通道数，输出数组hist,直方图的大小``histSize`` 和 ``ranges``:

.. code-block: java

    MatOfInt channels = new MatOfInt(0);
    Mat hist_b = new Mat();
    Mat hist_g = new Mat();
    Mat hist_r = new Mat();
    MatOfInt histSize = new MatOfInt(256);
    MatOfFloat histRange = new MatOfFloat(0, 256);

In the RGB case we will need all of the hist defined, in the grayscale case instead we will use just the ``hist_b`` one.
We are now ready to do the histogram calculation:

.. code-block: java

    Imgproc.calcHist(images.subList(0, 1), channels, new Mat(), hist_b, histSize, histRange, false);
    if (!gray){
	Imgproc.calcHist(images.subList(1, 2), channels, new Mat(), hist_g, histSize, 	histRange, false);
	Imgproc.calcHist(images.subList(2, 3), channels, new Mat(), hist_r, histSize, 	histRange, false);
    }

where ``gray`` is the flag we passed to the ``showHistogram`` method.

Draw the Histogram
------------------
Next step is to draw the calculated histogram in our GUI.
Open the fxml file with Scene Builder and add an ImageView above the "Controls" text in the right of the BP and set its id:

.. code-block:: xml

    <ImageView fx:id="histogram" />

Now back to the Controller class. Let's add a global variable to control the just added image view:

.. code-block:: java

    @FXML
    private ImageView histogram;

and continue to write the ``showHistogram`` method.
First thing first, let's create an image to display the histogram:

.. code-block:: java

    int hist_w = 150;
    int hist_h = 150;
    int bin_w = (int) Math.round(hist_w / histSize.get(0, 0)[0]);
    Mat histImage = new Mat(hist_h, hist_w, CvType.CV_8UC3, new Scalar(0, 0, 0));

before drawing, we first normalize the histogram so its values fall in the range indicated by the parameters entered:

.. code-block:: java

    Core.normalize(hist_b, hist_b, 0, histImage.rows(), Core.NORM_MINMAX, -1, new Mat());
    if (!gray){
       Core.normalize(hist_g, hist_g, 0, histImage.rows(), Core.NORM_MINMAX, -1, new Mat());
       Core.normalize(hist_r, hist_r, 0, histImage.rows(), Core.NORM_MINMAX, -1, new Mat());
    }

Now we can draw the histogram in our Mat:

.. code-block:: java

    for (int i = 1; i < histSize.get(0, 0)[0]; i++){
       Imgproc.line(histImage, new Point(bin_w * (i - 1), hist_h - Math.round(hist_b.get(i - 1, 0)[0])), new Point(bin_w * (i), hist_h - Math.round(hist_b.get(i, 0)[0])), new Scalar(255, 0, 0), 2, 8, 0);
       if (!gray){
          Imgproc.line(histImage, new Point(bin_w * (i - 1), hist_h - Math.round(hist_g.get(i - 1, 0)[0])),new Point(bin_w * (i), hist_h - Math.round(hist_g.get(i, 0)[0])), new Scalar(0, 255, 0), 2, 8, 0);
          Imgproc.line(histImage, new Point(bin_w * (i - 1), hist_h - Math.round(hist_r.get(i - 1, 0)[0])),Math.round(hist_r.get(i, 0)[0])), new Scalar(0, 0, 255), 2, 8, 0);
       }
    }

Let's convert the obtained Mat to an Image with our method ``mat2Image`` and update the ImageView with the returned Image:

.. code-block:: java

    histo = mat2Image(histImage);
    histogram.setImage(histo);

.. image:: _static/04-01.png

.. image:: _static/04-02.png

The source code of the entire tutorial is available on `GitHub <https://github.com/opencv-java/video-basics>`_.

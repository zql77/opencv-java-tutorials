=================
傅里叶变换
=================

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

目标
----
本教程中我们将创建一个JavaFX程序，可以从文件系统中加载图片并对其做傅里叶变换和傅里叶逆变换。

什么是傅里叶变换?
------------------------------
傅里叶变换将图像分解为其正弦和余弦分量。 换句话说，它会将图像从其空间域转换到其频域。 转换的结果是复数。通过实部图像和虚部图像或者通过幅度和相位图像来显示所有的可能。 然而,在图像处理过程中只有幅度图像是有用的，应为他包含了描述图像的所有信息。
对于本教程，我们将使用基本的灰度图像，其值通常介于0到255之间。因此，傅里叶变换也需要是离散类型，从而产生离散傅立叶变换（DFT）。 DFT是采样的傅立叶变换，因此不包含形成图像的所有频率，而只包含一组足够大的样本以充分描述空间域图像。 频率的数量对应于空间域图像中的像素的数量，即，空间域和傅里叶域中的图像具有相同的尺寸。

在本教程中我们会做什么
--------------------------------
在这个教程中我们将会:
 * 从*file chooser*中加载一张图像.
 * 对加载的图像进行傅里叶变换。
 * 对变换后的图像进行傅里叶逆变换。

现在开始
---------------
建立一个新的 JavaFX 工程。在 Scene Builder 中设置窗口元素，获得一个GUI窗口:

- 在这 **LEFT** ImageView 去加载一张图像:

.. code-block:: xml

    <ImageView fx:id="originalImage" />

- 在两个ImageView上,一个显示傅里叶变换图像和傅里叶逆变换图像;

.. code-block:: xml

    <ImageView fx:id="transformedImage" />
    <ImageView fx:id="antitransformedImage" />

-设置三个按钮（Button），第一个加载图片，第二个应用DFT并显示它，最后一个应用反变换并显示它。

.. code-block:: xml

    <Button alignment="center" text="Load Image" onAction="#loadImage"/>
    <Button fx:id="transformButton" alignment="center" text="Apply transformation" onAction="#transformImage" disable="true" />
    <Button fx:id="antitransformButton" alignment="center" text="Apply anti transformation" onAction="#antitransformImage" disable="true" />

这GUI将看起来像这样:

.. image:: _static/06-00.png

加载文件
-------------
首先，你需要在你的项目中添加一个文件夹 "resources" ，其中包含两个图像。 其中一个是正弦函数，另一个是圆形光圈。
在控制类中，为了将图像加载到我们的程序中，我们将使用* filechooser *：

.. code-block:: java

    private FileChooser fileChooser;

当我们点击加载按钮时，我们必须设置FC的初始目录并打开对话框。 FC将返回所选文件:

.. code-block:: java

    File file = new File("./resources/");
    this.fileChooser.setInitialDirectory(file);
    file = this.fileChooser.showOpenDialog(this.main.getStage());

一旦我们加载了文件，我们必须确保它将以灰度显示并将图像显示在图像视图中：

.. code-block:: java

    this.image = Imgcodecs.imread(file.getAbsolutePath(), Imgcodecs.CV_LOAD_IMAGE_GRAYSCALE);
    this.originalImage.setImage(this.mat2Image(this.image));

进行傅里叶变换
----------------
首先将图像展开为最佳尺寸。 DFT的性能取决于图像大小。 它往往是图像尺寸的最快速度，是数字2,3和5的倍数。 因此，为了达到最佳性能，通常将边界值填充到图像以获得具有这种特征的大小。 ``getOptimalDFTSize（）``返回这个最佳尺寸，我们可以使用``copyMakeBorder（）``函数来扩展图像的边界:

.. code-block:: java

    int addPixelRows = Core.getOptimalDFTSize(image.rows());
    int addPixelCols = Core.getOptimalDFTSize(image.cols());
    Core.copyMakeBorder(image, padded, 0, addPixelRows - image.rows(), 0, addPixelCols - image.cols(),Imgproc.BORDER_CONSTANT, Scalar.all(0));

附加像素用零初始化。

傅里叶变化的结果很复杂，所以我们必须为复杂和真实的价值都做好准备。 我们通常至少以浮点格式存储这些数据。 因此，我们会将输入图像转换为此类型，并使用另一个通道来展开以保存复杂值：

.. code-block:: java

    padded.convertTo(padded, CvType.CV_32F);
    this.planes.add(padded);
    this.planes.add(Mat.zeros(padded.size(), CvType.CV_32F));
    Core.merge(this.planes, this.complexImage);

现在我们可以应用傅里叶变换，然后从复杂图像中获取实部和虚部：

.. code-block:: java

    Core.dft(this.complexImage, this.complexImage);
    Core.split(complexImage, newPlanes);
    Core.magnitude(newPlanes.get(0), newPlanes.get(1), mag);

不幸的是，傅立叶系数的动态范围太大而不能显示在屏幕上。 要使用灰度值进行可视化，我们可以将线性比例转换为对数：

.. code-block:: java

    Core.add(Mat.ones(mag.size(), CVType.CV_32F), mag);
    Core.log(mag, mag);

请记住，在第一步，我们扩大了图像？ 那么，是时候抛弃新引入的值了。 为了可视化目的，我们还可以重新排列结果的象限，以便原点（零点，零点）与图像中心对应:

.. code-block:: java

    image = image.submat(new Rect(0, 0, image.cols() & -2, image.rows() & -2));
    int cx = image.cols() / 2;
    int cy = image.rows() / 2;

    Mat q0 = new Mat(image, new Rect(0, 0, cx, cy));
    Mat q1 = new Mat(image, new Rect(cx, 0, cx, cy));
    Mat q2 = new Mat(image, new Rect(0, cy, cx, cy));
    Mat q3 = new Mat(image, new Rect(cx, cy, cx, cy));

    Mat tmp = new Mat();
    q0.copyTo(tmp);
    q3.copyTo(q0);
    tmp.copyTo(q3);

    q1.copyTo(tmp);
    q2.copyTo(q1);
    tmp.copyTo(q2);

现在我们必须使用 ``normalize()`` 函数来标准化我们的值，以便将浮点值的矩阵转换为可见的图像形式:

.. code-block:: java

    Core.normalize(mag, mag, 0, 255, Core.NORM_MINMAX);

最后一步是在ImageView中显示幅度图像：

.. code-block:: java

    this.transformedImage.setImage(this.mat2Image(magnitude));

进项傅里叶逆变换
------------------------
要使用傅里叶逆变换，我们只需使用``idft（）``函数，用``split（）``函数从复杂图像中提取真实值，并用``normalize（）``归一化结果:

.. code-block:: java

    Core.idft(this.complexImage, this.complexImage);
    Mat restoredImage = new Mat();
    Core.split(this.complexImage, this.planes);
    Core.normalize(this.planes.get(0), restoredImage, 0, 255, Core.NORM_MINMAX);

最后，我们可以在适当的ImageView上显示结果：

.. code-block:: java

    this.antitransformedImage.setImage(this.mat2Image(restoredImage));

分析结果
---------------------
- *sinfunction.png*

.. image:: _static/06-01.png

图像是4个周期的水平正弦图。 请注意，DFT只有一个分量，由对称放置在DFT图像中心的2个亮点表示。 图像的中心是频率坐标系的原点。 x轴从左到右贯穿中心，代表频率的水平分量。 y轴从底部到顶部贯穿中心，代表频率的垂直分量。 中心有一个点表示图像的（0,0）频率项或平均值。 图像通常具有较大的平均值（如128）和大量的低频信息，因此FT图像通常在中心附近有明亮的组件。 水平方向的高频率将导致亮点离开水平方向的中心。

- *circle.png*

.. image:: _static/06-02.png

在这种情况下，我们有一个圆形光圈，什么是圆形光圈的傅里叶变换？ 衍射盘和环。 大光圈会产生紧凑的变换，而小光圈会产生更大的艾里图案; 因此光圈越小光盘越大; 根据傅立叶特性，从第一个暗环的中心到中间，距离是*（1.22 x N）/ d *; 在这种情况下，N是图像的大小，d是圆的直径。
艾里斑是圆孔理想衍射的中心亮斑。
光学系统; 几乎一半的光被包含在直径* 1.02 x 波长 x 焦距 *的亮斑中。

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。

=========================================
创建你的第一个使用OpenCV库的JavaFX程序
=========================================

.. note:: 我们现在假设你已经阅读过前面的教程。 如果没有，请查看 `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_的教程。你也可以在 `<https://github.com/opencv-java/>`_相关代码和资源。

创建你的第一个使用OpenCV库的JavaFX程序
--------------------------------
本教程将指导你在Eclipse中使用OpenCV库创建一个简单的JavaFX GUI应用程序。

本教程中我们将要进行的操作
--------------------------------
在本教程中，我们将会:
 *安装 ``e(fx)clipse`` 插件 和(可选) *Scene Builder*。
 * 使用 *Scene Builder*。
 * 编写并执行我们的应用程序。

你的第一个JavaFX程序
--------------------------------
本教程将教你编写从网络摄像头捕获视频流，然后将其显示在用户界面（GUI）上的应用程序。 我们将使用Scene Builder创建GUI：它将会有一个按钮，它允许我们启动和停止流，以及一个简单播放每一帧视屏的控件。

安装 e(fx)clipse 插件和 Scene Builder
-----------------------------------------------
在Eclipse中安装 ``e(fx)clipse`` 插件, 可以参照这个指导 `<http://www.eclipse.org/efxclipse/install.html#fortheambitious>`_.
如果你选择不安装这样的插件，你只能创建一个普通的 **Java project**项目。
从 `<http://www.oracle.com/technetwork/java/javafxscenebuilder-1x-archive-2199384.html>`_下载并安装 *JavaFX Scene Builder* 2.0 。

现在你可以创建一个新的 JavaFX 项目， ``Go to File > New > Project...`` 然后选择 ``JavaFX project...``.

.. image:: _static/03-00.png

为你的项目取一个名字然后点击 ``Next``.

.. image:: _static/03-01.png

然后为你的项目添加OpenCV库。然后点击 ``Next``.

.. image:: _static/03-02.png

为你的Java包、*FXML 文件* 和 *控制类 Class*取一个名字。
在*FXML 文件*中使用FXML语言定义你的GUI, * 控制 Class *将管理当用户与GUI组件进行交互时必须调用的所有方法和事件。

.. image:: _static/03-03.png

使用 Scene Builder
--------------------------
如果你安装了*Scene Builder* ，你可以在Eclipse中右击*FXML 文件*选择 ``Open with SceneBuilder``。
你可以通过 *Scene Builder* 的图像交互界面创建你的程序界面GUI; 这使您可以查看窗口的实时预览，并通过编辑图形预览来修改组件及其位置。具体操作。
首先一个 *FXML 文件* 只有一个 *AnchorPane*.
An AnchorPane allows the edges of child nodes to be anchored to an offset from the anchorpane's edges. If the anchorpane has a border and/or padding set, the offsets will be measured from the inside edge of those insets.
The anchorpane lays out each managed child regardless of the child's visible property value; unmanaged children are ignored for all layout calculations.
You can go ahead and delete the anchorpane and add a *BorderPane* instead.
BorderPane将子控件放置在顶部，左侧，右侧，底部和中间位置。

.. image:: _static/03-04.png

你可以从 ``Container`` 菜单拖动一个边界框来添加一个BorderPane，然后将其放入 ``Hierarchy`` 菜单中。
现在我们可以添加允许我们启动和停止流的按钮了。从 ``Controls`` 菜单中选取一个按钮组件放在我们BP的**BOTTOM**区域。
我们会看到，在右侧有三个属性菜单(Properties, Layout, Code)，用于定制我们选定的组件。
例如我们可以在 ``Properties`` 菜单的 ``Text`` 字段的"Start Camera" 中编辑我们按钮的文本值。编辑 ``Code`` 菜单下的 ``fx:id`` 字段的ID值 (例如： "start_btn")。

.. image:: _static/03-05.png

.. image:: _static/03-06.png

在*Controller 类*的方法中我们需要通过控件的ID值来修改控件的属性。
我们看到，我们的按钮离窗户边缘太近了，所以我们应该为它增加一些底部边距; 我们可以在 ``Layout`` 菜单中添加属性来达到目的。
为了使按钮生效，我们必须设置一个回调方法（例如：''startCamera'' ），它将执行我们想要在``Code``菜单下的``OnAction``字段中执行的动作。

.. image:: _static/03-07.png

现在，我们从``Controls``菜单中添加一个* ImageView *组件到我们BP的** CENTER **字段中。我们还要编辑* ImageView *的ID（例如“currentFrame”），并为其添加一些边距。

.. image:: _static/03-08.png

最后，我们必须设定一个Controller来管理GUI，我们可以通过在窗口左下角的Controller控制器菜单下的Controller中添加控制类的名字来实现。

我们使用Scene Builder创建了第一个GUI，如果保存文件并返回到Eclipse，我们会发现FXML文件已经自动生成。

JavaFX中的关键概念
----------------------
**场景（Stage）**属性设定应用程序将在哪里显示 (例如： Windows系统的窗口)。
**场景（Stage）** 是构成应用程序的“页面”的一个节点容器。
A **节点（Node）** 是场景中的一个元素，具有视觉交互能力，节点可以分层嵌套。
在 *Main class* 我们需要定义一个 *start* 方法来初始化我们的*主场景（primary stage）*:

.. code-block:: java

    public void start(Stage primaryStage)

加载fxml文件中的*根元素（root element）*和*控制类（controller class）*来填充我们的场景:

.. code-block:: java

    FXMLLoader loader = new FXMLLoader(getClass().getResource("FXHelloCV.fxml"));
    BorderPane root = (BorderPane) loader.load();
    FXController controller = loader.getController();

使用控制类（Controller Class）管理GUI交互
---------------------------------------------------
对于我们的应用程序，我们基本上需要做两件事：控制按钮推送和图像视图的更新。
为此我们需要在gui组件和控制类（controller class）之间创建一个引用:

.. code-block:: java

    @FXML
    private Button button;
    @FXML
    private ImageView currentFrame;

``@FXML`` 引用表示我们将一个变量链接到fxml文件的一个元素,并且用于声明变量的值必须等于为该特定元素设置的id。

``@FXML`` 引用与特定元素中设置的具体动作具有相同的含义。

例如:

.. code-block:: xml

    <Button fx:id="button" mnemonicParsing="false" onAction="#startCamera" text="Start Camera" BorderPane.alignment="CENTER">

我们可以设置:

.. code-block:: java

    @FXML
    protected void startCamera(ActionEvent event) { ...

捕捉视屏流
---------------
总的讲，视频处理所需的所有功能都集成在VideoCapture类中。

.. code-block:: java

    private VideoCapture capture = new VideoCapture();

这些功能都是构建在 FFmpeg 开源库上。视频是由一系列的图片构成。我们称他们为帧。在视频文件中，帧速率指定两帧之间的时间长度。然而对于摄像机，它每秒能够记录的帧数是有限的。
在我们的例子中，我们将帧速率设置为每秒30帧。 为此，我们初始化一个计时器 (例如：一个 ```ScheduledExecutorService```) 这将开启一个每* 33毫秒*执行一次的后台任务。

.. code-block:: java

    Runnable frameGrabber = new Runnable() { ... }
    this.timer = Executors.newSingleThreadScheduledExecutor();
		this.timer.scheduleAtFixedRate(frameGrabber, 0, 33, TimeUnit.MILLISECONDS);

通过调用 ``isOpened`` 方法检查类与视频源的绑定是否成功:

.. code-block:: java

    if (this.capture.isOpened()) { ... }

通常，当调用对象的析构函数时，视频流是自动关闭的。 但是，手动关闭它，需要调用release方法。

.. code-block:: java

    this.capture.release();

视频的帧只是简单的图像。 因此，我们只需要将它们从VideoCapture对象中提取出来并放入Mat中即可。

.. code-block:: java

    Mat frame = new Mat();

视频流是连续的。 我们可以一个帧一个帧的读取图像或者重载的>>运算符来实现。

.. code-block:: java

    this.capture.read(frame);

现在我们将把图像从* BGR *格式转换为*灰度*格式。OpenCV只需要改变一个参数就可以实现这个功能:

.. code-block:: java

    Imgproc.cvtColor(frame, frame, Imgproc.COLOR_BGR2GRAY);

讲解cvtColor方法的参数:
 - 源图像（帧）
 - 目标图像（帧），我们将在其中保存转换后的图像。
 - 一个额外的参数，指示将执行怎样的转换。在这里我们使用``COLOR_BGR2GRAY``参数 (因为 ``imread`` 的默认色彩通道是BGR)。

现在为了将捕获的帧放入ImageView中，我们需要将Mat转换为Image。
我们首先创建一个缓冲区来存储Mat。

.. code-block:: java

    MatOfByte buffer = new MatOfByte();

然后我们可以使用 ``imencode`` 方法将视频的帧放入缓存中:

.. code-block:: java

    Imgcodecs.imencode(".png", frame, buffer);

这将从新编码图像到图像缓冲区中。 这个方法将压缩图像并调整图像的大小以满足图像缓冲区的要求。

.. 注意:: ``imencode`` 返回的是将图像编码为字节数组组成的单行矩阵 ``CV_8UC1`` 。

它有三个参数：
 - (".png") 定义输出格式的文件扩展名。
 - (frame) 要写入的图像。
 - (buffer)调整输出缓冲区大小以适应压缩后的图像。

一旦我们将图像写入缓冲区，我们必须通过使用``ByteArrayInputStream``将它传输到图像流中:

.. code-block:: java

    new Image(new ByteArrayInputStream(buffer.toArray()));

现在我们可以将新图像放入ImageView中。
使用 *Java 1.8* 我们无法在与主线程不同的其他线程中更新GUI元素; 所以我们需要在第二个线程中获取新的帧并在主线程中刷新我们的ImageView：

.. code-block:: java

    Image imageToShow = grabFrame();
    Platform.runLater(new Runnable() {
	    @Override public void run() { currentFrame.setImage(imageToShow); }
    });

.. image:: _static/03-09.png

这个例子的源码在 `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_。

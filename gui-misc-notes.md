# gui-misc
## 1.pc-gui开发
```
    1.IDE选择
        visual studio 2017、qt-creator、c++builder、eclipse（JAVA）、IntelliJ IDEA（JAVA）、pycharm（python）、JetBrains WebStorm（前端）、c++builder等
        qt-creator、qt-designer的关系
        mfc、c#、qt、java、.net
    2.库的选择
        boost库，qt库，stl和mfc的关系，opengl
    3.开发基础
        ide使用
        语言语法的学习
        qt学习
        常用控件的学习（属性，方法和事件）
        库的选择
```
### 1.1 设计注意事项
```
    1.工程头文件路径填写时使用相对路径，或者宏，保证在其他电脑上也可以一次编译通过
    2.工程使用库路径使用相对路径
```
### 1.2 QT中常见后缀文件
```
    1.项目管理文件 .pro，存储项目设置的文件。
    2.主程序入口文件 main.cpp，实现 main()函数的程序文件。
    3.窗体界面文件 widget.ui，一个 XML 格式存储的窗体上的元件及其布局的文件。
    4.widget.h 是所设计的窗体类的头文件。
    5.widget.cpp 是widget.h里定义类的实现文件。
      C++ 中，任何窗体或界面组件都是用类封装的，一个类一般有一个头文件（.h 文件）和一个源程序文件（.cpp 文件）。
    6.qrc文件（qt resource），一个xml格式的资源配置文件，与应用程序关联的应用程序由 .qrc 文件来指定，它用XML记录硬盘上的文件和对应的随意指定的资源名称，应用程序通过资源名称来访问资源。指定的路径是 .qrc 文件所在目录的相对路径。
    注意：
        列出的资源文件必须位于 .qrc 文件所在目录或者其子目录下。
        图标文件需要与qrc文件位于同一目录下或者在该目录的子目录中。
        另外，如果qrc文件中使用了前缀（例如<qresource prefix="/myresources">），要确保图标路径正确无误；
        使用Qt designer添加图标文件时就是自动使用前缀/new/prefix的，但是该路径默认不存在。
    7.rc文件，资源文件，包括比如对话框、菜单、图标、字符串等资源信息。使用.rc资源文件的目的是为了对程序中用到的大量的资源进行统一的管理。
    8.clw文件，VC Class Wizard信息文件。存放了Class Wizard的信息。
    9.ncb文件，分析器信息文件，是由系统自动产生的。
    10.opt文件，IDE的Option文件。
    11.aps文件，资源文件的二进制版本。  
```
### 1.3 [VS中配置属性](https://blog.csdn.net/lp310018931/article/details/49110069)
* [包含目录的区别VC++目录和C/C++目录](https://blog.csdn.net/qq_35608277/article/details/80768660)
```
    1.常规中配置输出目录，中间目录
    2.调试中配置工作目录
    3.C/C++中，常规中配置附加包含目录（头文件目录）
    4.链接器中，常规中配置附件库目录（库目录）
    5.链接器中，输入中配置附加依赖项一般设置的就是xx.lib静态链接库的名称
    6.设置其他库的.h头文件的目录，一般放在库的include文件夹下。后者是设置其他库的lib以及.dll链接库的目录，一般放在库的lib下
    7.配置由Debug改变Release再将Release下的所有这些设置重新设置成Debug相同的就可以。
      路径推荐使用编译器提供给我们的宏变量，而尽量不要使用绝对的名称，这样程序更具有移植性。
      例如，如果某外部库的目录为Win32\Debug与Win32\Release或Win64\Debug与Win64\Release。这样我们使用$(Platform)\$(Configuration)进行设置的时候就不需要再去管什么平台以及是Debug还是Release版本。因为编译器会自动为我们切换，当选择Debug进行编译时，编译器会自动链接到Debug版本，当选择Release进行编译时，会自动链接到Release版本。
    8.目录添加
        附加包含目录---添加工程的头文件目录：
            项目->属性->配置属性->C/C++->常规->附加包含目录：加上头文件的存放目录；
        附加库目录---添加文件引用的lib静态库路径：
            项目->属性->配置属性->链接器->常规->附加库目录：加上lib文件的存放目录；
        附加依赖项---添加工程引用的lib文件名：
            项目->属性->配置属性->链接器->输入->附加依赖项：加上lib文件名。
    9.当需要向项目中添加.dll动态链接库时，直接将需要添加的.dll文件拖拽到项目生成的.exe所在的文件夹下即可（项目->属性->配置属性->常规->输出目录，可以看到.exe生成在哪个目录下）。
```
### 1.4 VS常见后缀名
```
    .sln：解决方案文件，为解决方案资源管理器提供显示管理文件的图形接口所需的信息。   
    .suo: 解决方案用户选项,记录所有将与解决方案建立关联的选项，以便在每次打开时，它都包含您所做的自定义设置。
    .csproj: 项目文件，创建应用程序所需的引用、数据连接、文件夹和文件的信息。
```
### 1.5 制作一个库
```
    下载源码，使用cmake生成sln解决方案（指定vs版本），使用vs打开，编译生成[debug/release]的库文件libd/lib[静态库]和dll[动态库]
    特别注意lib和dll的区别
```
### 1.6 [lib和dll](https://blog.csdn.net/googler_offer/article/details/88526962)
[lib和dll区别](http://www.cppblog.com/amazon/archive/2009/09/04/95318.html)
```
关于lib和dll的区别如下：
    （1）lib是编译时用到的，dll是运行时用到的。如果要完成源代码的编译，只需要lib；如果要使动态链接的程序运行起来，只需要dll。
    （2）如果有dll文件，那么lib一般是一些索引信息，记录了dll中函数的入口和位置，dll中是函数的具体内容；如果只有lib文件，那么这个lib文件是静态编译出来的，索引和实现都在其中。使用静态编译的lib文件，在运行程序时不需要再挂动态库，缺点是导致应用程序比较大，而且失去了动态库的灵活性，发布新版本时要发布新的应用程序才行。
    （3）动态链接的情况下，有两个文件：一个是LIB文件，一个是DLL文件。LIB包含被DLL导出的函数名称和位置，DLL包含实际的函数和数据，应用程序使用LIB文件链接到DLL文件。在应用程序的可执行文件中，存放的不是被调用的函数代码，而是DLL中相应函数代码的地址，从而节省了内存资源。DLL和LIB文件必须随应用程序一起发行，否则应用程序会产生错误。如果不想用lib文件或者没有lib文件，可以用WIN32 API函数LoadLibrary、GetProcAddress装载。
    注：
        如何区分lib是静态库还是dll的地址链接(如何判断lib文件是静态库还是导入库)
        lib.exe /list [文件名] 显示dll的是动态链接库，显示.obj或者.o是静态库
```
### 1.7 发布一个应用
```
    C:\Qt\Qt5.11.3\5.11.3\msvc2017_64\bin\windeployqt.exe 生成可执行程序并打包
```

## 2.单片机-gui开发
```
    1.emWin
    2.stemWin，Segger公司为ST公司定制的emWin
    3.LittlevGL，一个开源免费(MIT许可)的GUI，支持触摸屏操作，移植简单方便，开发者一直在不断完善更新。LittlevGL 自带了丰富的控件：窗口、按键、标签、list、图表等，还可以自定义控件；支持很多特效：透明、阴影、自动显示隐藏滚动条、界面切换动画、图标打开关闭动画、平滑的拖拽控件、分层显示、反锯齿、仅耗少量内存的字体等等
    4.TouchGFX，以界面华丽，流畅以及强劲的TouchGFX Designer著称。现在已经被ST收购，在ST MCU、MPU可免费使用。TouchGFX在MCU系统上运行的界面非常炫，堪比手机的APP界面.使用TouchGFX开发STM32界面，有2种方法：一是利用TouchGFX Designer软件，支持图片和控件拖拽、可快速生成在KEIL或IAR等IDE中可打开的项目工程；另一种方法是，STM32CUBEMX 5.0版本增加了对TouchGFX的支持，可以使用CubeMX开发TouchGFX应用。
    5.AWTK全称 Toolkit AnyWhere，是 ZLG 开发的开源 GUI 引擎，旨在为嵌入式系统、WEB、各种小程序、手机和 PC 打造的通用 GUI 引擎，为用户提供一个功能强大、高效可靠、简单易用、可轻松做出炫酷效果的 GUI 引擎。
    6.EmbeddedWizard，Embedded Wizard是德国TARA System公司开发的一个使用舒适、灵活的嵌入式系统GUI开发工具。主要特性：1）简化GUI开发 2）强大的模拟器，所见即所得编辑 3）图形很漂亮，2D、3D均能够支持，媲美Android界面 4）支持Broadcom、ST、TI等各家MCU 5）支持远程Web UI ，在任何常见的Web浏览器中作为单页面应用程序运行嵌入式GUI 6）收费贵
    7.Qt for MCU，Qt for MCU 将能够在没有操作系统的设备上运行，允许开发人员在具有成本效益的单片机上创建流畅的用户界面，基于 Qt 的应用程序现在可以部署在运行传统操作系统的系统以及基于 ARM Cortex M7 的微控制器上。目前，Qt forMCU 是专门为 ARM Cortex-M 单片机开发的，目前支持测试硬件平台：·   STM32F769i-DISCO、STM32F7508-DK、I.MX RT1050-EVKB、Renesas RH850。

```
## 3.linux设备-gui开发

## 4.[QT开发](http://shouce.jb51.net/qt-beginning/)
```
    1.字符串编码（QT5.11以后没有这个问题）
        * 代码中来设置按钮的中文文本出现了乱码，有两种方法来解决：
            第一种就是在编写程序时使用英文，当程序完成后使用Qt语言家来翻译整个软件中的显示字符串；
            第二种就是在代码中设置字符串编码，然后使用函数对要在界面上显示的中文字符串进行编码转换。
            因为翻译一个软件很麻烦，对于这些小程序，我们希望中文可以立即显示出来，所以下面来讲解第二种方法。
        * 设置字符串编码，可以使用QTextCodec类的setCodecForTr()函数，一般的使用方法就是在要进行编码转换之前调用该函数
            QTextCodec::setCodecForTr(QTextCodec::codecForLocale()); //设置编码，返回适合本地环境的编码
            QTextCodec::setCodecForTr(QTextCodec::codecForName("GB2312")); //指定编码
        * 当设置完编码后，就要在显示中文字符串的地方使用tr()函数
            ui->pushButton->setText(tr("新窗口"));
    2.Qdialog
        accept()
        exec()
        open()
        show()
        reject()
        注意关闭的操作，不一定是销毁
    3.QString类的trimmed()函数可以去除字符串前后的空白字符
    4.emit
        发射信号
    5.菜单操作
        Qt中的一个菜单被看做是一个Action
    6.QDebug类
        调试时输出
    7.foreach
        foreach (variable, container)
        注意，foreach 关徤字遍历一个容器变量是创建了容器的一个副本，所以不能修改原来容器变量的数据项。
    8.system
    9.sql（QSqlQuery类执行sql语句，SQL接口层）
    * 操作结果集
        使用query.exec("select * from student");查询出表中所有的内容。其中的SQL语句“select * from student”中“*”号表明查询表中记录的所有属性。而当query.exec("select * from student");这条语句执行完后，我们便获得了相应的执行结果，因为获得的结果可能不止一条记录，所以称之为结果集。
        结果集其实就是查询到的所有记录的集合，在QSqlQuery类中提供了多个函数来操作这个集合，需要注意这个集合中的记录是从0开始编号的。最常用的操作有：
            seek(int n) ：query指向结果集的第n条记录；
            first() ：query指向结果集的第一条记录；
            last() ：query指向结果集的最后一条记录；
            next() ：query指向下一条记录，每执行一次该函数，便指向相邻的下一条记录；
            previous() ：query指向上一条记录，每执行一次该函数，便指向相邻的上一条记录；
            record() ：获得现在指向的记录；
            value(int n) ：获得属性的值。其中n表示你查询的第n个属性，比方上面我们使用“select * from student”就相当于“select id, name from student”，那么value(0)返回id属性的值，value(1)返回name属性的值。该函数返回QVariant类型的数据，关于该类型与其他类型的对应关系，可以在帮助中查看QVariant。
            at() ：获得现在query指向的记录在结果集中的编号。
            需要特别注意，刚执行完query.exec("select *from student");这行代码时，query是指向结果集以外的，我们可以利用query.next()使得 query指向结果集的第一条记录。当然我们也可以利用seek(0)函数或者first()函数使query指向结果集的第一条记录。但是为了节省内存开销，推荐的方法是，在query.exec("select * from student");这行代码前加上query.setForwardOnly(true);这条代码，此后只能使用next()和seek()函数。
    * sql语句中使用变量
    * 批处理操作
    * 事务操作
        事务可以保证一个复杂的操作的原子性，就是对于一个数据库操作序列，这些操作要么全部做完，要么一条也不做，它是一个不可分割的工作单位。在Qt中，如果底层的数据库引擎支持事务，那么QSqlDriver::hasFeature(QSqlDriver::Transactions)会返回true。可以使用QSqlDatabase::transaction()来启动一个事务，然后编写一些希望在事务中执行的SQL语句，最后调用QSqlDatabase::commit()或者QSqlDatabase::rollback()。当使用事务时必须在创建查询以前就开始事务
    10.sql查询模型（QSqlQueryModel，只读的）
    11.sql表格模型（QSqlTableModel）
        QSqlTableModel提供了一个一次只能操作单个SQL表的读写模型，它是QSqlQuery的更高层次的替代品，可以浏览和修改独立的SQL表，并且只需编写很少的代码，而且不需要了解SQL语法。
    12.sql关系表格模型（QSqlRelationalTableModel）
        QSqlRelationalTableModel继承自QSqlTableModel，并且对其进行了扩展，提供了对外键的支持。一个外键就是一个表中的一个属性和其他表中的主键属性之间的一对一的映射。例如，student表中的course属性对应的是course表中的id属性，那么就称属性course是一个外键。因为这里的course属性的值是一些数字，这样的显示很不友好，使用关系表格模型，就可以将它显示为course表中的name属性的值
        * 使用外键
        * 使用委托
            有时我们也希望，如果用户更改课程属性，那么只能在课程表中有的课程中进行选择，而不能随意填写课程。Qt中还提供了一个QSqlRelationalDelegate委托类，它可以为QSqlRelationalTableModel显示和编辑数据。这个委托为一个外键提供了一个QComboBox部件来显示所有可选的数据，这样就显得更加人性化了。使用这个委托是很简单的，我们先在mainwindow.cpp文件中添加头文件#include <QSqlRelationalDelegate>，然后继续在构造函数中添加如下一行代码：
                ui->tableView->setItemDelegate(
                            new QSqlRelationalDelegate(ui->tableView));
    13.sql比较
        我们可以根据自己的需要来选择使用哪个模型。如果熟悉SQL语法，又不需要将所有的数据都显示出来，那么只需要使用QSqlQuery就可以了。对于QSqlTableModel，它主要是用来显示一个单独的表格的，而QSqlQueryModel可以用来显示任意一个结果集，如果想显示任意一个结果集，而且想使其可读写，那么建议子类化QSqlQueryModel，然后重新实现flags()和setData()函数。
    14.xml
        * XML(ExtensibleMarkup Language，可扩展标记语言)，是一种类似于HTML的标记语言，但它的设计目的是用来传输数据，而不是显示数据。XML的标签没有被预定义，用户需要在使用时自行进行定义。XML是W3C（万维网联盟）的推荐标准。相对于数据库表格的二维表示，XML使用的树形结构更能表现出数据的包含关系，作为一种文本文件格式，XML简单明了的特性使得它在信息存储和描述领域非常流行。
        * 在Qt中提供了QtXml模块来进行XML文档的处理，我们在Qt帮助中输入关键字QtXml Module，可以看到该模块的类表。
        这里主要提供了三种解析方法： 
            DOM方法，可以进行读写；
            SAX方法，可以进行读取；
            基于流的方法，分别使用QXmlStreamReader和QXmlStreamWriter进行读取和写入。
            要在项目中使用QtXml模块，还需要在项目文件（.pro文件）中添加QT += xml一行代码。
    15.xml，Dom（Document Object Model，即文档对象模型）
        * 把XML文档转换成应用程序可以遍历的树形结构，这样便可以随机访问其中的节点。它的缺点是需要将整个XML文档读入内存，消耗内存较多。    
        * 在Qt中使用QDomProcessingInstruction类来表示XML说明，元素对应QDomElement类，属性对应QDomAttr类，文本内容由QDomText类表示。所有的DOM节点，比如这里的说明、元素、属性和文本等，都使用QDomNode来表示，然后使用对应的isProcessingInstruction()、isElement()、isAttr()和isText()等函数来判断是否是该类型的元素，如果是，那么就可以使用toProcessingInstruction()、toElement()、toAttr()和toText()等函数转换为具体的节点类型。
    16.xml，SAX，QXmlSimpleReader
        * DOM实现起来很灵活，但是这样也就使得编程变得复杂了些，而且我们前面也提到过，DOM需要预先把整个XML文档都读入内存，这样就使得它不适合处理较大的文件。下面我们讲述另一种读取XML文档的方法，即SAX 。是的，如果你只想读取并显示整个XML文档，那么SAX是很好的选择，因为它提供了比DOM更简单的接口，并且它不需要将整个XML文档一次性读入内存，这样便可以用来读取较大的文件。
        * 使用SAX方法来解析XML文档比使用DOM方法要清晰很多，更重要的是它的效率要高很多，不过SAX方法只适用于读取XML文档。
    17.xml，使用流读写xml
        从Qt 4.3开始引入了两个新的类来读取和写入XML文档：QXmlStreamReader和QXmlStreamWriter。 QXmlStreamReader类提供了一个快速的解析器通过一个简单的流API来读取格式良好的XML文档，它是作为Qt的SAX解析器的替代品的身份出现的，因为它比SAX解析器更快更方便。QXmlStreamReader可以从QIODevice或者QByteArray中读取数据。流读取器的基本原理就是将XML文档报告为一个记号（tokens）流，这一点与SAX相似，而它们的不同之处在于XML记号被报告的方式。在SAX中，应用程序必须提供处理器（回调函数）来从解析器获得所谓的XML事件；而对于QXmlStreamReader，是应用程序代码自身来驱动循环，在需要的时候可以从读取器中一个接一个的拉出记号。这个是通过调用readNext()函数实现的，它可以读取下一个记号，然后返回一个记号类型，然后可以使用isStartElement()和text()等函数来判断这个记号是否包含我们需要的信息。使用这种主动拉取记号的方式的最大的好处就是可以构建递归解析器，也就是可以在不同的函数或者类中来处理XML文档中的不同记号。
    18.网络（http）
    19.网络（ftp） 
        使用 QNetworkAccessManager 可以实现 Ftp 的上传/下载功能（参考：Qt之FTP上传/下载），但有些原本 QFtp 有的功能 QNetworkAccessManager 却没有提供，例如：list、cd、remove、mkdir、rmdir、rename 等。这种情况下，就不得不使用 QFtp，值得庆幸的是 QFtp 一直在维护，只需要下载源码自行编译即可使用。
    20.网络（ssh）
    21.网络（获取本机网络信息）
        * 使用QHostInfo获取主机名和IP地址
        * 通过QNetworkInterface类来获取本机的IP地址和网络接口信息
    22.网络（udp）
    23.网络（tcp）
    24.进程和线程（QProcess和QThread）
    25.网络（webkit）
        WebKit是一个开源的浏览器引擎。Qt中提供了基于WebKit的QtWebKit模块，它包含了一组相关的类。QtWebKit提供了一个Web浏览器引擎，使用它便可以很容易的将万维网（WorldWide Web）中的内容嵌入到Qt应用程序中。与此同时，本地也可以对Web内容进行控制。QtWebKit可以呈现HTML（HyperTextMarkup Language，超文本标记语言）文档、XHTML（Extensible HyperTextMarkup Language，可扩展超文本标记语言）文档和SVG（Scalable VectorGraphics，可缩放矢量图形）文档，风格使用CSS（Cascading StyleSheets，层叠样式表），脚本使用JavaScript。在JavaScript执行环境和Qt对象模型间搭建的桥梁，实现了使用WebKit的JavaScript环境访问本地对象。关于这一点，大家可以在帮助中参考The QtWebKit Bridge关键字对应的文档。通过整合Qt的网络模块，实现了从Web服务器、本地文件系统甚至Qt资源系统中透明的加载Web页面。
    26.对象树与拥有权
        对象模型，Qt在标准C++对象模型的基础上添加了一些特性
            * 一个强大的无缝对象通信机制——信号和槽（signals and slots）；
            * 可查询和可设计的对象属性系统（object properties）；
            * 强大的事件和事件过滤器（events and event filters）；
            * 通过上下文进行国际化的字符串翻译机制（string translation for internationalization）；
            * 完善的定时器（timers）驱动，使得可以在一个事件驱动的GUI中处理多个任务；
            * 分层结构的、可查询的对象树（object trees），它使用一种很自然的方式来组织对象拥有权（object ownership）；
            * 守卫指针即QPointer，它在引用对象被销毁时自动将其设置为0；
            * 动态的对象转换机制（dynamic cast）；
        元对象系统
            Qt中的元对象系统（Meta-Object System）提供了对象间通信的信号和槽机制、运行时类型信息和动态属性系统。元对象系统是基于以下三个条件的：
                * 该类必须继承自QObject类；
                * 必须在类的私有声明区声明Q_OBJECT宏（在类定义时，如果没有指定public或者private，则默认为private）；
                * 元对象编译器Meta-Object Compiler（moc），为QObject的子类实现元对象特性提供必要的代码。
            其中moc工具读取一个C++源文件，如果它发现一个或者多个类的声明中包含有Q_OBJECT宏，便会另外创建一个C++源文件（就是在项目目录中的debug目录下看到的以moc开头的C++源文件），其中包含了为每一个类生成的元对象代码。这些产生的源文件或者被包含进类的源文件中，或者和类的实现同时进行编译和链接。元对象系统主要是为了实现信号和槽机制才被引入的。
        对象树与拥有权
            * Qt中使用对象树（object tree）来组织和管理所有的QObject类及其子类的对象。当创建一个QObject时，如果使用了其他的对象作为其父对象（parent），那么这个QObject就会被添加到父对象的children()列表中，这样当父对象被销毁时，这个QObject也会被销毁。实践表明，这个机制非常适合于管理GUI对象。例如，一个QShortcut（键盘快捷键）对象是相应窗口的一个子对象，所以当用户关闭了这个窗口时，这个快捷键也可以被销毁。
            * QWidget作为能够在屏幕上显示的所有部件的基类，扩展了对象间的父子关系。一个子对象一般也就是一个子部件，因为它们要显示在父部件的区域之中。例如，当关闭一个消息对话框（message box）后要销毁它时，消息对话框中的按钮和标签也会被销毁，这也正是我们所希望的，因为按钮和标签是消息对话框的子部件。当然，我们也可以自己来销毁一个子对象。
            * 当关闭窗口后，因为该窗口是顶层窗口，所以应用程序要销毁该窗口部件（如果不是顶层窗口，那么关闭时只是隐藏，不会被销毁），而当窗口部件销毁时会自动销毁其子部件。这也就是为什么在Qt中经常只看到new操作而看不到delete操作的原因。
                如：MyButton *button = new MyButton(this);    // 创建按钮部件，指定widget为父部件
            * 对于规范的Qt程序，我们要在main()函数中将主窗口部件创建在栈上，例如Widget w;，而不要在堆上进行创建（使用new操作符）。对于其他窗口部件，可以使用new操作符在堆上进行创建，不过一定要指定其父部件，这样就不需要再使用delete操作符来销毁该对象了。
            * Qt中的对象树很好地解决了父子部件的关系，对于Gui编程是十分方便的，在创建部件时我们只需要关注它的父部件，这样就不用再考虑其销毁问题了。
    27.信号和槽
        * 为了实现对象间的通信，一些工具包中使用了回调（callback）机制，而在Qt中，使用了信号和槽来进行对象间的通信。当一个特殊的事情发生时便可以发射一个信号，比如按钮被单击；而槽就是一个函数，它在信号发射后被调用，来响应这个信号。在Qt的部件类中已经定义了一些信号和槽，但是更多的做法是子类化这个部件，然后添加自己的信号和槽来实现想要的功能。
        * 使用信号和槽应该注意的几点
            * 需要继承自QObject或其子类；
            * 在类声明的最开始处添加Q_OBJECT宏；
            * 槽中的参数的类型要和信号的参数的类型相对应，且不能比信号的参数多；
            * 信号只用声明，没有定义，且返回值为void类型。
        * 信号和槽的自动关联
            * 信号和槽还有一种自动关联方式，例如前面程序中在设计模式直接生成的按钮的单击信号的槽，就是使用的这种方式：on_pushButton_clicked()，它由“on”、部件的objectName和信号三部分组成，中间用下划线隔开。这样组织的名称的槽就可以直接和信号关联，而不用再使用connect()函数。不过使用这种方式还要进行其他设置，而前面之所以可以直接使用，是因为程序中默认已经进行了设置。
            * 因为在setupUi()函数中调用了connectSlotsByName()函数，所以要使用自动关联的部件的定义都要放在setupUi()函数之前，而且还必须使用setObjectName()函数指定它们的objectName，只有这样才能正常使用自动关联。
            * 如果要使用信号和槽的自动关联，就必须在connectSlotsByName()函数之前进行部件的定义，而且还要指定部件的objectName。鉴于这些约束，虽然自动关联形式上很简单，但是实际编写代码时却很少使用。而且，在定义一个部件时，很希望明确的使用connect()函数来对其进行信号和槽的关联。
        * 信号和槽的高级应用
            在Qt中提供了QObject::sender()函数来返回发送该信号的对象的指针。但是如果有多个信号关联到了同一个槽上，而在该槽中需要对每一个信号进行不同的处理，使用上面的方法就很麻烦了。对于这种情况，便可以使用QSignalMapper类。QSignalMapper可以被叫做信号映射器，它可以实现对多个相同部件的相同信号进行映射，为其添加字符串或者数值参数，然后再发射出去。
        * 信号和槽机制的特色和优越性：
            信号和槽机制是类型安全的，相关联的信号和槽的参数必须匹配；
            信号和槽是松耦合的，信号发送者不知道也不需要知道接受者的信息；
            信号和槽可以使用任意类型的任意数量的参数。
        * 虽然信号和槽机制提供了高度的灵活性，但就其性能而言，还是慢于回调机制的。当然，这点性能差异通常在一个应用程序中是很难体现出来的。
    28.qt样式表
        * 一个完善的应用程序不仅应该有实用的功能，还要有一个漂亮的外观，这样才能使应用程序更加友善，更加吸引用户。作为一个跨平台的UI开发框架，Qt提供了强大而灵活的界面外观设计机制。 Qt样式表是一个可以自定义部件外观的十分强大的机制。Qt样式表的概念、术语和语法都受到了HTML的层叠样式表（Cascading StyleSheets，CSS）的启发，不过与CSS不同的是，Qt样式表应用于部件的世界。
        * 在设计模式中使用qt样式表
        * 在代码中设置qt样式表
            ui->pushButton->setStyleSheet("color: blue");
        * 样式表语法
            一个样式规则由一个选择符（selector）和一个声明（declaration）组成。选择符指定了受该规则影响的部件；声明指定了这个部件上要设置的属性。
            QPushButton{color:red}，QPushButton是选择符，{color:red}是声明，而color是属性，red是值。Qt样式表中一般不区分大小写，例如color、Color、COLOR和COloR表示相同的属性。只有类名，对象名和Qt属性名是区分大小写的。
        * 要想为软件设计一个漂亮的界面，需要灵活使用Qt样式表，不过这需要一定的CSS功底，还需要有美工经验。
    30.国际化
        在Qt中可以使用Qt Linguist工具来很容易的完成应用程序的翻译工作，在Qt中编写代码时要对需要显示的字符串调用tr()函数，完成代码编写后，对这个应用程序的翻译主要包含三步：
            * 运行lupdate工具从C++源代码中提取要翻译的文本，这时会生成一个.ts文件，这个文件是XML格式的；
            * 在Qt Linguist中打开.ts文件，并完成翻译工作；
            * 运行lrelease工具从.ts文件中获得.qm文件，它是一个二进制文件。这里的.ts文件是供翻译人员使用的，而在程序运行时只需要使用.qm文件，这两个文件都是与平台无关的。
    31.帮助系统
    32.3D绘图
    33.多媒体应用简介
    34.quick和qml
        为了适应手机移动应用开发， Qt5 将 QML 脚本编程提到与传统 C++ 部件编程相同的高度，力推 QML 界面编程，当然 QML 主要用于手机移动应用程序。 QML 包含大量使用手机移动设备的功能模块，比如基本部件（QtQuick 模块）、GPS 定位、渲染特效、蓝牙、NFC、WebkKit 等等。
```
## 5.[QT开发](http://c.biancheng.net/view/3876.html)
```
注意：控件和类的关系，控件也是一种类，但类不一定全是控件
    1.QSpinBox
    2.QComboBox
    3.QPlainTextEdit
    4.QToolBox
    5.QTabWidget
    6.QSplitter
    7.QListWidget 
    8.QToolButton
    9.QDockWidget
    10.QTreeWidget
    11.QT Model/View结构
    12.QFileSystemModel
    13.QStringListModel 
    14.QStandardItemModel
    15.对话框
        QFileDialog
        QcolorDialog
        QFontDialog
        QinputDialog
        QMessageBox
    16.自定义对话框
    17.多窗口
    18.多文档界面（Multi-document Interface，MDI）
    19.Splash 
        一般的大型应用程序在启动时会显示一个启动画面，即 Splash 窗口。
        Splash 窗口是一个无边对话框，一般显示一个图片，展示软件的信息。Splash 窗口显示时，程序在后台做一些比较耗时的启动准备工作，Splash 窗口显示一段时间后自动关闭，然后软件的主窗口显示出来。
    20.读写文件
        用 QFile 类的 IODevice 读写功能直接进行读写
        用 QFile 和 QTextStream 结合起来，用流（Stream)的方法进行文件读写。
    21.读写二进制文件
        
```




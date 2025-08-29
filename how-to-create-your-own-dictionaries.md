如何创建自己的字典

首先，阅读doc/DICTFILE_FORMAT。

其次，安装stardict-editer包。有关更多工具，请下载stardict-tools源代码压缩包并编译，您将在src/目录中找到许多工具，例如tabfile、babylon、stardict2txt、stardict_verify。

在大多数情况下，我推荐使用“tabfile”转换器来创建自己的字典。您需要先创建一个文本文件，它应以UTF-8编码，换行符应为“\n”，如果是DOS文件格式（“\r\n”），您可以使用“dos2unix”将其转换。

这是一个示例dict.tab文件：
============
a	1\n2\n3
b	4\\5\n6
c	789
============
它的意思是：首先写入搜索词，然后是一个Tab字符，接下来是定义。如果定义包含换行，直接写入\n，如果包含\字符，则写入\\。

然后使用“tabfile”编译它：
./tabfile dict.tab
您将找到tabfile生成的三个文件：“dict.ifo”、“dict.dict”和“dict.idx”，然后您可以使用dictzip压缩“dict.dict”文件：
dictzip dict.dict
您将获得“dict.dict.dz”文件。您可以在DICT项目网站http://www.dict.org找到dictzip，只需下载其源代码压缩包并编译，然后您就可以在其中找到“dictzip”。
StarDict也可以直接加载dict.dict，因此dictzip是可选的。

您可以使用gedit编辑“dict.ifo”文件，改变书名、描述等，您可以参考“src/example.ifo”文件作为示例。使用“ls -l dict.tab.idx”来获取idx文件大小。

现在您可以在/usr/share/stardict/dic/目录下创建一个目录，例如：
mkdir /usr/share/stardict/dic/example-dict/
并将dict.dict.dz、dict.idx、dict.ifo移动到此目录中：
mv dict.dict.dz dict.idx dict.ifo /usr/share/stardict/dic/example-dict/

最后，建议您通过以下方式验证此字典：
./stardict_verify /usr/share/stardict/dic/example-dict/dict.ifo

运行StarDict，您将发现由自己创建的字典。

StarDict推荐的另一种格式是babylon源文件格式，它的格式如下：
======
apple|apples
apple的意思

2dimensional|2dimensionale|2dimensionaler|2dimensionales|2dimensionalem|2dimensionalen
二次元的意思<br>第二行。

======
使用babylon进行编译。stardict-editer也可以进行编译。

如果您想在StarDict网站上分发您的字典，请与我联系。

胡正 <huzheng001@gmail.com>
http://www.huzheng.org
2006.6.29

[GPT-4o mini]


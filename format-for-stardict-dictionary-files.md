StarDict字典文件格式
------------------------------------

StarDict主页: http://stardict-4.sourceforge.net

{0}. 数字和字节顺序约定
在记录标识大小、偏移量等的数字时，您应使用32位数字，例如您可能使用guint32、glong表示。

为了使StarDict在不同平台上工作，这些数字必须采用网络字节顺序。您可以使用g_htonl()函数来确保字节顺序正确，以便创建字典文件。相反，当读取字典文件时，您应使用g_ntohl()。

字符串应以UTF-8编码。

{1}. 文件
每个字典由以下文件组成：
(1). somedict.ifo
(2). somedict.idx或somedict.idx.gz
(3). somedict.dict或somedict.dict.dz
(4). somedict.syn（可选）

您可以使用gzip -9压缩.idx文件。如果.idx文件没有压缩，则加载速度会更快并节省内存，而使用压缩则会使.idx文件加载到内存中，从而加快查询速度。

您可以使用dictzip来压缩.dict文件。
"dictzip"使用与gzip相同的压缩算法和文件格式，但提供一个表，用于随机访问文件中压缩块。使用50-64kB块进行压缩通常仅降低压缩率不到10%，同时保持对文件中所有数据的可接受随机访问能力。此外，使用dictzip压缩的文件可以用gunzip解压缩。
有关dictzip的更多信息，请参考DICT项目，请见：
http://www.dict.org
下载dictd软件包，解压并编译，即可获得dictzip工具。

创建字典时，您应正常使用.idx和.dict.dz。

StarDict会在以下预定义目录中搜索字典：
1) gStarDictDataDir + "/dic",
2) "/usr/share/stardict/dic",
3) g_get_home_dir() + "/.stardict/dic"。
其中gStarDictDataDir是Windows下stardict.exe文件的目录，STARDICT_DATA_DIR配置变量的值则在其他情况下使用。

StarDict会在每个预定义目录及其子目录中搜索.ifo文件。
StarDict不会深入包含.ifo文件的目录的“res”子目录。有关“res”目录，请参见“资源存储”部分。

找到.ifo文件后，StarDict会打开.idx或.idx.gz文件，以及与该.ifo文件所在目录具有相同基本名称的.dict.dz或.dict文件。

虽然可以将多个字典放在同一目录中，但建议为每个字典创建一个独立的目录。
如果您有多个字典，则每个目录只能有一个资源存储，并且将与该目录的所有字典共享。

{2}. 正常字典 ".ifo" 文件格式。
.ifo文件格式在“.ifo文件格式”部分中描述。

魔法数据: "StarDict's dict ifo file"
版本: 必须是 "2.4.2" 或 "3.0.0"。 
	自 "3.0.0" 起，StarDict接受 "idxoffsetbits"选项。

可用选项:

bookname=      // 必需
wordcount=     // 必需
synwordcount=  // 如果存在 ".syn" 文件，必需
idxfilesize=   // 必需
idxoffsetbits= // 3.0.0中的新选项
author=
email=
website=
description=	// 您可以使用<br>表示新行。
date=
sametypesequence= // 非常重要。
dicttype=

wordcount是.idx文件中词条的计数，它必须正确。

idxfilesize是.idx文件的字节大小，即使 .idx 压缩为 .idx.gz 文件，该条目也必须记录原始.idx文件的大小，而且它也必须正确。 .gz文件不包含原始大小信息，但了解原始大小可以加速提取到内存，因为您不需要多次调用realloc()。

idxoffsetbits可以是64或32。如果"idxoffsetbits=64"，.idx文件的偏移字段将是64位。

dicttype由一些特殊字典插件使用，例如wordnet。该值目前可以为"wordnet"。

“sametypesequence”选项在下面将进一步详细描述。

***
sametypesequence

您应该首先熟悉下一节中描述的.dict文件格式，以便理解此选项对.dict文件的影响。

如果设置了sametypesequence选项，它告诉StarDict每个单词在.dict文件中的数据将具有相同的数据类型序列。在这种情况下，我们期望的.dict文件经过了两种方式的优化：类型标识符应被省略，并且每个单词最后一个数据条目的大小标记应被省略。

让我们考虑一些具体的sametypesequence选项示例。

假设字典记录了许多.wav文件，因此设置：
        sametypesequence=W
在这种情况下，每个单词在.dict文件中的条目仅由一个wav文件组成。在.dict文件中，您可以在每个条目之前省略'W'字符，并且还应省略每个.wav条目前面的32位整数，通常用于给出该条目的长度。您可以这样做，因为长度可以从idx文件中的信息中得知。

另一个例子是，假设字典包含每个单词的语音信息和含义。该字典的sametypesequence选项将是：
        sametypesequence=tm
同样，您可以在.dict文件中的每个数据条目前省略't'和'm'字符。此外，您还应在.dict文件中省略每个单词“m”条目的结束'\0'，因为含义字符串的长度可以从语音字符串的长度（仍然由结束的'\0'指示）以及整个单词条目的长度（在.idx文件中列出）中推断出来。

因此，对于每个单词最后一个数据条目通常要求终止'\0'字符的情况，您应在dict文件中省略该字符。对于每个单词最后一个数据条目通常要求一个初始32位数字来给出该字段的长度（如WAV和PNG条目），您必须在字典中省略该数字。

每个字典都应尽量使用sametypesequence特性，以节省磁盘空间。

此功能实际上并没有节省太多空间。让我们考虑一个实际例子：
Wirtschaft Deutsch-Englisch字典。
单词数量: 46385
不使用sametypesequence的.dict文件大小：3244782字节
使用sametypesequence的.dict文件大小：     3152012字节
节省的空间为2.86%。 
真的值得吗？
***

{3}. ".idx" 文件格式。
.idx文件只是一个单词列表。

单词列表是一个排序的单词条目列表。

每个单词条目包含三个字段，依次为：
     word_str;  // 一个以'\0'终止的utf-8字符串。
     word_data_offset;  // 单词数据在.dict文件中的偏移量
     word_data_size;  // 单词数据在.dict文件中的总大小     

word_str给出表示该单词的字符串。这是StarDict将“查找”的字符串。

两个或多个条目可能具有相同的“word_str”，但具有不同的word_data_offset和word_data_size。这可能对某些字典有用。但此功能仅在StarDict-2.4.8及更新版本中得到良好支持。

“word_str”的长度应小于256。换句话说， (strlen(word) < 256)。

如果版本为“3.0.0”且“idxoffsetbits=64”，则word_data_offset将是以网络字节顺序存储的64位无符号数。否则，将为32位。
word_data_size应为以网络字节顺序存储的32位无符号数。

可能不同的word_str具有相同的word_data_offset和word_data_size，因此多个单词索引指向相同的定义。
但这不推荐，对于具有相同定义的多个单词，您可以为它们创建一个“.syn”文件，请参见下面第4节。

单词列表必须通过对“word_str”字段调用stardict_strcmp()进行排序。如果单词列表的顺序错误，StarDict将无法正确工作！

============
gint stardict_strcmp(const gchar *s1, const gchar *s2)
{
	gint a;
	a = g_ascii_strcasecmp(s1, s2);
	if (a == 0)
		return strcmp(s1, s2);
	else
		return a;
}
============
g_ascii_strcasecmp()是glib函数：
与BSD strcasecmp()函数不同，它仅识别标准ASCII字母并忽略区域设置，将所有非ASCII字符视为非字母字符。

stardict_strcmp()对于英文字符效果良好，但其他区域字符的排序并不好，在这种情况下，您可以启用排序特性，请见第6节。

{4}. ".syn" 文件格式。
此文件是可选的，您应该注意树字典不需要此文件。只有StarDict-2.4.8及更高版本支持此文件。

.syn文件包含同义词的信息，也就是说，当您输入一个同义词时，StarDict将搜索与其相关的另一个单词。

格式很简单。每个条目包含一个字符串和一个数字。
synonym_word;  // 一个以'\0'终止的utf-8字符串。
original_word_index; // 原始单词在.idx文件中的索引。
然后其他条目没有分隔。
当您输入synonym_word时，StarDict将搜索original_word;

“synonym_word”的长度应小于256。换句话说，(strlen(word) < 256)。
original_word_index是以网络字节顺序存储的32位无符号数字。
两个或多个条目可能具有相同的“synonym_word”，但具有不同的original_word_index。
条目必须按synonym_word的stardict_strcmp()进行排序。

{5}. 偏移量缓存文件格式。
StarDict-2.4.8开始支持缓存文件，此功能可以加速加载并通过mmap()缓存文件节省内存。缓存文件的名称为.idx.oft和.syn.oft，格式为：
- 一个32位数字 - 索引条目的数量；
- 许多32位数字作为wordoffset索引，该索引是稀疏的，“ENTR_PER_PAGE=32”；
- 一个以'\0'终止的utf-8字符串。
所有数字均以机器字节顺序存储。
将索引条目放在文件开头有助于避免对齐问题。
字符串必须以：
=====
StarDict's oft file
version=2.4.8
=====
开头。
然后是一行这样的内容：
url=/usr/share/stardict/dic/stardict-somedict-2.4.2/somedict.idx
该行最后应有一个'\n'。

StarDict将首先尝试在.ifo文件的同一目录中创建.oft文件，如果失败，则尝试在~/.cache/stardict/中创建，~/.cache由g_get_user_cache_dir()获取。
如果两个或多个字典具有相同的文件名，StarDict将分别为它们创建somedict.idx.oft、somedict(2).idx.oft、somedict(3).idx.oft等，每个文件中的"url="不同。

{6}. 排序文件格式。
StarDict-2.4.8开始支持排序，根据排序函数对单词列表进行排序。它将创建名为.idx.clt和.syn.clt的排序文件，格式类似于偏移量缓存文件：
- 一个32位数字 - 索引条目的数量；
- 许多32位数字作为按排序函数排序的索引；
- 一个以'\0'终止的utf-8字符串。
所有数字均以机器字节顺序存储。
将索引条目放在文件开头有助于避免对齐问题。
字符串必须以：
=====
StarDict's clt file
version=2.4.8
=====
开头。
然后两行如下：
url=/usr/share/stardict/dic/stardict-somedict-2.4.2/somedict.idx
func=0
第二行也应有一个'\n'结束。

StarDict目前支持以下排序函数：
typedef enum {
	UTF8_GENERAL_CI = 0,
	UTF8_UNICODE_CI,
	UTF8_BIN,
	UTF8_CZECH_CI,
	UTF8_DANISH_CI,
	UTF8_ESPERANTO_CI,
	UTF8_ESTONIAN_CI,
	UTF8_HUNGARIAN_CI,
	UTF8_ICELANDIC_CI,
	UTF8_LATVIAN_CI,
	UTF8_LITHUANIAN_CI,
	UTF8_PERSIAN_CI,
	UTF8_POLISH_CI,
	UTF8_ROMAN_CI,
	UTF8_ROMANIAN_CI,
	UTF8_SLOVAK_CI,
	UTF8_SLOVENIAN_CI,
	UTF8_SPANISH_CI,
	UTF8_SPANISH2_CI,
	UTF8_SWEDISH_CI,
	UTF8_TURKISH_CI,
	COLLATE_FUNC_NUMS
} CollateFunctions;
这些UTF8_*_CI函数实际上源自MySQL。

文件的定位路径与.oft文件相同。

注意，对于“somedict.idx.gz”文件，相应的排序文件是somedict.idx.clt，而不是somedict.idx.gz.clt，"url="是somedict.idx，而不是somedict.idx.gz。因此，在您对.idx文件进行压缩后，StarDict无需重新创建.clt文件。

{7}. ".dict" 文件格式。
.dict文件是一个纯数据序列，因为每个单词的偏移量和大小在相应的.idx文件中记录。

如果在.ifo文件中未使用“sametypesequence”选项，则.dict文件的字段顺序如下：
==============
word_1_data_1_type; // 标识数据类型的单个字符
word_1_data_1_data; // 数据
word_1_data_2_type;
word_1_data_2_data;
...... // 每个单词的数据条目数量由.idx文件中的word_data_size决定
word_2_data_1_type;
word_2_data_1_data;
......
==============
重要的是，要注意每个单词中的每个字段指示其自身的长度，如下所述。每个单词的可能字段数量不是固定的，而是通过读取数据，直到读取完字节数word_data_size为止的简单过程。

假设在.idx文件中使用了“sametypesequence”选项，且选项设置如下：
sametypesequence=tm
那么.dict文件将如下所示：
==============
word_1_data_1_data
word_1_data_2_data
word_2_data_1_data
word_2_data_2_data
......
==============
每个单词的第一个数据条目将以终止的'\0'结束，但第二个条目将没有终止的'\0'。省略类型字符和最后字段的大小信息是sametypesequence选项上述描述的优化。

如果"idxoffsetbits=64"，则.dict文件的文件大小将大于4G。因为我们通常需要对这个大文件进行mmap，而32位计算机的进程最大虚拟内存空间限制为4G，这将导致错误，因此实际上无法在32位机器中加载“idxoffsetbits=64”字典，StarDict在加载时将简单地打印一条警告，64位计算机则不应有此限制。

数据类型标识符
----------------
以下是可在.idx文件中与“sametypesequence”选项一起使用或在未使用“sametypesequence”选项的情况下出现在dict文件中的单字符类型标识符。

小写字母表示字段的大小由终止的'\0'决定，而大写字母表示数据以网络字节序的guint32开头，给出以下数据的大小（而不是整个大小，即大4个字节）。

'm'
单词的纯文本含义。
数据应为一个以'\0'结束的utf-8字符串。

'l'
单词的纯文本含义。
数据不是utf-8字符串，而是以本地编码格式的字符串，以'\0'结束。有时使用这种类型可以节省磁盘空间，但不鼓励其使用。这只是一个想法。

'g'
一个用Pango文本标记语言标记的utf-8字符串。
有关该标记语言的更多信息，请参见“Pango参考手册”。
您可能在本地安装：
file:///usr/share/gtk-doc/html/pango/PangoMarkupFormat.html
在线：
http://library.gnome.org/devel/pango/stable/PangoMarkupFormat.html

't'
英语音标字符串。
数据应为一个以'\0'结束的utf-8字符串。

这里有一些utf-8音标字符：
Î¸ÊƒÅ‹Ê§Ã°Ê’Ã¦Ä±ÊŒÊŠÉ’É›É™É‘ÉœÉ”ËŒËˆËË‘á¹ƒá¹‡á¸·
Ã¦É‘É’ÊŒÓ™Ñ”Å‹vÎ¸Ã°ÊƒÊ’ÉšËÉ¡ËËŠË‹

'x'
一个用xdxf语言标记的utf-8字符串。
见http://xdxf.sourceforge.net
StarDict拥有这些扩展：
<rref>可以具有"type"属性，可以是"image"、"sound"、"video"和"attach"。
<kref>可以具有"k"属性。

'y'
中文音标或日文假名。
数据应为一个以'\0'结束的utf-8字符串。

'k'
金山词霸数据。数据为一个以'\0'结束的utf-8字符串。
它是XML格式。

'w'
MediaWiki标记语言。
见http://meta.wikimedia.org/wiki/Help:Editing#The_wiki_markup

'h'
Html代码。

'n'
WordNet数据。

'r'
资源文件列表。
内容可以是：
img:pic/example.jpg	// 图像文件
snd:apple.wav		// 音频文件
vdo:film.avi		// 视频文件
att:file.bin		// 附件文件
支持多行作为可用文件的列表。
StarDict将在资源存储中查找文件。
图像将被显示，音频文件将有播放按钮。
您可以将附件文件“另存为”等。
文件列表必须是以'\0'结尾的utf-8字符串。
使用'\n'分隔新行。
使用'/'字符作为目录分隔符。

'W'
wav文件。
数据以网络字节顺序的guint32开头，以识别wav文件的大小，后面紧跟文件内容。
这仅仅是一个想法，在大多数情况下，最好使用'r'资源文件列表。

'P'
图片文件。
数据以网络字节顺序的guint32开头，以识别图片文件的大小，后面紧跟文件内容。
该功能已实现，因为stardict-advertisement-plugin需要它。
无论如何，在大多数情况下，最好使用'r'资源文件列表。

'X'
此类型标识符保留用于实验性扩展。

{8}. 资源存储
资源存储在'r'资源文件列表中存储外部文件，图像以html代码存储，图像、媒体和其他文件以wiki标签存储。
它有两种形式：
1. 直接目录和“res”子目录中的文件。
2. res.rifo、res.ridx和res.rdic数据库。
直接文件可能存在文件名编码问题，因为Linux使用UTF-8而Windows使用本地编码，因此您最好只使用ASCII文件名，或者使用数据库来存储UTF-8文件名。
数据库可能需要提取文件（如.wav文件）到临时文件，因此相比直接文件效率较低。但数据库具有压缩的优点。
您可以使用dir2resdatabase和resdatabase2dir工具在res目录和res数据库之间进行转换。
StarDict将首先尝试加载存储数据库，然后尝试直接文件形式。

res.rifo文件的格式在“.ifo文件格式”部分中描述。

魔法数据：“StarDict's storage ifo file”
版本：“3.0.0”

可用选项：

filecount=	// 必需。
ridxfilesize=	// 必需。
idxoffsetbits=	// 可选。

res.ridx文件的格式：
filename;	// 一个以'\0'终止的utf-8字符串。
offset;		// 以网络字节顺序存储的32或64位无符号数字。
size;		// 以网络字节顺序存储的32位无符号数字。
filename也可以包括路径，例如“pic/example.png”。使用'/'字符作为目录分隔符。filename是区分大小写的，所有条目中应没有两个相同的文件名。
如果“idxoffsetbits=64”，则偏移量为64位。
这三个条目重复作为每个条目的数组。
条目按filename字段的strcmp()函数进行排序。
不同的文件名可能具有相同的偏移量和大小。
您可以使用gzip -9压缩.ridx文件。有关压缩.idx文件的注意事项。

res.rdic文件格式：
它只是每个资源文件的连接。
您可以使用dictzip将此文件压缩为res.rdic.dz。

{9}. 树字典
树字典支持用于信息查看等。

树字典包含三个文件：sometreedict.ifo、sometreedict.tdx.gz和sometreedict.dict.dz。

最好压缩.tdx文件，因为它始终加载到内存中。

.ifo文件格式在“.ifo文件格式”部分中描述。

魔法数据：“StarDict's treedict ifo file”
版本：“2.4.2”

可用选项：

bookname=      // 必需
wordcount=     // 必需
tdxfilesize=   // 必需
author=
email=
website=
description=	// 您可以使用<br>表示新行。
date=
sametypesequence=

wordcount仅用于字典管理对话框中的信息视图，因此在树字典中并不重要。

.tdx文件只是单词列表。
-----------
单词列表是一个单词条目的树形列表。

每个单词条目依次包含四个字段：
     word_str;  // 一个以'\0'终止的utf-8字符串。
     word_data_offset;  // 单词数据在.dict文件中的偏移量
     word_data_size;  // 单词数据在.dict文件中的总大小。可以为0。
     word_subentry_count; // 该条目具有的子单词数量，0表示没有。

子条目紧接其父条目之后。这使得顺序与展开的树列表的顺序相同，然后从上到下进行排序。

word_data_offset、word_data_size和word_subentry_count应为以网络字节顺序存储的32位无符号数字。

.dict文件的格式与正常字典相同。

{10}. .ifo文件格式
.ifo文件格式用于描述正常和树字典、资源数据库。
所有文件具有共同的结构，但支持的选项集不同。

.ifo文件结构
<魔法数据>
version=<值>
<键1>=<值1>
<键2>=<值2>
...

文件以魔法数据开头，这是文件类型唯一的字符串。例如，“StarDict's dict ifo file”是普通字典的魔法数据。魔法数据占用文件的第一行。第一行前后不得有空格或其他字符。

接下来的行包含用等号分隔的键值对，每行一个对。
我们在该行中搜索第一个等号。等号左边的内容是键，右边的内容是值。
前导和尾随空格以及等号周围的空格都会被忽略。
每个键值对必须以换行结束（请记得在最后一个键值对后添加换行）。
键不能包含空格或制表符。它不应包含除字母数字字符、'-'和'_'之外的任何内容。
文件中的第一个键必须是“version”。这是所有格式的强制性选项。

空行被忽略。
编码。文件内容必须为UTF-8编码。
行结束。任何行结束转换都是可以的：LF、CR+LF、CR。

{11}. 关于StarDict文件格式设计。

当前StarDict文件格式设计是基于二分查找算法，有人可能认为b+树算法应该更快，但实际上，b+树在CPU上的速度更快，但使用更多的磁盘搜索时间（这对缩短磁盘寿命没有好处）。所以实际上二分查找比b+树更好（因为CPU越来越快，而磁盘搜索则较慢）。
但是...有人说Babylon的BGL文件格式是完美的！也许那就是StarDict-4！

关于.idx文件格式，有人可能认为只使用一个总长度数字就可以了，然而目前是偏移量和大小这两个数字。但设计中需要一些冗余，因此当前的设计仍然是最佳选择！

总结一下，当前StarDict文件格式设计或多或少是完美的！

{12}. 更多信息。
您可以阅读“src/lib.cpp”、“src/dictmanagedlg.cpp”和“src/tools/*.cpp”以获取更多信息。

在您构建字典之后，您可以使用“stardict_verify”来验证字典文件。您可以在“src/tools/”中找到它。

如果您有任何问题，请给我发电子邮件。 :)

感谢Will Robinson <wsr23@stanford.edu>清理此文件的英语。

胡铮 <huzheng001@gmail.com>
http://www.huzheng.org
2020.11.28

[GPT-4o mini]

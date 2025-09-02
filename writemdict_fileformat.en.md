## [fileformat.md](https://github.com/zhansliu/writemdict/blob/master/fileformat.md)

## 介绍

这是MDX和MDD文件格式版本2.0的描述，该格式用于[MDict](http://www.octopus-studio.com/product.en.htm)字典软件。该软件不是开源的，文件格式也没有公开的规范，因此以下描述是基于逆向工程的，细节上可能不完整或不准确。

大部分信息来自https://bitbucket.org/xwang/mdict-analysis。虽然xwang主要专注于能够读取这种未知格式，我已添加了必需的细节，以便也能写入MDX文件。

## 概念

MDX和MDD文件都设计用于存储一组关联数组（关键字，记录）。

对于MDX文件，存储的信息通常是字典。关键字和记录都是（Unicode）字符串，关键字是字典条目的头词，记录则描述该词。例如，一个MDX条目可以是：

- 关键字: "reverse engineering"
- 记录: "<i>名词：</i>分析和研究一个对象或设备的过程，以理解其内部工作原理"

而MDD文件则旨在存储二进制数据。通常，关键字是文件路径，记录是该文件的内容。例如，我们可能有：

- 关键字: "\image.png"
- 记录: 0x89 0x50 0x4e 0x47 0x0d 0x0a 0x1a 0x0a...

MDX文件旨在存储字典，即一组（关键字，记录）对，例如，关键字="reverse engineering"，记录="<i>名词：</i>分析和研究一个对象或设备的过程，以理解其内部工作原理"。

通常，MDD文件与同名的MDX文件相关联（但扩展名为.mdx而不是.mdd），并包含MDX文件文本中要包含的资源。例如，MDX文件中的条目可能包含HTML代码`<img src="/images/image.png" />`，在这种情况下，MDict软件将查找MDD文件中的条目"\image.png"。

## 文件结构

基本文件结构如下：

| MDX文件        |             |
|----------------|-------------|
| `header_sect`  | 头部部分，见下文“头部部分”。 |
| `keyword_sect` | 关键字部分，见下文“关键字部分”。 |
| `record_sect`  | 记录部分，见下文“记录部分”。 |

## 头部部分

| `header_sect` | 长度  |   |
|----------------|-----|-----------------|
| `length`       | 4字节 | `header_str`的长度（以字节为单位）。大端编码。 |
| `header_str`   | 可变 | XML字符串，UTF-16LE编码。见下文详细信息。 |
| `checksum`     | 4字节 | `header_str`的ADLER32校验和，存储为小端。 |

`header_str`由一个XML标签`dictionary`组成，具有各种属性。对于MDX文件，它们看起来如下（为清晰起见添加了换行符）：

```xml
<Dictionary 
GeneratedByEngineVersion="2.0" 
RequiredEngineVersion="2.0" 
Encrypted="2" 
Encoding="UTF8"
Format="Html"
CreationDate="2015-01-01"
Compact="No"
Compat="No"
KeyCaseSensitive="No"
Description="这是一个<i>测试字典</i>。"
Title="我的字典"
DataSourceFormat="106"
StyleSheet=""
RegisterBy="Email"
RegCode="0102030405060708090A0B0C0D0E0F"/>
```

对于MDD文件，则为：

```xml
<Library_Data 
GeneratedByEngineVersion="2.0" 
RequiredEngineVersion="2.0" 
Encrypted="2" 
Format=""
CreationDate="2015-01-01"
Compact="No"
Compat="No"
KeyCaseSensitive="No"
Description="这是一个<i>测试字典</i>。"
Title="我的字典"
DataSourceFormat="106"
StyleSheet=""
RegisterBy="Email"
RegCode="0102030405060708090A0B0C0D0E0F"/>
```

这些属性的含义在下方进行解释：

| 属性 | 说明 |
|-----------|-------------|
|`GeneratedByEngineVersion`| 文件格式的版本。本文档描述版本2.0。除了这一版本，版本1.2也是可能的。|
|`RequiredEngineVersion` | 假定与此版本兼容的最低格式版本。 |
|`Encrypted` | 一个介于0和3（包含）之间的整数。如果最低位被设置，表示关键字部分的第一部分被加密，如[关键字头加密](#keyword-header-encryption)一节所述。如果最高位被设置，表示关键字索引已加密，使用[关键字索引加密](#keyword-index-encryption)中描述的方案。 |
|`Encoding`| 仅用于MDX文件。文档中文本的编码。可能的值包括“UTF-8”、“UTF-16”（使用小端编码）、“GBK”和“Big5”。对于MDD文件，关键字（文件路径）的编码始终为UTF-16，记录由二进制数据组成。 |
|`Format`| 字典条目文本的格式。可能的值包括“Html”和“Text”。对于MDD文件，此项必须为空。 |
|`CreationDate` | 字典创建的日期。 |
|`Compact` | 如果此项为“是”，则表示字典条目采用一种特定于Mdict的紧凑格式，其中某些字符串根据在`StyleSheet`中指定的方案进行替换。有关详细信息，请参考官方MdxBuilder客户端的文档。 |
|`Compat` | 似乎是对`Compact`的拼写错误，官方Mdict客户端的某些版本查找此项而不是`Compact`。 |
|`KeyCaseSensitive` | 指示字典读取器是否应区分关键字的大小写。 |
|`Description` | 字典的描述，在官方MDict客户端中作为“：关于”页面出现。 |
|`Title` | 字典的标题。 |
|`DataSourceFormat` | 未知。 |
|`StyleSheet` | 与`Compact`选项结合使用。有关详细信息，请参考官方MdxBuilder客户端的文档。 |
|`RegisterBy` | 可以是“EMail”或“DeviceID”。仅在`Encrypted`的最低位被设置时使用。指示用于加密密钥的用户标识数据。有关详细信息，请参见[关键字头加密](#keyword-header-encryption)部分。 |
|`RegCode` | 当使用关键字头加密时（参见[关键字头加密](#keyword-header-encryption)），这是传递加密密钥的一种方式。在这种情况下，这是由32个十六进制数字组成的字符串。 |

## 关键字部分

关键字部分包含字典中的所有关键字，分为块，以及这些块的大小信息。

| `keyword_sect` | 长度|      |
|----------------|------|------|
| `num_blocks`   | 8字节 | `key_blocks`中的项目数量。大端编码。可能被加密，见下文。 |
| `num_entries`  | 8字节 | 关键字的总数。大端编码。可能被加密，见下文。 |
| `key_index_decomp_len` | 8字节 | `key_index`的解压缩版本的字节数。大端编码。可能被加密，见下文。 |
| `key_index_comp_len`   | 8字节 | `key_index`压缩版本的字节数（包括`comp_type`和`checksum`部分）。大端编码。可能被加密，见下文。 |
| `key_blocks_len`       | 8字节 | `key_blocks`占用的总字节数。大端编码。可能被加密，见下文。 |
| `checksum`             | 4字节 | 前40字节的ADLER32校验和。如果这些被加密，则为解密版本的校验和。大端编码。 |
| `key_index`            | 可变 | 关键字索引，经过压缩和可能加密。见下文。 |
| `key_blocks[0]`       | 可变 | 包含关键字的压缩块。见下文。  |
| ...                    |    ...  | ...|
| `key_blocks[num_blocks-1]` | 可变 |... |

### 关键字头加密：

如果头部中参数`Encrypted`的最低位被设置（即`Encrypted | 1`不为零），那么来自`num_blocks`的40字节块将被加密。使用的加密为Salsa20/8（Salsa20仅进行8轮而不是20轮）。伪Python代码如下：

```python
def encrypt(message, key):
	salsa20_8_init(key_length=128, # 128位
				   iv_length=64, # 64位
				   ivs=b"\0\0\0\0\0\0\0\0"), # 64位的零）
	return salsa20_8_encrypt(message, key)

encrypted_block = encrypt(unencrypted_block, key=ripemd128(encryption_key))
```

这里，`encryption_key`是创建字典时指定的字典密码。

此`encryption_key`并不直接分发。相反，使用与用户或客户端机器特定的数据`user_id`进一步加密，如下所示：

```python
reg_code = encrypt(ripemd128(encryption_key), ripemd128(user_id))
```

字符串`user_id`可以是用户输入到其MDict客户端的电子邮件地址（"example@example.com"），也可以是设备ID（"12345678-90AB-CDEF-0123-4567890A"），MDict客户端根据平台的不同以不同方式获得该ID。选择使用哪一种取决于文件头中属性`RegisterBy`。 （见[头部部分](#header-section)。）在某些平台上，官方MDict客户端似乎将DeviceID的默认值设置为空字符串。

然后将128位的`reg_code`传递给用户。这可以通过两种方式完成：

* 如果MDX文件名为`dictionary.mdx`，则字典读取器应在同一目录中查找名为`dictionary.key`的文件，其中包含`reg_code`作为32位十六进制字符串。
* 否则，`reg_code`可以作为MDX文件头的一部分，作为属性`RegCode`包含。

### 关键字索引

关键字索引列出有关关键字块的一些基本数据。它是经过压缩的（见“压缩”），可能被加密（见“关键字索引加密”）。在解压缩和解密后，结果如下所示：

| `decompress(key_index)` | 长度 |  |
|----------------------------|------|----|
| `num_entries[0]`          | 8字节 | 第一个关键字块中的关键字数。 |
| `first_size[0]`           | 2字节 | `first_word[0]`的长度，不包含尾随的空字符。以“基本单位”的数量表示，UTF-8为字节数，UTF-16为2字节单位。 |
| `first_word[0]`           | 可变 | 第一关键字（按字母顺序排列）在`key_blocks[0]`关键字块中的位置。编码由头部中的`Encoding`属性指定。 |
| `last_size[0]`            | 2字节 | `last_word[0]`的长度，不包含尾随的空字符。以“基本单位”的数量表示，UTF-8为字节数，UTF-16为2字节单位。 |
| `last_word[0]`            | 可变 | `key_blocks[0]`关键字块中最后一个关键字（按字母顺序排列）的位置。编码由头部中的`Encoding`属性指定。 |
| `comp_size[0]`            | 8字节 | `key_blocks[0]`的压缩大小。 |
| `decomp_size[0]`          | 8字节 | `key_blocks[0]`的解压缩大小。 |
| `num_entries[1]`          | 8字节 |... |
| ...                       |      ...|... |
| `decomp_size[num_blocks-1]` | 8字节 |... |

#### 关键字索引加密：

如果头部中参数`Encrypted`的次低位被设置（即`Encrypted | 2`不为零），则关键字索引进一步加密。在这种情况下，`comp_type`和`checksum`字段将保持不变（见“压缩”部分），将使用以下C函数对`compressed_data`部分进行加密，在压缩之后。

```c
#define SWAPNIBBLE(byte) (((byte)>>4) | ((byte)<<4))
void encrypt(unsigned char* buf, size_t buflen, unsigned char* key, size_t keylen) {
	unsigned char previous = 0x36;
	for(size_t i = 0; i < buflen; i++) {
		buf[i] = SWAPNIBBLE(buf[i] ^ ((unsigned char)i) ^ key[i % keylen] ^ previous);
		previous = buf[i];
	}
}
```

用于加密的密钥为`ripemd128(checksum + "\x95\x36\x00\x00")`，其中+表示字符串连接。

### 关键字块

每个关键字是压缩的（见“压缩”）。解压缩后，它们的结构如下：

| `decompress(key_blocks[0])` | 长度  |   |
|-----------------|---------|------|
| `offset[0]`     | 8字节 | 记录对应于`key[0]`的位置，见下文。大端编码。 |
| `key[0]`        | 可变 | 字典中的第一个关键字，使用`Encoding`编码并以null结尾。  |
| `offset[1]`     | 8字节 | ... |
| `key[1]`        | 可变 | ... |
| ...             |   ... | ... |

偏移量应如下解释：解压缩所有记录块，并将其连接在一起，令`records`表示结果字节数组。与`key[i]`对应的记录将从`records[offset[i]]`开始。

## 记录部分

记录部分的结构如下：

| `record_sect`   | 长度  |    |
|-----------------|---------|----|
| `num_blocks` | 8字节 | `record_blocks`中的项目数量。无需与关键字块的数量相等。大端编码。 |
| `num_entries` | 8字节 | 字典中的记录总数。应等于`keyword_sect.num_entries`。大端编码。 |
| `index_len` | 8字节 | `comp_size[i]`和`decomp_size[i]`变量的总大小，以字节为单位。换句话说，应等于16倍的`num_blocks`。大端编码。 |
| `blocks_len` | 8字节 | `rec_block[i]`部分的总大小，以字节为单位。大端编码。 |
| `comp_size[0]` | 8字节 | `rec_block[0]`的长度，以字节为单位。大端编码。 |
| `decomp_size[0]` | 8字节 | `rec_block[i]`的解压缩大小，以字节为单位。大端编码。 |
| `comp_size[1]` | 8字节 | `rec_block[1]`的长度，以字节为单位。大端编码。 |
| ...           |   ...    |  ... |
| `decomp_size[num_blocks-1]` | 8字节 | ... |
| `rec_block[0]` | 可变 | 包含记录的压缩块。见下文。 |
| ...           |     ... | ... |
| `rec_block[num_blocks-1]` | 可变 |... |

### 记录块

每个记录块是压缩的（见“压缩”）。解压缩后，结构如下：

| `decompress(rec_block[0])` | 长度 | |
|----------------------------|--------|-----|
| `record[0]`                | 可变 | 第一条记录。如果为MDX文件，则以null结尾并使用`Encoding`编码。 |
| `record[1]`                | 可变 |...|
| ...                        |   ...   |...|

## 压缩：

各种数据块使用相同的方案进行压缩。它们的结构如下：

| `compress(data)`  | 长度 |  |
|-------------------|--------|------|
| `comp_type`       | 4字节 | 压缩类型。见下文。 |
| `checksum`        | 4字节 | 未压缩数据的ADLER32校验和。大端编码。 |
| `compressed_data` | 可变 | `data`的压缩版本。|

压缩类型可以通过`comp_type`指示。 有三种选择：

* 如果`comp_type`为`'\x00\x00\x00\x00'`，则不应用任何压缩，并且`compressed_data`等于`data`。
* 如果`comp_type`为`'\x01\x00\x00\x00'`，则使用LZO压缩。
* 如果`comp_type`为`'\x02\x00\x00\x00'`，则使用zlib压缩。zlib压缩格式附加ADLER32校验和，因此在这种情况下，`checksum`将等于`compressed_data`的最后四个字节。

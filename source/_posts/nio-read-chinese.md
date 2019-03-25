---
title: Java NIO下使用ByteBuffer读取文本时解决UTF-8概率性中文乱码的问题
date: 2019-03-25
---
场景：
读取一个大文本文件，并输出到控制台。

在这里我们选择使用nio进行读取文本文件，在输出的过程中，有些文件中英文都显示正常，有些则偶尔出现中文乱码，经思考发现，在 ByteBuffer.allocate 时分配空间，如果中英混合的文件中就会出现中文字符只读取了一部分的问题，如果文本为等长编码字符集的时候，可以根据编码集 byte 长度进行 allocate ，例如 GBK 为2 byte ，所以我们 allocate 时未2的倍数即可，但像 UTF-8 这类变长的编码字符集时则没那么简单了。

下面就是 UTF-8 的编码方式

```
0xxxxxxx
110xxxxx 10xxxxxx
1110xxxx 10xxxxxx 10xxxxxx
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```

- 对于 UTF-8 编码中的任意字节 B ，如果 B 的第一位为0，则 B 为 ASCII 码，并且 B 独立的表示一个字符；
- 如果 B 的第一位为1，第二位为0，则B为一个非 ASCII 字符（该字符由多个字节表示）中的一个字节，并且不是字符的第一个字节编码；
- 如果 B 的前两位为1，第三位为0，则B为一个非 ASCII 字符（该字符由多个字节表示）中的第一个字节，并且该字符由两个字节表示；
- 如果 B 的前三位为1，第四位为0，则B为一个非 ASCII 字符（该字符由多个字节表示）中的第一个字节，并且该字符由三个字节表示；
- 如果 B 的前四位为1，第五位为0，则B为一个非 ASCII 字符（该字符由多个字节表示）中的第一个字节，并且该字符由四个字节表示；

通过分析我们发现，在读取中我们通过处理临界值来解决 UTF-8 编码字符读取问题。

> 示例代码如下：

```
	public static void read() throws Exception {
		RandomAccessFile rf = new RandomAccessFile("zh.txt", "rw");
		FileChannel channel = rf.getChannel();

		// 至少为4，因为UTF-8最大为4字节
		ByteBuffer buffer = ByteBuffer.allocate(4);

		while (channel.read(buffer) != -1) {
			byte b;
			int idx;
			out:
			for (idx = buffer.position() - 1; idx >= 0; idx--) {
				b = buffer.get(idx);

				// 0xxxxxxx
				if ((b & 0xff) >> 7 == 0) {
					break;
				}
				// 11xxxxxx，110xxxxx、1110xxxx、11110xxx
				if ((b & 0xff & 0xc0) == 0xc0) {
					idx -= 1;
					break;
				}
				if ((b & 0xff & 0x80) == 0x80) {
					for (int i = 1; i < 4; i++) {
						b = buffer.get(idx - i);
						if ((b & 0xff & 0xc0) == 0xc0) {
							if ((b & 0xff) >> (5 + 1 - i) == 0xf >> (3 - i)) {
								break out;
							} else {
								idx = idx - 1 - i;
								break out;
							}
						}
					}
				}
			}

			buffer.flip();
			int limit = buffer.limit();
			// 阻止读取跨界数据
			buffer.limit(idx + 1);
			System.out.print(Charset.forName("UTF-8").decode(buffer).toString());

			// 恢复limit
			buffer.limit(limit);
			buffer.compact();
		}

		channel.close();
		rf.close();
	}
```


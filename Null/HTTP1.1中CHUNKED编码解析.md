# **HTTP1.1中CHUNKED编码解析**

一般*HTTP*通信时，会使用 *Content-Length* 头信息来告知发送的内容长度，但是当不能预先确定报文体的长度时，不可能在头中包含*Content-Length*域来指明报文体长度，此时就需要通过 *Transfer-Encoding* 域来确定报文体长度。

Chunked*编码一般使用若干个*chunk*串连而成，最后由一个标明长度为* 0 的 chunk 标示结束。每个chunk分为头部和正文两部分，**头部内容**指定下一段正文的字符总数（非零开头的十六进制的数字）和数量单位（一般不写,表示字节）.**正文部分**就是指定长度的实际内容，两部分之间用回车换行(CRLF)隔开。在最后一个长度为 0 的chunk中的内容是称为 footer 的内容，是一些附加的 Header 信息（通常可以直接忽略）。

具体格式如下*(BNF*文法*)*：

```
Chunked-Body   = *chunk            //0至多个chunk

				last-chunk         //最后一个chunk

				trailer            //尾部

				CRLF               //结束标记符

chunk          = chunk-size [ chunk-extension ] CRLF

				chunk-data CRLF

chunk-size     = 1*HEX

last-chunk     = 1*("0") [ chunk-extension ] CRLF

chunk-extension= *( ";" chunk-ext-name [ "=" chunk-ext-val ] )

chunk-ext-name = token

chunk-ext-val  = token | quoted-string

chunk-data     = chunk-size(OCTET)

trailer        = *(entity-header CRLF)

```

解释：

l         Chunked-Body表示经过chunked编码后的报文体。报文体可以分为chunk, last-chunk，trailer和结束符四部分。chunk的数量在报文体中最少可以为0，无上限；

l        每个chunk的长度是自指定的，即，起始的数据必然是16进制数字的字符串，代表后面chunk-data的长度（字节数）。这个16进制的字符串第一个字符如果是“0”，则表示chunk-size为0，该chunk为last-chunk,无chunk-data部分。

l         可选的chunk-extension由通信双方自行确定，如果接收者不理解它的意义，可以忽略。

l         trailer是附加的在尾部的额外头域，通常包含一些元数据（metadata, meta means "about information"），这些头域可以在解码后附加在现有头域之后

示例：AWSs3 copyObject操作请求与返回数据

```http
PUT /Windows.iso1 HTTP/1.1
Host: mul.s3.amazonaws.com
x-amz-content-sha256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
Authorization: AWS4-HMAC-SHA256 Credential=AKIAIXNWJVSO5YDVX5OQ/20190828/us-east-1/s3/aws4_request, SignedHeaders=amz-sdk-invocation-id;amz-sdk-retry;content-length;content-type;host;user-agent;x-amz-content-sha256;x-amz-copy-source;x-amz-date, Signature=32b32a3779b7215aaed3c7ff36c6586bd908cb20e4b47c17538e89cd69b30ad5
X-Amz-Date: 20190828T021126Z
User-Agent: aws-sdk-java/1.11.605 Windows_10/10.0 Java_HotSpot(TM)_64-Bit_Server_VM/25.201-b09 java/1.8.0_201 vendor/Oracle_Corporation
amz-sdk-invocation-id: 81ecb401-1403-c6b4-6232-31f6b5a05fcc
amz-sdk-retry: 0/0/500
x-amz-copy-source: /juyan/Windows.iso
Content-Type: application/octet-stream
Content-Length: 0
Connection: Keep-Alive
```

```http
HTTP/1.1 200 OK
x-amz-id-2: aE8u/Lf+wnhGzkfZTfu9VKL8ddz5zaKCHGPEZokS40AiysnkXLH54I6Lptn89v34Lau+JL2o8VI=
x-amz-request-id: B7C89D5F6806669F
Date: Wed, 28 Aug 2019 02:11:27 GMT
x-amz-version-id: null
Content-Type: application/xml
Transfer-Encoding: chunked
Server: AmazonS3

26
<?xml version="1.0" encoding="UTF-8"?>
1


1


1


1


1


1


c4

<CopyObjectResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><LastModified>2019-08-28T02:11:27.000Z</LastModified><ETag>&quot;058814ea8b2012a217c0abbda8bf6a74&quot;</ETag></CopyObjectResult>
0

```


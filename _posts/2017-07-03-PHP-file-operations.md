---
layout: post
title: PHP 文件操作的各种姿势
categories: development
tags: [PHP, File, IO]
---

## 使用 SPL 库

SPL 是 PHP 标准库，用于解决典型问题的一组接口与类的集合。

### 迭代器 FilesystemIterator

官方文档：http://php.net/manual/zh/class.filesystemiterator.php

类摘要：

```
FilesystemIterator extends DirectoryIterator implements SeekableIterator {

/* 常量 */
const integer CURRENT_AS_PATHNAME = 32 ;
const integer CURRENT_AS_FILEINFO = 0 ;
const integer CURRENT_AS_SELF = 16 ;
const integer CURRENT_MODE_MASK = 240 ;
const integer KEY_AS_PATHNAME = 0 ;
const integer KEY_AS_FILENAME = 256 ;
const integer FOLLOW_SYMLINKS = 512 ;
const integer KEY_MODE_MASK = 3840 ;
const integer NEW_CURRENT_AND_KEY = 256 ;
const integer SKIP_DOTS = 4096 ;
const integer UNIX_PATHS = 8192 ;

/* 方法 */
public __construct ( string $path [, int $flags = FilesystemIterator::KEY_AS_PATHNAME | FilesystemIterator::CURRENT_AS_FILEINFO | FilesystemIterator::SKIP_DOTS ] )
public mixed current ( void )
public int getFlags ( void )
public string key ( void )
public void next ( void )
public void rewind ( void )
public void setFlags ([ int $flags ] )

/* 继承的方法 */
public DirectoryIterator DirectoryIterator::current ( void )
public int DirectoryIterator::getATime ( void )
public string DirectoryIterator::getBasename ([ string $suffix ] )
public int DirectoryIterator::getCTime ( void )
public string DirectoryIterator::getExtension ( void )
public string DirectoryIterator::getFilename ( void )
public int DirectoryIterator::getGroup ( void )
public int DirectoryIterator::getInode ( void )
public int DirectoryIterator::getMTime ( void )
public int DirectoryIterator::getOwner ( void )
public string DirectoryIterator::getPath ( void )
public string DirectoryIterator::getPathname ( void )
public int DirectoryIterator::getPerms ( void )
public int DirectoryIterator::getSize ( void )
public string DirectoryIterator::getType ( void )
public bool DirectoryIterator::isDir ( void )
public bool DirectoryIterator::isDot ( void )
public bool DirectoryIterator::isExecutable ( void )
public bool DirectoryIterator::isFile ( void )
public bool DirectoryIterator::isLink ( void )
public bool DirectoryIterator::isReadable ( void )
public bool DirectoryIterator::isWritable ( void )
public string DirectoryIterator::key ( void )
public void DirectoryIterator::next ( void )
public void DirectoryIterator::rewind ( void )
public void DirectoryIterator::seek ( int $position )
public string DirectoryIterator::__toString ( void )
public bool DirectoryIterator::valid ( void )
}
```

使用这个迭代器的方法很简单，实例化的时候传入的是目录路径，这个迭代器继承自 DirectoryIterator 迭代器，而 DirectoryIterator 迭代器又继承自 SplFileInfo 类，这个类我们后面介绍。

```
$fSI = new FileSystemIterator('.');
/** @var FileSystemIterator $fInfo */
// 模拟 windows dir
foreach ($fSI as $fInfo) {
    printf("%8s%8s%8s %8s\n",
        date("Y-m-d H:i:s", $fInfo->getMtime()),
        $fInfo->isDir() ? '<DIR>' : '',
        number_format($fInfo->getSize()),
        $fInfo->getFileName()
    );
}
// 模拟 linux ls
foreach ($fSI as $fInfo) {
    printf("%8s  ",
        $fInfo->getFileName()
    );
}
```

以上代码模拟了 windows 的 dir 和 linux 的 ls 功能。还有许多功能可以使用，看一下官方文档里的方法列表就知道了，都是比较语义化的方法命名，比如 isFile 就是判断一个文件是不是文件， getMTime 是获取文件的修改时间。

### 文件处理类 SplFileInfo 

官方文档：http://php.net/manual/zh/class.splfileinfo.php

SplFileInfo 这个类可以用来获取文件的基本信息，如修改时间，大小。

SplFileObject 这个类操作文件的内容，如读取、写入。

类摘要

```
SplFileInfo {

/* 方法 */
public __construct ( string $file_name )
public int getATime ( void )
public string getBasename ([ string $suffix ] )
public int getCTime ( void )
public string getExtension ( void )
public SplFileInfo getFileInfo ([ string $class_name ] )
public string getFilename ( void )
public int getGroup ( void )
public int getInode ( void )
public string getLinkTarget ( void )
public int getMTime ( void )
public int getOwner ( void )
public string getPath ( void )
public SplFileInfo getPathInfo ([ string $class_name ] )
public string getPathname ( void )
public int getPerms ( void )
public string getRealPath ( void )
public int getSize ( void )
public string getType ( void )
public bool isDir ( void )
public bool isExecutable ( void )
public bool isFile ( void )
public bool isLink ( void )
public bool isReadable ( void )
public bool isWritable ( void )
public SplFileObject openFile ([ string $open_mode = "r" [, bool $use_include_path = false [, resource $context = NULL ]]] )
public void setFileClass ([ string $class_name = "SplFileObject" ] )
public void setInfoClass ([ string $class_name = "SplFileInfo" ] )
public void __toString ( void )
}
```

```
$file = new SplFileInfo('README.md');
echo 'File is created at ' . date('Y-m-d H:i:s', $file->getCTime()) . PHP_EOL;
echo 'File is modified at ' . date('Y-m-d H:i:s', $file->getMTime()) . PHP_EOL;
echo 'File size is ' . $file->getSize() . ' bytes' . PHP_EOL;

// openFile 方法返回的 SplFileObject 对象
// 读取文件里面的内容
$fileObj = $file->openFile('r');
while($fileObj->valid()){
    echo $fileObj->fgets(); // 读取文件里的一行数据
}

// 关闭打开的文件
$fileObj = null;
$file = null;
```


### 函数

目录相关函数：http://php.net/manual/zh/ref.dir.php
文件系统函数：http://php.net/manual/zh/ref.filesystem.php

以上官方文档都比较完善，并且翻译完整使用自行学习。同时也有助于理解 SPL 的文件操作，因为 SPL 里的方法与这里的函数命名是一致的。









[TOC]

<!-- File -->

# 第十七章 文件
>在丑陋的 Java I/O 编程方式诞生多年以后，Java终于简化了文件读写的基本操作。

这种"困难方式"的全部细节都在 [Appendix: I/O Streams](./Appendix-IO-Streams.md)。如果你读过这个部分，就会认同 Java 设计者毫不在意他们的使用者的体验这一观念。打开并读取文件对于大多数编程语言来说是非常常用的，由于 I/O 糟糕的设计以至于
很少有人能够在不依赖其他参考代码的情况下完成打开文件的操作。

好像 Java 设计者终于意识到了 Java 使用者多年来的痛苦，在 Java7 中对此引入了巨大的改进。这些新元素被放在 **java.nio.file** 包下面，过去人们通常把 **nio** 中的 **n** 理解为 **new** 即新的 **io**，如今却意味着 **non-blocking** 非阻塞 **io**(**io**就是*input/output输入/输出*)。**java.nio.file** 库终于将 Java 文件操作带到与其他编程语言相同的水平。最重要的是 Java8 新增的 streams 与文件结合使得文件操作编程变得更加优雅。我们将看一下文件操作的两个基本组件：

1. 文件或者目录的路径；
2. 文件自身。

<!-- File and Directory Paths -->
## 文件和目录路径

一个 **Path** 对象表示一个文件或者目录的路径，是一个跨操作系统（OS）和文件系统的抽象，目的是在构造路径时不必关注底层操作系统，代码可以在不进行修改的情况下运行在不同的操作系统上。**java.nio.file.Paths** 类包含一个重载方法 **static get()**，该方法接受一系列 **String** 字符串或一个*统一资源标识符*(URI)作为参数，并且进行转换返回一个 **Path** 对象：

```java
// files/PathInfo.java
import java.nio.file.*;
import java.net.URI;
import java.io.File;
import java.io.IOException;

public class PathInfo {
    static void show(String id, Object p) {
        System.out.println(id + ": " + p);
    }

    static void info(Path p) {
        show("toString", p);
        show("Exists", Files.exists(p));
        show("RegularFile", Files.isRegularFile(p));
        show("Directory", Files.isDirectory(p));
        show("Absolute", p.isAbsolute());
        show("FileName", p.getFileName());
        show("Parent", p.getParent());
        show("Root", p.getRoot());
        System.out.println("******************");
    }
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        info(Paths.get("C:", "path", "to", "nowhere", "NoFile.txt"));
        Path p = Paths.get("PathInfo.java");
        info(p);
        Path ap = p.toAbsolutePath();
        info(ap);
        info(ap.getParent());
        try {
            info(p.toRealPath());
        } catch(IOException e) {
           System.out.println(e);
        }
        URI u = p.toUri();
        System.out.println("URI: " + u);
        Path puri = Paths.get(u);
        System.out.println(Files.exists(puri));
        File f = ap.toFile(); // Don't be fooled
    }
}

/* 输出:
Windows 10
toString: C:\path\to\nowhere\NoFile.txt
Exists: false
RegularFile: false
Directory: false
Absolute: true
FileName: NoFile.txt
Parent: C:\path\to\nowhere
Root: C:\
******************
toString: PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: false
FileName: PathInfo.java
Parent: null
Root: null
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: true
FileName: PathInfo.java
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Root: C:\
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Exists: true
RegularFile: false
Directory: true
Absolute: true
FileName: files
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples
Root: C:\
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: true
FileName: PathInfo.java
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Root: C:\
******************
URI: file:///C:/Users/Bruce/Documents/GitHub/onjava/
ExtractedExamples/files/PathInfo.java
true
*/
```

我已经在 **main()** 方法的第一行用于展示操作系统的名称，因此你可以看到不同操作系统之间存在哪些差异。理想情况下，差别会相对较小，诸如 **/** 还是 **\\** 路径分隔符进行分隔之类的。你可以看到我运行在Windows 10 上的程序输出。

 **toString()** 生成完整路径， **getFileName()** 方法总是返回当前文件名。
通过使用 **Files** 工具类(我们接下来将会更多地使用它)，可以测试一个文件是否存在、是否"普通"文件、是否目录等等。"Nofile.txt"这个示例展示我们描述的文件可能并不在指定的位置；这样就允许你创建一个新的路径。"PathInfo.java"存在于当前目录中，最初它只是没有路径的文件名，但它仍然被检测为"存在"。一旦我们将其转换为绝对路径，我们将会得到一个从"C:"盘(因为我们是在Windows机器下进行测试)开始的完整路径，现在它也拥有一个父路径。“真实”路径的定义在文档中有点模糊，因为它取决于具体的文件系统。例如，如果文件名不区分大小写，即使路径由于大小写的缘故而不是完全相同，也可能得到肯定的匹配结果。在这样的平台上，**toRealPath()** 将返回实际情况下的 **Path**，并且还会删除任何冗余元素。

这里你会看到 **URI** 看起来只能用于描述文件，实际上 **URI** 可以用于描述更多的东西；通过 [维基百科](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) 可以了解更多细节。现在我们成功地将 **URI** 转为一个 **Path** 对象。

最后，你会在 **Path** 中看到一些有点欺骗性的定义，这就是调用 **toFile()** 生成的 **File** 对象。这听起来好像得到了一个类似文件的东西(毕竟被称为 **File** )，但是这种描述仅仅是为了兼容老版本。**File** 对象 实际上表示文件或目录本身——应该被称为"路径**Path**"，这是个非常草率并且令人困惑的命名，但是由于 **java.nio.file** 的存在我们可以安全地忽略它的存在。 （这个 **File** 对象是 **java.io.File**， 不兼容旧版本一般用不上）

### 选取路径部分片段
**Path** 对象可以非常容易地生成路径的某一部分：

```java
// files/PartsOfPaths.java
import java.nio.file.*;

public class PartsOfPaths {
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PartsOfPaths.java").toAbsolutePath();
        for(int i = 0; i < p.getNameCount(); i++)
            System.out.println(p.getName(i));
        System.err.println("ends with '.java'1: " + p.endsWith(".java")); //false
        System.err.println("ends with '.java'2: " + p.endsWith("PartsOfPaths.java"));  //true      
        for(Path pp : p) {
            System.out.print(pp + ": ");
            System.out.print(p.startsWith(pp) + " : ");
            System.out.println(p.endsWith(pp));
        }
        System.out.println("Starts with " + p.getRoot() + " " + p.startsWith(p.getRoot()));
    }
}

/* 输出:
Windows 10
Users
Bruce
Documents
GitHub
on-java
ExtractedExamples
files
PartsOfPaths.java
ends with '.java': false
Users: false : false
Bruce: false : false
Documents: false : false
GitHub: false : false
on-java: false : false
ExtractedExamples: false : false
files: false : false
PartsOfPaths.java: false : true
Starts with C:\ true
*/

```
对于 **Path** 中的路径分隔符分隔的几个部分的名字（组件名），可通过 **getName(i)** 来索引，直至上限 **getNameCount()**。还可以通过 for-in 循环进行遍历 (**Path** 也实现了 **Iterable** 接口)。请注意，即使路径的确以 **.java** 结尾，使用 **endsWith()** 也返回 **false**。这是因为使用 **endsWith()** 比较的是 **Path** 的各个完整的组件名，而不仅是某个组件名的子串**.java**。这里展示了使用 **startsWith()** 和 **endsWith()** 在 for-in 循环中检测 **Path** 的当前部分。但是我们可以看到，遍历 **Path** 对象没有包含根路径(C盘D盘之类的盘符)，只有使用 **startsWith()** 检测 **getRoot()** 时才会返回 **true**。

### 路径分析
**Files** 工具类包含一系列完整的方法用于获得 **Path** 相关的信息。

```java
// files/PathAnalysis.java
import java.nio.file.*;
import java.io.IOException;

public class PathAnalysis {
    static void say(String id, Object result) {
        System.out.print(id + ": ");
        System.out.println(result);
    }
    
    public static void main(String[] args) throws IOException {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PathAnalysis.java").toAbsolutePath();
        say("Exists", Files.exists(p));
        say("Directory", Files.isDirectory(p));
        say("Executable", Files.isExecutable(p));
        say("Readable", Files.isReadable(p));
        say("RegularFile", Files.isRegularFile(p));
        say("Writable", Files.isWritable(p));
        say("notExists", Files.notExists(p));
        say("Hidden", Files.isHidden(p));
        say("size", Files.size(p));
        say("FileStore", Files.getFileStore(p));
        say("LastModified: ", Files.getLastModifiedTime(p));
        say("Owner", Files.getOwner(p));
        say("ContentType", Files.probeContentType(p));
        say("SymbolicLink", Files.isSymbolicLink(p));
        if(Files.isSymbolicLink(p))
            say("SymbolicLink", Files.readSymbolicLink(p));
        if(FileSystems.getDefault().supportedFileAttributeViews().contains("posix"))
            say("PosixFilePermissions",
        Files.getPosixFilePermissions(p));
    }
}

/* 输出:
Windows 10
Exists: true
Directory: false
Executable: true
Readable: true
RegularFile: true
Writable: true
notExists: false
Hidden: false
size: 1631
FileStore: SSD (C:)
LastModified: : 2017-05-09T12:07:00.428366Z
Owner: MINDVIEWTOSHIBA\Bruce (User)
ContentType: null
SymbolicLink: false
*/
```
在调用最后一个测试方法 **getPosixFilePermissions()** 之前我们需要确认一下当前文件系统是否支持 **Posix** 接口，否则会抛出运行时异常。

### **Paths**的增减修改
我们必须能通过对 **Path** 对象增加或者删除一部分来构造一个新的 **Path** 对象。
删除：使用 **relativize()** 移除 **Path** 的根路径；
添加：使用 **resolve()** 添加到 **Path** 的末尾(不一定是“可发现”的名称)。（可以是不存在的目录和文件）

对于下面代码中的示例，我使用 **relativize()** 方法从所有的输出中移除根路径，为了示范，也为了简化输出结果。事实证明只有绝对**isAbsolute()**才能使用 **relativize()** 方法。
这个版本的代码中包含 **id** 以便跟踪输出结果：

```java
// files/AddAndSubtractPaths.java
import java.nio.file.*;
import java.io.IOException;

public class AddAndSubtractPaths {
    static Path base = Paths.get("..", "..", "..").toAbsolutePath().normalize();
    
    static void show(int id, Path result) {
        if(result.isAbsolute())
            System.out.println("(" + id + ")r " + base.relativize(result));
        else
            System.out.println("(" + id + ") " + result);
        try {
            System.out.println("RealPath: " + result.toRealPath());
        } catch(IOException e) {
            System.out.println(e);
        }
    }
    
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        System.out.println(base);
        Path p = Paths.get("AddAndSubtractPaths.java").toAbsolutePath();
        show(1, p);
        Path convoluted = p.getParent().getParent()
        .resolve("strings").resolve("..")
        .resolve(p.getParent().getFileName());
        show(2, convoluted);
        show(3, convoluted.normalize());
        Path p2 = Paths.get("..", "..");
        show(4, p2);
        show(5, p2.normalize());
        show(6, p2.toAbsolutePath().normalize());
        Path p3 = Paths.get(".").toAbsolutePath();
        Path p4 = p3.resolve(p2);
        show(7, p4);
        show(8, p4.normalize());
        Path p5 = Paths.get("").toAbsolutePath();
        show(9, p5);
        show(10, p5.resolveSibling("strings"));
        show(11, Paths.get("nonexistent"));
    }
}

/* 输出:
Windows 10
C:\Users\Bruce\Documents\GitHub
(1)r onjava\
ExtractedExamples\files\AddAndSubtractPaths.java
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\AddAndSubtractPaths.java
(2)r on-java\ExtractedExamples\strings\..\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(3)r on-java\ExtractedExamples\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(4) ..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(5) ..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(6)r on-java
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(7)r on-java\ExtractedExamples\files\.\..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(8)r on-java
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(9)r on-java\ExtractedExamples\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(10)r on-java\ExtractedExamples\strings
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\strings
(11) nonexistent
java.nio.file.NoSuchFileException:
C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\nonexistent
*/
```
我还对 **toRealPath()** 做了进一步测试，它会一直扩展和规则化 **Path** ，路径不存在时抛出运行时异常。

<!-- Directories -->

## 目录
**Files** 工具类包含大部分我们需要的目录操作和文件操作方法。出于某种原因，它们没有包含删除目录树相关的方法，因此我们将实现并将其添加到 **onjava** 库中。

```java
// onjava/RmDir.java
package onjava;

import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.io.IOException;

public class RmDir {
    public static void rmdir(Path dir) throws IOException {
        Files.walkFileTree(dir, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                Files.delete(file);
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                Files.delete(dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
```
删除目录树的方法实现依赖于 **Files.walkFileTree()**，"walking" 目录树意味着遍历每个子目录和文件。 设计模式—— *访问者模式* *Visitor* 提供了一种标准机制来访问集合中的每个对象，然后需要你提供（实现）在每个对象上执行的操作。
此操作的定义取决于实现的 **FileVisitor** 的四个抽象方法，包括：

    1.  **preVisitDirectory()**：在目录上运行，在其内容被访问之前。 
    2.  **visitFile()**：运行目录中的每一个文件。  
    3.  **visitFileFailed()**：调用无法访问的文件。   
    4.  **postVisitDirectory()**：在目录上运行，在其内容（包括所有的子目录）被访问之后。

为了简化，**java.nio.file.SimpleFileVisitor** 提供了所有方法的默认实现。这样，在我们的匿名内部类中，我们只需要重写非标准行为的方法：**visitFile()** 和 **postVisitDirectory()** 实现删除文件和删除目录。两者都返回“继续访问”标志(这样就可以一直访问，直至找到所需为止)。
作为我们探索创建和填充目录的一部分，现在我们可以有条件地删除已存在的目录。在以下例子中，**makeVariant()** 接受基本目录测试，并通过旋转组件名列表生成不同的子目录路径。这些旋转(组件名)和路径分隔符 **sep** 用 **String.join()** 粘贴在一起，然后返回一个 **Path** 对象。

```java
// files/Directories.java
import java.util.*;
import java.nio.file.*;
import onjava.RmDir;

public class Directories {
    static Path test = Paths.get("test");
    static String sep = FileSystems.getDefault().getSeparator();
    static List<String> parts = Arrays.asList("foo", "bar", "baz", "bag");
    
    static Path makeVariant() {
        Collections.rotate(parts, 1);
        return Paths.get("test", String.join(sep, parts));
    }
    
    static void refreshTestDir() throws Exception {
        if(Files.exists(test))
        RmDir.rmdir(test);
        if(!Files.exists(test))
        Files.createDirectory(test);
    }
    
    public static void main(String[] args) throws Exception {
        refreshTestDir();
        Files.createFile(test.resolve("Hello.txt"));
        Path variant = makeVariant();
        // Throws exception (too many levels):
        try {
            Files.createDirectory(variant);
        } catch(Exception e) {
            System.out.println("Nope, that doesn't work.");
        }
        populateTestDir();
        Path tempdir = Files.createTempDirectory(test, "DIR_");
        Files.createTempFile(tempdir, "pre", ".non");
        Files.newDirectoryStream(test).forEach(System.out::println);
        System.out.println("*********");
        Files.walk(test).forEach(System.out::println);
    }
    
    static void populateTestDir() throws Exception  {
        for(int i = 0; i < parts.size(); i++) {
            Path variant = makeVariant();
            if(!Files.exists(variant)) {
                Files.createDirectories(variant);
                Files.copy(Paths.get("Directories.java"),
                    variant.resolve("File.txt"));
                Files.createTempFile(variant, null, null);
            }
        }
    }
}

/* 输出:
Nope, that doesn't work.
test\bag
test\bar
test\baz
test\DIR_5142667942049986036
test\foo
test\Hello.txt
*********
test
test\bag
test\bag\foo
test\bag\foo\bar
test\bag\foo\bar\baz
test\bag\foo\bar\baz\8279660869874696036.tmp
test\bag\foo\bar\baz\File.txt
test\bar
test\bar\baz
test\bar\baz\bag
test\bar\baz\bag\foo
test\bar\baz\bag\foo\1274043134240426261.tmp
test\bar\baz\bag\foo\File.txt
test\baz
test\baz\bag
test\baz\bag\foo
test\baz\bag\foo\bar
test\baz\bag\foo\bar\6130572530014544105.tmp
test\baz\bag\foo\bar\File.txt
test\DIR_5142667942049986036
test\DIR_5142667942049986036\pre7704286843227113253.non
test\foo
test\foo\bar
test\foo\bar\baz
test\foo\bar\baz\bag
test\foo\bar\baz\bag\5412864507741775436.tmp
test\foo\bar\baz\bag\File.txt
test\Hello.txt
*/
```
首先，**refreshTestDir()** 用于检测 **test** 目录是否已经存在。若存在，则使用我们新工具类 **rmdir()** 删除其整个目录。在这里检查是否 **exists()** 做了2次，是重复的，这是因为我是想阐明，如果你对于已存在的目录调用 **createDirectory()** 将会抛出异常。**createFile()** 以 **Path** 为参数创建了一个空文件; **resolve()** 将文件名添加到该目录路径的末尾。

我们尝试使用 **createDirectory()** 来创建多级路径，但是这样会抛出异常，因为这个方法只能创建单级路径。

我已经将 **populateTestDir()** 作为一个单独的方法，因为它将在后面的例子中被重用。对于每一个 **variant** ( **Path** 对象)，我们都能使用 **createDirectories()** 创建完整的目录路径(目录树)，然后使用此文件(Directories.java)的拷贝以另一个的名称(File.txt)填充该目录树的终端。然后我们使用 **createTempFile()** 生成一个临时文件。(这里我们给 **createTempFile()** 传的参数是两个 **null** )

在调用 **populateTestDir()** 之后，下一步的操作是在 **test** 目录下面创建一个临时目录。请注意，**createTempDirectory()** 只有名称的前缀(设置)选项，这一点和 **createTempFile()** 不同。再下一步，我们使用 **createTempFile()** 将临时文件放入新的临时目录中。你可以从输出中看到，如果未指定后缀，默认使用" **.tmp** "作为后缀。

为了列出上述操作的结果，我们开始试用很有前途的 **newDirectoryStream()**，但事实证明这个方法只是流式处理了(stream化) **test** 这一级的目录内容，并没有再深入。要获取目录树的全部内容的流，请使用 **Files.walk()**。

<!-- File Systems -->

## 文件系统
为了完整性，我们需要一种方法来找出有关文件系统的其余信息。在这里，我们可以:
- 使用静态的 **FileSystems** 工具类获取"默认"的文件系统；`FileSystems.getDefault()`
- 对 **Path** 对象调用 **getFileSystem()** 以获取创建该 **Path** 的文件系统；`xx.getFileSystem()`
- 获得给定 *URI* 对应的文件系统；`FileSystems.getFileSystem(URI)`
- 构建新的文件系统(对于支持该操作的操作系统)。`FileSystems.newFileSystem(URI,map)`
```java
// files/FileSystemDemo.java
import java.nio.file.*;

public class FileSystemDemo {
    static void show(String id, Object o) {
        System.out.println(id + ": " + o);
    }
    
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        FileSystem fsys = FileSystems.getDefault();
        for(FileStore fs : fsys.getFileStores())
            show("File Store", fs);
        for(Path rd : fsys.getRootDirectories())
            show("Root Directory", rd);
        show("Separator", fsys.getSeparator());
        show("UserPrincipalLookupService",
            fsys.getUserPrincipalLookupService());
        show("isOpen", fsys.isOpen());
        show("isReadOnly", fsys.isReadOnly());
        show("FileSystemProvider", fsys.provider());
        show("File Attribute Views",
        fsys.supportedFileAttributeViews());
    }
}
/* 输出:
Windows 10
File Store: SSD (C:)
Root Directory: C:\
Root Directory: D:\
Separator: \
UserPrincipalLookupService:
sun.nio.fs.WindowsFileSystem$LookupService$1@15db9742
isOpen: true
isReadOnly: false
FileSystemProvider:
sun.nio.fs.WindowsFileSystemProvider@6d06d69c
File Attribute Views: [owner, dos, acl, basic, user]
*/
```
一个 **FileSystem** 对象也能生成 **WatchService** 和 **PathMatcher** 对象，会在接下来两节中讲解。

<!-- Watching a Path -->

## 路径监听
通过 **WatchService** 可以设置一个进程，对目录中的更改做出响应。在这个例子中，**delTxtFiles()** 作为一个单独的任务执行，该任务将遍历整个目录并删除以 **.txt** 结尾的所有文件，**WatchService** 会对文件删除操作做出反应：

```java
// files/PathWatcher.java
// {ExcludeFromGradle}
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;
import java.util.concurrent.*;

public class PathWatcher {
    static Path test = Paths.get("test");
    
    static void delTxtFiles() {
        try {
            Files.walk(test)
            .filter(f ->
                f.toString()
                .endsWith(".txt"))
                .forEach(f -> {
                try {
                    System.out.println("deleting " + f);
                    Files.delete(f);
                } catch(IOException e) {
                    throw new RuntimeException(e);
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    public static void main(String[] args) throws Exception {
        Directories.refreshTestDir();
        Directories.populateTestDir();
        Files.createFile(test.resolve("Hello.txt"));
        WatchService watcher = FileSystems.getDefault().newWatchService();
        test.register(watcher, ENTRY_DELETE);
        Executors.newSingleThreadScheduledExecutor()
        .schedule(PathWatcher::delTxtFiles,
        250, TimeUnit.MILLISECONDS);
        WatchKey key = watcher.take();
        for(WatchEvent evt : key.pollEvents()) {
            System.out.println("evt.context(): " + evt.context() +
            "\nevt.count(): " + evt.count() +
            "\nevt.kind(): " + evt.kind());
            System.exit(0);
        }
    }
}
/* Output:
deleting test\bag\foo\bar\baz\File.txt
deleting test\bar\baz\bag\foo\File.txt
deleting test\baz\bag\foo\bar\File.txt
deleting test\foo\bar\baz\bag\File.txt
deleting test\Hello.txt
evt.context(): Hello.txt
evt.count(): 1
evt.kind(): ENTRY_DELETE
*/
```

**delTxtFiles()** 中的 **try** 代码块看起来有重复，因为它们捕获的是同一种类型的异常，似乎只要外部的 **try** 语句已经足够了。然而出于某种原因，Java 要求两者都在(这也可能是一个 bug)。还要注意的是在 **filter()** 中，我们必须显式地使用 **f.toString()** 转为字符串，否则 **endsWith()** 会与整个 **Path** 对象进行比较，而不是只比较路径的字符串名称这部分。

一旦我们从 **FileSystem** 中得到了 **WatchService** 对象，就将 **WatchService** 对象及感兴趣项的参数列表一起注册到 **test** 对象，参数列表可选 **ENTRY_CREATE**，**ENTRY_DELETE** 或 **ENTRY_MODIFY**(其中创建 *CREATE* 和删除 *DELETE* 不属于修改)。

因为接下来对 **watcher.take()** (开始监听事件)的调用会在监听到增删改之前一直阻塞线程，所以我们希望 **deltxtfiles()** 能够并行运行以便生成我们感兴趣的事件。(删完再监听,晚了,一直监听不到；先监听再删，删操作被阻塞。因此只能多线程并发)

为了这一目的，我通过调用 **Executors.newSingleThreadScheduledExecutor()** 产生一个 **ScheduledExecutorService** 对象，然后调用 **schedule()** 方法传递所需函数的方法引用，并设置在开始监听（ **watcher.take()** ）之后延时（250ms）发生。

此时，**watcher.take()** 将等待并阻塞在这里。当目标事件发生时，会返回一个包含 **WatchEvent** 的 **Watchkey** 对象。展示的这三种方法是能对 **WatchEvent** 执行的全部操作。

查看输出的具体内容。即使我们正在删除以 **.txt** 结尾的文件，在 **Hello.txt** 被删除之前，**WatchService** 也不会被触发。你可能认为，如果说"监视这个目录"，自然会包含整个目录和下面子目录，但实际上：只会监视给定的目录，不包括下面的。如果需要监视整个树目录，必须在整个树的每个子目录上放置一个 **Watchservice**。

```java
// files/TreeWatcher.java
// {ExcludeFromGradle}
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;
import java.util.concurrent.*;

public class TreeWatcher {

    static void watchDir(Path dir) {
        try {
            WatchService watcher =
            FileSystems.getDefault().newWatchService();
            dir.register(watcher, ENTRY_DELETE);
            Executors.newSingleThreadExecutor().submit(() -> {
                try {
                    WatchKey key = watcher.take();
                    for(WatchEvent evt : key.pollEvents()) {
                        System.out.println(
                        "evt.context(): " + evt.context() +
                        "\nevt.count(): " + evt.count() +
                        "\nevt.kind(): " + evt.kind());
                        System.exit(0);
                    }
                } catch(InterruptedException e) {
                    return;
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    public static void main(String[] args) throws Exception {
        Directories.refreshTestDir();
        Directories.populateTestDir();
        Files.walk(Paths.get("test"))
            .filter(Files::isDirectory)
            .forEach(TreeWatcher::watchDir);
        PathWatcher.delTxtFiles();
    }
}

/* Output:
deleting test\bag\foo\bar\baz\File.txt
deleting test\bar\baz\bag\foo\File.txt
evt.context(): File.txt
evt.count(): 1
evt.kind(): ENTRY_DELETE
*/
```

在 **watchDir()** 方法中给 **WatchSevice** 提供参数 **ENTRY_DELETE**，并启动一个独立的线程来监视该**Watchservice**。这里我们没有使用 **schedule()** 进行启动，而是使用 **submit()** 启动线程。我们遍历整个目录树，并将 **watchDir()** 应用于每个子目录。现在，当我们运行 **deltxtfiles()** 时，其中的一个 **Watchservice** 会检测到最开始的一次删除。

<!-- Finding Files -->

## 文件查找
到目前为止，为了找到文件，我们一直使用相当粗糙的方法，在 `path` 上调用 `toString()`，然后使用 `string` 相关操作查看结果。事实证明，`java.nio.file` 有更好的解决方案：`PathMatcher`。其通过在 `FileSystem` 对象上调用 `getPathMatcher()` 获得，然后传入您感兴趣的模式。模式有两个选项：`glob` 和 `regex`。`glob` 看似简单实际上功能十分强大，因此您可以使用 `glob` 解决许多问题。如果您的问题更复杂，可以使用 `regex`(这将在接下来的 `Strings` 一章中解释)。

在这里，我们使用 `glob` 查找以 `.tmp` 或 `.txt` 结尾的所有 `Path`：

```java
// files/Find.java
// {ExcludeFromGradle}
import java.nio.file.*;

public class Find {
    public static void main(String[] args) throws Exception {
        Path test = Paths.get("test");
        Directories.refreshTestDir();
        Directories.populateTestDir();
        // Creating a *directory*, not a file:
        Files.createDirectory(test.resolve("dir.tmp"));

        PathMatcher matcher = FileSystems.getDefault()
          .getPathMatcher("glob:**/*.{tmp,txt}");
        Files.walk(test)
          .filter(matcher::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        PathMatcher matcher2 = FileSystems.getDefault()
          .getPathMatcher("glob:*.tmp");
        Files.walk(test)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        Files.walk(test) // Only look for files
          .filter(Files::isRegularFile)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
    }
}
/* Output:
test\bag\foo\bar\baz\5208762845883213974.tmp
test\bag\foo\bar\baz\File.txt
test\bar\baz\bag\foo\7918367201207778677.tmp
test\bar\baz\bag\foo\File.txt
test\baz\bag\foo\bar\8016595521026696632.tmp
test\baz\bag\foo\bar\File.txt
test\dir.tmp
test\foo\bar\baz\bag\5832319279813617280.tmp
test\foo\bar\baz\bag\File.txt
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
dir.tmp
5832319279813617280.tmp
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
5832319279813617280.tmp
*/
```

在 `matcher` 定义中，`glob` 表达式开头的 `**/` 表示“当前目录及所有子目录”，这在当你不仅仅要匹配当前目录下特定结尾的 `Path` 时非常有用。单 `*` 表示“任何东西”，然后是一个点，然后大括号表示一系列的可能性---我们正在寻找以 `.tmp` 或 `.txt` 结尾的东西。您可以在 `getPathMatcher()` 文档中找到更多详细信息。

`matcher2` 只使用 `*.tmp`，通常不匹配任何内容，但是添加 `map()` 操作会将完整路径减少到末尾的名称 ( 因此现在有匹配内容了 )。

注意，在这两种情况下，输出中都会出现 `dir.tmp`，即使它是目录而不是文件。要只查找文件，必须像在最后 `files.walk()` 中那样对其进行筛选—— `.filter(Files::isRegularFile)`。

<!-- Reading & Writing Files -->
## 文件读写
到此，我们可以对路径和目录做任何事情。 现在让我们看一下如何操纵文件本身的内容。

如果一个文件很“小”，也就是说“它运行得足够快且占用内存小”，那么 `java.nio.file.Files` 类中的实用程序将帮助你轻松读写文本和二进制文件。

`Files.readAllLines()` 一次读取整个文件（因此，“小”文件很有必要），产生一个`List<String>`。 作为例子，我们又重用`streams/Cheese.dat`：

```java
// files/ListOfLines.java
import java.util.*;
import java.nio.file.*;

public class ListOfLines {
    public static void main(String[] args) throws Exception {
        Files.readAllLines(
        Paths.get("../streams/Cheese.dat"))
        .stream()
        .filter(line -> !line.startsWith("//"))
        .map(line ->
            line.substring(0, line.length()/2))
        .forEach(System.out::println);
    }
}
/* Output:
Not much of a cheese
Finest in the
And what leads you
Well, it's
It's certainly uncon
*/
```

跳过注释行，其余的内容每行只打印一半。 这实现起来很简单：你只需将 `Path` 传递给 `readAllLines()` （以前的 java 实现这个功能很复杂）。`readAllLines()` 有一个重载版本，包含一个 `Charset` 参数来存储文件的 Unicode 编码。

`Files.write()` 被重载以写入 `byte` 数组或任何 `Iterable` 对象（它也有 `Charset` 选项）：

```java
// files/Writing.java
import java.util.*;
import java.nio.file.*;

public class Writing {
    static Random rand = new Random(47);
    static final int SIZE = 1000;
    
    public static void main(String[] args) throws Exception {
        // Write bytes to a file:
        byte[] bytes = new byte[SIZE];
        rand.nextBytes(bytes);
        Files.write(Paths.get("bytes.dat"), bytes);
        System.out.println("bytes.dat: " + Files.size(Paths.get("bytes.dat")));

        // Write an iterable to a file:
        List<String> lines = Files.readAllLines(
          Paths.get("../streams/Cheese.dat"));
        Files.write(Paths.get("Cheese.txt"), lines);
        System.out.println("Cheese.txt: " + Files.size(Paths.get("Cheese.txt")));
    }
}
/* Output:
bytes.dat: 1000
Cheese.txt: 199
*/
```

我们使用 `Random` 来创建一个随机的 `byte` 数组; 你可以看到生成的文件大小是 1000。

一个 `List` 被写入文件，任何 `Iterable` 对象也可以这么做。

如果文件大小有问题怎么办？ 比如：

1. 文件太大，如果你一次性读完整个文件，你可能会耗尽内存。

2. 您只需要通过文件进行局部操作就可以得到您想要的结果，因此读取整个文件会浪费时间。

`Files.lines()` 方便地将文件转换以 **"行"** 为单位元素的 `Stream`：

```java
// files/ReadLineStream.java
import java.nio.file.*;

public class ReadLineStream {
    public static void main(String[] args) throws Exception {
        Files.lines(Paths.get("PathInfo.java"))
          .skip(13)
          .findFirst()
          .ifPresent(System.out::println);
    }
}
/* Output:
    show("RegularFile", Files.isRegularFile(p));
*/
```

这对本章开头的示例代码`PathInfo.java`做了流式处理，跳过 13 **行** ，然后选择下一行并将其打印出来。

`Files.lines()` 对于处理文件（转为 **"行"** 为单位元素的输入流）非常有用，但如果你想读取、处理或写入全在一个 `Stream` 中又怎么做呢？这就需要稍微复杂的代码：

```java
// files/StreamInAndOut.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class StreamInAndOut {
    public static void main(String[] args) {
        try(
          Stream<String> input =
            Files.lines(Paths.get("StreamInAndOut.java"));
          PrintWriter output =
            new PrintWriter("StreamInAndOut.txt")
        ) {
            input.map(String::toUpperCase)
              .forEachOrdered(output::println);
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

因为我们在同一个块中执行所有操作，所以这两个文件都可以在相同的 try-with-resources 语句中打开。`PrintWriter` 是一个旧式的 `java.io` 类，允许你“打印”到一个文件，所以它是这个应用的理想选择。如果你看一下 `StreamInAndOut.txt`，你会发现它里面的内容确实是大写的。

<!-- Summary -->

## 本章小结
尽管本章对文件和目录操作做了相当全面的介绍，但还有类库中的功能未被介绍——一定要研究 `java.nio.file` 的 Javadocs，尤其是 `java.nio.file.Files` 这个类。

Java 7 和 8 对于处理文件和目录的类库做了大量改进。如果您刚刚开始使用 Java，那么您很幸运。在过去，它令人非常不愉快，我确信 Java 设计者以前对于文件操作不够重视才没做简化。对于初学者来说这是一件很棒的事，对于教学者来说也一样。我不明白为什么花了这么长时间来解决这个明显的问题，但不管怎么说它被解决了，我很高兴。使用文件现在很简单，甚至很有趣，这是你以前永远想不到的。

<!-- 分页 -->

<div style="page-break-after: always;"></div>

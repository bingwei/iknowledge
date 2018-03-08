# JPF 安装和配置

## 下载

JPF的源码使用Mercurial进行版本控制，从NASA自己的[仓库](http://babelfish.arc.nasa.gov/hg/jpf/jpf-core)上下载的代码是最新版本。不要下载sourceforge上的代码。

+ Install Mercurial

```Shell
sudo apt-get install mercurial
```

+ Clone the jpf-core source code

```Shell
hg clone http://babelfish.arc.nasa.gov/hg/jpf/jpf-core
```

## 编译

### 使用Eclipse

最方便的编译方式是将项目导入Eclipse。这需要首先安装Eclipse的JPF插件，详见[这里](http://babelfish.arc.nasa.gov/trac/jpf/wiki/install/eclipse-plugin#installfromupdatesite)。默认情况下，Eclipse导入项目后会自动编译。

### 命令行

JPF项目中包含了Ant的构建脚本build.xml，可以直接使用Ant进行构建：

```Shell
ant build
```

我第一次build的时候，ant提示出现“cannot find symbol”错误。编译器无法找到sun.misc.SharedSecret.java中引用的`JavaObjectInputStreamAccess`类。但此时Eclipse的编译是成功的。我发现这个类在JRE的lib/rt.jar中有，于是在build.xml中添加了几行：

```
  <path id="lib.path">
    <pathelement location="build/main"/>
    <pathelement location="build/peers"/>
    <pathelement location="build/annotations"/>
    <fileset dir=".">
  	  <include name="lib/*.jar"/>
    </fileset>
+   <fileset dir="/home/william/software/jre1.8.0_91">
+     <include name="lib/*.jar"/>
+   </fileset>
  </path>
```

添加之后，ant构建成功了（虽然有很多警告）。

## 运行

关于运行的详细说明可以看[这里](http://babelfish.arc.nasa.gov/trac/jpf/wiki/user/run)。

以下例子都假设运行src/examples/Racer.jpf

### 命令行

```Shell
./bin/jpf src/examples/Racer.jpf
```

### Eclipse

在src/examples/Racer.jpf文件上单击右键，选择“Verify”项。

### 在Java程序中运行

```Java
import gov.nasa.jpf.Config;
import gov.nasa.jpf.JPF;

public class Main {

	public static void main(String[] args) {

		Config conf = JPF.createConfig(args);
		JPF jpf = new JPF(conf);
		jpf.run();

	}
}
```

编译后用`java Main src/examples/Racer.jpf`运行。

## 报告

用JPF分析src/examples/Racer.jpf后可以看到控制台输出详细的报告。这个报告是可以进行配置的，不仅可以配置报告包含哪些内容，还可以控制报告的格式（纯文本、XML）和报告输出的地方（标准输出、文件）。

JPF Wiki中的[Report](http://babelfish.arc.nasa.gov/trac/jpf/wiki/user/output#Reports)章节介绍了JPF输出的不同阶段。阅读gov.nasa.jpf.report.Publisher类，可以看到，实际上的阶段比Wiki中更多。大阶段和小阶段有：

+ start
  + jpf —— 输出JPF的版本号
  + platform —— 输出平台信息，包括主机名、架构、操作系统、Java版本
  + user —— 用户名
  + dtg —— 开始时间
  + config —— JPF此次运行的所有配置，包括site.properties, jpf.properties和.jpf配置
  + sut —— 被测程序（System Under Test）的入口
+ transition
  + statistics —— 统计数据，包括花费的时间和内存，经过的状态数统计，下同
+ probe
  + statistics
+ constraint
  + constraint —— 上一个search constraint，似乎没多大用
  + trace —— 程序轨迹（路径）
  + snapshot —— VM当前的状态
  + output —— 程序轨迹的输出
  + statistics
+ property violation
  + error —— 当前错误的细节
  + trace
  + snapshot
  + output
  + statistics
+ finished
  + result —— 总结发现的错误
  + statistics

Publisher类中对每一个大阶段和小阶段都定义了各自的方法，如start大阶段对应`publishStart()`，jpf小阶段对应`publishJPF()`。每个大阶段的方法调用下面所有的小阶段的方法。在Publisher类中，小阶段的方法是抽象方法，需要子类来实现。默认提供了Publisher类的两个子类，ConsolePublisher和XMLPublisher，分别定义了不同的输出行为。

配置文件中可以设置哪些大阶段小阶段的内容进行输出。例如jpf-core下默认的配置文件jpf.properties内容如下：

```
report.console.start=jpf,sut

report.console.transition=
report.console.constraint=constraint,snapshot

report.console.probe=statistics

report.console.property_violation=error,snapshot
report.console.show_steps=true
report.console.show_method=true
report.console.show_code=false

report.console.finished=result,statistics
```

这个配置文件说明了：当使用ConsolePublisher时，start阶段输出jpf和sut小阶段，property violation阶段输出error和snapshot小阶段，以此类推。

# JAR File Specification

# Java规范系列：JAR文件规范

[TOC]


## Introduction

JAR file is a file format based on the popular ZIP file format and is used for aggregating many files into one. A JAR file is essentially a zip file that contains an optional `META-INF` directory. A JAR file can be created by the command-line jar tool, or by using the [`java.util.jar`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/jar/package-summary.html) API in the Java platform. There is no restriction on the name of a JAR file, it can be any legal file name on a particular platform.

## JAR文件格式

JAR文件基于ZIP文件格式, 用于将多个文件打包成一个压缩文件。
本质上JAR文件就是一个zip文件, 其中包含一个可选的 `META-INF` 目录。
可以通过命令行工具 `jar` 打包jar文件, 也可以在程序中使用[`java.util.jar`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/jar/package-summary.html)里面的工具类来创建jar文件。
JAR文件的名称没有限制, 只要符合所在平台上的文件命名规则即可。

![](01_jar_meta_inf.jpg)

## Modular JAR files

A modular JAR file is a JAR file that has a module descriptor, `module-info.class`, in the top-level directory (or root) directory. The module descriptor is the binary form of a module declaration. (Note the section on [multi-release JAR files](#multi-release-jar-files) further refines the definition of modular JAR files.)

A modular JAR file deployed on the module path, as opposed to the class path, is an *explicit* module. Dependences and service providers are declared in the module descriptor. If the modular JAR file is deployed on the class path then it behaves as if a non-modular JAR file.

A non-modular JAR file deployed on the module path is an *automatic module*. If the JAR file has a main attribute `Automatic-Module-Name` (see [Main Attributes](#main-attributes)) then the attribute's value is the module name, otherwise the module name is derived from the name of the JAR file as specified in [`ModuleFinder.of(Path...)`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/module/ModuleFinder.html#automatic-modules).

## 模块化JAR文件

模块化JAR文件也是JAR文件格式, 其内部的顶级路径（根路径）下有一个模块描述文件 `module-info.class`。
模块描述文件是模块声明的二进制形式。  更详细的模块化JAR文件定义请参考: [multi-release JAR files](#multi-release-jar-files)。

相对于类路径, 部署在模块路径下的模块化JAR文件是 **显式的** 模块。
依赖项和服务提供者都在模块描述符中声明。
如果将模块化JAR文件部署到类路径中, 则其行为就如同非模块化的JAR文件一样。

部署在模块路径下的非模块化JAR文件则是 **自动模块**。
如果JAR文件具有主属性 [`Automatic-Module-Name`](#main-attributes), 那么模块名称就是该属性的值, 否则模块名称根据 Jar 文件名推导得出, 具体推导规则请参考: [`ModuleFinder.of(Path...)`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/module/ModuleFinder.html#automatic-modules).

## Multi-release JAR files

A multi-release JAR file allows for a single JAR file to support multiple major versions of Java platform releases. For example, a multi-release JAR file can depend on both the Java 8 and Java 9 major platform releases, where some class files depend on APIs in Java 8 and other class files depend on APIs in Java 9. This enables library and framework developers to decouple the use of APIs in a specific major version of a Java platform release from the requirement that all their users migrate to that major version. Library and framework developers can gradually migrate to and support new Java features while still supporting the old features.

A multi-release JAR file is identified by the main attribute:

## 多版本JAR文件

多版本JAR文件, 允许单个JAR文件支持多个JDK版本。
例如, 一个多版本的JAR文件可以同时依赖Java 8和Java 9版本, 其中, 有一些类文件是兼容Java 8中的API, 另一些类文件则兼容Java 9的API。
这个特性主要是第三方库和框架的开发人员在使用, 以便兼容不同的JDK版本。
第三方库和框架的开发人员可以逐步迁移并支持新的Java特性, 同时也兼容旧的特性。

多版本JAR文件由main属性来标识：

```
Multi-Release: true
```

declared in the main section of the [JAR Manifest](#jar-manifest).

Classes and resource files dependent on a major version, 9 or greater, of a Java platform release may be located under a *versioned directory* instead of under the top-level (or root) directory. The versioned directory is located under the [the META-INF directory](#the-meta-inf-directory) and is of the form:

在 [JAR Manifest](#jar-manifest) 的main这一节中声明。

依赖于Java 9 或更高版本的类和资源文件可以放到 **特定版本目录** 下, 而不是顶级目录中。
版本目录位于 [META-INF](#the-meta-inf-directory) 下面, 其格式为：

```
META-INF/versions/N
```

where N is the string representation of the major version number of a Java platform release. Specifically `N` must conform to the specification:

其中 `N` 是Java平台大版本号的字符串表示形式。 具体来说, `N`的格式需要满足：

| N: | `{1-9}{0-9}*` |
| :--- | ------------------ |
|      |                    |


Any versioned directory whose value of `N` is less than `9` is ignored as is a string representation of `N` that does not conform to the above specification.

A class file under a versioned directory, of version `N` say, in a multi-release JAR must have a class file version less than or equal to the class file version associated with `N`th major version of a Java platform release. If the class of the class file is public or protected then that class must *preside over* a class of the same fully qualified name and access modifier whose class file is present under the top-level directory. By logical extension this applies to a class of a class file, if present, under a versioned directory whose version is less than `N`.

如果N的值小于9 将被忽略, 不符合上述规范的版本目录也会被忽略。

在多版本JAR中, 版本目录`N`下面的 class 文件, 其 class file version 必须小于等于Java平台第N个大版本对应的 major version。
如果类文件中的类是 public 或 protected 的, 那么根目录下也必须能够找到具有完全限定名和访问修饰符的class。
通过逻辑扩展, 这同样适用于版本小于“ N”的类文件。

If a multi-release JAR file is deployed on the class path or module path (as an automatic module or an explicit [multi-release module](#modular-multi-release-jar-files)) of major version `N` of a Java platform release runtime, then a class loader loading classes from that JAR file will first search for class files under the `N`th versioned directory, then prior versioned directories in descending order (if present), down to a lower major version bound of `9`, and finally under the top-level directory.

The public API exported by the classes in a multi-release JAR file must be *exactly* the same across versions, hence at a minimum why versioned public or protected classes for class files under a versioned directory must preside over classes for class files under the top-level directory. It is difficult and costly to perform extensive API verification checks as such tooling, such as the `jar` tool, is not required to perform extensive verification and a Java runtime is not required to perform any verification. A future release of this specification may relax the exact same API constraint to support careful evolution.

如果将多版本JAR文件部署到 class path 或 module path 中, 假设JDK版本为 `N`,  那么类加载器从该JAR文件加载class的时候, 将优先搜索版本目录`N`, 找不到则递减, 搜索 `N-1`, 直到下限9为止, 最后才会搜索顶级目录。

多版本JAR文件中, 各个版本暴露的 public API 必须 “完全一致”,  这就解释了为什么特定版本目录下的 public 和 protected 类文件, 都必须在根目录下存在相同限定名的类。
执行扩展API的校验非常困难而且开销很大, 所以并不要求 `jar` 之类的工具来验证,  也不要求Java运行时来执行这类验证。
本规范未来的版本可能会放宽完全一致的API约束, 以支持谨慎的演进。


Resources under the `META-INF` directory cannot be versioned (such as for service configuration).

A multi-release JAR file can be signed.

Multi-release JAR files are not supported by the boot class loader of a Java runtime. If a multi-release JAR file is appended to the boot class path (with the `-Xbootclasspath/a` option) then the JAR is treated as if it is an ordinary JAR file.



`META-INF` 目录下的资源无法进行版本控制（例如服务配置）。

多版本JAR文件也可以进行签名。

Java运行时的引导类加载器（ boot class loader）不支持多版本JAR文件。
如果将多版本JAR文件放到引导类路径（使用 `-Xbootclasspath/a` 选项）, 则该JAR将被当做一个普通的JAR文件。

### Modular multi-release JAR files

A modular multi-release JAR file is a multi-release JAR file that has a module descriptor, `module-info.class`, in the top-level directory (as for a [modular](#modular-jar-files) JAR file), or directly in a versioned directory.

A public or protected class in a non-exported package (that is not declared as exported in the module descriptor) need not preside over a class of the same fully qualified name and access modifier whose class file is present under the top-level directory.

A module descriptor is generally treated no differently to any other class or resource file. A module descriptor may be present under a versioned area but not present under the top-level directory. This ensures, for example, only Java 8 versioned classes can be present under the top-level directory while Java 9 versioned classes (including, or perhaps only, the module descriptor) can be present under the `9` versioned directory.

Any versioned module descriptor that presides over a lesser versioned module descriptor or that at the top-level, `M` say, must be identical to `M`, with two exceptions:

1. the presiding versioned descriptor can have different non-`transitive` `requires` clauses of `java.*` and `jdk.*` modules; and
2. the presiding versioned descriptor can have different `uses` clauses, even of service types defined outside of `java.*` and `jdk.*` modules.

Tooling, such as the `jar` tool, should perform such verification of versioned module descriptors but a Java runtime is not required to perform any verification.

### 模块化的多版本JAR文件

模块化的多版本JAR文件, 内部兼容多个JDK版本, 通过描述模块信息的 `module-info.class` 文件来描述, 这个描述文件位于模块化JAR文件的顶层目录, 或者位于版本目录中。

在模块描述符中未声明为导出的包称为非导出包, 其中的 public 或 protected 类, 不需要具有相同的完全限定名和访问修饰符的类, 其 class 文件直接放到顶级目录下。

模块描述符与其他的class文件或资源文件并不区别对待。 模块描述符可以存放在版本化区域下, 而不必存放在顶级目录下。 这样就确保了只有 Java 8 版本的类会出现在顶级目录下, 而Java 9版本的类（包括模块描述符）会出现在`9`版本目录下。

更小版本的模块描述符, 或者顶层的模块描述符, 都必须与`M`相同, 但有两个例外：

1. 主版本描述符可以和 `java.*` 和 `jdk.*` 模块具有不同的 non-`transitive` `requires` 子句；
2. 主导版本描述符可以具有不同的 `uses` 子句, 即使是在 `java.*` 和 `jdk.*` 模块之外定义的服务类型也是如此。

例如 `jar` 之类的工具, 应该执行版本化模块描述符的验证, 但 Java 运行时不需要执行任何验证。


## The META-INF directory

The following files/directories in the META-INF directory are recognized and interpreted by the Java Platform to configure applications, class loaders and services:

## `META-INF` 目录

Java平台通过解析 `META-INF` 目录下的文件和目录, 自动配置应用程序, 类加载器以及服务：

- `MANIFEST.MF`

  The manifest file that is used to define package related data.

  清单文件, 用于定义与Jar包相关的数据。

- `INDEX.LIST`

  This file is generated by the new "`-i`" option of the jar tool, which contains location information for packages defined in an application. It is part of the JarIndex implementation and used by class loaders to speed up their class loading process.

  这个文件由 jar 工具的新选项 "`-i`" 生成, 包含应用程序中定义的 package 的位置信息。 它是 JarIndex 实现的一部分, 用来加快 class loader 的类加载过程。

- `xxx.SF`

  The signature file for the JAR file. 'xxx' stands for the base file name.

  JAR文件的签名文件。 'xxx' 代表基本文件名。

- `xxx.DSA`

  The signature block file associated with the signature file with the same base file name. This file stores the digital signature of the corresponding signature file.

  与具有相同文件名的 `xxx.SF` 文件关联, 表示其签名块。 该文件用来存储相应签名文件的数字签名。

- `services/`

  This directory stores all the service provider configuration files for JAR files deployed on the class path or JAR files deployed as automatic modules on the module path. See the specification of [service provider development](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/ServiceLoader.html#developing-service-providers) for more details.

  这个目录用来保存 service provider 的配置文件, 包括 class path 上的JAR文件, 或者 module path 上的自动模块。  更多细节请参见 [service provider development](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/ServiceLoader.html#developing-service-providers)。

- `versions/`

  This directory contains underneath it versioned class and resource files for a [multi-release](#multi-release-jar-files) JAR file.

  这个目录下保存的就是 [multi-release](#multi-release-jar-files) JAR文件中, 对应版本的类和资源文件。

## Name-Value pairs and Sections

Before we go to the details of the contents of the individual configuration files, some format convention needs to be defined. In most cases, information contained within the manifest file and signature files is represented as so-called "name: value" pairs inspired by the RFC822 standard. We also call these pairs headers or attributes.

Groups of name-value pairs are known as a "section". Sections are separated from other sections by empty lines.

Binary data of any form is represented as base64. Continuations are required for binary data which causes line length to exceed 72 bytes. Examples of binary data are digests and signatures.

Implementations shall support header values of up to 65535 bytes.

All the specifications in this document use the same grammar in which terminal symbols are shown in fixed width font and non-terminal symbols are shown in italic type face.

## 名值对与Section

在介绍每个配置文件内容之前, 需要明确一些格式约定。 清单文件和签名文件中包含的信息格式, 受RFC822标准启发, 在大部分情况下, 表示为 "名:值" 对。 我们也将其称为报头(header)或属性(attribute)。

一组名/值对, 称为一“段(Section)”。 各段之间用空行分隔。

任何形式的二进制数据都使用 base64 表示。 二进制数据需要连续, 这会导致行长超过72个字节。 例如摘要(digest)和签名(signature)。

JVM实现需要支持最大65535字节的报头值。

本文档中的所有规范都使用相同的语法, 其中终结符以固定宽度的字体显示, 非终结符以斜体显示。

### Specification:

| *section:*          | **header +newline*                                |
| :------------------ | ------------------------------------------------- |
| *nonempty-section:* | *+header +newline*                                |
| *newline:*          | `CR LF | LF | CR` (*not followed by* `LF`)        |
| *header:*           | *name* `:` *value*                                |
| *name:*             | *alphanum \*headerchar*                           |
| *value:*            | SPACE **otherchar newline \*continuation*         |
| *continuation:*     | SPACE **otherchar newline*                        |
| *alphanum:*         | {`A-Z`} | {`a-z`} | {`0-9`}                       |
| *headerchar:*       | *alphanum* \| `-` | `_`                           |
| *otherchar:*        | *any UTF-8 character except* `NUL, CR` *and* `LF` |

> Note: To prevent mangling of files sent via straight e-mail, no header will start with the four letters "From".

Non-terminal symbols defined in the above specification will be referenced in the following specifications.

> 注意: 为防止直接通过电子邮件发送的文件损坏, 没有标题时则以四个字母 "From" 开头。

以上规范中定义的非终结符, 将在后面的规范中引用。

## JAR Manifest

### Overview

A JAR file manifest consists of a main section followed by a list of sections for individual JAR file entries, each separated by a newline. Both the main section and individual sections follow the section syntax specified above. They each have their own specific restrictions and rules.

- The main section contains security and configuration information about the JAR file itself, as well as the application. It also defines main attributes that apply to every individual manifest entry. No attribute in this section can have its name equal to "`Name`". This section is terminated by an empty line.
- The individual sections define various attributes for packages or files contained in this JAR file. Not all files in the JAR file need to be listed in the manifest as entries, but all files which are to be signed must be listed. The manifest file itself must not be listed. Each section must start with an attribute with the name as "`Name`", and the value must be a relative path to the file, or an absolute URL referencing data outside the archive.
- If there are multiple individual sections for the same file entry, the attributes in these sections are merged. If a certain attribute have different values in different sections, the last one is recognized.
- Attributes which are not understood are ignored. Such attributes may include implementation specific information used by applications.

## JAR文件清单

### 概述

JAR文件的清单中,有一个 main section, 后面是各个条目的 section 列表, 各个部分之间用换行符分隔。 main section 和其他 section 都遵循上面指定的语法。 每个 section 都有自己特定的限制和规则。

- main section 主要是安全和配置信息, 包含JAR文件以及应用程序的。 还定义了适用于每个清单实体的主要属性。 这部分的任何属性名, 都不能为“`Name`”。 以空行结束。
- 各个部分定义了JAR文件中包含的程序包或文件的各种属性。 JAR文件中的所有文件并不是都要在清单条目中列出,  但所有要签名的文件都必须列出。 清单文件自身不能列出。 每个部分都必须以名为 "`Name`" 的属性开头, 对应的值必须是文件的相对路径, 或者是引用外部数据的绝对URL。
- 如果同一文件条目有多个 section, 则这些section中的属性将被合并。 如果某个属性在不同section中具有不同的值, 则将取最后一个的值。
- 无法解析的属性将被忽略。这样的属性主要用于保存某些特定实现的信息。

### Manifest Specification:

### 清单格式

| *manifest-file:*      | *main-section newline \*individual-section*     |
| :-------------------- | ----------------------------------------------- |
| *main-section:*       | *version-info newline \*main-attribute*         |
| *version-info:*       | `Manifest-Version :` *version-number*           |
| *version-number:*     | *digit+{*`.`*digit+}**                          |
| *main-attribute:*     | *(any legitimate main attribute) newline*       |
| *individual-section:* | `Name :` *value* *newline \*perentry-attribute* |
| *perentry-attribute:* | *(any legitimate perentry attribute) newline*   |
| *newline:*            | `CR LF | LF | CR` (*not followed by* `LF`)      |
| *digit:*              | `{0-9}`                                         |

In the above specification, attributes that can appear in the main section are referred to as main attributes, whereas attributes that can appear in individual sections are referred to as per-entry attributes. Certain attributes can appear both in the main section and the individual sections, in which case the per-entry attribute value overrides the main attribute value for the specified entry. The two types of attributes are defined as follows.

在以上格式中，可以出现在 main section 中的属性被称为主属性(main attribute)， 而出现在各个部分中的属性被称为每个条目的属性。 某些属性可以同时出现在 main section 和 单个部分 中，在这种情况下，每个条目的属性值将覆盖指定条目的主属性值。 两种属性的定义如下。

### Main Attributes

Main attributes are the attributes that are present in the main section of the manifest. They fall into the following different groups:

### 主属性

主属性是清单文件中, main section 部分中存在的属性。 它们可分为下面这些不同的组：

- general main attributes

  - Manifest-Version: Defines the manifest file version. The value is a legitimate version number, as described in the above spec.
  - Created-By: Defines the version and the vendor of the java implementation on top of which this manifest file is generated. This attribute is generated by the `jar` tool.
  - Signature-Version: Defines the signature version of the jar file. The value should be a valid *version-number* string.
  - Class-Path: The value of this attribute specifies the relative URLs of the libraries that this application needs. URLs are separated by one or more spaces. The application class loader uses the value of this attribute to construct its internal search path. See [Class-Path Attribute](#class-path-attribute) section for details.
  - Automatic-Module-Name: Defines the module name if this JAR file is deployed as an automatic module on the module path. For further details see the specification of [`automatic modules`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/module/ModuleFinder.html#automatic-modules).
  - Multi-Release: This attribute defines whether this JAR file is a [multi-release](#modular-multi-release-jar-files) JAR file. If the value is "true" , case is ignored, then the JAR file will be processed by the Java runtime and tooling as a multi-release JAR file. Otherwise, if the value is anything other than "true" then this attribute is ignored.

- attribute defined for stand-alone applications: This attribute is used by stand-alone applications that are bundled into executable jar files which can be invoked by the java runtime directly by running "`java -jar x.jar`".

  - Main-Class: The value of this attribute is the class name of the main application class which the launcher will load at startup time. The value must *not* have the `.class` extension appended to the class name.
  - Launcher-Agent-Class: If this attribute is present then its value is the class name of a *java agent* that is started before the application main method is invoked. This attribute can be used for cases where a java agent is packaged in the same executable JAR file as the application. The agent class defines a public static method name `agentmain` in one of the two forms specified in the [`java.lang.instrument`](https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/package-summary.html) package summary. Additional attributes (such as `Can-Retransform-Classes`) can be used to indicate capabilities needed by the agent.

- attributes defined for [package versioning and sealing](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/Package.html) information: The value of these attributes apply to all the packages in the JAR file, but can be overridden by per-entry attributes.

  - Implementation-Title: The value is a string that defines the title of the extension implementation.
  - Implementation-Version: The value is a string that defines the version of the extension implementation.
  - Implementation-Vendor: The value is a string that defines the organization that maintains the extension implementation.
  - Specification-Title: The value is a string that defines the title of the extension specification.
  - Specification-Version: The value is a string that defines the version of the extension specification.
  - Specification-Vendor: The value is a string that defines the organization that maintains the extension specification.
  - Sealed: This attribute defines whether this JAR file is sealed or not. The value can be either "true" or "false", case is ignored. If it is set to "true", then all the packages in the JAR file are defaulted to be sealed, unless they are defined otherwise individually. See also the [Package Sealing](#package-sealing) section.

### Per-Entry Attributes

Per-entry attributes apply only to the individual JAR file entry to which the manifest entry is associated with. If the same attribute also appeared in the main section, then the value of the per-entry attribute overwrites the main attribute's value. For example, if JAR file a.jar has the following manifest content:

```
Manifest-Version: 1.0
Created-By: 1.8 (Oracle Inc.)
Sealed: true
Name: foo/bar/
Sealed: false
```

It means that all the packages archived in a.jar are sealed, except that package foo.bar is not.

The per-entry attributes fall into the following groups:

- attributes defined for file contents:
  - Content-Type: This attribute can be used to specify the MIME type and subtype of data for a specific file entry in the JAR file. The value should be a string in the form of *type/subtype.* For example "image/bmp" is an image type with a subtype of bmp (representing bitmap). This would indicate the file entry as an image with the data stored as a bitmap. RFC [1521](http://www.ietf.org/rfc/rfc1521.txt) and [1522](http://www.ietf.org/rfc/rfc1522.txt) discuss and define the MIME types definition.
- attributes defined for package versioning and sealing information: These are the same set of attributes defined above as main attributes that defines the extension package versioning and sealing information. When used as per-entry attributes, these attributes overwrites the main attributes but only apply to the individual file specified by the manifest entry.
- attribute defined for beans objects:
  - Java-Bean: Defines whether the specific jar file entry is a Java Beans object or not. The value should be either "true" or "false", case is ignored.
- attributes defined for signing: These attributes are used for signing and verifying purposes. More details here.
  - x-Digest-y: The name of this attribute specifies the name of the digest algorithm used to compute the digest value for the corresponding jar file entry. The value of this attribute stores the actual digest value. The prefix 'x' specifies the algorithm name and the optional suffix 'y' indicates to which language the digest value should be verified against.
  - Magic: This is an optional attribute that can be used by applications to indicate how verifier should compute the digest value contained in the manifest entry. The value of this attribute is a set of comma separated context specific strings. Detailed description is here.

## Signed JAR File

### Overview

A JAR file can be signed by using the command line jarsigner tool or directly through the `java.security` API. Every file entry, including non-signature related files in the `META-INF` directory, will be signed if the JAR file is signed by the jarsigner tool. The signature related files are:

- `META-INF/MANIFEST.MF`
- `META-INF/*.SF`
- `META-INF/*.DSA`
- `META-INF/*.RSA`
- `META-INF/SIG-*`

Note that if such files are located in `META-INF` subdirectories, they are not considered signature-related. Case-insensitive versions of these filenames are reserved and will also not be signed.

Subsets of a JAR file can be signed by using the `java.security` API. A signed JAR file is exactly the same as the original JAR file, except that its manifest is updated and two additional files are added to the `META-INF` directory: a signature file and a signature block file. When jarsigner is not used, the signing program has to construct both the signature file and the signature block file.

For every file entry signed in the signed JAR file, an individual manifest entry is created for it as long as it does not already exist in the manifest. Each manifest entry lists one or more digest attributes and an optional [Magic attribute](#the-magic-attribute).

### Signature File

Each signer is represented by a signature file with extension `.SF`. The major part of the file is similar to the manifest file. It consists of a main section which includes information supplied by the signer but not specific to any particular jar file entry. In addition to the `Signature-Version` and `Created-By` attributes (see [Main Attributes](#main-attributes)), the main section can also include the following security attributes:

- x-Digest-Manifest-Main-Attributes (where x is the standard name of a `java.security.MessageDigest` algorithm): The value of this attribute is the digest value of the main attributes of the manifest.
- x-Digest-Manifest (where x is the standard name of a `java.security.MessageDigest` algorithm): The value of this attribute is the digest value of the entire manifest.

The main section is followed by a list of individual entries whose names must also be present in the manifest file. Each individual entry must contain at least the digest of its corresponding entry in the manifest file.

Paths or URLs appearing in the manifest file but not in the signature file are not used in the calculation.

### Signature Validation

A successful JAR file verification occurs if the signature(s) are valid, and none of the files that were in the JAR file when the signatures were generated have been changed since then. JAR file verification involves the following steps:

1. Verify the signature over the signature file when the manifest is first parsed. For efficiency, this verification can be remembered. Note that this verification only validates the signature directions themselves, not the actual archive files.

2. If an `x-Digest-Manifest` attribute exists in the signature file, verify the value against a digest calculated over the entire manifest. If more than one `x-Digest-Manifest` attribute exists in the signature file, verify that at least one of them matches the calculated digest value.

3. If an `x-Digest-Manifest` attribute does not exist in the signature file or none of the digest values calculated in the previous step match, then a less optimized verification is performed:

   1. If an `x-Digest-Manifest-Main-Attributes` entry exists in the signature file, verify the value against a digest calculated over the main attributes in the manifest file. If this calculation fails, then JAR file verification fails. This decision can be remembered for efficiency. If an `x-Digest-Manifest-Main-Attributes` entry does not exist in the signature file, its nonexistence does not affect JAR file verification and the manifest main attributes are not verified.
   2. Verify the digest value in each source file information section in the signature file against a digest value calculated against the corresponding entry in the manifest file. If any of the digest values don't match, then JAR file verification fails.

   One reason the digest value of the manifest file that is stored in the `x-Digest-Manifest` attribute may not equal the digest value of the current manifest file is that it might contain sections for newly added files after the file was signed. For example, suppose one or more files were added to the JAR file (using the jar tool) after the signature (and thus the signature file) was generated. If the JAR file is signed again by a different signer, then the manifest file is changed (sections are added to it for the new files by the jarsigner tool) and a new signature file is created, but the original signature file is unchanged. A verification on the original signature is still considered successful if none of the files that were in the JAR file when the signature was generated have been changed since then, which is the case if the digest values in the non-header sections of the signature file equal the digest values of the corresponding sections in the manifest file.

4. For each entry in the manifest, verify the digest value in the manifest file against a digest calculated over the actual data referenced in the "Name:" attribute, which specifies either a relative file path or URL. If any of the digest values don't match, then JAR file verification fails.

Example manifest file:

```
Manifest-Version: 1.0
Created-By: 1.8.0 (Oracle Inc.)

Name: common/class1.class
SHA-256-Digest: (base64 representation of SHA-256 digest)

Name: common/class2.class
SHA1-Digest: (base64 representation of SHA1 digest)
SHA-256-Digest: (base64 representation of SHA-256 digest)
```

The corresponding signature file would be:

```
Signature-Version: 1.0
SHA-256-Digest-Manifest: (base64 representation of SHA-256 digest)
SHA-256-Digest-Manifest-Main-Attributes: (base64 representation of SHA-256 digest)

Name: common/class1.class
SHA-256-Digest: (base64 representation of SHA-256 digest)

Name: common/class2.class
SHA-256-Digest: (base64 representation of SHA-256 digest)
```

### The Magic Attribute

Another requirement to validate the signature on a given manifest entry is that the verifier understand the value or values of the Magic key-pair value in that entry's manifest entry.

The Magic attribute is optional but it is required that a parser understand the value of an entry's Magic key if it is verifying that entry's signature.

The value or values of the Magic attribute are a set of comma-separated context-specific strings. The spaces before and after the commas are ignored. Case is ignored. The exact meaning of the magic attributes is application specific. These values indicate how to compute the hash value contained in the manifest entry, and are therefore crucial to the proper verification of the signature. The keywords may be used for dynamic or embedded content, multiple hashes for multilingual documents, etc.

Here are two examples of the potential use of Magic attribute in the manifest file:

```
Name: http://www.example-scripts.com/index#script1
SHA-256-Digest: (base64 representation of SHA-256 hash)
Magic: JavaScript, Dynamic

Name: http://www.example-tourist.com/guide.html
SHA-256-Digest: (base64 representation of SHA-256 hash)
SHA-256-Digest-French: (base64 representation of SHA-256 hash)
SHA-256-Digest-German: (base64 representation of SHA-256 hash)
Magic: Multilingual
```

In the first example, these Magic values may indicate that the result of an http query is the script embedded in the document, as opposed to the document itself, and also that the script is generated dynamically. These two pieces of information indicate how to compute the hash value against which to compare the manifest's digest value, thus comparing a valid signature.

In the second example, the Magic value indicates that the document retrieved may have been content-negotiated for a specific language, and that the digest to verify against is dependent on which language the document retrieved is written in.

## Digital Signatures

A digital signature is a signed version of the `.SF` signature file. These are binary files not intended to be interpreted by humans.

Digital signature files have the same filenames as the .SF files but different extensions. The extension varies depending on the type of digital signature.

- `.RSA` (PKCS7 signature, SHA-256 + RSA)
- `.DSA` (PKCS7 signature, DSA)

Digital signature files for signature algorithms not listed above must reside in the `META-INF` directory and have the prefix "`SIG-`". The corresonding signature file (`.SF` file) must also have the same prefix.

For those formats that do not support external signed data, the file shall consist of a signed copy of the `.SF` file. Thus some data may be duplicated and a verifier should compare the two files.

Formats that support external data either reference the `.SF` file, or perform calculations on it with implicit reference.

Each `.SF` file may have multiple digital signatures, but those signatures should be generated by the same legal entity.

File name extensions may be 1 to 3 *alphanum* characters. Unrecognized extensions are ignored.

## Notes on Manifest and Signature Files

Following is a list of additional restrictions and rules that apply to manifest and signature files.

- Attributes:
  - In all cases for all sections, attributes which are not understood are ignored.
  - Attribute names are case insensitive. Programs which generate manifest and signature files should use the cases shown in this specification however.
  - Attribute names cannot be repeated within a section.
- Versions:
  - Manifest-Version and Signature-Version must be first, and in exactly that case (so that they can be recognized easily as magic strings). Other than that, the order of attributes within a main section is not significant.
- Ordering:
  - The order of individual manifest entries is not significant.
  - The order of individual signature entries is not significant, except that the digests that get signed are in that order.
- Line length:
  - No line may be longer than 72 bytes (not characters), in its UTF8-encoded form. If a value would make the initial line longer than this, it should be continued on extra lines (each starting with a single SPACE).
- Errors:
  - If a file cannot be parsed according to this spec, a warning should be output, and none of the signatures should be trusted.
- Limitations:
  - Because header names cannot be continued, the maximum length of a header name is 70 bytes (there must be a colon and a SPACE after the name).
  - NUL, CR, and LF can't be embedded in header values, and NUL, CR, LF and ":" can't be embedded in header names.
  - Implementations should support 65535-byte (not character) header values, and 65535 headers per file. They might run out of memory, but there should not be hard-coded limits below these values.
- Signers:
  - It is technically possible that different entities may use different signing algorithms to share a single signature file. This violates the standard, and the extra signature may be ignored.
- Algorithms:
  - No digest algorithm or signature algorithm is mandated by this standard. However, at least one of SHA-256 and SHA1 digest algorithm must be supported.

## JAR Index

### Overview

Since 1.3, JarIndex is introduced to optimize the class searching process of class loaders for network applications, especially applets. Originally, an applet class loader uses a simple linear search algorithm to search each element on its internal search path, which is constructed from the "ARCHIVE" tag or the "Class-Path" main attribute. The class loader downloads and opens each element in its search path, until the class or resource is found. If the class loader tries to find a nonexistent resource, then all the jar files within the application or applet will have to be downloaded. For large network applications and applets this could result in slow startup, sluggish response and wasted network bandwidth. The JarIndex mechanism collects the contents of all the jar files defined in an applet and stores the information in an index file in the first jar file on the applet's class path. After the first jar file is downloaded, the applet class loader will use the collected content information for efficient downloading of jar files.

The existing `jar` tool is enhanced to be able to examine a list of jar files and generate directory information as to which classes and resources reside in which jar file. This directory information is stored in a simple text file named `INDEX.LIST` in the `META-INF` directory of the root jar file. When the classloader loads the root jar file, it reads the `INDEX.LIST` file and uses it to construct a hash table of mappings from file and package names to lists of jar file names. In order to find a class or a resource, the class loader queries the hashtable to find the proper jar file and then downloads it if necessary.

Once the class loader finds a `INDEX.LIST` file in a particular jar file, it always trusts the information listed in it. If a mapping is found for a particular class, but the class loader fails to find it by following the link, an unspecified Error or RuntimeException is thrown. When this occurs, the application developer should rerun the `jar` tool on the extension to get the right information into the index file.

To prevent adding too much space overhead to the application and to speed up the construction of the in-memory hash table, the INDEX.LIST file is kept as small as possible. For classes with non-null package names, mappings are recorded at the package level. Normally one package name is mapped to one jar file, but if a particular package spans more than one jar file, then the mapped value of this package will be a list of jar files. For resource files with non-empty directory prefixes, mappings are also recorded at the directory level. Only for classes with null package name, and resource files which reside in the root directory, will the mapping be recorded at the individual file level.

### Index File Specification

The `INDEX.LIST` file contains one or more sections each separated by a single blank line. Each section defines the content of a particular jar file, with a header defining the jar file path name, followed by a list of package or file names, one per line. All the jar file paths are relative to the code base of the root jar file. These path names are resolved in the same way as the current extension mechanism does for bundled extensions.

The UTF-8 encoding is used to support non ASCII characters in file or package names in the index file.

#### Specification

| *index file:*     | *version-info blankline section**                         |
| :---------------- | --------------------------------------------------------- |
| *version-info:*   | `JarIndex-Version:` *version-number*                      |
| *version-number:* | *digit+{.digit+}**                                        |
| *section:*        | *body blankline*                                          |
| *body:*           | *header name**                                            |
| *header:*         | *char+*`.jar` *newline*                                   |
| *name:*           | *char+ newline*                                           |
| *char:*           | *any valid Unicode character except* `NULL, CR` *and*`LF` |
| *blankline:*      | *newline newline*                                         |
| *newline:*        | `CR LF | LF | CR` (*not followed by* `LF`)                |
| *digit:*          | {`0-9`}                                                   |

The `INDEX.LIST` file is generated by running `jar -i.` See the jar man page for more details.

### Backward Compatibility

The new class loading scheme is totally backward compatible with applications developed on top of the current extension mechanism. When the class loader loads the first jar file and an `INDEX.LIST` file is found in the `META-INF` directory, it would construct the index hash table and use the new loading scheme for the extension. Otherwise, the class loader will simply use the original linear search algorithm.

## Class-Path Attribute

The manifest for an application can specify one or more relative URLs referring to the JAR files and directories for other libraries that it needs. These relative URLs will be treated relative to the code base that the containing application was loaded from (the "*context JAR*").

An application (or, more generally, JAR file) specifies the relative URLs of the libraries that it needs via the manifest attribute `Class-Path`. This attribute lists the URLs to search for implementations of other libraries if they cannot be found on the host Java Virtual Machine. These relative URLs may include JAR files and directories for any libraries or resources needed by the application. Relative URLs not ending with '/' are assumed to refer to JAR files. For example,

```
Class-Path: servlet.jar infobus.jar acme/beans.jar images/
```

At most one `Class-Path` header may be specified in a JAR file's manifest.

A `Class-Path` entry is valid if the following conditions are true:

- It can be used to create a [`URL`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/net/URL.html#(java.net.URL,java.lang.String)), by resolving it against the context JAR’s URL.
- It is relative, not [absolute](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/net/URI.html#isAbsolute()), i.e. it does not contain a scheme component, except for the case when the context JAR is loaded from the file system, in which case the `file` scheme is permitted for compatibility reasons.
- The location of the JAR file or directory represented by this entry is contained within the containing directory of the context JAR. Use of "`../`" to navigate to the parent directory is not permitted, except for the case when the context JAR is loaded from the file system.

Invalid entries are ignored. Valid entries are resolved against the context JAR. If the resulting URL is invalid or refers to a resource that cannot be found, then it is ignored. Duplicate URLs are ignored.

The resulting URLs are inserted into the class path, immediately following the URL of the context JAR. For example, given the following class path:

```
a.jar b.jar
```

If `b.jar` contained the following `Class-Path` manifest attribute:

```
Class-Path: lib/x.jar a.jar
```

Then the effective search path of such a `URLClassLoader` instance would be:

```
a.jar b.jar lib/x.jar
```

Of course, if `x.jar` had dependencies of its own then these would be added according to the same rules and so on for each subsequent URL. In the actual implementation, JAR file dependencies are processed lazily so that the JAR files are not actually opened until needed.

## Package Sealing

JAR files and packages can be optionally *sealed*, so that an package can enforce consistency within a version.

A package sealed within a JAR specifies that all classes defined in that package must originate from the same JAR. Otherwise, a `SecurityException` is thrown.

A sealed JAR specifies that all packages defined by that JAR are sealed unless overridden specifically for a package.

A sealed package is specified via the manifest attribute, `Sealed`, whose value is `true` or `false` (case irrelevant). For example,

```
Name: javax/servlet/internal/
Sealed: true
```

specifies that the `javax.servlet.internal` package is sealed, and that all classes in that package must be loaded from the same JAR file.

If this attribute is missing, the package sealing attribute is that of the containing JAR file.

A sealed JAR is specified via the same manifest header, `Sealed`, with the value again of either `true` or `false`. For example,

```
Sealed: true
```

specifies that all packages in this archive are sealed unless explicitly overridden for a particular package with the `Sealed` attribute in a manifest entry.

If this attribute is missing, the JAR file is assumed to *not* be sealed, for backwards compatibility. The system then defaults to examining package headers for sealing information.

Package sealing is also important for security, because it restricts access to package-protected members to only those classes defined in the package that originated from the same JAR file.

The unnamed package is not sealable, so classes that are to be sealed must be placed in their own packages.

## API Details

## API 详情

- Package [java.util.jar](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/jar/package-summary.html)


## See Also

## 相关链接

- Package [java.security](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/security/package-summary.html)
- Package [java.util.zip](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/zip/package-summary.html)


- [JAR File Specification](https://docs.oracle.com/en/java/javase/14/docs/specs/jar/jar.html)

- [JDK 14 Documentation](https://docs.oracle.com/en/java/javase/14/)

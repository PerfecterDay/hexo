---
title: java基础-IO
date: 2019-03-05  22:30:29
tags: IO
category: java
---

## File类
`File` 类看上去是指代文件，其实它既能代表一个特定文件，也能代表一个目录。

### 目录列表
+ `String[] list()`: 返回所有文件名的字符串数组
+ `String[] list(FileNameFilter filter)`: 返回 `FileNameFilter` 过滤后的字符串数组
+ `File[] listFiles()`: 返回所有的文件数组
+ `File[] listFiles(FilenameFilter filter)`: 返回 `FilenameFilter` 过滤后的文件数组
假如 `File` 指代的是一个目录，那么就可以使用 `list()` 方法获取目录下的文件列表，如果想获取所有文件列表，使用不带参数的 `list()` 的方法即可；如果想获得一个受限列表，那么就要使用“目录过滤器”了。 `FilenameFilter` 接口的 `boolean accept(File dir,String name)` 返回 `true` 的才会返回到数组中。 `listFiles` 同理。

```
public class FileList {
    public static void main(String[] args) {****
        File f = new File("/usr/local/etc");
        File[] allFiles = f.listFiles();
        File[] filterFiles = f.listFiles((dir,name)->{
            return name.contains("a"); 
        });
        for (File allFile : allFiles) {
            System.out.print(allFile.getName()+", ");
        }
        System.out.println();
        for (File filterFile : filterFiles) {
            System.out.print(filterFile.getName()+", ");
        }
        System.out.println();
        String[] allNames = f.list();
        String[] filterNames = f.list((dir,name)-> {
                return name.contains(".");
        });
        for (String file : allNames) {
            System.out.print(file+", ");
        }
        System.out.println();
        for (String file : filterNames) {
            System.out.print(file+", ");
        }
    }
}```

### 目录/文件的检查及创建
可以获取文件/目录相关的信息：
+ `String getAbsolutePath()` :获取绝对路径
+ `String getName()` :获取名字
+ `File getParent()` :获取父目录
+ `long length()` :获取目录/文件大小
+ `long lastModified()` :获取最后修改时间
+ `boolean canExecute()` :是否可执行
+ `boolean canRead()` :是否可读
+ `boolean canWrite()` :是否可写
+ `boolean createNewFile()` :创建新文件当 `File` 代表一个文件时
+ `boolean mkdir()` :创建新目录当 `File` 代表一个目录时


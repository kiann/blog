## 通过jcifs库使用smb协议访问共享文件(以下代码全都伪代码)

----
>smb协议是微软和intel制定的协议，最常见的地方就是windows局域网网络共享了。通过jcifs这个库，我们能够像操作本地文件一样操作共享目录里的文件。

jcifs官方地址：[https://jcifs.samba.org](https://jcifs.samba.org)

这里还是使用最快速的导入jar包的方式使用该库。将官网下载到的jar文件放到libs目录并引用到classpath。然后对于远程共享目录来说我们需要一个网络地址和一个有访问权限的账户，所以用以下方式创建一个auth对象用来访问:
```java
NtlmPasswordAuthentication auth = new NtlmPasswordAuthentication("192.168.31.11","soar","111111");
```

然后对于smb协议来说，并不能使用常用的File对象，而是使用专门的SmbFile对象。以下是一个迭代远程目录的范例(必须以“**smb://**”开头)：
```java
SmbFile rootDir = new SmbFile("smb://192.168.31.11",auth);
if(rootDir != null && rootDir.exists()){
    listFiles(rootDir, auth);
}

public static void listFiles(SmbFile file, NtlmPasswordAuthentication auth){
    if(file.isFile()){
        System.out.println("file:" + file.getPath());
    }else{
        System.out.println("dir:" + file.getPath());
        SmbFile[] files = file.listFiles();
        for(SmbFile f: files){
            listFiles(f, auth);
        }
    }
}
```

对于SmbFile对象的其他用法可以自己看官方文档，非常的简单和清晰。

接下来说一下从远程复制文件到本地以及复制本地文件到远程的方法：
```java
SmbFile remoteFile = new SmbFile(remoteUrl, auth);
InputStreem inputStreem = remoteFile.getInputStream();
//这里的inputStreem就可以用来读取byte[]流写文件到本地了

OutputStreem outputStreem = remoteFile.getOutputStream();
//这里的outputStreem就可以用来卸乳byte[]流到远程了
````
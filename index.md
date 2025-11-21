---
layout: default
title: "个人笔记"
permalink: /
---


# 🚀 个人技术学习与实战笔记

> 本笔记库汇集了我在日常开发、逆向分析以及环境配置中积累的经验、操作步骤和关键“踩坑”记录。旨在系统化整理知识，并快速查阅解决方案。



## 💻 1、Git & GitHub 配置流程

### 1.1 本地密钥配置（SSH）

用于本地与 GitHub 仓库的安全连接。

```bash
# 生成 SSH 密钥，使用你的邮箱作为注释
ssh-keygen -t rsa -C "your_email@youremail.com" 
```

### 1.2 全局用户名和邮箱配置

用于 Git 提交记录的身份标识。

```bash
$git config --global user.name "your name"$ git config --global user.email "your_email@youremail.com" 
```

### 1.3 仓库操作流程

| 操作 | 命令 | 描述 |
| :--- | :--- | :--- |
| **初始化** | `git init` | 在当前目录创建一个 Git 仓库 |
| **推送** | `git add <文件>`<br>`git commit -m "提交信息"`<br>`git remote add origin <仓库地址>`<br>`git push origin master` | 提交代码到远程仓库 |
| **同步/拉取** | `git remote add origin <仓库地址>`<br>`git pull origin master` | 同步远程代码到本地 |


## 🛠️ 2、Xposed 模块开发基础操作

### 2.1 `AndroidManifest.xml` 配置

添加 Xposed 模块识别所需的 `meta-data`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)"
    package="com.example.batchathook">
    <application
        android:allowBackup="true"
        android:theme="@style/Theme.BatChatHook">
        <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="这是一个Xposed例程" />
        <meta-data
            android:name="xposedminversion"
            android:value="53" />
    </application>
</manifest>
```

### 2.2 修改 `build.gradle` (Module: app)

配置依赖库。**注意：** 配置 Gradle 报错时，请检查 Gradle 版本是否兼容。

```groovy
repositories {
    jcenter()
}
// Xposed 模块 API 依赖
compileOnly 'de.robv.android.xposed:api:82'
compileOnly 'de.robv.android.xposed:api:82:sources'
```

> **注意：** 配置 Gradle 报错时，可能需要修改 Gradle 版本为 7.0 或适应当前 Android Studio 版本。

### 2.3 核心 Hook 代码示例

实现 `IXposedHookLoadPackage` 接口，编写 Hook 逻辑。

```java
public class batChatHook implements IXposedHookLoadPackage {
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if (loadPackageParam.packageName.equals("com.example.root.xposd_hook_new")) {
            XposedBridge.log(" has Hooked!");
            Class clazz = loadPackageParam.classLoader.loadClass(
                    "com.example.root.xposd_hook_new.MainActivity");
            XposedHelpers.findAndHookMethod(clazz, "toastMessage", new XC_MethodHook() {
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                    //XposedBridge.log(" has Hooked!");
                }
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    param.setResult("你已被劫持"); // 劫持并修改方法返回值
                }
            });
        }
    }
}
```

### 2.4 Xposed 入口配置

新建 `assets/xposed_init` 文件，包含 Hook 类的完整路径。

> **操作步骤：** 右键 `main` 文件夹 -\> New -\> Folder -\> Assets Folder。在 `assets` 文件夹下新建文件 `xposed_init`，写入 Hook 类的完整路径。

-----

## 🔐 3、逆向工程：脱壳与解密操作

### 3.1 APK 脱壳与重打包流程

1.  **dex -\> smali**：使用工具将 DEX 文件反编译为 Smali 代码 (`dex->smali`, `dex2->smali_classes2` 等)。
2.  **Smali 塞回**：将修改后的 Smali 代码替换回 APK 结构。
3.  **修改入口**：调整 `AndroidManifest.xml` 中的启动入口。
4.  **Apktool 打包**：使用 Apktool 重新打包。
    ```bash
    java -jar apktool.jar b apkFile/
    ```

### 3.2 Frida 脱壳基本操作

> **关键日期：** 2021年7月7日 汤德源 记录

| 步骤 | 操作命令/描述 | 注意事项 |
| :--- | :--- | :--- |
| **1. 安装环境** | `pip install frida-tools` | 确保 Python 环境已配置 |
| **2. 下载 Server** | [Frida GitHub Releases](https://github.com/frida/frida/releases/tag/15.0.0) | **版本必须一致！** 找到对应架构的 `frida-server` |
| **3. 启动 Server** | 将 `frida-server` 放到 `/data/local/tmp`，`chmod 777 frida-server`<br>`./frida-server` | `frida-server` 和 `frida` 版本号必须一致 |
| **4. 检查运行** | `frida-ps -U` | 成功会显示进程 PID 和名称 |
| **5. 下载脱壳脚本** | `pip install frida-dexdump` | 安装脱壳工具 |
| **6. 开始脱壳** | **手机运行目标应用后**，执行 `frida-dexdump` | 脱壳文件存放路径：`C:\Users\xxx\package_name` |

### 3.3 蝙蝠聊天数据库解密步骤

1.  **定位数据库**：在 `database` 目录下找到 `batchatsql+uid.db`。
2.  **获取密钥**：运行 Java 工具获取数据库密钥。
    ```bash
    java -jar 蝙蝠聊天数据库密钥-大写.jar
    ```
3.  **使用 SQLCipher 解密**：
    ```bash
    # 启动 SQLCipher
    sqlcipher.exe 加密数据库.db

    # 输入密钥
    PRAGMA key = 'xxxxxx';

    # 设置页大小
    PRAGMA cipher_page_size = 4096;

    # 导出解密数据库
    ATTACH DATABASE 'batchatsql.db' AS batchatsql KEY '';
    SELECT sqlcipher_export('batchatsql');
    DETACH DATABASE batchatsql;

    # 退出
    .exit
    ```
4.  解密后的 `batchatsql.db` 会出现在当前目录下。

-----

## 🧩 4、开发环境与工具配置

### 4.1 JDK 免登录下载技巧

1.  **获取带重定向的链接**：右击要下载的版本，复制链接。
      * *示例:* `https://www.oracle.com/webapps/redirect/signon?nexturl=https://download.oracle.com/otn/java/jdk/...`
2.  **替换直链**：将链接中的 `otn` 替换为 `otn-pub`，即可得到真实直链进行免登录下载。
      * *示例:* `https://download.oracle.com/otn-pub/java/jdk/.../jdk-8u271-windows-x64.exe`

### 4.2 小米手机版本查看 (ADB/Fastboot)

| 环境 | 命令 | 作用 |
| :--- | :--- | :--- |
| **ADB 下** | `adb shell getprop ro.product.name` | 查看设备的产品名称 |
| **Fastboot 下** | `fastboot getvar product` | 查看设备的产品型号 |

### 4.3 VS Code Markdown 高效写作配置

| 插件名称 | 功能描述 |
| :--- | :--- |
| **MarkDown All in One** | 提供快捷键、目录生成、预览等一体化支持。 |
| **MarkDown Preview Github Styling** | 优化预览样式，使其接近 GitHub 网页效果。 |
| **MarkDown PDF** | 用于导出各种格式的文档 (PDF, HTML)。 |

> **导出操作：** `CTRL+Shift+P` 调出控制台，输入 `MarkDown PDF`，选择导出的文件格式。

### 4.4 Python 代码模板 (PyCharm/IDEA)

**路径：** 设置 -\> 编辑器 -\> 文件和代码模板 -\> Python Script

```python
# -*- coding:UTF-8 -*-
#@TIME : ${DATE} ${TIME}
#@FILE : ${NAME}.py
#@Software : ${PRODUCT_NAME}
```

### 4.5 Android 控件分析：`uiautomatorviewer`

`uiautomatorviewer` 可用于对连接到电脑的手机屏幕进行快照，查看页面层级关系和控件属性，是编写 UI 自动化测试用例的基础。

> **路径：** 在 `..\sdk\tools\` 目录下打开 `uiautomatorviewer.bat` (需确保手机已开启 USB 调试并连接电脑)。

-----

## 🛠️ 5、逆向工具链增强

### 5.1 IDA PRO 插件配置

| 插件名称 | 依赖安装 | 作用 |
| :--- | :--- | :--- |
| **Keypatch** | 安装 `keystone-0.9.1-python-win64.msi` | 用于修改二进制文件。 |
| **Findcrypt-YARA** | `pip3 install yara-python` | 用于快速定位程序中的加密算法和密钥。 |

> **安装步骤：** 将插件文件和依赖库 (如 `keystone`, `keypatch`, `findcrypt3.py` 和 `findcrypt3.rules`) 放入 IDA Pro 的 `plugins` 目录。使用时，打开 **Edit \> Plugins \> Findcrypt**。

-----

## 🤖 6、Magisk 源码同步与编译

### 6.1 镜像站推荐

  * **GitHub 镜像站：** `https://github.com.cnpmjs.org/` / `https://github.sunflyer.cn`
  * **Google Android 源码：** `https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/`

### 6.2 源码同步与更新

1.  **拉取源码：** `git clone --recurse-submodules https://github.com.cnpmjs.org/topjohnwu/Magisk.git`
2.  **修改配置：** 将 `.gitmodules` 和 `.git\config` 的 URL 地址改为镜像站地址。
3.  **获取更新：** `git fetch` (取回所有分支的更新)。
4.  **强制更新子模块：** `git submodule update -f`。

### 6.3 编译流程

1.  **配置环境：**
      * `ANDROID_SDK_ROOT`: 设置为你的 SDK 路径。
      * `Path`: 确保 JDK/JRE 的 `bin` 目录已加入环境变量。
2.  **执行编译脚本：**
      * `python build.py ndk`
      * `python build.py stub`
      * `python build.py binary`
      * `python build.py app`

> **编译出错检查：** 检查 JDK 版本是否正确，以及依赖文件夹（如 NDK）是否为空。

-----

## 🚨 7、常见问题与解决方案

### 7.1 安卓高版本刷面具 (Magisk)

需要刷入空 `vbmeta` 文件。

```
00000000 41 56 42 30
00000128 61 76 62 74 6F 6F 6C 20 31 2E 31 2E 30
```

### 7.2 pip 安装错误找不到解决办法

使用国内清华大学镜像源和 `--trusted-host` 参数。

```bash
pip install -i [https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple) --trusted-host pypi.tuna.tsinghua.edu.cn <要安装的包名>
```


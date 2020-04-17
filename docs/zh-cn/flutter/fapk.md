# 应用签名
## 1. 创建一个密钥库

> 警告： keystore文件自己管理，不要跟应用代码放一起。

linux/mac
```
keytool -genkey -v -keystore ~/key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias key
```

window
```
keytool -genkey -v -keystore c:/Users/USER_NAME/key.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias key

```
> keytool命令可能不在你的全局环境中，它是Java的一部分，而Java是作为Android Studio的一部分安装的。对于具体路径，运行flutter doctor -v并找到在“ Java binary at：”之后打印的路径。把打印的路径最后的java替换为keytool。如果您的路径包含以空格分隔的名称（例如）Program Files，请在以空格分隔的名称周围加上引号。例如：/"Program Files"/


---


## 2. 在应用引用上一步创建的密钥库

> 警告： key.properties文件自己管理，不要推到git上，里面的引用都是自己本地的keystore, 别人也用不了。

在项目目录下，创建一个名为的文件`<app dir>/android/key.properties` ，其中包含对密钥库的引用：

```
storePassword=<password from previous step>  //创建密钥库时设置的
keyPassword=<password from previous step>  //创建密钥库时设置的
keyAlias=key
storeFile=<location of the key store file, such as /Users/<user name>/key.jks>  // 上一步创建的keystore文件的路径
```

## 3.在gradle中配置签名


> 警告： 改了gradle之后，需要运行`flutter clean`,更新一下，避免缓存影响

在项目目录下，编辑<app dir>/android/app/build.gradle文件为您的应用配置签名

- 1. 在android代码块之前添加代码：
```
    // 添加前
   android {
      ...
   }
```
```
    // 添加后 使用key.properties文件中的密钥库信息：
   def keystoreProperties = new Properties()
   def keystorePropertiesFile = rootProject.file('key.properties')
   if (keystorePropertiesFile.exists()) {
       keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
   }

   android {
         ...
   }
```


- 2. 在buildTypes代码块之前添加代码：
```
    // 添加前
   buildTypes {
       release {
           signingConfig signingConfigs.debug
       }
   }

```
```
    // 添加后 使用key.properties文件中的密钥库信息
   signingConfigs {
       release {
           keyAlias keystoreProperties['keyAlias']
           keyPassword keystoreProperties['keyPassword']
           storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) :null
           storePassword keystoreProperties['storePassword']
       }
   }
   buildTypes {
       release {
           signingConfig signingConfigs.release
       }
   }

```

## 4.走了上面三步，你在编译的时候就会自动签名啦

```
flutter build apk --split-per-abi

```

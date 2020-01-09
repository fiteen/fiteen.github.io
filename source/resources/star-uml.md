---
title: StarUML Mac 版破解
---

> ⚠️⚠️⚠️ **请支持[正版](http://staruml.io)，仅供技术交流。**

### 破解步骤

1. [下载](https://nodejs.org/en/)并安装 Node.js，如已安装可以跳过。

2. [下载](http://staruml.io/download)并安装 dmg 包。

3. 安装 asar：

   ```
   sudo npm install -g asar
   ```

4. 进入 app.asar 目录：

   ```
   cd /Applications/StarUML.app/Contents/Resources/
   ```

5. 解压 app.asar

   ```
   asar extract app.asar app
   ```

6. 修改源码绕过注册弹窗

   ```
   open app/src/engine/license-manager.js
   ```

   找到 `checkLicenseValidity` 方法，改成下图所示：

   ![](license-manager.png)

7. 重新打包，替换 app.asar

   ```
   asar pack app app.asar
   ```
   

至此，破解完成。
   

   
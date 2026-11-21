---
title: "如何使用git拉取项目并部署到自己的仓库（自存）"
date: 2025-12-14T15:02:06+08:00
draft: false
---

## **拉取项目代码到本地**
### **1. 环境依赖准备**
先看这个项目需要什么环境和依赖，再进行之后的步骤。
### **2. 开始构建**

#### **第一步：克隆代码**
```
git clone 网址
cd 拉取到的文件夹
```
#### **第二步：构建（打包）**
```
npm run build
```
#### **第三步：结果与调试**
构建结果：完成后在dist文件夹中生成网页文件。
本地开发预览：
```
npm run serve
```
此时运行的是**开发模式**，这种模式下代码没有压缩，包含调试信息，确认无误就可以接下来的步骤，把项目部署到自己的仓库了。此处以github为例：
## **把项目部署到自己的仓库**
### **第一步：准备GitHub仓库**
1.登录你的Github。
2.创建一个新的仓库。（不用勾选添加README文件，添加.gitignore，添加许可证，因为是拉取了别人的项目，有现成的文件）。
3.记住你的仓库名字。
### **第二步：在本地终端连接并推送**
现在，回到你的 VS Code 或者命令行窗口（确保路径是在你的项目文件夹根目录下）。

#### 情况 A
如果你以前从来没在这个项目里运行过 git 命令 （或者你不确定有没有，就按这个顺序完整执行一遍）

复制下面的命令，一行一行执行：
```
# 1. 初始化 git 仓库（如果显示 Reinitialized... 也没关系）
git init

# 2. 把所有文件放入暂存区
git add .

# 3. 提交文件到本地仓库
git commit -m "first commit"

# 4. 把分支重命名为 main（GitHub 现在默认叫 main，不叫 master 了）
git branch -M main

# 5. 关联远程仓库
# ⚠️ 把下面的网址换成你刚才在 GitHub 创建完看到的那个 HTTPS 地址
git remote add origin https://github.com/你的用户名/你的仓库名.git

# 6. 推送到 GitHub
git push -u origin main
```
#### 情况 B
执行第 5 步报错 "fatal: remote origin already exists" 这说明你之前可能尝试关联过（也许关联错了地址）。

解决方法：
```
# 先删除旧的关联
git remote remove origin

# 再重新执行关联命令
git remote add origin https://github.com/你的用户名/你的仓库名.git

# 最后推送
git push -u origin main
```
### **第三步：如何修改代码并推送到 GitHub**
Git 的逻辑是：你在本地做的任何操作（删除、修改、新增），都需要“告诉”Git，然后“推送”上去。也就是说，如果你想修改自己的项目，就必须要走三个步骤：
#### 1.把修改“装箱”
```
git add .
```
#### 2.给箱子贴个条（这步最关键，有时候可能会加载的很慢）
```
git commit -m "修改配置文件并删除无用文件"
```
_(执行完这句，你应该会看到终端列出了一堆你修改或删除的文件名)_
#### 3.把箱子发走
```
git push
```
如果执行完之后报错了，你可以先输一句`git status`。如果你看到一堆红色的文件名，说明你可能忘了add或commit。
但还有可能会遇到以下报错：
> $ git push

fatal: unable to access 'https://github.com/你的用户名/你的仓库名.git': Failed to connect to github.com port 443 after 21033 ms: Couldn't connect to server

这就说明是国内的墙发力了（汗），此处只给出你有VPN的情况：

如果你电脑上开着代理软件（比如 Clash, v2ray 等），Git 是不会自动使用这个代理的，你需要手动告诉 Git 去走这个代理通道。

查看你的代理端口号： 打开你的代理软件设置，找到 “端口” 或 “Port”（通常 HTTP/S 端口是 7890，或者是 10809，具体看你的软件）。

在终端设置 Git 代理： 假设你的端口是 7890（如果不是，请把下面的 7890 改成你的端口号），在终端执行这两行命令：
```
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```
然后再试着推送：
```
git push
```


![](https://ipfs.io/ipfs/QmcLY2DBQhrY1Pc2qUinBPRcZTojj4u4y1xiJqKT9Fjdfz)

---
title: Git分支合并详解
---

> “合并前文件还在的，合并后就不见了”、“我遇到 Git 合并的 bug 了” 是两句经常听到的话，但真的是 Git 的 bug 么？或许只是你的预期不对。本文通过讲解三向合并和 Git 的合并策略，step by step 介绍 Git 是怎么做一个合并的，让大家对 Git 的合并结果有一个准确的预期，并且避免发生合并事故。

<a name="965gh"></a>
### **
<a name="nSJh2"></a>
### **故事时间**
在开始正文之前，先来听一下这个故事。<br />如下图，小明从节点 A 拉了一条 dev 分支出来，在节点 B 中新增了一个文件 http.js，并且合并到 master 分支，合并节点为 E。这个时候发现会引起线上 bug，赶紧撤回这个合并，新增一个 revert 节点 E'。过了几天小明继续在 dev 分支上面开发新增了一个文件 main.js，并在这个文件中 import 了 http.js 里面的逻辑，在 dev 分支上面一切运行正常。可当他将此时的 dev 分支合并到 master 时候却发现，**http.js 文件不见了**，导致 main.js 里面的逻辑运行报错了。但这次合并并没有任何冲突。他又得重新做了一下 revert，并且迷茫的怀疑是 Git 的 bug。<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952132-be17f0fc-6299-4f55-9c49-bac3439401af.jpeg#align=left&display=inline&height=441&margin=%5Bobject%20Object%5D&originHeight=347&originWidth=566&size=0&status=done&style=none&width=720)<br />两句经常听到的话：<br />—— ”合并前文件还在的，合并后就不见了“<br />—— ”我遇到 Git 的 bug 了“<br />相信很多同学或多或少在不熟悉 Git 合并策略的时候都会发生过类似上面的事情，明明在合并前文件还在的，为什么合并后文件就不在了么？一度还怀疑是 Git 的 bug。这篇文章的目的就是想跟大家讲清楚 Git 是怎么去合并分支的，以及一些底层的基础概念，从而避免发生如故事中的问题，并对 Git 的合并结果有一个准确的预期。
<a name="fwDFT"></a>
### **
<a name="k4QxW"></a>
### **如何合并两个文件**
在看怎么合并两个分支之前，我们先来看一下怎么合并两个文件，因为两个文件的合并是两个分支合并的基础。<br />大家应该都听说过“三向合并”这个词，不知道大家有没有思考过为什么两个文件的合并需要三向合并，只有二向是否可以自动完成合并。如下图<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952188-5803ae4f-411d-49c7-8921-b0471e119f60.jpeg#align=left&display=inline&height=281&margin=%5Bobject%20Object%5D&originHeight=195&originWidth=500&size=0&status=done&style=none&width=720)<br />
<br />很明显答案是不能，如上图的例子，Git 没法确定这一行代码是我修改的，还是对方修改的，或者之前就没有这行代码，是我们俩同时新增的。此时 Git 没办法帮我们做自动合并。<br />
<br />所以我们需要三向合并，所谓三向合并，就是找到两个文件的一个合并 base，如下图，这样子 Git 就可以很清楚的知道说，对方修改了这一行代码，而我们没有修改，自动帮我们合并这两个文件为 Print("hello")。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952167-905de7b8-e0df-42ce-90fa-f53f9a7b1968.jpeg#align=left&display=inline&height=579&margin=%5Bobject%20Object%5D&originHeight=317&originWidth=394&size=0&status=done&style=none&width=720)<br />
<br />
<br />接下来我们了解一下什么是冲突？冲突简单的来说就是三向合并中的三方都互不相同，即参考合并 base，我们的分支和别人的分支都对同个地方做了修改。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951898-e0b7fb19-cdbc-4b38-9132-8a96890d2d93.jpeg#align=left&display=inline&height=310&margin=%5Bobject%20Object%5D&originHeight=258&originWidth=600&size=0&status=done&style=none&width=720)
<a name="7djk9"></a>
### **
<a name="CtPqA"></a>
### **Git 的合并策略**
![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951898-d37c89f5-1030-4aea-b2b9-03085c138677.jpeg#align=left&display=inline&height=393&margin=%5Bobject%20Object%5D&originHeight=254&originWidth=465&size=0&status=done&style=none&width=720)<br />了解完怎么合并两个文件之后，我们来看一个使用 git merge 来做分支合并。如上图，将 master 分支合并到 feature 分支上，会新增一个 commit 节点来记录这次合并。<br />Git 会有很多合并策略，其中常见的是 Fast-forward、Recursive 、Ours、Theirs、Octopus。下面分别介绍不同合并策略的原理以及应用场景。默认 Git 会帮你自动挑选合适的合并策略，如果你需要强制指定，使用git merge -s <策略名字><br />了解 Git 合并策略的原理可以让你对 Git 的合并结果有一个准确的预期。
<a name="lHTQt"></a>
####
<a name="4Imf2"></a>
#### Fast-forward
![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951887-bdcd5e60-0dfb-45e5-8cf3-a187fcecf865.jpeg#align=left&display=inline&height=371&margin=%5Bobject%20Object%5D&originHeight=330&originWidth=640&size=0&status=done&style=none&width=720)<br />Fast-forward 是最简单的一种合并策略，如上图中将 some feature 分支合并进 master 分支，Git 只需要将 master 分支的指向移动到最后一个 commit 节点上。<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951880-b1ef28ad-a43b-4514-ac7c-5f36b5880516.jpeg#align=left&display=inline&height=381&margin=%5Bobject%20Object%5D&originHeight=339&originWidth=640&size=0&status=done&style=none&width=720)<br />Fast-forward 是 Git 在合并两个没有分叉的分支时的默认行为，如果不想要这种表现，想明确记录下每次的合并，可以使用git merge --no-ff。
<a name="ElugR"></a>
####
<a name="c2qJP"></a>
#### Recursive
Recursive 是 Git 分支合并策略中最重要也是最常用的策略，是 Git 在合并两个有分叉的分支时的默认行为。其算法可以简单描述为：递归寻找路径最短的唯一共同祖先节点，然后以其为 base 节点进行递归三向合并。说起来有点绕，下面通过例子来解释。<br />如下图这种简单的情况，圆圈里面的英文字母为当前 commit 的文件内容，当我们要合并中间两个节点的时候，找到他们的共同祖先节点（左边第一个），接着进行三向合并得到结果为 B。（因为合并的 base 是“A”，下图靠下的分支没有修改内容仍为“A”，下图靠上的分支修改成了“B”，所以合并结果为“B”）。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951881-0d8fa42d-5e5e-48fe-8b96-dccd272823ea.jpeg#align=left&display=inline&height=337&margin=%5Bobject%20Object%5D&originHeight=201&originWidth=430&size=0&status=done&style=none&width=720)<br />
<br />但现实情况总是复杂得多，会出现历史记录链互相交叉等情况，如下图：<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951905-abfae49b-59a5-4b9b-9a8c-3b45bbb7f0f4.jpeg#align=left&display=inline&height=194&margin=%5Bobject%20Object%5D&originHeight=196&originWidth=728&size=0&status=done&style=none&width=720)<br />
<br />
<br />当 Git 在寻找路径最短的共同祖先节点的时候，可以找到两个节点的，如果 Git 选用下图这一个节点，那么 Git 将无法自动的合并。因为根据三向合并，这里是是有冲突的，需要手动解决。（base 为“A“，合并的两个分支内容为”C“和”B“）<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951896-3c63c70c-df8f-4970-8d31-f7c9afffde7e.jpeg#align=left&display=inline&height=192&margin=%5Bobject%20Object%5D&originHeight=194&originWidth=728&size=0&status=done&style=none&width=720)<br />
<br />
<br />而如果 Git 选用的是下图这个节点作为合并的 base 时，根据三向合并，Git 就可以直接自动合并得出结果“C”。（base 为“B“，合并的两个分支内容为”C“和”B“）<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951915-231323ac-73f6-442c-853c-fcbdbac2547b.jpeg#align=left&display=inline&height=192&margin=%5Bobject%20Object%5D&originHeight=194&originWidth=728&size=0&status=done&style=none&width=720)<br />
<br />
<br />作为人类，在这个例子里面我们很自然的就可以看出来合并的结果应该是“C”（如下图，节点 4、5 都已经是“B”了，节点 6 修改成“C”，所以合并的预期为“C”）<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952167-9416f09f-0145-4e61-92e9-e06df32e7ce6.jpeg#align=left&display=inline&height=194&margin=%5Bobject%20Object%5D&originHeight=196&originWidth=728&size=0&status=done&style=none&width=720)<br />
<br />
<br />那怎么保证 Git 能够找到正确的合并 base 节点，尽可能的减少冲突呢？答案就是，Git 在寻找路径最短的共同祖先节点时，如果满足条件的祖先节点不唯一，那么 Git 会继续递归往下寻找直至唯一。还是以刚刚这个例子图解。<br />如下图所示，我们想要合并节点 5 和节点 6，Git 找到路径最短的祖先节点 2 和 3。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952187-d7ce95ed-cceb-4ae6-a939-454f587fc20a.jpeg#align=left&display=inline&height=358&margin=%5Bobject%20Object%5D&originHeight=150&originWidth=302&size=0&status=done&style=none&width=720)<br />
<br />
<br />因为共同祖先节点不唯一，所以 Git 递归以节点 2 和节点 3 为我们要合并的节点，寻找他们的路径最短的共同祖先，找到唯一的节点 1。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951886-bfc8cb75-035c-4065-9fbd-21ecf63c55fc.jpeg#align=left&display=inline&height=261&margin=%5Bobject%20Object%5D&originHeight=150&originWidth=414&size=0&status=done&style=none&width=720)<br />
<br />
<br />接着 Git 以节点 1 为 base，对节点 2 和节点 3 做三向合并，得到一个临时节点，根据三向合并的结果，这个节点的内容为“B”。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952025-82e86752-33cb-4fa7-8824-b5fec84801a1.jpeg#align=left&display=inline&height=261&margin=%5Bobject%20Object%5D&originHeight=150&originWidth=414&size=0&status=done&style=none&width=720)<br />
<br />
<br />再以这个临时节点为 base，对节点 5 和节点 6 做三向合并，得到合并节点 7，根据三向合并的结果，节点 7 的内容为“C”<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952146-0d5e06b5-f526-4cf2-acf0-7e49b20be085.jpeg#align=left&display=inline&height=212&margin=%5Bobject%20Object%5D&originHeight=128&originWidth=435&size=0&status=done&style=none&width=720)<br />
<br />
<br />至此 Git 完成递归合并，自动合并节点 5 和节点 6，结果为“C”，没有冲突。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952199-ede29c8c-bfa8-4813-be3d-dc2ce7439147.jpeg#align=left&display=inline&height=193&margin=%5Bobject%20Object%5D&originHeight=110&originWidth=410&size=0&status=done&style=none&width=720)<br />
<br />
<br />Recursive 策略已经被大量的场景证明它是一个尽量减少冲突的合并策略，我们可以看到有趣的一点是，对于两个合并分支的中间节点（如上图节点 4，5），只参与了 base 的计算，而最终真正被三向合并拿来做合并的节点，只包括末端以及 base 节点。<br />
<br />需要注意 Git 只是使用这些策略尽量的去帮你减少冲突，如果冲突不可避免，那 Git 就会提示冲突，需要手工解决。（也就是真正意义上的冲突）。
<a name="QUaOp"></a>
####
<a name="VxV1x"></a>
#### Ours & Theirs
Ours 和 Theirs 这两种合并策略也是比较简单的，简单来说就是保留双方的历史记录，但完全忽略掉这一方的文件变更。如下图在 master 分支里面执行git merge -s ours dev，会产生蓝色的这一个合并节点，其内容跟其上一个节点（master 分支方向上的）完全一样，即 master 分支合并前后项目文件没有任何变动。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952013-bd1ad94f-3fbd-4ddb-b1a0-c6f03d0fe803.jpeg#align=left&display=inline&height=190&margin=%5Bobject%20Object%5D&originHeight=150&originWidth=568&size=0&status=done&style=none&width=720)<br />
<br />
<br />而如果使用 theirs 则完全相反，完全抛弃掉当前分支的文件内容，直接采用对方分支的文件内容。<br />这两种策略的一个使用场景是比如现在要实现同一功能，你同时尝试了两个方案，分别在分支是 dev1 和 dev2 上，最后经过测试你选用了 dev2 这个方案。但你不想丢弃 dev1 的这样一个尝试，希望把它合入主干方便后期查看，这个时候你就可以在 dev2 分支中执行git merge -s ours dev1。<br />

<a name="3gemz"></a>
#### Octopus
这种合并策略比较神奇，一般来说我们的合并节点都只有两个 parent（即合并两条分支），而这种合并策略可以做两个以上分支的合并，这也是 git merge 两个以上分支时的默认行为。比如在 dev1 分支上执行git merge dev2 dev3。<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951920-93866877-5beb-4e70-a84c-ec0b501563ed.jpeg#align=left&display=inline&height=285&margin=%5Bobject%20Object%5D&originHeight=247&originWidth=623&size=0&status=done&style=none&width=720)<br />
<br />他的一个使用场景是在测试环境或预发布环境，你需要将多个开发分支修改的内容合并在一起，如果不用这个策略，你每次只能合并一个分支，这样就会导致大量的合并节点产生。而使用 Octopus 这种合并策略就可以用一个合并节点将他们全部合并进来。
<a name="HX9Px"></a>
### **
<a name="OzwAj"></a>
### **Git rebase**
git rebase 也是一种经常被用来做合并的方法，其与 git merge 的最大区别是，他会更改变更历史对应的 commit 节点。<br />
<br />如下图，当在 feature 分支中执行 rebase master 时，Git 会以 master 分支对应的 commit 节点为起点，新增两个**全新**的 commit 代替 feature 分支中的 commit 节点。其原因是新的 commit 指向的 parent 变了，所以对应的 SHA1 值也会改变，所以没办法复用原 feature 分支中的 commit。（这句话的理解需要**这篇文章**的基础知识）<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687951891-5cfcdece-c0ed-447f-90f3-19fc6993bdd6.jpeg#align=left&display=inline&height=490&margin=%5Bobject%20Object%5D&originHeight=350&originWidth=514&size=0&status=done&style=none&width=720)<br />
<br />对于合并时候要使用 git merge 还是 git rebase 的争论，我个人的看法是没有银弹，根据团队和项目习惯选择就可以。git rebase 可以给我们带来清晰的历史记录，git merge 可以保留真实的提交时间等信息，并且不容易出问题，处理冲突也比较方便。唯一有一点需要注意的是，不要对已经处于远端的多人共用分支做 rebase 操作。<br />我个人的一个习惯是：对于本地的分支或者确定只有一个人使用的远端分支用 rebase，其余情况用 merge。<br />
<br />rebase 还有一个非常好用的东西叫 interactive 模式，使用方法是git rebase -i。可以实现压缩几个 commit，修改 commit 信息，抛弃某个 commit 等功能。比如说我要压缩下图 260a12a5、956e1d18，将他们与 9dae0027 合并为一个 commit，我只需将 260a12a5、956e1d18 前面的 pick 改成“s”，然后保存就可以了。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952141-4e076938-58c0-4a92-9721-5de79919d8a8.jpeg#align=left&display=inline&height=493&margin=%5Bobject%20Object%5D&originHeight=374&originWidth=546&size=0&status=done&style=none&width=720)<br />
<br />限于篇幅，git rebase -i 还有很多实用的功能暂不展开，感兴趣的同学可以自己研究一下。
<a name="VW6pF"></a>
### **
<a name="U7mRA"></a>
### **总结**
![](https://cdn.nlark.com/yuque/0/2020/jpeg/203222/1608687952146-88215c35-62cd-4d79-8b32-4553b2b5d511.jpeg#align=left&display=inline&height=441&margin=%5Bobject%20Object%5D&originHeight=347&originWidth=566&size=0&status=done&style=none&width=720)<br />现在我们再来看一下文章开头的例子，我们就可以理解为什么最后一次 merge 会导致 http.js 文件不见了。根据 Git 的合并策略，在合并两个有分叉的分支（上图中的 D、E‘）时，Git 默认会选择 Recursive 策略。找到 D 和 E’的最短路径共同祖先节点 B，以 B 为 base，对 D，E‘做三向合并。B 中有 http.js，D 中有 http.js 和 main.js，E’中什么都没有。根据三向合并，B、D 中都有 http.js 且没有变更，E‘删除了 http.js，所以合并结果就是没有 http.js，没有冲突，所以 http.js 文件不见了。<br />
<br />这个例子理解原理之后解决方法有很多，这里简单带过两个方法：1. revert 节点 E'之后，此时的 dev 分支要抛弃删除掉，重新从 E'节点拉出分支继续工作，而不是在原 dev 分支上继续开发节点 D；2. 在节点 D 合并回 E’节点时，先 revert 一下 E‘节点生成 E’‘（即 revert 的 revert），再将节点 D 合并进来。<br />
<br />Git 有很多种分支合并策略，本文介绍了 Fast-forward、Recursive、Ours/Theirs、Octopus 合并策略以及三向合并。掌握这些合并策略以及他们的使用场景可以让你避免发生一些合并问题，并对合并结果有一个准确的预期。<br />希望这篇文章对大家有用，感兴趣的同学可以逛一逛我的博客www.lzane.com 或看看我的其他文章。<br />
<br />**参考**

- 三向合并 [http://blog.plasticscm.com/2016/02/three-way-merging-look-under-hood.html](http://blog.plasticscm.com/2016/02/three-way-merging-look-under-hood.html)
- Recursive 合并【视频】[https://www.youtube.com/watch?v=Lg1igaCtAck](https://www.youtube.com/watch?v=Lg1igaCtAck)
- 书籍 **Scott Chacon, Ben Straub - Pro Git-Apress (2014)**
- 书籍 **Jon Loeliger, Matthew McCullough - Version Control with Git, 2nd Edition - O’Reilly Media (2012)**


<br />原文 [https://www.jiqizhixin.com/articles/2020-05-28-8](https://www.jiqizhixin.com/articles/2020-05-28-8)

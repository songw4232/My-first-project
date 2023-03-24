#Github

[向 RT-Thread 贡献代码](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/development-guide/github/github)  
还写啥笔记啊！！这个RT-Thread官方的文档要多详细就有多详细！！直接去官网文档看去
###GitHub 的 Pull Request 是指什么意思？[知识点链接](https://www.zhihu.com/question/21682976/answer/79489643)


我尝试用类比的方法来解释一下 pull reqeust。想想我们中学考试，老师改卷的场景吧。你做的试卷就像仓库，你的试卷肯定会有很多错误，就相当于程序里的 bug。老师把你的试卷拿过来，相当于先 fork。在你的卷子上做一些修改批注，相当于 git commit。最后把改好的试卷给你，相当于发 pull request，你拿到试卷重新改正错误，相当于 merge。  
当你想更正别人仓库里的错误时，要走一个流程：  
1.先 fork 别人的仓库，相当于拷贝一份，相信我，不会有人直接让你改修原仓库的  
2.clone 到本地分支，做一些 bug fix  
3.发起 pull request 给原仓库，让他看到你修改的 bug  
4.原仓库 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中至此，整个 pull request 的过程就结束了。

###格式：git clone https://github.com/RT-Thread/rt-thread/


###fork
简单的说明：GitHub中Fork 是 服务端的代码仓库克隆（即 新克隆出来的代码仓库在远程服务端），可以包含了原来的仓库（即upstream repository，上游仓库）所有内容，如分支、Tag、提交。代码托管服务（如Github、BitBucket）都提供了方便完成Fork操作的功能（在仓库页面点一下Fork按钮）。这样有了一个你自己的 可以自由提交的远程仓库，然后可以通过的 Pull Request 把你的提交贡献回 原仓库。而对于原仓库Owner来说，鼓励别人Fork他的仓库，通过Pull Request 能给他的仓库做贡献，也是提升了原仓库的知名度。  

当你想更正别人仓库里的错误时，要按照下面的流程进行：   

先 fork 别人的仓库，相当于拷贝一份别人的资料。因为不能保证你的修改一定是正确的，对项目有利的，所以你不能直接在别人的仓库里修改，而是要先fork到自己的git仓库中。  
clone 到自己的本地分支，做一些 bug fix，然后发起 pull request给原仓库，让原仓库的管理者看到你提交的修改。  
原仓库的管理者 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中。merge 的意思就是合并，将你修改的这部分代码合并到原来的仓库中添加代码或者替换掉原来的代码。至此，整个 Pull Request 的过程就结束了。

##torotiseGit的下载安装和使用
官网下载TGit就行，还有中文包，安装一路next
###使用！重点
现在可能会，一段时间不用又不会了！！这个软件是结合Git一起使用的，就是Git的一种图形化软件，后面会仔细说说这个软件结合Git的用法！！

管理Git的仓库很好的工具

###第一步使用这个方法在本地克隆一个Git仓库 
 
	格式：git clone https://github.com/RT-Thread/rt-thread/
后面有SSH密钥了，不这样下载克隆Git仓库了！！  
上面的克隆就是下载，这一步，就是使我们的磁盘上有一个仓库  

###第二步就是上传Code到我们的GitHub仓库里面  
这里使用SSH密钥，方法：  
使用tortoisegit软件实现产生密钥，在GitHub网站上添加。具体如下：
使用tortoisegit软件产生的子软件PuTTYgen软件生成密钥。将密钥复制到GitHub网站SSH中。
还要保存Save private key,这个文件非常重要。使用Git时需要添加，同一个密钥，在不同电脑上是可以运行的，只要是同一个账号，自己添加Git远端的相关设置就行。  
![](\Picture\tu1.png)
注：上面这个图片必须这样设置！！我在选择实际生效选项时，并不能将本地Code发送到GitHub仓库！！

###第三步如何发送，怎样设置Git和tortoiseGit
![](\Picture\tu2.png)
注意：必须在本地仓库代码中右键，点击TGit-->设置，Git下面的远端才会出现！！  

PuTTY密钥！！生成的那个文件！！！

###从本地上传代码到Git步骤
####修改的Code先到TGit-->添加；然后Git提交（写好修改日志后才能提交）；最后TGit推送！！！






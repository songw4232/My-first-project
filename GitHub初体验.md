#Github



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




Hifive  -->   sifive  
[Kendryte]  -->   [嘉楠Kendryte]  
[Kendryte]  -->[Canaan Kendryte]  
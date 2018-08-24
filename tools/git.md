安装
---
1. Linux

    Debian、Ubuntu :

        sudo apt-get install git
        
    Old Debian、Ubuntu :
    
        sudo apt-get install git-core
        
    Others : need to down load source code.
    
        ./config
        make
        sudo make install

2. Mac OS X
    * homebrew
    * Xcode : Preferences -> Downloads -> Command Line Tools

Setting : 

        git config --global user.name "Your Name"
        git config --global user.email "email@example.com"

Git基本命令
---

        git init                    //初始化git仓库
        git status                  //仓库当前状态
        git diff file1              //查看修改内容
        git add file1 file2         //添加文件到暂存区（stage）
        git commit -m "message"     //把暂存区的所有内容提交到当前（master）分支（保存快照）
        
版本回退
---
Git是**分布式**的版本控制系统，因此 commit id 是十六进制表示的SHA1计算出的大数字，而不是1,2,3，否则多人协作时候会冲突。

        git log                     //显示从最近到最远的提交日志（当前版本:HEAD）
        git log --pretty=oneline    //精简显示提交日志
        git reset --hard HEAD^      //回退到（上个版本:HEAD^、上上个版本:HEAD^^、上100个版本:HEAD~100）
        git log                     //回退后之后的版本看不到了，找到想前进的版本号可以回到未来的某个版本。
        git reset --hard id         //版本号id可以只写前几位，git会自动去找。
        git reflog                  //若找不到未来版本号。通过记录每一次命令的reflog找到未来版本号。

管理修改
---

git跟踪并管理的是修改，而非文件。  
第一次修改 -> git add -> 第二次修改 -> git commit（第一次的修改被提交了，第二次的修改不会被提交）
每次修改，如果不用git add到暂存区，那就不会加入到commit中。

        git diff HEAD -- file       //查看工作区和版本库里面最新版本的区别：

撤销修改
---
git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。用HEAD时，表示最新的版本。

        git checkout -- file            //丢弃工作区的修改（回到最近一次git commit或git add时的状态）
        git reset HEAD <file>           //把暂存区的修改撤销掉（unstage），重新放回工作区。（现在暂存区是干净的，工作区有修改，再运行上条命令丢弃工作区的修改）

删除文件
---

        git rm file                 //从版本库中删除该文件，再git commit
        git checkout -- file        //删错了恢复。git checkout是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

远程仓库
---
先有本地库，后有远程库：

        git remote add origin git@server-name:path/repo-name.git       //远程库的名字就是origin（如果关联多个远程库，起不同名字）
        git push -u origin master       //把本地库的所有内容（当前分支master）推送到远程库（第一次需要有 -u 进行关联，之后可省略 -u）
        
先有远程库，本地从远程库克隆：

        git clone git@server-name:path/repo-name.git                //使用ssh协议克隆一个本地库（速度快）
        git clone https://server-name/path/repo-name.git            //使用https协议（速度慢，每次推送必须输入口令）

分支管理、解决冲突
---

        git checkout -b dev                 //-b : 创建dev分支，并切换到dev分支。等价于一下两条命令：
        git branch dev                      //创建dev分支
        git checkout dev                    //切换到dev分支
        git branch                          //查看当前分支（列出所有分支，当前分支前面会标一个*号）
        git checkout master                 //切换到master分支
        git merge dev                       //切换到master分支后，把dev分支合并到master分支上（合并指定分支到当前分支）（Fast-forward, 快进模式，删除分支后，会丢掉分支信息）
        git merge --no-ff -m "msg" dev      //禁用Fast forward时，生成一个新的commit
        git status                          //若有冲突，查看冲突的文件。修改冲突文件后add、commit。
        git log --graph --pretty=oneline --abbrev-commit                //查看分支合并图。
        git branch -d dev                   //删除dev分支
        git branch -D branchname            //若分支没有合并，强制丢弃分支

保存现场stash
---

        git stash                           //保存当前工作现场，先去处理紧急bug，等回复后继续工作
        git checkout master                 //假设bug在master分支上
        git checkout -b issue-101           //从master创建临时分支issue-101。修复bug，git add，git commit
        git checkout master                 //再切换到master
        git merge --no-ff -m "msg" issue-101            //合并分支。删除分支
        git checkout dev                    //回到原来分支继续干活
        git stash list                      //查看已保存的工作现场
        git stash apply                     //恢复现场，但stash内容不删除。有多次stash时，恢复指定的stash最后加上stash@{0}
        git stash drop                      //删除stash
        git stash pop                       //恢复现场，同时删除stash内容

多人协作
---
从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin。

        git remote                          //查看远程库信息
        git remote -v                       //显示更详细信息
        git push origin master              //推送分支，把该分支上的所有本地提交推送到远程库。如果要推送其他分支，修改master。
        git remote rm origin                //删除已有的远程库关联
        
* master分支是主分支，因此要时刻与远程同步；
* dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
* bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
* feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。


        git clone git@server-name:path/repo-name.git                    //默认抓取master分支
        git checkout -b branch-name origin/branch-name                  //创建远程origin的dev分支到本地
        git push origin branch-name                                     //把本地修改的dev分支push到远程。若推送有冲突，需把远程新提交pull下来在本地合并后再推送
        git branch --set-upstream-to=origin/branch-name branch-name     //设置本地dev分支与远程origin/dev分支的链接
        
变基Rebase
---
把本地未push的分叉提交历史整理成直线

        git checkout experiment                 //当前分支（需合并到master）
        git rebase master                       //变基操作的目标基底分支master
        git checkout master
        git merge experiment                    //合并分支
        
标签管理
---
标签是版本库的快照，也是指向某个commit的指针（与分支不同的是，分支可以移动，标签不能移动）。
标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。

        git checkout master                     //切换到需要打标签的分支上
        git tag tag-name                        //打标签，如v1.0。默认标签是打在最新提交的commit上的
        git tag tag-name commit-id              //对历史某次提交打标签
        git tag -a tag-name -m "msg" commit-id  //创建带有说明的标签。-a指定标签名，-m指定说明文字
        git tag                                 //查看所有标签（不是按时间顺序，而是按字母排序）
        git show tag-name                       //查看标签信息
        git tag -d tag-name                     //删除本地标签
        git push origin :refs/tags/tag-name     //删除远程标签（需先删除本地标签）
        git push origin tag-name                //推送某个标签到远程
        git push origin --tags                  //推送全部尚未推送的标签到远程
        
自定义git
---

        git config --global color.ui true                   //显示字体颜色
        git check-ignore -v file-name                       //查找file-name由于哪个规则被忽略的
        git config --global alias.st status                 //配置别名，以后st就表示status
        git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
        
忽略文件的原则：(<https://github.com/github/gitignore>)
* 忽略操作系统自动生成的文件，比如缩略图等；
* 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；
* 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

配置Git的时候，加上--global是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。  
每个仓库的Git配置文件都放在.git/config文件中，要删除别名，直接把对应的行删掉即可。  
当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中。

搭建Git服务器
---

        sudo apt-get install git                    //第一步，安装git
        sudo adduser git                            //第二步，创建一个git用户，用来运行git服务
        /home/git/.ssh/authorized_keys              //第三步，创建证书登录：一行一个id_rsa.pub存储
        sudo git init --bare sample.git             //第四步，初始化Git仓库（裸仓库，没有工作区）
        sudo chown -R git:git sample.git            //        把owner改为git
        /etc/passwd                                 //第五步，禁用shell登录：git:x:1001:1001:,,,:/home/git:/bin/bash 改为 git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
        
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

        git clone git@server:/srv/sample.git        //第六步，克隆远程仓库
        
管理公钥：Gitosis <https://github.com/res0nat0r/gitosis>  
管理权限：Gitolite <https://github.com/sitaramc/gitolite>

用在hexosrc分支上的说明，用于管理博客源文件。

# 总体原则
github pages默认反映repo的master分支内容，这也是hexo最后生成的结果。因此用master分支跟踪hexo的输出，另用一个分支hexosrc跟踪源文件、主题、配置等。

# 建站过程
1. 在github建立`silaoA.github.io`的repo，不添加任何文件；
2. 本地配置生成好ssh key，复制一份public key到github账户[设置](https://github.com/settings/keys)中，如果已在github账户部署好public key，本步跳过；

``` bash
ssh-keygen -t rsa -C <Email> # "stsilaoa@gmail.com" 
cat ~/.ssh/id_rsa.pub # 查看public key，复制内容到ithub账户设置页
ssh -T git@github.com # 测试ssh认证
```

3. 使用git协议克隆repo到本地 
``` bash
git clone git@github.com:silaoA/silaoA.github.io.git
```

4. 进入本地repo，并将.git移到别处，保证暂时是空目录
``` bash
cd silaoA.github.io
mv .git <path_elsewhere>
```

5. hexo初始化并安装必要软件包，写好配置、博客源文件 
``` bash
hexo init
mv .git <path_elsewhere>
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant #安装maupassant-hexo主题
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass --save
npm install hexo-deployer-git --save
npm install hexo-hey --save
```

6. 本地生成web页面并预览，确认无误后则部署至github
``` bash
hexo s -g  # 本地生成web页面并预览
hexo d -g  # 本地生成web页面并部署至远端`silaoA.github.io`的master分支
```

7. 切换至hexosrc分支，并追踪以下文件，确认无误后提交并推送至远端`silaoA.github.io`的hexosrc分支
   
   + .gitignore
   + README.md
   + _config.yml
   + source/
   + themes/
   
``` bash
mv <path_elsewhere>/.git ./ #把刚才移出去的.git再放进来
git branch #查看本地repo的分支
git remote -v #查看repo远程主机信息，可以看到只有名为origin的主机
git checkout -b hexosrc #创建并切换至hexosrc分支
git add . #编辑完源文件后加入追踪，其中.gitignore写明忽略追踪文件的规则；也可以手动逐个文件（夹）添加，themes下maupassant中存在.git目录，即仓库中有子仓库，git add会失败，可以删除maupassant中的.git目录再添加，**注意**不是`silaoA.github.io`下的.git
git status #确认下repo的状态
git commit -m "hexosrc分支首次提交，追踪源文件、theme和配置" 
git push origin hexosrc:hexosrc #推送本地hexosrc内容至远程主机的hexosrc分支
```

# 在其他机器上工作
在多台机器上协作，最简单直接的办法就是把整个`silaoA.github.io`复制到另一台机器上，编辑修改文档之后提交并推送到远程主机的hexosrc分支。远程主机repo更新了，在任一机器上，直接拉取即可实现同步更新。

若是现实环境没法直接复制整个repo，那就按照上述2~7的步骤，拼凑一个本地repo来，但注意以下事项：
+ 本次克隆第2步所指远端`silaoA.github.io`的**hexosrc分支**，如果在工作机器上已经有克隆过repo，那么只需要拉取远端`silaoA.github.io`的**hexosrc分支**的最新内容即可；

``` bash
git clone -b hexosrc git@github.com:silaoA/silaoA.github.io.git # 克隆hexosrc分支时会自动进入hexosrc分支
```
or
``` bash
git checkout hexosrc && git pull origin hexosrc #切换到hexosrc分支再拉取更新
```
+ 第4步要把repo中**所有内容**暂移别处，不仅仅是.git；
``` bash
mv ./ <path_elsewhere>
```

+ 第5步不需要再安装maupassant-hexo主题，因克隆repo时已包含；
+ 第7步不仅仅是将暂移别处的.git目录放回来，而是**所有内容**；

``` bash
mv <path_elsewhere> ./ 
``` 

+ 第7步编辑修改文档之后，再回到第6步预览，直至确认无误再提交和推送。

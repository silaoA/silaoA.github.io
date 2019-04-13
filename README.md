用在hexosrc分支上的说明，用于管理博客源文件。

# 总体原则
github pages默认反映repo的master分支内容，这也是hexo最后生成的结果。因此用master分支跟踪hexo的输出，另用一个分支hexosrc跟踪源文件、主题、配置等。

# 建站过程
1. 在github建立`silaoA.github.io`的repo，不添加任何文件；
2. 本地配置生成好ssh key，复制一份public key到github账户[设置](https://github.com/settings/keys)中，如果已在github账户部署好public key，本步跳过；
  ``` shell
  ssh-keygen -t rsa -C <Email> # "stsilaoa@gmail.com" 
  cat ~/.ssh/id_rsa.pub # 查看public key，复制内容到ithub账户设置页
  ssh -T git@github.com # 测试ssh认证
  ```
3. 使用ssh协议克隆repo到本地 
   ``` shell
   git clone git@github.com:silaoA/silaoA.github.io.git
   ```
4. 进入本地repo，并将.git移到别处，保证暂时是空目录
   ``` shell
   cd silaoA.github.io
   mv .git <path_elsewhere>
   ```
5. hexo初始化并安装必要软件包，写好配置、博客源文件 
   ``` shell
   hexo init
   mv .git <path_elsewhere>
   git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant #安装maupassant-hexo主题
   npm install hexo-renderer-pug --save
   npm install hexo-renderer-sass --save
   npm install hexo-deployer-git --save
   npm install hexo-hey --save
   ```
6. 本地生成web页面并预览，确认无误后则部署至github
   ``` shell
   hexo s -g  # 本地生成web页面并预览
   hexo d -g  # 本地生成web页面并部署至github
   ```
7. 切换至hexosrc分支，并追踪以下文件，确认无误后提交并推送至远端`silaoA.github.io`的hexosrc分支
   
   + .gitignore
   + README.md
   + _config.yml
   + source/
   + themes/
   
   ``` shell
   mv <path_elsewhere>/.git ./ #把刚才移出去的.git再放进来
   git remote -v查看远程repo
   git commit -m "hexosrc分支首次提交，追踪源文件、theme和配置"
   git push origin hexosrc:hexosrc
   ```

# 在其他机器上工作

重复前一节2~7过程，但注意以下事项：
+ 本次克隆第2步所指远端`silaoA.github.io`的hexosrc分支
   ``` shell
   git clone -b hexosrc git@github.com:silaoA/silaoA.github.io.git
   ```
+ 第4步要把repo中**所有内容**暂移别处，不仅仅是.git；
   ``` shell
   mv ./ <path_elsewhere>
   ```
+ 第5步不需要再安装maupassant-hexo主题，因repo中已包含；
+ 第7步不仅仅是将暂移别处的.git目录放回来，而是**所有内容**；
   ``` shell
   mv <path_elsewhere> ./ 
   ``` 
+ 第7步编写修改博客后，再回到第6步预览，直至确认无误；

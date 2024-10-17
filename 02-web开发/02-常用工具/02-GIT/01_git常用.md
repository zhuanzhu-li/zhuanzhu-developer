* **不同分支，合并某个提交**
    参考：[cherry-pick](https://blog.csdn.net/weixin_44792849/article/details/125807934)
    git  cherry-pick commit_hash(IEDA git 插件可以使用界面操作)

* 忽略本地提交，同步为远程分支

  ~~~ sh
  git reset --hard origin/master
  ~~~

* 通过tag创建分支

  ~~~ shell
  git branch {new_branch_name} {tag_name}
  ~~~

* 批量修改提交人以及email

  ~~~shell
  git filter-branch --env-filter '
  
  OLD_EMAIL="278956314@qq.com"
  OLD_NAME="liwenqiKeep"
  CORRECT_NAME="10244"
  CORRECT_EMAIL="li.wenqi@datasw.com"
  
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
  
  then
  
      export GIT_COMMITTER_NAME="$CORRECT_NAME"
      export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
  
  fi
  
  if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
  
  then
  
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  
  fi
  
  if [ "$GIT_AUTHOR_NAME" = "$OLD_NAME" ]
  
  then
  
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  
  fi
  
  ' --tag-name-filter cat -- --branches --tags
  ~~~

  
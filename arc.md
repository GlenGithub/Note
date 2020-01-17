5、代码审核-preview流程
以下为先审核再提交的preview流程:

a.) 本地提交代码
$ git add .
$ git ci -am '* [cps-bk] 提交信息'
b.) 提交审核
运行arc diff，弹出vi编辑窗口里Reviewers项添加审核人，多个审核人中间用逗号隔开，保存退出，生成一个审核url，拷贝发给审核人或让审核人查收邮件，完成审核

$ arc diff
若审核未通过需要修改，或之前忘记了啥，想再修改修改，可以先按步骤a.)本地提交好，然后运行

$ arc diff --update D(n)    # 更新之前那条审核，再次发起提交
c.) 审核通过后，运行以下命令完成提交
$ arc land   # 已包括git push的动作，所以无需再提交了
6、  使用Arcanist提交Revision
　　$ arc help                      # 获得arc中包装的可用指令/工具
   $ arc diff                      # 提交代码去审核
　　$ arc diff --update D(n)        # 审核未通过，修改后，再次提交审核
　　$ arc diff --create             # 创建一个新的提交审核
　　$ arc land                      # 审核通过后提交，已包括git push的动作，所以无需再push了
　　$ arc amend                     # 审核Git更新提交后的信息
　　$ arc list                      # 显示未提交修改的代码信息
　　$ arc lint                      # 检查代码的语法
　　$ arc get-config                # 查看已设置过的配置
　　$ arc set-config <key> <value>  # 修改配置，使用--local参数为全局配置
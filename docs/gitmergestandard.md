# git 分支合并规范

||master||
|------|------|------|
|hotfix_2.15.1(组长拥有代码合并权限)(修复上个迭代的线上bug，当有需要发布时新建) ||release(系统测试)(唯一)(组长拥有代码合并权限)(测试人员在这里测试)|
|wzy_hotfix_2.15.1(个人hot分支，修复线上bug)(发布完及时合并迭代分支)||moduleTest(集成测试)(唯一)(组长拥有代码合并权限)(测试人员在这里测试)|
|||delivery_OMSWebOnline_2.16(开发人员拥有合并代码权限)(开发自测环境，没有问题再提交到集成环境)(每次迭代发布后，需要由组长新建下个迭代分支)|
|||wzy_delivery_2.16(根据迭代分支新建个人开发分支)|
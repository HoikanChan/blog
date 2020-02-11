## Linux -  Chmod

means : change mode 

用于修改权限

Linux权限会根据三种角色进行区分

- u(author)  作者
- g(group)   小组成员
- o(others)  其他用户

每种角色的权限又有三种

- r(read)读
- w(write)写
- x(execute)执行

语法：

chmod [mode] [filename]

其中mode有两种写法

1. ugo+-rwx：即权限组 +(增加)或-(减少) 权限，e.g. 给作者权限增加执行权限即为 chmod u+x [filename]，给所有人去掉读取权限即为chomd ugo-r [filename]
2. 二进制写法：将rwx当作一个二进制数，e.g. rwx都有权限即为111，等于十进制的7，只是可读即为010，十进制为4。将三种角色的权限凑到一起既可以组成我们的权限mode，e.g. chmod  777 [filename]为所有人都拥有一切权限，chmod  740 [filename]意思是作者有一切权限，小组内只可读，其他人没权限
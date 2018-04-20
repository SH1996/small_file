chapter01.print informations of file
    
    1.stat 文件
    2.ll 
    3.cat | tac |   参数 More | less

chapter02.stat—parameters

    1.stat -f 显示文件在系统中的信息
    3.stat -t 显示简短信息

chapter03.stat-info

    1.Access:最后访问文件时间
    2.Modify:最后修改文件内容时间
    3.Change:最后修改文件属性时间

    test：
        1.cat后文件access时间改变
        2.echo后modify和change改变
        3.chmod后change改变
        
    修改这三个属性时间；
        1.touch -a 2000-10-10 文件名
        2.touch -m 文件名  ------------默认就是添加系统当前时间
        3.touch -d  文件名  --------修改access和modify的时间
    
chapter04.chmod——operations：

      1.chmod是对文件进行权限设置的命令
      2.对文件有三个权限：可读，可写，写执行，但是一般用数字表示分别为4，2，1， 那么7就表示三个权限都有
      3.对文件的权限还分为root拥有者，群组，其他人，比如：chmod 764 文件名，表示：文件拥有者三着，群组用户不可以执行，其他用户只能读

chapter05.print file——chmod：  ls -l  ------------在目录之下输出全部权限信息


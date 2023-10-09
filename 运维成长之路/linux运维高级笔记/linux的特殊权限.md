三种特殊权限

SUID SGID STICKY

000 对管理员不受限

安全上下文：

​	进程是以某用户的身份运行，是以此用户的身份和权限，shell的子进程

​	进程的用户对访问文件属主权限、属组权限、其他权限



chmod u+s

拥有SUID 的权限，运行的时候是以文件属主的身份运行

其他是以进程的用户运行

-rwsr-xr-x. 1 **root** **root** 27856 Apr  1  2020 /bin/passwd

比如 passwd 命令，其他用户也能使用这个命令，这个命令是root权限



SGID 

​	功能： 目录属组有写权限，且有SGID权限

在该目录下，新建的文件属组都是这个属组身份

chmod g+s

test 目录下，创建的属组都是这个组



STICKY

​	chmod o+t

每个用户都能创建文件，只能删自己的文件

比如/tmp目录



0777 1777  2777 4777



facl 

文件的额外赋权机制

在u,g,o之外 ，让普通用户能控制赋权给其他用户或组

getfacl 

setfacl  -m u:user1:rw file1

setfacl -m g:group1:rw file1



撤销赋权 -x



隐藏权限，防止重要文件被删除

chattr  +i

​			+a

lsattr 
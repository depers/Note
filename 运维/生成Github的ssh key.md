# 生成Github的ssh key

1. 检查本地主机是否已经存在ssh key

    ```
    cd ~/.ssh
    ls
    //看是否存在 id_rsa 和 id_rsa.pub文件，如果存在，说明已经有SSH Key
    ```

2. 如果不存在ssh key，使用如下命令生成`ssh-keygen -t rsa -C "xxx@xxx.com"`，接着一直回车，然后就会生成两个秘钥文件，一个是id_rsa、一个是id_rsa.pub。

    * `-t` ：指定密钥类型，默认是 rsa ，可以省略。

    * `-C` ：设置注释文字，比如邮箱。

3. 然后去~/.ssh目录下拷贝 id_rsa.pub 文件的内容，在Github的
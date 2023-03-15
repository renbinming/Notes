
## SFTP/FTP sync插件
ctrl + shift + p
调出 sftp:config 配置
```json
{

    "name": "My Server",

    "host": "192.168.1.165",

    "protocol": "sftp",

    "port": 22,

    "username": "root",

    "privateKeyPath": "~/.id_rsa",

    "passphrase": "password",

    "remotePath": "/",

    "uploadOnSave": false,

    "useTempFile": false,

    "openSsh": false

}
```
## python 插件
配置自动补全

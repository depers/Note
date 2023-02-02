github相关api调试



1. 获取仓库的内容
    * 接口地址：/repos/{owner}/{repo}/contents/{path}
    
    * 头信息：Authorization：Bearer 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
    
    * 注意：
        1. path若是文件路径，则会返回文件的内容，文件内容是以base64进行编码的。
        1. path若是目录路径，则会返回目录的相关信息。
        
    * 问题：
        1. 我的文章页面涉及创建时间和标签，这个该怎么处理呢？github返回的报文中，没有这两个字段。这个问题其实也好解决，我通过目录名称就可以解决这个问题了。文章的目录就是**年-月-日-标签-文章内容**
        
    * 根目录响应
        
        ```json
        [
            {
              "name": ".gitignore",
              "path": ".gitignore",
              "sha": "6d19323ee26946ed98b66ac81abc6abeaa0c4d32",
              "size": 478,
              "url": "https://api.github.com/repos/depers/jasper-db/contents/.gitignore?ref=main",
              "html_url": "https://github.com/depers/jasper-db/blob/main/.gitignore",
              "git_url": "https://api.github.com/repos/depers/jasper-db/git/blobs/6d19323ee26946ed98b66ac81abc6abeaa0c4d32",
              "download_url": "https://raw.githubusercontent.com/depers/jasper-db/main/.gitignore?token=ADZ4M5IQ455MVF6IGS3HJYTD3IMNK",
              "type": "file",
              "_links": {
                "self": "https://api.github.com/repos/depers/jasper-db/contents/.gitignore?ref=main",
                "git": "https://api.github.com/repos/depers/jasper-db/git/blobs/6d19323ee26946ed98b66ac81abc6abeaa0c4d32",
                "html": "https://github.com/depers/jasper-db/blob/main/.gitignore"
              }
            },
            {
              "name": "2023",
              "path": "2023",
              "sha": "770c076b024482811eba43726762044b0f9fb362",
              "size": 0,
              "url": "https://api.github.com/repos/depers/jasper-db/contents/2023?ref=main",
              "html_url": "https://github.com/depers/jasper-db/tree/main/2023",
              "git_url": "https://api.github.com/repos/depers/jasper-db/git/trees/770c076b024482811eba43726762044b0f9fb362",
              "download_url": null,
              "type": "dir",
              "_links": {
                "self": "https://api.github.com/repos/depers/jasper-db/contents/2023?ref=main",
                "git": "https://api.github.com/repos/depers/jasper-db/git/trees/770c076b024482811eba43726762044b0f9fb362",
                "html": "https://github.com/depers/jasper-db/tree/main/2023"
              }
            },
            {
              "name": "README.md",
              "path": "README.md",
              "sha": "6bb4003f508d5301502b58abd4c1c8d70bba810d",
              "size": 12,
              "url": "https://api.github.com/repos/depers/jasper-db/contents/README.md?ref=main",
              "html_url": "https://github.com/depers/jasper-db/blob/main/README.md",
              "git_url": "https://api.github.com/repos/depers/jasper-db/git/blobs/6bb4003f508d5301502b58abd4c1c8d70bba810d",
              "download_url": "https://raw.githubusercontent.com/depers/jasper-db/main/README.md?token=ADZ4M5IU3SPXYCLOJWK4CQ3D3IMNK",
              "type": "file",
              "_links": {
                "self": "https://api.github.com/repos/depers/jasper-db/contents/README.md?ref=main",
                "git": "https://api.github.com/repos/depers/jasper-db/git/blobs/6bb4003f508d5301502b58abd4c1c8d70bba810d",
                "html": "https://github.com/depers/jasper-db/blob/main/README.md"
              }
            }
          ]
          
        ```
        
        



# 参考文章

* github API文档地址：https://docs.github.com/zh/rest?apiVersion=2022-11-28
* [一篇文章搞定Github API 调用 (v3)](https://segmentfault.com/a/1190000015144126)
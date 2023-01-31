github相关api调试



1. 获取仓库的内容
    * 接口地址：/repos/{owner}/{repo}/contents/{path}
    * 头信息：Authorization：Bearer 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
    * 注意：
        1. path若是文件路径，则会返回文件的内容，文件内容是以base64进行编码的。
        1. path若是目录路径，则会返回目录的相关信息。
    * 问题：
        1. 我的文章页面涉及创建时间和标签，这个该怎么处理呢？github返回的报文中，没有这两个字段。这个问题其实也好解决，我通过目录名称就可以解决这个问题了



# 参考文章

* github API文档地址：https://docs.github.com/zh/rest?apiVersion=2022-11-28
* [一篇文章搞定Github API 调用 (v3)](https://segmentfault.com/a/1190000015144126)
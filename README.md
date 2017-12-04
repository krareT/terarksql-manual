# 文档协作说明

该仓库是用来生成 gitbook 文档用的

## 结构说明
- en/  英文文档路径
- zh-hans/  中文文档路径
  - SUMMARY.md  文档目录，该文件中的目录需要引用同一目录下的其他 md 文档
  - xxx.md 每个独立的文档
  
  
## 添加新文档

1. 请首先在 zh-hans/SUMMARY.md(中文) 中找到合适的位置把新文档添加到目录中
2. 在 zh-hans/ 目录下创建新文档
3. 新的文档 push 到仓库后，可以通过 gitbook(https://terark.gitbooks.io/mysql-on-terarkdb-manual/content/) 来访问查看效果
4. 由于 gitbook 国内访问较慢，所以我们在自己的官网上独立部署了文档，但需要手动更新，请编写完成后通知管理员

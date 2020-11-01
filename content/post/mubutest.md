---
title: "Mubutest"
date: 2020-11-01T10:54:13+08:00
draft: true
---



<div class="export-wrapper"><div style="font-size: 22px; padding: 0 15px 0;"><div style="padding-bottom: 24px">LevelDB</div><div style="background: #e5e6e8; height: 1px; margin-bottom: 20px;"></div></div><ul style="list-style: disc outside;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">概述</span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">持久化键值存储</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">特性</span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">1. key和value都是<span class="bold" style="font-weight: bold;">任意长度</span>的字节数组；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">2. entry（即一条K-V记录）默认是<span class="bold" style="font-weight: bold;">按照key的字典顺序存储</span>的，当然开发者也<span class="bold" style="font-weight: bold;">可以重载这个排序函数</span>；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">3. 提供的基本操作接口：<span class="bold" style="font-weight: bold;">Put()、Delete()、Get()、Batch()</span>；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">4. 支持<span class="bold" style="font-weight: bold;">批量操作以原子操作进行</span>；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">5. 可以创建数据全景的<span class="bold" style="font-weight: bold;">snapshot(快照)</span>，并允许在快照中查找数据；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">6. 可以通过前向（或后向）迭代器<span class="bold" style="font-weight: bold;">遍历数据</span>（迭代器会隐含的创建一个snapshot）；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">7. 自动<span class="bold" style="font-weight: bold;">使用Snappy压缩数据</span>；</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">8. 可移植性；</span></li></ul></li></ul></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">核心要点</span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">API</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">LSM结构</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">sstable</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">boolm filter</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">缓存</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">压缩 compaction</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">版本控制</span></li></ul></li><li style="line-height: 24px;"><span class="content mubu-node" images="%5B%7B%22id%22%3A%2237e17581b23cd00e4-2603194%22%2C%22uri%22%3A%22document_image%2F9a892e2b-43e6-40e9-9880-46cf0b6ae754-2603194.jpg%22%2C%22ow%22%3A482%2C%22oh%22%3A444%2C%22w%22%3A353%7D%5D" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">架构</span><div style="padding: 3px 0"><img src="https://img.mubu.com/document_image/9a892e2b-43e6-40e9-9880-46cf0b6ae754-2603194.jpg" style="max-width: 720px; width: 353px;" class="attach-img"></div></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">资料</span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">github-leveldb</span><br><span class="note" style="display: inline-block; color: rgb(136, 136, 136); line-height: 22px; min-height: 22px; font-size: 14px; padding-bottom: 2px;"><a class="content-link" target="_blank" href="https://github.com/google/leveldb" style="text-decoration: underline; opacity: 0.6; color: inherit;">https://github.com/google/leveldb</a>有时间可以学习源码, ​</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">levelDB-handbook</span><br><span class="note" style="display: inline-block; color: rgb(136, 136, 136); line-height: 22px; min-height: 22px; font-size: 14px; padding-bottom: 2px;"><a class="content-link" target="_blank" href="https://leveldb-handbook.readthedocs.io/zh/latest/basic.html" style="text-decoration: underline; opacity: 0.6; color: inherit;">https://leveldb-handbook.readthedocs.io/zh/latest/basic.html</a></span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" images="%5B%7B%22id%22%3A%2226e17526b3c69c079-2603194%22%2C%22oh%22%3A595%2C%22ow%22%3A404%2C%22uri%22%3A%22document_image%2F268274e6-acb8-474f-bfbc-3ae9523cd3af-2603194.jpg%22%2C%22w%22%3A212%7D%5D" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">很好的书, 相对于博客更系统, 内容更全</span><div style="padding: 3px 0"><img src="https://img.mubu.com/document_image/268274e6-acb8-474f-bfbc-3ae9523cd3af-2603194.jpg" style="max-width: 720px; width: 212px;" class="attach-img"></div></li></ul></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">LevelDB深入浅出之整体架构:&nbsp;<a class="content-link" target="_blank" href="https://zhuanlan.zhihu.com/p/67833030" style="text-decoration: underline; opacity: 0.6; color: inherit;">https://zhuanlan.zhihu.com/p/67833030</a></span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">讲到了 levelDB名称由来</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">为什么用LSM树(将磁盘的随机操作变为顺序操作, 以及LSM的结构)</span></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">LevelDB基础架构, 包括哪些重要文件(部分)</span></li></ul></li><li style="line-height: 24px;"><span class="content mubu-node" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">庖丁解LevelDB之概览</span><br><span class="note" style="display: inline-block; color: rgb(136, 136, 136); line-height: 22px; min-height: 22px; font-size: 14px; padding-bottom: 2px;"><a class="content-link" target="_blank" href="https://catkang.github.io/2017/01/07/leveldb-summary.html" style="text-decoration: underline; opacity: 0.6; color: inherit;">https://catkang.github.io/2017/01/07/leveldb-summary.html</a></span><ul class="children" style="list-style: disc outside; padding-bottom: 4px;"><li style="line-height: 24px;"><span class="content mubu-node" images="%5B%7B%22id%22%3A%2235217581afb45005b-2603194%22%2C%22uri%22%3A%22document_image%2Fb4b3de30-2bdb-494f-a3fa-59c09d5f147f-2603194.jpg%22%2C%22ow%22%3A492%2C%22oh%22%3A866%2C%22w%22%3A245%7D%5D" style="line-height: 24px; min-height: 24px; font-size: 16px; padding: 2px 0px; display: inline-block; vertical-align: top;">这个人写了一系列博客, 值得学习</span><div style="padding: 3px 0"><img src="https://img.mubu.com/document_image/b4b3de30-2bdb-494f-a3fa-59c09d5f147f-2603194.jpg" style="max-width: 720px; width: 245px;" class="attach-img"></div></li></ul></li></ul></li></ul></div>
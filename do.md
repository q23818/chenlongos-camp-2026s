
# Learning Notes for ChenLongOS Camp 2026 Spring

2026-04-18 11:53:48

## 0. about the course

https://opencamp.cn/ChenLongOS/camp/2026S

**2026 春季辰龙操作系统训练营**

**时间**  ： 2026/04/19-2026/05/17
**主办方**： 辰龙操作系统开源社区

-------------------------------------------------------
## 1. init repo

**创建仓库**

create repo: https://github.com/q23818/chenlongos-camp-2026s

```bash
cd /Users/liu/opencamp/chenlongos/2026s/
echo "# chenlongos-camp-2026s" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:q23818/chenlongos-camp-2026s.git
git push -u origin main

echo "*.swp\n*.un~\n*~" > .gitignore
git add .gitignore
git commit -m "add gitignore"
git push

```

-------------------------------------------------------
## 2. guidance learning

**课程导学**

### tasks

1. 按照下面这个仓库的 readme 文档在 qemu riscv 下跑起来 arceos helloworld。

考核：截图启动命令和 helloworld 输出。 https://github.com/arceos-org/arceos

2. （选做）深入了解 spi 协议，

考核：产出一份详细学习的文档。

3. （选做）学习 tpu-mlir 的基本使用方法（如果对神经网络不熟悉的同学可以先了解一下神经网络基础知识）

考核：跑通 readme 中 dog.jpg 的识别，得到一个画框结果图片

参考链接：https://github.com/sophgo/tpu-mlir

4. 导学作业提交：

【腾讯文档】题目大家给出链接就可以，腾讯文档或者是github等
https://docs.qq.com/sheet/DUEVHd3hQUkxoWGR6?tab=BB08J2


-------------------------------------------------------



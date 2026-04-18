
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

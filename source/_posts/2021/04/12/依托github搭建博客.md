---
title: 依托github搭建博客--(1)
date: 2021-04-12 22:45:11 
keyword: github pages hexo next actions gittalk
tags: github pages hexo next actions gittalk
categories: 建站 博客
---

依托于github来创建属于个人的博客，并开启评论，以及使用自定义域名访问。
一、创建博客

<!--more-->

# 依托github搭建博客

## 准备

1. 新建一个名为`xxx.github.io`的仓库，其中`xxx`为你登录后`https://github.com/xxx`显示的用户名
2. 对仓库进行配置，配置如下
    ![githubpages](githubpages.png)

3. 安装hexo

   ```js
   npm install hexo-cli -g
   ```

4. 创建博客工程

   ```shell
   hexo init xxx.github.io
   cd init xxx.github.io
   npm install
   ```

5. 下载主题插件

   ```shell
   cd init xxx.github.io
   git clone https://github.com/ themes/next
   ```

6. 创建用户数据文件

   ```shell
   cd init xxx.github.io/source
   mkdir _data
   cd _data
   ```

   将`themes/next/_config.yml`复制到`source/_data`目标，并改名为`next.yml`  
   将`next.yml`中的`override: false`改为`override: true`

7. 修改配置hexo的住配置文件`xxx.github.io/_config.yml`, 修改项如下

   ```yml
    title: xxxx  #博客标题
    subtitle: xxxx  ## 副标题
    description: xxxx  ## 博客说明
    keywords: xxxx  ##SEO用
    author: xxxx  ##作者名称，显示用
    language: zh-CN
    timezone: Asia/Shanghai

    url: https://xxx.github.io

    theme: next
   ```

8. 推送到`github`上

    ```shell
        git init
        git add .
        git commit -m "first commit"
        git remote add origin https://github.com/xxx/xxx.github.io
        git push -u origin master
    ```

9. 本地新建`source`分支，切换到`source`分支上，并将该分支推到`github`

    ```shell
    git checkout -b source
    git push origin source:source
    ```

10. 在本地创建部署公私钥`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
11. 将公私钥填入到`github`中，并确认私钥的名称，假设私钥名称为`GENERATE`
    ![public_key](public_key.png)
    ![secret](secret.png)
12. 在`github`上创建流水线文件`.github/workflows/pages.yml`,内容为

    ```yml
    name: Pages

    on:
    push:
        branches:
        - source  # default branch


    jobs:
    build:
        runs-on: ubuntu-latest
        name: A job to deploy blog.
        steps:
        - name: Checkout
        uses: actions/checkout@v2
        with:
            submodules: true # Checkout private submodules(themes or something else).
        - name: Checkout tools repo
        uses: actions/checkout@v2
        with:
            repository: theme-next/hexo-theme-next
            path: themes/next
        - name: Setup Node.js environment
        uses: actions/setup-node@v2.1.5
        with:
            node-version: 12.x
        # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
        - name: Cache node modules
        uses: actions/cache@v1
        id: cache
        with:
            path: node_modules
            key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
            restore-keys: |
            ${{ runner.os }}-node-
        - name: Install Dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
        - name: Build
        run: npm run build
        - name: Deploy
        uses: peaceiris/actions-gh-pages@v3.7.3
        with:
            deploy_key: ${{ secrets.GENERATE }}
            #github_token: ${{ secrets.GENERATE }}
            publish_dir: ./public
            publish_branch: master
        # Deploy hexo blog website.
        #- name: Deploy
        #  id: deploy
        # uses: sma11black/hexo-action@v1.0.3
        #with:
        # deploy_key: ${{ secrets.GENERATE }}
            #user_name: piggsoft  # (or delete this input setting to use bot account)
            #user_email: piggsoft@163.com  # (or delete this input setting to use bot account)
            #commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
            #publish_dir: ./public
            #publish_branch: master  # deploying branch
        # Use the output from the `deploy` step(use for test action)
        - name: Get the output
        run: |
            echo "${{ steps.deploy.outputs.notify }}"

    ```

13. 修改`source/helloworld.md`中任意内容，观察`Github Actions`,待完成后，浏览器中输入`xxx.github.io`，观察博客


# Github Actions自动部署hugo博客




### 创建Github Actions流水线
```bash
mkdir -p .github/workflows/
touch hugo.yaml
```
#### 发布到本仓库
<!--more-->
```yaml
name: GitHub Pages Deploy Local REPOSITORY

on:
  push:
    branches:
      - master  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 1    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.91.2'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

#### 发布到另外仓库
```yaml
name: CI #自动化的名称
on:
  push: # push的时候触发
    branches: # 那些分支需要触发
      - master
jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 1    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        run: hugo --minify
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.EXTERNAL_REPOSITORY_TOKEN }}
          publish_dir: ./public
          external_repository: kbsonlong/devops.alongparty.cn
```
> 注意: 发布到本仓库github_token不需要手动创建GITHUB_TOKEN, 发布到其他仓库personal_token需要提前创建并配置secret变量
#### 创建SSH证书
```bash
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```
#### 配置SSH公钥
![](https://raw.githubusercontent.com/kbsonlong/picgo-imgs/main/images20220523213724.png)

#### 配置SSH私钥
![](https://raw.githubusercontent.com/kbsonlong/picgo-imgs/main/images20220523213820.png)
#### 使用SSH私钥发布
```yaml
name: CI #自动化的名称
on:
  push: # push的时候触发
    branches: # 那些分支需要触发
      - master
jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 1    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        run: hugo --minify
      - name: Deploy with PRIVATE KEY
        uses: peaceiris/actions-gh-pages@v3
        with:
          cname: devops.alongparty.cn
        env:
          ACTIONS_DEPLOY_KEY: ${{ secrets.HUGO_DEPLOY_PRIVATE_KEY }}
          EXTERNAL_REPOSITORY: kbsonlong/devops.alongparty.cn
          PUBLISH_BRANCH: gh-pages
          PUBLISH_DIR: ./public
```
> 注意: 部署到另外仓库时ssh私钥配置在源仓库, ssh公钥配置在目标仓库或者全局

### 创建GITHUB Personal TOKEN
![](https://raw.githubusercontent.com/kbsonlong/picgo-imgs/main/images20220523213508.png)



### 创建仓库Secrets变量
![](https://raw.githubusercontent.com/kbsonlong/picgo-imgs/main/images20220523213955.png)

### Github Page配置自定义域名
```bash
echo "<自定义域名>" >static/CNAME
```


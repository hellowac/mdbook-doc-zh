# 在持续集成中运行 `mdbook`

虽然以下示例使用 Travis CI，但它们的原则也应该直接转移到其他持续集成提供者。

## 确保您的图书 build 和 test 通过

这是一个示例 Travis CI `.travis.yml` 配置，可确保 `mdbook build` 和 `mdbook test` 成功运行。
快速 CI 运转时间的关键是缓存 `mdbook` 安装，这样您就不会在每次 CI 运行时编译 `mdbook`。

```yaml
language: rust
sudo: false

cache:
  - cargo

rust:
  - stable

before_script:
  - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
  - (test -x $HOME/.cargo/bin/mdbook || cargo install --vers "^0.3" mdbook)
  - cargo install-update -a

script:
  - mdbook build && mdbook test # In case of custom book path: mdbook build path/to/mybook && mdbook test path/to/mybook
```

## 将您的书部署到 GitHub Pages

遵循这些说明将导致在您仓库的`master`分支上成功运行 CI 后，您的书将被发布到 GitHub pages。

首先，使用“public_repo”权限（或“repo”用于私有仓库）创建一个新的 GitHub “Personal Access Token”(个人访问令牌)。 转到仓库的 Travis CI 设置页面并添加名为 `GITHUB_TOKEN` 的环境变量，该变量标记为安全且未显示在日志中。

在您仓库的设置页面中，导航到 Options 并将 GitHub Pages 上的 Source 更改为 `gh-pages`。

然后，将此代码段附加到您的 `.travis.yml` 并更新 `book` 目录的路径：

```yaml
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  local-dir: book # In case of custom book path: path/to/mybook/book
  keep-history: false
  on:
    branch: main
```

仅此而已!

注意: Travis 有一个新的 [dplv2] 配置，目前处于测试阶段。 要使用这种新格式，请将 `.travis.yml` 文件更新为：

[dplv2]: https://blog.travis-ci.com/2019-08-27-deployment-tooling-dpl-v2-preview-release

```yaml
language: rust
os: linux
dist: xenial

cache:
  - cargo

rust:
  - stable

before_script:
  - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
  - (test -x $HOME/.cargo/bin/mdbook || cargo install --vers "^0.3" mdbook)
  - cargo install-update -a

script:
  - mdbook build && mdbook test # In case of custom book path: mdbook build path/to/mybook && mdbook test path/to/mybook
  
deploy:
  provider: pages
  strategy: git
  edge: true
  cleanup: false
  github-token: $GITHUB_TOKEN
  local-dir: book # In case of custom book path: path/to/mybook/book
  keep-history: false
  on:
    branch: main
  target_branch: gh-pages
```

### 手动部署到 GitHub Pages

如果您的 CI 不支持 GitHub Pages，或者您正在使用 Github Pages 等集成部署到其他地方：

*注意: 您可能想要使用不同的 tmp 目录*:

```console
$> git worktree add /tmp/book gh-pages
$> mdbook build
$> rm -rf /tmp/book/* # this won't delete the .git directory
$> cp -rp book/* /tmp/book/
$> cd /tmp/book
$> git add -A
$> git commit 'new book message'
$> git push origin gh-pages
$> cd -
```

或者将其放入 Makefile 规则中：

```makefile
.PHONY: deploy
deploy: book
 @echo "====> deploying to github"
 git worktree add /tmp/book gh-pages
 rm -rf /tmp/book/*
 cp -rp book/* /tmp/book/
 cd /tmp/book && \
  git add -A && \
  git commit -m "deployed on $(shell date) by ${USER}" && \
  git push origin gh-pages
```

## 部署你的书到 GitLab Pages

在仓库的项目根目录中，创建一个名为 `.gitlab-ci.yml` 的文件，其中包含以下内容：

```yml
stages:
    - deploy

pages:
  stage: deploy
  image: rust
  variables:
    CARGO_HOME: $CI_PROJECT_DIR/cargo
  before_script:
    - export PATH="$PATH:$CARGO_HOME/bin"
    - mdbook --version || cargo install mdbook
  script:
    - mdbook build -d public
  rules:
    - if: '$CI_COMMIT_REF_NAME == "master"'
  artifacts:
    paths:
      - public
  cache:
    paths:
      - $CARGO_HOME/bin
```

在您提交并推送这个新文件后，GitLab CI 将自动运行并且您的书将可用！

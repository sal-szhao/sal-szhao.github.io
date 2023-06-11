---
title: Hexo Github Deployment for Personal Blogs
date: 2023-06-06 10:03:56
tags: [hexo, tech]
---

This is the very first blog for myself. So I would rather begin with the process of how to deploy personal hexo blogs on Github. Hexo is a light-weighted blog framework, supporting theme customizations. Check [documentation](https://hexo.io/docs/) for more info.

## Preparations

### Basic packages

Install `Git` and `NodeJS` in local environment.

### Create and clone Github repository
Create a new `repository` on Github. The format of the repository name should be `<username>.github.io`. Use `Github Pages` to check if the website of the repository can be visited.

## Install Hexo

Install `Hexo`. `npm install` tends to require root access.

``` bash
$ (sudo) npm install -g hexo-cli
```

For China users, `npm install` command can be very slow. Common solution of using mirror sites of `taobao` have been deprecated. Refer to the [github issue](https://github.com/npm/cli/issues/4876).

``` bash
$ npm cache clear --force
$ npm config set registry https://registry.npmmirror.com
```

Check the whether Hexo is installed and the current version.

``` bash
$ hexo -v
```

Create a new directory `hexo-blog` and initialize.

``` bash
$ hexo init hexo-blog
$ cd hexo-blog
```

Launch the blog page locally.

``` bash
$ (sudo) npm install
$ hexo generate
$ hexo server
```

Or the abbreviated code is

``` bash
$ hexo g && hexo s
```

Use the browser to visit http://localhost:4000/. One should get access to the Hexo blog page with default theme.


## Deploy to Github

Stay in the root directory of the blog folder and install `hexo-deployer-git`.

``` bash
$ (sudo) npm install hexo-deployer-git --save
```

Then modify the configure file `_config.yml` in the root directory, where `token` is the `Personal Access Token` for `Github`.

``` yaml
deploy:
  type: git
  repo: https://github.com/sal-szhao/sal-szhao.github.io.git
  branch: main
  token: your_personal_access_token
```

Then run the following code to deploy the website.

``` bash
$ hexo g
$ hexo deploy / hexo d
```

Notice that running `hexo d` will generate a `public` folder, which is consistent with the folder pushed to `Github repository`.

![public folder](public.png)

``` bash
$ hexo clean
```

Cleans the cache file `db.json` and generated folder `public`.

## Additional Hexo Usage

### Update blog on new machines

Since the codes generated for rendering the page are different from the original codes, they could be stored in different branches.

The `public` folder is stored in `main` branch of the repository.

The original codes are stored in `hexo` branch of the repository.

``` bash
$ git clone -b hexo git@github.com:JoeyBling/hexo-theme-yilia-plus.git
```

Only clone the hexo branch and rerun

``` bash
$ (sudo) npm install
$ hexo g && hexo s
```

### New post

``` bash
$ hexo new post testing
```
creates a new post.

### Add image to post

Install `hexo-renderer-marked`.

``` bash
$ (sudo) npm install hexo-renderer-marked --save
```

Modify the `_config.yml` file in the root directory.

``` yaml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

Setting `post_asset_folder` to be `true` will create a folder with the same name as the post when creating a new post.

``` markdown
![alt text](GithubPage.png)
```
will add image to the post.

![insert image](insertimage.png)

### Change theme

Here is a simple tutorial for how to change the default theme to [yilia-plus](https://github.com/JoeyBling/hexo-theme-yilia-plus) theme.

Clone the repository to the `themes` folder.

``` bash
$ git clone https://github.com/JoeyBling/hexo-theme-yilia-plus.git
```

Or try the following if cloning is too slow.

``` bash
$ git clone https://gitclone.com/github.com/JoeyBling/hexo-theme-yilia-plus.git
```

Modify the `_config.yml` file in the root directory.

``` yaml
theme: yilia-plus
```

<font color='red' size=5>Delete all the `.git` folders inside the cloned themes folder! ! !</font>

Otherwise there will be conflicts when pusing into the repository.

![final demo](finaldemo.png)

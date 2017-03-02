# hexo-blog
http://ouyangyewei.me/

# Mark

## step 1
```
https://github.com/ouyangyewei/hexo-blog.git hexo-blog
```

## step 2
```
hexo init blog
cd blog
```

## step 3
***copy `hexo-blog/CNAME` to `blog`***
copy `hexo-blog/_config.yaml` to `blog`
copy `hexo-blog/source/*` to `blog/source`
copy `hexo-blog/themes/maupassant` to `blog/themes`

## step 4
```
npm install hexo-renderer-jade --save

npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install hexo-renderer-sass --save

npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
npm install hexo-deployer-git --save
```

## step 5
```
hexo clean
hexo generate
hexo deploy
```

# Resource
image: http://www.qiniu.com/

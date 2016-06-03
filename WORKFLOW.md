# Writer's workflow

## 1. Setup & Get started

```
# clone this repository
git clone https://github.com/fenneclab/blog.fenneclab.com.git blog.fenneclab.com
cd $_

# install dependencies
npm i

# run hugo server
npm run serve
```

## 2. Add new blog post

```
# change your git-branch
git checkout -b feat/my-new-post origin/master
npm run new -- 'blog/my-new-post.md'
```

### :memo: Understanding content structure

```
content/ # saved all posts
  blog/  # saved default posts (show in blog top page)
static/
  img/   # saved all post's images
    2016/
      06/
        my-newpost-1.jpg
        my-newpost-2.jpg
        my-otherpost-1.jpg
        my-otherpost-2.jpg
        ...
      07/
        my-specialpost-1.jpg
        ...
    2017/
    ...
```

## 3. Write/Edit your content

## 4. Set `featured image`

\#-1. Save featured image name as `static/img/2016/06/my-newpost-featured.jpg`.

\#-2. Set your post setting fields like below.

```
---
title: my-newpost-featured
date: '2015-03-19T19:01:14.000Z'
draft: false
type: post
featured: my-newpost-featured.jpg
featuredalt: my-newpost-featured
featuredpath: date
...
---
```

## 5. Commit your post

```
git add .
git commit -m "feat(content): my new post!"
git push origin feat/my-new-post
```

## 6. Create Pull Request on github

eg. feat/my-new-post -> master

# :bulb: Tips;

## Linting

All contents text are checked via [textlint/textlint](https://github.com/textlint/textlint).

If you use sublime text 3 for writing, textlint plugin is available.

1. <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> > type `install package`
2. Search: `textlint`
3. Install

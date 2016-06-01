# blog.fenneclab.com

[![Build Status](https://travis-ci.org/fenneclab/blog.fenneclab.com.svg?branch=master)](https://travis-ci.org/fenneclab/blog.fenneclab.com)

## Get started

```
# clone this repository
git clone https://github.com/fenneclab/blog.fenneclab.com.git blog.fenneclab.com
cd $_

# install dependencies
npm i

# run hugo server
npm run serve
```

## Add blog post

```
npm run new -- 'blog/new-post-name.md'
# edit content/blog/new-post-name.md
```

## Create site environment

:warning: Buy site's domanin before run.

```
export SITE_DOMAIN="fenneclab.com"
export SITE_HOSTNAME="blog.fenneclab.com"
aws cloudformation validate-template --template-body "$(cat ./formation.json)"
aws cloudformation create-stack \
  --stack-name ${SITE_HOSTNAME//./-} \
  --parameters \
  ParameterKey=DomainRoot,ParameterValue=${SITE_DOMAIN} \
  ParameterKey=HostName,ParameterValue=${SITE_HOSTNAME} \
  --template-body "$(cat ./formation.json)" \
  --capabilities CAPABILITY_IAM \
  --tags Key=site,Value=${SITE_HOSTNAME}
```

## Update site environment

```
export SITE_HOSTNAME="blog.fenneclab.com"
aws cloudformation validate-template --template-body "$(cat ./formation.json)"
aws cloudformation update-stack \
  --stack-name ${SITE_HOSTNAME//./-} \
  --parameters \
  ParameterKey=DomainRoot,UsePreviousValue=true \
  ParameterKey=HostName,UsePreviousValue=true \
  --capabilities CAPABILITY_IAM \
  --template-body "$(cat ./formation.json)"
```

## Generate favicons

```
npm run favicon
# copy and paste if needed
# see: themes/hugo-future-imperfect/layouts/partials/favicon.html
# also: https://github.com/audreyr/favicon-cheat-sheet
export favVer="1"
cp ./.favicon/apple-touch-icon-precomposed.png ./static/favicon/apple-touch-icon-precomposed-${favVer}.png
cp ./.favicon/mstile-144x144.png ./static/favicon/mstile-${favVer}.png
cp ./.favicon/favicon-32x32.png ./static/favicon/favicon-${favVer}.png
```

## Deploy manually

**more dependencies**

- [aws-cli](http://docs.aws.amazon.com/ja_jp/cli/latest/userguide/installing.html)

```
# --dryrun on s3 sync
npm run deploy:dryrun

npm run deploy
```

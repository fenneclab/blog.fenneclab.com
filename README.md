# blog.fenneclab.com [![Build Status](https://travis-ci.org/fenneclab/blog.fenneclab.com.svg?branch=master)](https://travis-ci.org/fenneclab/blog.fenneclab.com)

> [blog.fenneclab.com](https://blog.fenneclab.com) template powered by Hugo

## Writer's workflow

see: [WORKFLOW.md](WORKFLOW.md)

## Create/Update site environment

:warning: Buy site's domanin before run.

```
export SITE_DOMAIN="fenneclab.com"
export SITE_HOSTNAME="blog.fenneclab.com"
export SITE_ACM_ARN="***"
aws cloudformation validate-template --template-body "$(cat ./formation.json)"
aws cloudformation deploy \
  --stack-name ${SITE_HOSTNAME//./-} \
  --no-execute-changeset \
  --parameter-overrides \
  DomainRoot=${SITE_DOMAIN} \
  HostName=${SITE_HOSTNAME} \
  AcmArn=${SITE_ACM_ARN} \
  --template-file ./formation.json \
  --capabilities CAPABILITY_IAM
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

## License

Apache 2.0 @ [fenneclab](https://blog.fenneclab.com/)

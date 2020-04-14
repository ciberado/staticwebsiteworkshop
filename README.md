# Build your static blog in 25 simple steps

## Tooling

* [hub](https://github.com/github/hub/releases)
* Configure it with `alias git=hub"
* [AWS cli](https://aws.amazon.com/es/cli/)

```bash
sudo apt update
sudo apt-get install python3-pip python3 --yes
pip3 install awscli
aws --version
```

* Set AWS credentials and preferred region

```bash
aws configure --profile blog
export AWS_PROFILE=blog
```

* [Hugo](https://github.com/gohugoio/hugo/releases)

## Hosting configuration

* Create the bucket

```bash
BLOG_NAME=blognumber$RANDOM
aws s3 mb s3://$BLOG_NAME

cat<< EOF > policy.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Public s3 access",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::$BLOG_NAME/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy --bucket $BLOG_NAME --policy file://policy.json
```

* Configure it

```bash
cat<< EOF > statichost.json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "404.html"
  },
  "RoutingRules": [
    {
      "Redirect": {
        "ReplaceKeyWith": "index.html"
      },
      "Condition": {
        "KeyPrefixEquals": "/"
      }
    }
  ]
}
EOF

aws s3api put-bucket-website --bucket $BLOG_NAME --website-configuration file://statichost.json
```

## Create your blog

* Create the blog

```bash
hugo new site $BLOG_NAME
cd $BLOG_NAME
sed -i "s/http:\/\/example.org\///i" config.toml

git init
git add -A
git commit -m "New blog"

echo public >> .gitignore

git submodule add https://github.com/azmelanar/hugo-theme-pixyll themes/pixyll
echo 'theme = "pixyll"' >> config.toml

git add config.toml
git commit -m "Added new theme"
```

* Add a post and change the `draft = true` to `draft = false`

```bash
hugo new posts/hello.md
vim posts/hello.md
git add content/posts
git commit -m "First post"
```

* Test the site

```bash
hugo serve -D
```

* Publish the site

```bash
rm public -fr
hugo
aws s3 cp --cache-control max-age=3600 --acl public-read  --recursive public/ s3://$BLOG_NAME
```

* Open your website!

```bash
REGION=$(aws configure get region)
echo The website url is: http://$BLOG_NAME.s3-website-$REGION.amazonaws.com
```

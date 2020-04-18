# Build your static blog in 25 simple steps

## Tooling

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
mkdir web
wget https://pastebin.com/raw/Y1D5zYNR -O web/index.html
wget https://pastebin.com/raw/jxEqVD29 -O web/404.html
```

* Publish the site

```bash
aws s3 cp --cache-control max-age=3600 --acl public-read  --recursive web/ s3://$BLOG_NAME
```

* Open your website!

```bash
REGION=$(aws configure get region)
echo The website url is: http://$BLOG_NAME.s3-website-$REGION.amazonaws.com
```

# CD-CI-Hugo-pipeline

## Installing Hugo
From https://github.com/gohugoio/hugo/releases select the version of hugo
then in the terminal cloud9
```
wget https://github.com/gohugoio/hugo/releases/download/v0.88.1/hugo_extended_0.88.1_Linux-64bit.tar.gz
tar xzvf hugo_extended_0.88.1_Linux-64bit.tar.gz
mkdir -p ~/bin
mv hugo ~/bin/
hugo version
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
echo 'theme = "ananke"' >> config.toml
hugo new posts/my-first-post.md
```
Then deploying live in the server but before a need to open the port 8080
```
hugo serve --bind=0.0.0.0 --port=8080 --baseURL=http://3.15.165.62/
```


## Deploying to S3
Create a public s3 that can serve static web site, then add the policies
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::nombredelbucket/*"
            ]
        }
    ]
}
```
Copy the link of de web static  ej: http://hugowebsiteduke.s3-website.us-east-2.amazonaws.com y Then copy into config.toml
```
baseURL = "http://hugowebsiteduke.s3-website.us-east-2.amazonaws.com"
languageCode = "en-us"
title = "My New Hugo CD"
theme = "ananke"

[[deployment.targets]]
# An arbitrary name for this target.
name = "awsbucket"
URL = "s3:// hugowebsiteduke/?region=us-east-2" #your bucket here
```
Lets create the public files
```
hugo
```
The deploying in s3
```
hugo deploy 
```

## Create a repository
To create a pipeline we need to create a repository

Before create a ssh key to upload in github
```
create the key
ssh-keygen -t rsa and cat the directory
cat /home/ec2-user/.ssh/id_rsa.pub
```
Then push the code
```
git remote add origin git@github.com:josebarrientosq/CD-CI-Hugo-pipeline.git     //SSH
git pull --rebase origin master
git add .
git commit –m “inicio”
git push origin master
```

## CI/CD aws website
In AWS create a new pipeline en codebuild
Origin github account
Activate hook
Select linux2 for the environment
The add the Role
Administratiraccess

Then create the buildspec.yml file
```
version: 0.2

environment_variables:
  plaintext:
    HUGO_VERSION: “0.88.1”
    
phases:
  install:
    runtime-versions:
      docker: 18
    commands:                                                                 
      - cd /tmp
      - wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz
      - tar xzvf hugo_extended_${HUGO_VERSION}_Linux-64bit.tar.gz
      - mv hugo /usr/bin/hugo
      - cd - 
      - rm -rf /tmp/*
  build:
    commands:
      - rm -rf public
      - hugo
      - aws s3 sync public/ s3://hugowebsiteduke/ --region us-east-2 --delete
  post_build:
    commands:
      - echo Build completed on `date`
 ```
 Then when the code is pushing , it build the enviroment and sync to s3

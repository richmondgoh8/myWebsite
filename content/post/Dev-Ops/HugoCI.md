+++
title = "Hugo CI"
date = "2017-05-05T13:50:46+02:00"
tags = ["theme"]
description = "Your Guide to Building a Web Crawler from scratch using Golang"
keywords = ["Programming","Go","Hugo","Website","Blog"]
draft = false
+++

[1]: https://gohugo.io/
[2]: http://bezdelev.com/post/hugo-aws-lambda-static-website/
[3]: https://github.com/s3tools/s3cmd
[4]: https://kramerc.com/2013/10/23/deploying-to-s3-upon-git-push/

## Sneak Preview
Today's Topic will be all about Continuous Integration for [Hugo][1] Deployment! Click on the link to find out more but in a nutshell, it is a pretty neat tool for hosting a static website! The thing is that now that you are done creating your website with Hugo tool in localhost. You are ready to let it run public for the whole wide world to know! After browsing tutorials after tutorials and scanning through providers, i realized that hosting providers are definitely not the way to go and to keep cost low, I have decided to use Amazon S3 Storage.

# Manual Labor Sucks!
Essentially, If you do it manually, what you would do is create a bucket in S3 with the name "example.com". Followed by which you would proceed to run the command
```
// Generate Public Folder
hugo
```

You would then place the public folder into the bucket, if you enable static website hosting under the properties tab of the bucket, it will now provide you an endpoint for you to use to access to your website! This is all good unless you realize that you have to constantly log into the amazon cloud console and run the hugo command, empty the bucket and place the files all over again.

Follow this optional [tutorial][2] to get a domain name to be integrated with your bucket. Note that you need not follow the part where it requires the lambda functions. You would only need example.com as the storage for the public content and www.example.com as a redirect to example.com.

What we want now is to use a tool called [s3cmd][3] which seeks to helps us to configure our command line to [sync content][4] across from our local repository to our s3 bucket.

```
#!/bin/bash
#
# Deploy to s3 when master gets updated.
#
# This expects (and does NOT check for) s3cmd to be installed and configured!
# This expects (and does NOT check for) hugo to be installed and on your $PATH

bucket='example.com'
prefix=''

branch=$(git rev-parse --abbrev-ref HEAD)

if [[ "$branch" == "master" ]]; then
  hugo
  echo "Syncing public/* with s3://$bucket/$prefix."
  s3cmd --acl-public --delete-removed --no-progress sync public/* s3://$bucket/$prefix
  echo -e "\nUpdated s3://$bucket/$prefix."
else
  echo "*** s3://$bucket/$prefix only syncs when master branch is updated! ***"
fi

exit 0
```
</br>
Above is a sample post-commit code from git which helps to sync content to s3 bucket which is trigger by git push. Thats all folks and you have a updated website just by running the commands!
git add.
git commit -m "First Commit"

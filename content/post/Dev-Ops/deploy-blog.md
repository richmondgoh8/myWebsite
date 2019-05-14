+++
title = "Deploying Hugo On AWS"
date = "2017-03-31T13:50:46+02:00"
categories = ["Development"]
description = "Your Guide to deploying Hugo with AWS Services"
keywords = ["Game","Engine","Corona","Business","Mobile","Application"]
menu = "main"
cover = "/images/pexel_small.jpg"
+++

[1]: http://bezdelev.com/post/hugo-aws-lambda-static-website/
[2]: https://gohugo.io/overview/quickstart/
[3]: http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html
[4]: http://blog.el-chavez.me/2015/11/26/go-hugo-seo/
[5]: https://gohugo.io/overview/introduction/
## Introduction
<p>Initially, I have been using Web Hosting Hub with wordpress installed with it to run my website. I got my domain name from OpenSRS. I am quite happy with their services and support. However, It has come to the point that I am clearly paying too much for just hosting a blog which stood at about 13 bucks a month.</p>

<!--more-->
<p>This is where I chanced upon Amazon Web Services. Not being a good front-end developer, I went around in search of a template where I found Hugo, a static web builder developed with Golang. Below list the ways i went around trying to switch from web hosting hub to AWS. It was a rather fruitful experience as my overhead became less than 1 dollar a month.</p>

Feel free to view a step by step tutorial on deploying hugo with Amazon Web Services by [Ilya Bezdelev][1].  

In a nutshell, key points that you should note involves:  

+ Hugo website
  - Follow Hugo Tutorial on Creating Website [here][2]
+ example.com bucket
  - serves the website content
+ www.example.com bucket
  - redirects to example.com endpoint
+ input.example.com bucket
  - contains all hugo related files

+ Default AWS Endpoint Domain Name
  - Storage Buckets (used for containing Hugo files and serving web pages to visitors to site)
  - Lambda ( piece of code that triggers downloading of static files from hugo input to output bucket)
</br>
+ Custom Domain Name
  - CloudFront Distribution (CDN)
  - Route 53
</br>
* Security of AWS resources will be managed by Identity & Access Management(IAM)
* AWS CloudFront consist of CDN which helps to speed up your website by
* Route 53 is used as a DNS Service as well as Domain Name Registration

Amazon Route 53 is an authoritative Domain Name System (DNS) service. DNS is the system that translates human-readable domain names (example.com) into IP addresses (192.0.2.0). With authoritative name servers in data centers all over the world, Route 53 is reliable, scalable, and fast.
TLD = Top Level Domain

With all these in hand, Begin your journey to hosting your very own blog.

Remember --> Create Hugo Website --> Create AWS Web Service --> Create S3 to store Hugo Files --> DNS with hosted zone to serve your example.com to the world.  
</br>

*Disclaimer: feel free to read up more about [Hugo][5] to find out about the other ways to host Hugo files. A free alternative would be netlify!*
*PS - Web Hosting Hub Support is Excellent*

+++
title = "China Infrastructure Deployment"
date = "2017-04-14T13:50:46+02:00"
tags = ["theme"]
description = "Your Guide to Losing Weight Permanently"
keywords = ["Deploy","Infrastructure","China","Aliyun","CS","Terminal","Docker","Docker-Compose","Services"]
draft = false
+++

[1]: https://docs.docker.com/engine/getstarted/step_four/
[2]: https://hub.docker.com/_/haproxy/
[3]: https://m.aliyun.com/yunqi/articles/9102
[4]: https://docs.docker.com/engine/getstarted/step_one/
[5]: https://docs.docker.com/compose/install/
[6]: https://docs.docker.com/compose/compose-file/compose-file-v2/#dependson
In this blog, we will be exploring the way of deploying the current service infrasture that you have built locally to deploying it on<!--more--> a server in china via Alibaba Cloud aka Aliyun.  

*Before starting on this tutorial, the following should be done:*

1.	Services Created and running locally.
2.	Ensure DockerFile have been build.
3.	Understanding of [Docker][1] & [HAProxy][2]

*Successful Implementation*

1. Register an account with both Aliyun CN and INTL Account
2. Login into the CN account & Create an [Image WareHouse][3]
3. At this point, you should have successful push the built docker image to aliyun registry store.
4. Now, we proceed to Creating an ECS Instance (2 core, 8GB Ram, Ubuntu 16.0.4)
5. After creating an instance, you will be given an Internet Address of that Instance.
6. Proceed to SSH (or Putty for Windows) into the virtual machine.
7. Install [Docker][4] & [Docker Compose][5] in the ECS Machine
8. Create a [docker-compose.yml][6] file in a directory in the Ubuntu Machine
9. Ensure that HAProxy is the first image you build. You can check the statistics after building the image via http://”Internet IP”/stats/haproxy
10.	Run```Docker-Compose Up``` and ensure that services are connected and working.
11.	Enjoy



*FAQ*

*Why do we have both an intl and cn aliyun account?*

+ Currently the intl account do not have the the registry image warehouse and is only available in the cn aliyun account. The reason why we do not do everything in the cn aliyun account is because it requires a "real-name" authentication that we currently do not possess and is unable to do.

*Why do we not use container service?*

+ Facing difficulties with Container Services linking multi containers services. Creating a custom image from the services provided in the international account could not pull from the registry properly. Orchestration templates formats in building the multi containers link file have to be in Aliyun’s format context.


*Why did we have to set up OpenVPN through Digital Ocean?*

+ Accessing Aliyun Cloud Website from Singapore would redirect us to the Internal Version of the website which did not provide us with the features we require. Proxy would not work as it does not enable scripting.

*Why didn’t we use third party registry such as docker hub?*

+ The main reason behind not using third-party registry is that they are costly. Another reason is that we will never know when the china’s firewall will block connection from external sources. Integration with third party registry might not be as optimised. Moreover, ensuring all registries have to be in ‘private’ mode.

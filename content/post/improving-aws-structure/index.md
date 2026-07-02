+++
date = '2025-10-04T13:55:52+03:00'
draft = false
title = 'Improving Aws Design Architecture'
description = 'Creating scalable and reliable architecture with low cost'
tags = ['cloud', 'aws', 'design']
categories = ['cloud', 'aws']
+++

In the begining of May 2025 i was assigned to improve the architecture a Fincial Startup company that was using aws at that time i was very familiar with cloud and
i have been using digital ocean for deployments and i was even azure certified but didn't have any expieriance with aws so before i started working on the project I prepared to take the
aws solution architecture exam as it was also having a discount at the tie and i have passed the exam in 21 may.

Now I was the only one assigned to be manager of the migration. Since the previous developer who setup the aws wasn’t working with the company anymore and all the mainters only had knowledge of the basics, I took the first day to understand their architecture, how they operated their codebase and the other stuff. The company had multiple vpns setup and their server ip address was used by old partners so they needed that ip address. Also their operation didn’t want to be shutdown while we were working on it.

On my first assessment of their old architecture the only thing that came to my mind was it was very fragile. They were using a very basic server, just a single EC2 instance to host both their codebase project and the database. They had a natgateway and public setup and also the vpn connected to the vpcs but nothing much else. They were only taking snapshots every night which was very risky because if there was an issue with the server then the whole date of 24 hours could be missing in worst cases. That was bad. Also it was a single point of failure since there was no autoscaling or health checks.

{{< figure src="/assets/old_aws_architecture.jpg" width="300" caption="Old architecture — the first design I wrote when checking it" >}}

Old - Architecture — the first design I wrote when checking it.

{{< figure src="/assets/month.jpg" alt="New Architecture" width="600" caption="New Architecture i have designed" >}}  

After some time I have come up with my first ever aws design architecture for them and I have put my AWS SAA certification to the limit. Firstly I setup a vpc with two availability zones in the region then created one public subnet and one private subnet in each availability zone. I made the natgateway so the ec2 in private can use it only for updates and so. I also moved the project codebase to an EFS file system in the private subnet and made it available to the instances. I moved all the static assets to s3 bucket and connected the ec2 to an autoscaling group. I also moved the database to a managed RDS service with automated backups enabled and point in time restore so they can recover to any second if needed. I made the application load balancer the entry point to the whole project while I kept the existing vpn connections and ip address.

I have planed to use the cloufront for chaching but it would have added overhead and it was not worth it for us.

Now the costing by looking at my new architecture you will see it as much higher cost like double or even more but I have used multiple things I learned from the initial assessment and analysis. That’s why it’s important, on my first assessment I monitored and studied their usage of network, database and other resources and also checked old datas as well. What I found is that from morning 9a to 3pm is their most usage and all the remaining 18 hours of the day their usage is less than 25 percent of the instance. Even in those 9a to 3pm the 12‑1a is the most important of it but I stretched it. So by that logic I setup my instance in scheduled autoscaling so that in those hours the server scales horizontally by adding more instances. Even if the load is more than the scheduled it will also add vertically but the limit was 3 instances. I utilized that a lot almost making the new architecture only increase the monthly by 15% even tho it’s way more reliable, more durable and scalable and handles much more load than previously.

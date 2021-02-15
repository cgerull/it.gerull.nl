---
layout: post
title: Run a WordPress site on AWS
description: "Setup a WordPress site with Bitnami and Lightsail."
tag: AWS Wordpress Lightsail
category: AWS
date: 2020-06-13 20:13:17
---

I don’t want to purchase a hosting packet, as they are often quite restrictive. On the other hand I want to run a WordPress site but not on my local servers as that will require a lot of effort and investment in security setup.

As a DevOps engineer it is a nice a challenge to run my WordPress instance in the cloud. The starting point will be AWS Lightsail platform, as the monthly costs are quite reasonable. This will be my MVP, let’s see where we go from here.

## Setup
The (manual) setup is straight forward. AWS has a nice Tutorial that will guide you through the setup.

In short it will involve these steps.

- Sign up / sign in to AWS
- Set up an admin account and security group in IAM.
- Create a Lightsail account
- Choose a machine template, for the ease of use go for one of the bitnami/wordpress templates.
- Create or reuse a ssh key
- SSH into your new instance and retrieve the Bitnami password.
- If you own a domain and want to use url in that domain is good to setup a static IP address.
- Create a subdomain in your name server or Domain management tool. Alternatively you can also - transfer DNS management to Lightsail and use the Amazon name servers.
- Login to your wp-admin.php page and configure your WP instance.
- With WP it’s always a good idea to keep up with the updates.
- Now configure your site and write your first blog post.
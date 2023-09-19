---
layout: post
title: "TIL: AWS network trapdoors"
subtitle: "Why are you so quiet, dear load balancer?"
date: 2023-09-19
background: '/images/default_post.jpg'
---
As I started to get my hands dirty with Terraform and AWS services I thought a couple of times that I understood things just to destroy this glimmer of confidence again by a another service that was not reachable.

<iframe src="https://giphy.com/embed/EeTHrilLYVcTilqTqU" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/brooklynninenine-nbc-brooklyn-nine-99-EeTHrilLYVcTilqTqU">via GIPHY</a></p>

Currently, in this exact minute, I am again in a state of confidence which allows me to write down what I have learned.

## Subnets

It might be obvious to some, I had to learn it and found its definition surprisingly difficult to find. What exactly is a private subnet?

At first, I thought and was told that it is a subnet in which services have a public IP so they can be reached from outside the subnet.

However, the ALB deployed in a subnet with public IP was not reachabel, no logs, no access attempts, complete silence.

Then, after digging in I understood that its definition is if it has a bi-directional route to the internet.

So a subnet is either
1. Private if it has a NAT configured in its routing table to connect to the internet, only allowing outbound connections from services in the subnet.
2. Public if it has a Internet Gateway in its routing table to connect bi-directionally to the internet and allowing in-bound connections from the internet.

The ALB was not so silent anymore, at least most of the times.

## Firewalls

Even if the service is in a public subnet and thus could be accessed, there are two firewalls that could be still blocking. Probably there are more that I will be surprised by at a later point.

1. Security groups are virtual firewalls on an instance level. That means each service, like ALB or ECS service are attached to a security group which can deny access from certain IPs.
2. Access control lists (ACLs) are virtual firewalls on a subnet level. That means a subnet might have the right route to the internet but you could still deny access from certain IPs to services in this subnet.

I guess both have their place and can be confusing at times.

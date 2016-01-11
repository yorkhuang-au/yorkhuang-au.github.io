---
layout: post
title: Install CKAN On Centos 7
---

# Install CKAN On Centos 7

## Background

CKAN document, release 2.6.0a, gives three options to install CKAN.

* Installing CKAN from package.
* Installing CKAN from source.
* Install using Docker.

In this post, I will discuss the 2nd option, Installing CKAN from source, as it is more generic in many aspects.
Please refer to CKAN document for details.

This post will show you how to install CKAN in a corporation production environment.

The OS is Centos 7 on a vmware virtual machine.

## Install Centos 7

I use the image CentOS-7-x86_64-Everything-1503-01.iso to install Centos 7.
Set up network proxy and date/time properly.

In /etc/sudoers add the ckan user for sudo.  
```
	ckan-user   ALL(=ALL)   ALL
```

In /etc/yum.conf, add the following to enable proxy for yum.
```
	proxy=http://proxy_server:port_number
	proxy_username=username
	proxy_password=password
```

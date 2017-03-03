---
layout: post
title: Using MongoDB for message store
tags: [MongoDB]
categories: [Monitoring]
image:
  background: triangular.png
---

Being an organization whose bread and butter is to help people meet their love of their life, its essential that we keep the users happy and engaged.

Communication is an important aspect of relationship. It's the only way two people can understand about each other and decide if they are compatible with each other.

The current features in the application for user to user communication are nice but a new age calls for a new approach. This year, one of the goals was to revamp the communication module to be more real time and CHAT like.
I will explain the architecture of the new communication system in a separate post. In this blog, I will focus on the storage aspect of the system.


## Choosing A Data Store
This is what we wanted from the storage aspects of the system :

	1. To move away from RDBMS.
	
		Expensive licensing cost along with transactional, locking and normalized data storage will not help us build a highly performing application which can scale to support large number of DB ops/sec. 

2. Adopt a NoSQL storage system

a. Should have flexible schema.
	
	We like the storage model to be flexible and anticipate it to evolve with every new feature we add and requirement for this communicaiton module.
	
b. Horizontally scalable to meet increased load.
	
	Should be able to scale up or across to meet increased demand in load.
	
c. High performance.
	
	Fast inserts and retrival of data.	
	
	
3. Easy maintenance.

Should not be tons of work trying to take back up, restore , scale up or down.
	

4. Monitoring.

Must have an out of box monitoring tool or be easily integrable into another tool.

	
  









 



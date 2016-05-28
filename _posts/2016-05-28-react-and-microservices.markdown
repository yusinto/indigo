---
published: true
title: "React and Microservices Architectural Style"
layout: post
date: 2016-05-28 20:45
tag:
- react
- microservices
- conway's law
- team structure
blog: true
---

I am going to digress a little today and talk about how react fits into the microservice architectural style. Of course microservices and 
react are two buzz words we hear all the time these days. While it might look like I am just shamelessly meshing these two things 
together to get more visitors to my site, you can't be further from the truth (*cough). 

Seriously though, I first learnt of microservices from <a href="http://udidahan.com" target="_blank">Udi Dahan</a>. For those who don't know him, he is the 
creator of NServiceBus and founder of Particular Software. I have nothing but great respect and admiration for him. So you can 
understand my jubilation and joy when I had the chance to meet him in person today at <a href="http://dddsydney.com.au" target="_blank">DDD Sydney</a>. We talked
about many a thing, one of them microservices and team structure. The two might seem unrelated but they actually are, more so than you think.

I was very star-struck. He seemed taller than in the videos. Maybe I was imagining things? Anyway I digress (again). So what is a service? 
I am going to shamelessly rip off Udi's quote and say:

*"A service is the technical authority for a specific business capability. It is autonomous. It has an explicit boundary. All data
and business rules reside within the service."*

I used to think that a service is just a wcf or a webapi endpoint. For the longest time, I naively thought I was doing soa and microservices just
because I expose wcf and webapi endpoints in my application for consumption by other applications. This is incorrect. 

A service is an autonomous 
piece of your domain that is self contained. For example, you might have a product service and a customer service in your domain. The product
service has its own aggregates and its own database schema separate and independent from the customer service. The two services have explicit
boundaries and are autonomous. Each can use a completely different technology stack from the other. 

For example the product service might use
a relational database like sql server whereas the customer service might use a nosql database like a graph db. This is a very big topic which deserves
an entire blog post or technical course in its own right so I'm going to quit while I'm ahead and point you in the right direction more for information:
  
<a href="https://www.youtube.com/watch?v=Irlw-LGIJO4" target="_blank">Martin Fowler - Microservices@Sydney Yow!</a>

<a href="http://udidahan.com/training/" target="_blank">Udi Dahan - Advanced Distributed Systems Design</a><br/>
*<small>See <a href="#freeVideos">below</a> for free samples of Udi's training videos</small>*

Wait a minute, what's all this got to do with React? And what the heck has it got to do with team structure? Why am I still reading this crap when I could 
just easily watch Game of Thrones?

When you write that react component that displays a product's image, description, seller details, you become, or I should say you ARE a part of
the Product Service team. That team comprises of you plus the backend guys which implement the endpoint that talks to the product database. So you 
the react developer and the backend guys are all in the same Product Service team.

### Conway's Law
*Software architecture follows the organisational structure of people who design it*.

What does this mean? It means that if you are truly practising microservices, the react/angular guys working on the product widget on the front end,
are in the same team as the backend .net/java guys who implement the wcf/webapi product service endpoints.

In reality though, this is often not the case. Companies continue to organise teams by physical tiers i.e. front end team, back end services team, 
mobile team, database team, etc irrespective of service orientation. 

This could be because of political reasons or historical reasons, but make no
mistake, you can't say you are doing microservices unless you are also structurally organised in a service oriented way. Conway's law is both unambiguous and
definitive. If you try to fight it, you will lose.

### <span id="freeVideos">Udi Dahan's Sample Training Videos</span>
If you are inspired by this blog and want to know more about microservices or soa, I highly recommend you watch Udi's training videos. He
teaches a five day course on soa and for a limited time you can watch 2 full days of the course for free. Go <a href="http://go.particular.net/DSY16" target="_blank">here</a>
and use the activation code **DSYU**. This free offer expires 12 June 2016.



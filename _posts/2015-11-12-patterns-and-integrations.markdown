---
layout: post
title:  "Patterns and Integrations"
date:   2015-11-12 22:30:00
categories: personal recurse-center
comments_enabled: true
---

I admit that I am upset that I have to consume an API from a front-end application that reads like methods from something like Java methods. Something like `whatever-domain.com/getBusinessAreas`.

I hear arguments like no one will consume the link directly, or access this address directly. **I am still not convinced it is a good practice**.

# Patterns, patterns, patterns...

Why do we have Patterns in the Software Industry? **to communicate with other developers**.

Just like [design patterns][1], there are other types of patterns. The king of Pattern that was violated by the API mentioned before is called an **Integration Pattern**.

There are a lot of Integration Patterns, just to name some: RMI, SOAP, REST, Message Channels, Pipes, and many others.  The ones I am interested here are [SOAP][2] and [REST][3].

# Choosing between SOAP and REST

##SOAP##

SOAP exposes a service, by using a Service Descriptor (right, the WSDL file). All communication is via this service, and it uses an XML "Envelope". 

SOAP in the web came in disuse. I will not say it is dead, a lot of services still use it. Eg: Jira.

I still believe it might be useful in some scenarios. **But I am quite sure it not designed for UI-centric consumptions** such as browser consumption. So, if a front-end in HTML5 will consume, do not go for SOAP.

## REST ##

It is an Architecture Style. Every RESTful service must respect certain constraints: Client-Server, Stateless, Cacheable, Layered System and Uniform Interface.

A simple example: putting the previous service to REST.

The example from the beginning of this post, could have easily have been named as: `GET /BusinessAreas`. It would retrieve the list of "Business Areas" that exist in the system.

We could have retrieved the details of one specific, like BusinessArea "1" simply by `GET /BusinessAreas/1`.

To create a new "Business Area", we could use `POST /BusinessAreas/`. In the body of the POST we send the details required to create one.

To update the details for a given business area, we could use `PUT /BusinessAreas/1`

At last, in order to delete one Business Area, we could use `DELETE /BusinessAreas/1`. The example would have updated Business Area 1.

#Conclusion 

The REST approach allowed to remove the ugly *camelCase Java like*. But actually, the architectural style significantly improves the readability of the service. Imagine you have more entities, they would all follow the same pattern for access.

**Does it look simpler?** To me at least, **yes**. It looks a lot better thanks to uniform access pattern gained by following a simple standard.

[1]: https://blog.cleancoder.com/uncle-bob/2014/06/30/ALittleAboutPatterns.html
[2]: https://en.wikipedia.org/wiki/SOAP
[3]: https://en.wikipedia.org/wiki/Representational_state_transfer
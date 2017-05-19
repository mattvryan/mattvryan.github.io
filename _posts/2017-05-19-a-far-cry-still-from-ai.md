---
layout: post
title: A Far Cry (Still) From AI
published: True
---
Not long ago I attended a conference at which there were two talks about the current state of AI.  One was extremely optimistic about the state of AI today, and more or less told us that the big breakthroughs in AI were right around the corner.  This person also happened to work in a division of a company focused around selling AI as a service.  The other speaker was much less optimistic, essentially claiming that despite all the advances we've made in AI, the AI we are really hoping for is still very far away.

I tend to agree more with the second speaker.  I'm a fan of AI generally, although not of the Skynet-type of AI.  I studied it in college years ago and even wrote my own neural network.  But I think we are still far from what we would call true AI.

Here's a simple example.  I present to you the following picture of a red custom 1967 Chevrolet Camaro RS.

![Red 1967 Chevy Camaro RS](https://s3.amazonaws.com/seepingmatter/images/1967_chevy_camaro_rs.jpg)

At a single glance I can tell a number of things about this:
* It is a car.
* It is a red car.
* It is a muscle car.
* More specifically, it is a pony car.
* It is made by Chevrolet.
* It is a Camaro.
* It is a 1967 model.
* It is an RS.  (I can tell that even without the stripe around the nose.)
* It is a custom car, not stock.

I don't expect that every human who sees this picture would know all of that information, but someone familiar with first-generation Camaros definitely would know all of that at a single glance.  It would be hard to contend that an entity would have intelligence about this picture if they didn't know at least most of those things.

Microsoft's image recognition software is available online at [CaptionBot](https://www.captionbot.ai).  Cortana runs the show here; she's the one looking at the pictures and providing the intelligence.  Go there and then paste in the link to that picture:  https://s3.amazonaws.com/seepingmatter/images/1967_chevy_camaro_rs.jpg

If you get the same result I get, Cortana tells you:  "I think it's a red car parked on the side of a road."

It is amazing that today we have software that can look at a picture and determine that it is a car, let alone a red car.  So that's amazing, and yet still almost of no real use.  We would hardly call it intelligent.

Can you imagine trying to auto-tag this image based on the adjectives and nouns extracted from it?  You'd get red, and car, and road.  This would be grouped with all red cars, which isn't bad although not super helpful (there must be millions of red cars), but also grouped with things by the side of a road.  Most cars are on roads so this categorization is really not helpful at all.  Meanwhile, the tags you'd really want for this, like "Chevrolet", "Camaro", "1967", "Pony Car", or "Custom Car" wouldn't be obtained at all.

In fact Cortana can't even recognize a picture of herself.  Try https://s3.amazonaws.com/seepingmatter/images/cortana.jpg.  You would think they would at least train Cortana to recognize a picture of herself.

All of this to say, I guess we don't have to worry about the robot apocalypse any time soon.

---
title: Is a Sustainable Web worth talking about?
description: 
date: 2024-01-20
tags:
  - Engineering
---
A few months ago, I worked on a sustainable web project called [Overbrowsing](https://overbrowsing.com) with my friend [Headless Horse](https://headless.horse/). I started the project with an open-minded skepticism that a carbon-friendly website could ever be anything more than a marketing gimmick. I've found that at scale, bloated websites can be hugely inefficient. Reducing their size is a win-win companies and users. Inefficient websites are usually a result of lazy development practices, and making them more efficient also improves user experience for users, and saves a lot of money. 

Developers need to be aware of the keyword *at scale*. It doesn't matter how much your mom looks at your website, no matter how bloated a site is with only a few users, reducing the size of your site won't be worth it. 

For an extreme example at scale, cnn.com has a massively bloated site. The home page loads a series of mp4s weighing several megabytes each, bringing the site weight of the homepage to 48mb on page load. It is tough to find accurate stats, but I pulled some data from an aggregator that claims CNN.com averages 400,000,000 page views per month. 

According to the Paris Climate Agreement, per person, we should be consuming 1.5 tonnes of carbon per year. For context, 1.5 tonnes of carbon is equivalent to your portion of a roundtrip from SFO to JFK. People in countries like the United States on average produce more than 15 tonnes of carbon per year. The operation of cnn.com just through page views to their website produces around 120,000 tonnes per year. 

It doesn't need to be this way. For context, if our site [Overbrowsing](https://overbrowsing.com)had the same number of visitors per month as CNN, we would generate around 680 tonnes per year. This is not a perfect analogy, we expect cnn.com to have a little more going on behind the hood than this small site, but we should be able to avoid 50 MB of audio files on page load. cnn.com if you are reading this, making your site smaller could save you a fortune on your AWS bill. 

Recently, COP 28's website got called out on Twitter for having a "carbon neutral" toggle button on the homepage of the site. If you click the toggle, images on the website will be hidden from the user. The problem was, that even though they were hidden, they were still being downloaded, netting zero energy savings. There are some great write-ups about what the COP website did wrong that is worth a read from [Fershad Irani](https://fershad.com/writing/cop28-uae-a-low-carbon-website-review/) and [Michelle Barker](https://css-irl.info/greenwashing-and-the-cop28-website/). 

At previous jobs working as a developer on larger websites, the idea of making sustainable sites has been brought up multiple times and quickly pushed aside as a "nice to have, but not a priority" item. If there is an expectation of public punishment for laziness, I can see that shooting is at the top of the list of priorities for CMO types. To emphasize again, this is something developers are aware of. If a developer were to make a site follow standard best practices, these issues would exist in the first place. 

A few more points on why this is only a good idea. Other sustainability practices involve swapping carbon loads. Choosing to take public transit vs. driving still involves generating C02. Whereas, when you make a website more efficient, that is energy that will never be used. 

I'll leave you with some recommendations if you're a developer wanting to make your site more sustainable: 

**Reduce image asset size** - This is the most important thing you can do to make your site smaller, and what creates the most bulk on a website. Compress images before caching them, and set max-size restrictions on image uploads in your CMS. Ask yourself as well if you need high-res images at all. 

**Do you even need analytics?** From my informal polling approximately 0% of people know how to use their analytics services or how to correctly interpret the data they collect. Are you just using that analytics tool because you feel like you should? If you absolutely need analytics, you could also try switching to something a little less taxing, and a little less evil, like [Cabin Analytics](https://withcabin.com). 

**Implement Caching** - APIs don't need to be called on page load. Come up with a minimum viable API call, say once a day, and cache your data between calls. 

**Reduce JS load** to what is needed page by page. Make sure JavaScript is only loaded in component-by-component when needed. 

**Green Hosting** - This is the last item on the list for a reason. It is the least important. Why use green energy when you can use *no* energy? While this hosting is probably slightly better than doing nothing, you will always use gas to run electricity no matter what.  

**No JS** - Developers, all users on this planet hate flashy, unnecessary JavaScript. Join the movement and stop developing it. I am not just saying that because I am bad at making JavaScript animations, it's also good for the environment. 

**No Website** - Do we even need a website for that? Save your developer some time, and let's all just go outside.
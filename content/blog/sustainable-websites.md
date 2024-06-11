---
title: On Web Sustainability
description: 
date: 2024-01-20
tags:
  - Engineering
---
My friend [Headless Horse](https://headless.horse/) and I built a sustainable website called [Overbrowsing](https://overbrowsing.com) a few months ago. Before we started the project, I was skeptical. Greenwashing is a hot topic in web development, and I was unconvinced that making a website more sustainable could make a significant enough impact to be anything more than a marketing gimmick. 

  

I discovered that user volume was the crucial variable. At scale, high-traffic websites use an enormous amount of resources. Bloated websites with high traffic can be hugely inefficient. Reducing their size is a win-win for companies, user enjoyment, and the environment. The good news is that inefficient websites are usually the result of failing to follow industry best practices, and improving the efficiency of your website doesn't require reinventing the wheel. Developers need to be aware of the keyword *at scale*. If only a small number of people look at your site, no matter how bloated it is, the impact of making it more efficient will be negligible.

  

CNN.com is a fantastic example of a massively bloated site at scale. The home page loads dozens of mp4s weighing several megabytes each, bringing the site weight of the homepage to 48mb on page load. They probably did this so audio files are pre-loaded in case a user chooses to click on them. There is an adage on the internet that if you make a user wait for even one second, you've lost their attention. 

  

  

To understand how this data loading decision scales, I pulled up some data from an aggregator that claims CNN.com averages 400,000,000 page hits per month. According to the Paris Climate Agreement, individuals should consume no more than 1.5 tonnes of carbon per person per year. For context, 1.5 tonnes of carbon is equivalent to your portion of a roundtrip from SFO to JFK. On average, people living in countries like the United States produce more than 15 tonnes of carbon per year. I live in Europe and work primarily with American companies. At work, I used to joke that if you want to make your company more efficient, the best thing to do would be not to invite me to your conferences. However, if I had worked at CNN, that wouldn't have been the case. CNN.com, solely through page hits to their website, produces around 120,000 tonnes per year. 

  

It doesn't need to be this way. By contrast, if our site [Overbrowsing](https://overbrowsing.com) had the same 400,000,000 visitors per month as CNN, we would generate only 680 tonnes annually. This is not a perfect analogy; we expect CNN.com to have more significant needs than those of our small site. But, 50 MB of audio files on page load is something that doesn't reflect industry best practices and is something that could be easily remedied. CNN.com, if you are reading this, making these changes could save a fortune on your AWS bill. 

  

Last year, COP 28, the infamous environment conference got in some hot water on Twitter. Their website had a "carbon neutral" toggle on its homepage. If you clicked the toggle, images on the website would be hidden from the user. Because of lazy development practices, although hidden, they were still being downloaded in the background, netting zero energy savings. It's highly possible the developer didn't understand they hadn't programmed the site correctly. There are many ways to hide images on a website. Some of them still load content in the background, and some of them don't. It's not immediately obvious which one is which.  There are a few great write-ups about what the COP website did wrong that are worth a read from [Fershad Irani](https://fershad.com/writing/cop28-uae-a-low-carbon-website-review/) and [Michelle Barker](https://css-irl.info/greenwashing-and-the-cop28-website/). 



The COP 28 website drama hopefully reflects a change in expectations about developers' responsibility to consider the environment. Recently, sustainability has been discussed more often at work. The current consensus is that carbon-friendly websites are a "nice to have"but not a major priority. I would like to see increased pushback from the public about bloated websites. CMO types respond very well to public pressure, and I could see the winds turning and sustainability leading the list of must-have items on new development projects. 
  
The biggest benefit of making websites more sustainable is that reducing C02 generation requires no tradeoffs. Other sustainability practices involve swapping carbon loads. Choosing to take public transit instead of driving still generates C02. When you make a website more efficient, you eliminate energy use. 


In the next section, I will explain our technical choices while developing this site. For everyone whose eyes glaze over when they see code, I'll leave you with some general recommendations if you're hoping to make your site more sustainable: 

  
**Reduce asset size**. Websites are weighted based on how much is downloaded each time a user visits a site. JavaScript is often blamed for bloating sites, but the biggest bulk culprit is usually in image, video, and audio assets. Reducing asset size is an important thing you can do to make your site smaller. Compress images before caching them. Set max-size upload restrictions in your CMS. Ask yourself if you need high-res images at all. While building Overbrowsing, I tested some client work I'd recently built on the [Website Carbon Calculator](https://www.websitecarbon.com). A few of the sites I built scored a low rating. When I looked into the problem, I discovered that the client had uploaded large images to the site. I realized developers should always set limits in my CMS and compress images that are uploaded. 

**Implement Caching** - APIs don't need to be called on page load. Come up with a minimum viable API call, say once a day, and cache your data between calls.   

**Do you even need analytics?** From my informal polling, approximately 0% of people know how to use their analytics services or correctly interpret the data they collect. Are you just using that analytics tool because you feel like you should? If you need analytics, you could also try switching to something less taxing and evil, like [Cabin Analytics](https://withcabin.com). 

**Reduce JS load** to what is needed page by page. Make sure JavaScript is only loaded component by component when required. 

**Green Hosting** - This is the last major item on the list for a reason. It is the least important. Why use green energy when you can use *no* energy? While this hosting is better than doing nothing, you will always use gas to run electricity no matter what.  


**No JS** - Users hate flashy, unnecessary JavaScript. Join the movement and stop developing it. I am not just saying that because I am bad at making JavaScript animations; it's also good for the environment. 

**No Website** - Do we even need a website for that? Save your developer some time, and let's all go outside.

---

  

# How we built Overbrowsing 

  
When developing Overbrowsing, we had two primary goals. Our first goal was to make the site as small as possible, and our second goal was to store our data using the Are.na API. My first instinct was the obvious one. Make an HTML-only site with a couple of small lines of JavaScript to pull in the API data. After some trial and error, we realized that it didn't actually matter at all if we used only HTML or a JavaScript library because we were compiling JavaScript to a static site anyway. 

The API was the biggest source of weight on the site. Because we knew we wanted to occasionally update the site, we set the API to be called once a week, depending on Air Quality Index (AQI). Our workflow is triggered via a YAML file in two ways:

1. **When there's a push to the `main` branch**.

2. **On a schedule**, specifically at 7:30 AM every Monday. The scheduled build only executes scheduled build stops if the AQI is below a certain threshold, the scheduled build stops.  

```yaml

- name: Check AQI and stop if necessary

  run: |

    aqi=$(curl -s "https://api.openweathermap.org/{myAPI}')

    echo "AQI today is $aqi"

    if [ "$aqi" -lt 2 ] && [ "${{ github.event_name }}" == 'schedule' ]; then            

      echo "AQI today is $aqi. Stopping the workflow."

      exit 78  # 78 is a special exit code indicating the workflow should be stopped

    else

      echo "AQI today is $aqi. Workflow can continue."

    fi

```

If the build executes, the APIs are called and saved in a JSON file. The Are.na API had a huge amount of data, but we only needed a small amount for Overbrowsing. Therefore, before saving the data, we reduced it to only a few items. 

```javascript
async function processChannel(channel) {
	const data = await fetch(`https://api.are.na/v2/channels/${channel.name}`)
  .then(resp => resp.json());
  const category = channel.category;

  ...

  const mergedJson = [];

  await Promise.all(data.contents.map(async element => {
    const reducedData = {
      title: element.title,
      content: element.content,
      contents: element.contents,
      id: element.id,
      image: element.image,
      date: element.updated_at,
      category,
    };

    if (element.source) {
      reducedData.source = {
        url: element.source.url,
        title: element.source.title,
        provider: element.source.provider,
      };
    }

...

  return mergedJson;
}
```

Before saving images, we compressed them massively, which adds a nice visual effect to the website. Choosing how to compress the images took a lot of trial and error. This configuration resulted in the smallest, best-looking images. 

```javascript
async function processAndOptimizeImage(inputFilePath, outputFilePath) {

await sharp(inputFilePath)
	.resize(450) 
	.toFormat('png')	
	.png({	
		dither: 0,	
		colors: 3, 
		quality: 1,	
	})

.toFile(outputFilePath);
}
```


There are a lot of fun little easter eggs on this site. The background changes based on the air quality that day. The favicon changes based on the moon's phase. We needed to make many tiny API calls and used the same method as with Are.na. Here is the code: 

```javascript

async function fetchAqiAndSave() {

try {

	const [aqi, moonPhase] = await Promise.all(
		[getAirQualityData(), getMoonPhase()]
	);
	const aqiValue = aqi.list[0].main.aqi;
	const airQualityDescription = getAirQualityDescription(aqiValue);
	const backgroundColor = calculateBackgroundColor(aqiValue);
	
	const data = {
		aqiValue,
		airQualityDescription,
		backgroundColor,
		moonPhase,
	};
	...
}

async function getMoonPhase() {

try {

	const response = await fetch('https://api.openweathermap.org/{myAPI}');
	
	const data = await response.json();
	
	const moonPhaseEmoji = 
	['🌑', '🌒', '🌓', '🌔', '🌕', '🌖', '🌗', '🌘']
	[Math.floor(data.daily[0].moon_phase * 8)] || '🌎';
	
	return moonPhaseEmoji;
	
	} 
	
	catch (error) {	
		console.error('Error fetching moon phase:', error);
		throw error;
	}
}

```

And it's output:

```json
{
	"aqiValue": 2,
	"airQualityDescription": "fair",
	"backgroundColor": "#272a24",
	"moonPhase": "🌖"
}
```

Overbrowsing is open-sourced on GitHub](https://github.com/overbrowsing/overbrowsing.com). If you want to learn more, I highly recommend checking it out. There were so many interesting little choices made when developing this site that I have yet to mention, including building the enormous homepage animation from scratch with D3.js to scrape off tiny amounts of weight. The resources themselves are also highly worth looking at. Compiling them was a labor of love done mostly by [Headless Horse](https://headless.horse/) and the resources and a lot of stuff that changed how I view the internet, including [Black Google](ecoiron.blogspot.com/2007/01/black-google-would-save-3000-megawatts.html). This blog talks about the enormous amount of energy the search engine would save if it swapped the default color of its homepage. You can also review the resources or follow Overbrowsing on [Are.na]( https://www.are.na/overbrowsing/channels).
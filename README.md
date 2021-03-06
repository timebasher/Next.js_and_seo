# This is my Tutorial on how to do SEO with a Next.js App

Since Next.js Apps are serveside rendered. Google CrawlBot got issues with catching the all the Data, your App provides.
Next.js uses a special Method call getPropsOnInitialLoad, this method provides a Component with data before the initial-load. The data catched, wil provide the Component with data, to render the whole Content on the Server.

`Problem is:` Google Crawl Bot cannot catch this data correctly (even it is very clever), this is a huge Problem when it comes to SEO

You can provide the Crawl Bot an Sitemap.xml and robots.txt like every other Website, to help 'him' out, this will be generated dynamicly
For this we are using [a third Party Package](https://github.com/ekalinin/sitemap.js)
  
----

Inside the sitemap.xml we specify which routes to index (url), the change freqency (changefreg) and the priority of this Page (priority)

### This example is taken from the offical Docs:

```javascript
var sm = require("sitemap");
// Creates a sitemap object given the input configuration with URLs
var sitemap = sm.createSitemap({ options });
// Generates XML with a callback function
sitemap.toXML(function(err, xml) {
  if (!err) {
    console.log(xml);
  }
});
// Gives you a string containing the XML data
var xml = sitemap.toString();
```

### Since we are using [Express]('https://www.npmjs.com/package/express') as our Sever:

```javascript
var express = require("express"),
  sm = require("sitemap");

var app = express(),
  sitemap = sm.createSitemap({
    hostname: "http://example.com",
    cacheTime: 600000, // 600 sec - cache purge period
    urls: [
      { url: "/page-1/", changefreq: "daily", priority: 0.3 },
      { url: "/page-2/", changefreq: "monthly", priority: 0.7 },
      { url: "/page-3/" }, // changefreq: 'weekly',  priority: 0.5
      { url: "/page-4/", img: "http://urlTest.com" }
    ]
  });

app.get("/sitemap.xml", function(req, res) {
  sitemap.toXML(function(err, xml) {
    if (err) {
      return res.status(500).end();
    }
    res.header("Content-Type", "application/xml");
    res.send(xml);
  });
});

app.listen(3000);
```

This is a simple setup.
But image you got an Online Shop wich provides Data from your DB.
We can provide all our ShopItems to the Sitemap.xml, by filtering out the Data from
the Database and provide it to GoogleCrawler. So you don't need provide all Routes on your own.

## Crawl from the Database

```javascript
SHOPITEMS.find({}, "_id").then(SEO_TITLE => {
  SEO_TITLE.forEach(item_id => {
    sitemap.add({
      url: `/shop/items/${item_id}`,
      changefreq: "daily",
      priority: 1
    });
  });
});
```

### Providing Robots.txt

Create a robots.txt file and porvide it as a static File in ../static/robots.txt.

1.) Add an express Route

```javascript
server.get("/robots.txt", (req, res) => {
  res.sendFile(path.join(__dirname, "../static", "robots.txt"));
});
```

path.join(\_\_dirname) will directly point to the directory you are using

2.) content of this File

```javascript
User-agent: *
Allow: /books/builder-book/
Disallow: /admin
```

User-agent: this tells the indexing Bots to crawl all Routes
Allow: ...wich are behind a specific Route bsp /shop/items/
Disallow: ...this disallow a specfic Route for example /admin

## Setup initial SEO_Data for every Page

There is also a way to provide inital SEO Page to your \_document-File or to your Layout HOC with [next-seo]('https://www.npmjs.com/package/next-seo')
This Package simple Provdies default-Metadata from your Website like:

```HTML
<meta name="description" content="Entdecken, shoppen und einkaufen bei Amazon.de: Günstige Preise für Elektronik &amp; Foto, Filme, Musik, Bücher, Games, Spielzeug, Sportartikel, Drogerie &amp; mehr bei Amazon.de">
<meta name="keywords" content="xxxx.de, Bücher, Elektronik, Filme, Games, Sportartikel, Schuhe, Spielzeug, Drogerie, Musik, MP3">
```

### Create your DEFAULT_SEO and save it to a variable

```javascript
const DEFAULT_SEO = {
  title: "Next.js SEO Plugin",
  description: "SEO made easy for Next.js projects",
  openGraph: {
    type: "website",
    locale: "de_IE",
    url: "xxxxx",
    title: "your Title",
    description: "SEO made easy for Next.js projects",
    image: "xxxxx",
    site_name: "xxxx",
    imageWidth: 600,
    imageHeight: 600
  },
  twitter: {
    handle: "@xxxx",
    cardType: "summary_large_image"
  }
};
```

### The create a `<NextSEO />` component and fill the `config-Prop` with the `DEFAUL_SEO`

```javascript
import NextSeo from 'next-seo';

render() {
    return(
      <NextSeo config={DEFAULT_SEO} />
    )
}
```

if you want to Change the Data on a specific Site, you can Import the ``NextSeo` Component and update the config Data:

````javascript
import NextSeo from 'next-seo';

export default () => (
  <div>
    <NextSeo
      config={{
        title: 'About us',
        description: 'an other Description'
      }}
    />About us
  </div>
);


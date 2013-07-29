TrafficTracker v1.5
==============
A simple PHP class created and maintained by [@adamdehaven](http://about.adamdehaven.com/) to track and log all of a website's traffic using `PHP` and `MySQL`. The class utilizes cookies set by the typical Google Analytics tracking code initialization. In addition, if you deploy Google AdWords PPC campaigns to drive traffic to your website, the class will (along with custom URL parameters added to your destination URLs) track the hits from your AdWords campaigns.

## HOW IT WORKS
The class writes to a MySQL database table, saving values for every page visit for the following attributes:

Key|Description|Value
---|---|---
`id`|Auto-increment id|**_example:_** `987654`
`identifyUser`|Unique ID for each visitor, generated by Google Analytics.|**_example:_** `1234567890`
`medium`|Every referral to a website has a medium of arrival|`organic` - Unpaid search.<br>`cpc` - Cost per click, i.e. paid search.<br>`referral`- Referral.<br>`email` - Name of a custom medium you created.<br>`none`- Direct visits. 
`source`|The origin (or source) of the referral|<br>`direct`- Visits from people who typed your URL directly into their browser, or who had bookmarked your site.<br>`google` - The name of a referring search engine.<br>`example.com` - The name of a referring site.
`content`|Identifies a specific link or content item, such as a specific page on a referring site or link in a custom campaign.|**_example:_** `/page-with-link-to-you.html`
`campaign`|The name of the referring AdWords campaign or a custom campaign that you have created.|**_example:_** `July Banner Ad`
`keyword`|The **actual exact search term(s)** the user entered into a search engine to eventually reach your website (excluding Google AdWords keywords)|**_example:_** `bike storage hanger for wall`
`pageViewed`|The page on your website viewed for the recorded visit.|**_example:_** `http://www.example.com/page-one.php`
`adwordsKeyword`|The keyword or keyword phrase in your Google AdWords campaign that was triggered by the visitors actual search term(s) (see `keyword`).|**_example:_** `bike storage hanger`
`adwordsMatchType`|The type of match that triggered your AdWords keyword.|`e` - exact<br>`p`- phrase<br>`b` - broad
`adwordsPosition`|The position on the page that your ad appeared in, with a value such as `1t2`, which is equivalent to page 1, top, pos 2.|**_example:_** `1t2`
`firstVisit`|The timestamp indicating the visitor's **first** visit to your website when their cookies were created for your website.|date with format: `0000-00-00 00:00:00`
`previousVisit`|The timestamp for the visitor's **last** visit to your website, besides the current one.|date with format: `0000-00-00 00:00:00`
`currentVisit`|The timestamp of the visitor's **current** visit to your website which is being logged.|date with format: `0000-00-00 00:00:00`
`timesVisited`|The number of times the visitor viewed a specific page on your site.|**_example:_** `3`
`pagesViewed`|The total number of pages visited.|**_example:_** `12`
`userIp`|The IP address of the visitor|**_example:_** `10.0.0.1`
`timestamp`|The timestamp of the logged visit.|date with format: `0000-00-00 00:00:00`

## REQUIREMENTS
* Web Server / Hosting Account running PHP 5+.
* Website pages capable of running `PHP` server-side code (usually ending in `.php`) as well as JavaScript.
* [Google Analytics](https://www.google.com/intl/en_ALL/analytics/index.html) tracking code initialized on all website pages utilizing the asynchronous snippet.
* If using Google AdWords, you must append each keyword's Destination URL with the [custom URL parameters](#setup-google-adwords-destination-urls-if-using-adwords) outlined below.

## SETUP
Create a new table on your database called `trafficTracker` using the SQL statement below:
```php
CREATE TABLE IF NOT EXISTS `trafficTracker` (
  `id` int(10) NOT NULL auto_increment,
  `identifyUser` varchar(65) character set utf8 collate utf8_unicode_ci NOT NULL,
  `medium` varchar(50) character set utf8 collate utf8_unicode_ci NOT NULL,
  `source` varchar(60) character set utf8 collate utf8_unicode_ci NOT NULL,
  `content` varchar(150) character set utf8 collate utf8_unicode_ci NOT NULL,
  `campaign` varchar(50) character set utf8 collate utf8_unicode_ci NOT NULL,
  `keyword` varchar(100) character set utf8 collate utf8_unicode_ci NOT NULL,
  `pageViewed` varchar(350) character set utf8 collate utf8_unicode_ci NOT NULL,
  `adwordsKeyword` varchar(75) character set utf8 collate utf8_unicode_ci NOT NULL,
  `adwordsMatchType` varchar(10) character set utf8 collate utf8_unicode_ci NOT NULL,
  `adwordsPosition` varchar(5) NOT NULL,
  `firstVisit` timestamp NOT NULL default '0000-00-00 00:00:00',
  `previousVisit` timestamp NOT NULL default '0000-00-00 00:00:00',
  `currentVisit` timestamp NOT NULL default '0000-00-00 00:00:00',
  `timesVisited` int(4) NOT NULL,
  `pagesViewed` int(6) NOT NULL,
  `userIp` varchar(40) character set utf8 collate utf8_unicode_ci NOT NULL,
  `timestamp` timestamp NOT NULL default '0000-00-00 00:00:00',
  PRIMARY KEY  (`id`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8 ;
```

Open the `class.TrafficTracker.php` file and edit the top section variables shown here:
```php
/* -------------------------------------------------------------------------
------------------------------- SET DEFAULTS -------------------------------
--------------------------------------------------------------------------*/
private $urlPrefix          = 'http'; // Set URL prefix for your website.
private $replaceInUrl       = array('?customer=new','?version=mobile'); // strip out any custom strings from URL.
private $myIp               = array('10.0.0.1'); // Put IP addresses you would like to filter out (not track) in array.
private $reportingTimezone  = 'America/Kentucky/Louisville'; // http://www.php.net/manual/en/timezones.america.php
private $dateFormat         = 'Y-m-d H:i:s'; // Preferred date format - http://php.net/manual/en/function.date.php
private $cookieExpire       = 30; // Set number of days for AdWords tracking cookies to be valid.
```

Simply include the class above the `<head>` of your `PHP` page with:
```php
<?php include_once('../path/to/class.TrafficTracker.php'); ?>
```
Next, initialize the class with arguments:
```php
<?php $trafficTracker = new TrafficTracker($dbHost, $dbUsername, $dbPassword, $dbDatabase, $trackPrefix, $deleteRollingDays); ?>
```
The first 4 arguments define the connection to your database. The 4th argument `$trackPrefix` will be used as the prefix for your Google  AdWords destination URL custom parameters, as well as for the cookies set in the visitor's browser. The default value (if unset) is `ttcpc`. The 5th argument `$deleteRollingDays` is an **integer** value that designates the number of rolling days to go back before deleting logged visits. The default value for `$deleteRollingDays` is 30, meaning that any record created greater than 30 days ago from *now* will be deleted.

#### Setup Google AdWords Destination URLs (if using AdWords)
If using Google AdWords campaigns to drive traffic to your website, add the following parameters to the end of **each** destination URL, changing `ttcpc` in the code below to the value of `$trackPrefix` if you defined a different string.
```
?ttcpc=true&ttcpc_kw={keyword}&ttcpc_pos={adposition}&ttcpc_mt={matchtype}
```
The above URL parameters will cause AdWords to dynamically insert the keyword, ad position, and match type into your URL as values for their allocated parameter.

* `{keyword}` - For the search sites, the specific keyword that triggered your ad; for content sites, the best-matching keyword.
* `{adposition}` - The position on the page that your ad appeared in, with a value such as `1t2`, which is equivalent to page 1, top, pos 2.
* `{matchtype}` - The matching option of the keyword that triggered your ad: exact, phrase, or broad.

*Note: If your AdWords destination URLs already have parameters (i.e. `http://www.example.com/index.php?view=3`) then just add the TrafficTracker parameters to the end, like this:
```
http://www.example.com/index.php?view=3&ttcpc=true&ttcpc_kw={keyword}&ttcpc_pos={adposition}&ttcpc_mt={matchtype}
```

## USAGE EXAMPLE

1. Include the class.
2. Initialize the class with your database connection details; optionally changing 'ttcpc' to your desired prefix, and changing the default rolling 30 days delete to 60 days instead.

The top of each page on your site will look similar to this:
```php
<?php
include_once('../path/to/class.TrafficTracker.php'); // Include class
$trafficTracker = new TrafficTracker('your_host', 'your_db_username', 'your_db_password', 'your_database', 'ttcpc', 60);
?>
```

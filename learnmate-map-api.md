# LearnMate Wordpress Website

The wordpress website has the learnmate map plugin (https://github.com/learnmatetutors/learnmate-map-wordpress) which is a custom made plugin for the learnmate website. 

# Learnmate Map Plugin

## Search Box Function

In `search.js` we ensure that:
- When a customer does a search, they fill out the search boxes. 
- When they start typing 3 letters in the address field:
    - The google maps plugin is injected into the page
    - The dropdown for the address shows and we are charged for each keystroke after
- When the search button is clicked
    - Save all the search fields in the local browser

## Search Results Page

In `search-results.js`, the functions of this page are split into:
- Tutor results
- Saved Tutors
- Returning Student/Parent Question

The search results are shown for the suburb, subject, selection for in-person/online that was done on the search page.

These selections are stored in localstorage.

- The user selects a tutor on the search result page
    - In `search-results.js` we request the webpage for that tutor (eg. https://learnmate.com.au/meet-our-tutors/deep/)
    - All scripts are deleted from the data received for security reasons
    - The contents after the second blue line are displayed to the user (see `search-results.js` for more information)
- The user saves a tutor
    - The object of this tutor is added to localstorage
    - There is also a removal process which removes the tutor from localstorage and updates the UI
- The in person search function has a map which zooms into available tutors. The rules for this zoom levels are determined as follows:

```
IF 
    there are less than 10 tutors in a 2km radius => zoom to all tutors with distance less than or equal to 5km
OR 
    If there are more than 10 tutors in a 2km radus => zoom to all tutors with distance less than or equal to 2km
ELSE 
    If there are no tutors within 5 km radius, zoom to all tutors
```

- The online search results page is a clone of the in-person search results page:
    - The param online is set to `true` in the request.
    - It hides the map.
    - It randomises the tutors using the algorithm located in `search-results.js`.
    - All other functionality for saving tutors and getting tutors is the same.

## Injecting Headers and Footers

There is some code that needs to be injected into each page. There are two types of these that exist:
- Only inject the html code into some pages with a given url (eg. all pages containing /meet-our-tutors/)
- On all pages

There was no plugin which was able to meet these requirements. Hence, we needed to bake this into the map plugin.

# Other Website Tools

## CDN

The learnmate website uses cloudfront for the CDN. There are three levels of caching in place:
- Sucuri caches everything, but it seems cache invalidation is not handled correctly on their side. Hence, requests are not generally handled on sucuri servers.
- Amazon CloudFront caches all website assets including full pages, images and scripts. This is located in https://d3e4w3i6bal5ta.cloudfront.net/
- W3 Total Cache does some local caching on the server.

In order to clear the cache:
- Hover on W3 Total Cache in the top bar
- Clear Cache (locally)
- Clear CDN Cache (on CloudFront)
- We do not need to clear Sucuri Cache - Invalidation has never been a problem here
- Cache expiry is set to one day

## Staging Site

In order to test plugins and wordpress site updates, a clone of the actual learnmate website exists https://learnmate.com.au/1589700928403/

In order to clone the current website to the staging site (before we can test updates and plugins):
- In the sidebar on the dashboard click on WP Staging Plugin
- Follow the process to clone the current wordpress website
- Access the staging site https://learnmate.com.au/1589700928403/
- Log in with same username and password as the production website
- Deactivate plugins which are mentioned in [404 Issues](#404-issues)
- Make sure plugin updates do not cause site problems
- Make any changes in staging separately on production as well

## 404 Issues

There have been 404 Issues brought to my attention which occur after an admin logs into the website. While I haven't been able to trace the root cause of the issue yet, disabling these plugins seems to eliviate the issue:
- W3 Total Cache
- WP Simple Pay Pro 3
- Query Monitor

There are also some unrelated 404 issues which require further investigation when we go to edit certain text fields on tutor pages. This is not fixed by disabling the plugins above.

## Favicon

Changing the favicon of the website must be done in two places:
- In customise menu option on the left hand side
- In Yoast SEO Plugin 

After changing it in these, the new favicon should be live on the learnmate website. 

## Top Bar on LearnMate

The top bar is generally fixed to the top of the page (the code for this is in the `header.html` which is injected into all pages on the website).

There is a special rule to this on the 15 ATAR Success Tips page. Here, the top bar is no longer fixed since the quiz plugin scrolls to the top of the quiz each time a user inputs the answer to a question.

Hence, the top bar appears on top of the quiz, hiding the contents underneath. We had to make the top bar not fixed on this page in order to accomodate for this. 


## School Success Guide Popup

A plugin called Yeoli Exit Popup manages the school success guide popup. This popup occurs when the user tries to exit the parent/student enquire form. 

In order to edit this popup, please see the Yeloni Exit Popup documentation.

## Affiliate Submissions

Affiliate WP looks for a ref in the url to register success. In order to fake this, I have used the email address of the form submitter as the ref, which means if the user had been referred, he/she would be marked as a successful referral.

Example:
`https://learnmate.com.au/returning-student-success/?ref=TEST@TEST.com`

See the Affiliate WP Plugin for more details.

## Site Speedup

I have found that the follow query helps with site speedup. It deletes entries which are not necessary.

```sql
DELETE 
FROM `wp_ycsm_options` 
WHERE `autoload` = 'yes'
AND `option_name` LIKE '%transient%';
```
Proceed with caution but the following can also help:
- Delete the line with the name "cron" in wp_ycsm_options (it can get quite lengthy)
- Update wordpress version and plugins

# Getting Started - Setting Up A Local Copy of The Wordpress Website

First, download a fresh copy of the website from wordpress and the database from PHP My Admin. I cannot host it on github since there are regular updates to it that are going to be done directly on the site.

4 Things You Must Do To Remove SSL

- In the database in the wp_ycsm_options table the first two entries are set to https://learnmate.com.au. Change these to http://localhost:8000
- Edit the old .htaccess file, put this in its place:

```
    # BEGIN WordPress
    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.php [L]
    </IfModule>
    # END WordPress
```
- You should now be able to access http://localhost:8000/wp-admin
- Deactivate the really simple ssl plugin



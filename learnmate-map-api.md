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
- WP Total Cache does some local caching on the server.

In order to clear the cache:
- 


- The staging site location and how to use/access it.
- The favicon installation and how to use/access it.
- The bespoke nature of the top bar on LearnMate (15 ATAR Success Tips)
- The bespoke nature of the enquire form SCHOOL SUCCESS GUIDE popup that occurs when you try to exit the enquire form page.
- The bespoke nature of AffiliateWP and how student affiliate submissions are processed.
- The staging site and if applicable how to do site speedup, updates etc.


# Getting Started

## Setting Up A Local Copy of The Wordpress Website

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


## Things to try when clearing database (to make website faster)

```sql
DELETE 
FROM `wp_ycsm_options` 
WHERE `autoload` = 'yes'
AND `option_name` LIKE '%transient%';
```

Delete the line with the name "cron" in wp_ycsm_options
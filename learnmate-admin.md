# Learnmate Admin Console

## Required Knowledge Before Starting

- A thorough understanding of how to set up and administer a PostgreSQL database, as well as the extension PostGIS (which is used to serve map queries from the WordPress plugin).
- A thorough understanding of Ruby on Rails
- A thorough understanding of ReactJS and ES6 JavaScript
- An idea of how Redux works on a conceptual level, knowing how to use it extensively is a bonus, but not completely required
- A conceptual understanding of REST APIs
- How token-based authentication works between a client and server
- An intermediate understanding of ES5 Javascript & JQuery (Used on the WordPress plugin)
- An intermediate understanding of WordPress and PHP
- How to deploy to Heroku and understand log/configuration messages
- An intermediate understanding of how the React-on-Rails gem works, and how it ties the client and server together
- How to use webpack and yarn to bundle code for production
- An understanding of how to unit test with Rspec
- An understanding of paperclip/imagemagick (for storing images)

## Introduction

### What is this codebase?
From the website (learnmate.com.au):

> LearnMate is Australia’s leading tutoring agency offering private lessons in all high school subjects (Years 7 – 10) and HSC, IB, QCE, SACE, WACE & VCE subjects (Years 11 and 12) including English, maths, science, humanities, foreign languages, and so much more.

In short, Learnmate is a vehicle for effectively connecting students with tutors. \
The main way we do this is via our search function (learnmate.com.au/search). A prospective student or parent will visit the search, input their postcode or locality, and be returned with a list of tutors closest to them. \
It's probably best you take 5-10 minutes to familiarise yourself with how the search function works on a user level, as it is crucial to understanding the point of this codebase.\
Before this application existed, the search function was a WordPress plugin tacked on to the main Wordpress site. It was effective to start with, but as Learnmate grew as an organisation, it quickly became a bottleneck in terms of speed and flexibility to add new features. \
We decided in November 2017 to basically start from scratch, and engineer an API based solution to interface with WordPress, and abstract the functionality of searching and adding tutors to the map from WordPress, and into a totally new application. \
From there, we have added several new features, including the ability to search for online tutors, the ability to interface with Teachworks to dynamically remove unavailable tutors from the search, and several other things that Dmitri will familiarise you with (more about Teachworks later on in this guide).
\
\
**Quick Start Guide**\
If using MacOSX, all of these programs can be installed using homebrew. Ditto with linux and apt/rpm/pacman/etc. This has not been tested on a Windows development environment, so proceed at your own risk if you are deciding to go down that road. 

### Dependencies:

- PostgreSQL v10.3
- Heroku CLI
- Ruby v2.5.0 (Use Rbenv or RVM to install locally to a directory)
- Bundler
- Yarn
- NodeJS
- PostGIS extension

### Setting Up Ruby on Rails Application

1. Get NodeJS (8.0) and Ruby 2.5.0 on a linux or mac distribution. If you run windows, you must run a linux subsystem. There is currently no support for deploying on Windows. Use rvm (not rbenv) to install ruby
   NOTE: If you wish to have a local database, please install postgresql as well. You will also need the POSTGIS Extension for postgresql. If you dont wish to go down this route, feel free to make a hosted copy of the production database and use that!
   Install rvm using brew/homebrew. Remember its ruby 2.5.0!
2. Clone the repo from heroku or github.
3. Run `yarn install` and `bundle install` to install the dependencies. Make sure you run these in the root folder of the project
4. Modify the database details in the database.yml file.
5. Run the server!

For development on ruby on rails:
1. In one command window, run ```rails s```, which will host the main server on localhost:3000
2. Then in another command window, run ```webpack-dev-server``` which will run the development server in the background with hot module reloading enabled. You may then modify any of the react code and if you visit localhost:3000 in the browser, it will auto refresh on saving!
3. For the database, download a backup of the database from heroku, then use a program like DBeaver to import it into your local machine.

### To Deploy to Heroku:
1. ```git push heroku master```
2. If you have made new migrations in the database, be sure to use ```heroku run rails db:migrate```
3. Please go here: https://chromedriver.chromium.org/. Copy the latest version number for chrome's latest STABLE release (eg. 81.0.4044.69).
4. Go into settings in the heroku dashboard and update the config var ```CHROMEDRIVER_VERSION``` to the version on the ChromeDriver google site.

### To Insert a New Field
1. Run ```rails generate migration add_field_to_model field:type```
2. ```rails db:migrate```
3. Make sure you run ```heroku run rails db:migrate``` when you deploy to rails

### Locally trying automation functionality

1. Follow this guide if on windows: https://gist.github.com/danwhitston/5cea26ae0861ce1520695cff3c2c3315
2. Install webdriver and chromedriver (make sure it's in the PATH variable)
3. Launch the selenium server using ```java -jar .\selenium-server-standalone-3.0.1.jar -port 4445```

### Setting up a local copy of the wordpress Application

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
- Delete the SSL javascript redirect. This is located in the Insert Headers and Footers plugin options inside the settings tab on the left. You will see a piece of code like the following:

```javascript
<script>
console.log(location.protocol);
if (location.protocol != 'https:') {
 location.href = 'https:' + window.location.href.substring(window.location.protocol.length);
}
</script>
```

Delete this and you should be set!


## Things to try when clearing database (to make website faster)

```sql
DELETE 
FROM `wp_ycsm_options` 
WHERE `autoload` = 'yes'
AND `option_name` LIKE '%transient%';
```

Delete the line with the name "cron" in wp_ycsm_options

# LearnMate MAP Api Wordpress Plugin

## Getting Started

### Setting Up A Local Copy of The Wordpress Website

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


### Things to try when clearing database (to make website faster)

```sql
DELETE 
FROM `wp_ycsm_options` 
WHERE `autoload` = 'yes'
AND `option_name` LIKE '%transient%';
```

Delete the line with the name "cron" in wp_ycsm_options
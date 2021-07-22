# Learnmate Gatsby Configuration

Welcome to a quick guide on what is happening with the Gatsby frontend used at Learnmate.com.au

Gatsby has been used on the front end in order to create a reliable and fast fexperience for potential and existing clients of Learnmate. Data is sourced from several locations. Pages built with flexible content, forms, blog posts and knowledge base article all pull data from a wordpress server. Tutors pages, checkout process and the search engine are sourced from the app built in heroku. All content is compiled at build time aside from the search results.

### Required Wordpress plugins

First up there is abespoke plugin built which will build all necessary custom post types in wordpress. This Plugin is called Learnmate Gatsby Addons

- Advanced Custom Fields PRO = This is used for all custom content in our pages. The Flexible content field has been ised extensively to allow admin to build pages with various types of content.
- Gravity Forms = Handle form creation and submission via the wordpress Rest API.
- WPGraphQL Yoast SEO Addon - used to supply some data from YOAST in our post objects via graphql.
- Deploy with NetlifyPress = Allows compilation on Netlify via the wordpress admin interface.   
- SVG Support - Allows for SVG files to be uploaded to wordpress media.
- WP GraphQL - Generates a GraphQL endpoint on our wordpress installation at /graphql.
- WP Gatsby = Required for our gatsby connection to wordpress.
- WPGraphQL for Advanced Custom Fields - Used to allow ACF fields to be available in GraphQL.
- WPGraphQL for Gravity Forms - Allows for access to Gravity Forms objects via GraphQL.
- Yoast SEO

### last Tested Node Versions

Node v15.14.0
Yarn 1.22.10
Netlify CLI 3.31.7
Gatsby CLI 3.5.0

### Start developing

Ensure you have a .env.development file in the root of your folder with the GATSBY_GRAPHQL_URL and GATSBY_SITE_URL fields. there shuold be an example of this in the repo.

Navigate into your project root and run the following

```shell
yarn install
gatsby develop
```

### Clearing Development Cache

You may find the latest wordpress updates are not being pulled into your development environment. In such a case, try cleaning Yarns cache with the following command.

`yarn clean`

or `gatsby clean`


### Installing a new package

Please ensure you use yarn to install all packages.

eg.

`yarn add <package-name>`

or for development only packages

`yarn add -D <package-name>`


### Essential Links

    - [Guides](https://www.gatsbyjs.com/tutorial/?utm_source=starter&utm_medium=readme&utm_campaign=minimal-starter)

    - [API Reference](https://www.gatsbyjs.com/docs/api-reference/?utm_source=starter&utm_medium=readme&utm_campaign=minimal-starter)

    - [react-slick](https://react-slick.neostack.com/)

    - [gatsby-source-wordpress](https://www.gatsbyjs.com/plugins/gatsby-source-wordpress/)

    - [gatsby-plugin-image](https://www.gatsbyjs.com/plugins/gatsby-plugin-image)

    - [gatsby-background-image](https://www.gatsbyjs.com/plugins/gatsby-background-image/)

    - [wp-graphql](https://github.com/wp-graphql/wp-graphql)

    - [wp-gatsby](https://github.com/gatsbyjs/wp-gatsby)

    - [graphql](https://graphql.org/)

    - [html-react-parser](https://www.npmjs.com/package/html-react-parser)

    - [react-helmet](react-helmet)


### Test Deploy Process from Command Line

To emulate the deploy process in a development environemt run the following. This can help in diagnosing deployment issues.

`gatsby build && gatsby serve`


### Netlify SErverless Lambda Functions

We are using a lambda function as a middle man for serveral items in this project. This functions can be found in `src/lambda` and is compilied to `serverless`. To compile these files run `yarn lambda`.

#### Files used in serverless folder

1. file-upload - This script will generate a signed URL to allow file uploads into Amazon S3 via the file upload fiels in Gravity Froms. As we cannot upload files via url encoded using Ntlify's Serverless function, we are uploading files straight to S3 from the browser, and saving the URL back to a hidden field in Gravity Forms.

2. new-gf-entry - This file will take the fields entered into a gravity forms field, add the required authentication for make a post to wordpress and forward it on, the response from Wordpress is then sent back to the client.

3. stripe-proxy - A proxy to stripe which will add necessary authentication fields to create a credic card token in stripe. This is use in our paymeny gateway.


### Testing Serverless Functions in development

In order to test our serverless functions, we can run the following with the netlify CLI installed. This will emulate the lamdda endpoints locally.

`netlify dev`

To test compilation of our serverless files (found in `src/lamda/*`), run the following. This mode will give better error message than the above method.

`yarn lambda`
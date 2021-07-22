# Required Knowledge Before Starting

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

# Codebase Overview and Functionality

## Learnmate Admin API Documentation

These API routes are defined in the `routes.rb` file in learnmate-admin rails application. They can be called either from the React Admin Application or directly from the learnmate website, hence I have split them up into these categories:

## Website APIs

These APIs can be called from https://learnmate.com.au only in the production build of the learnmate-admin website.

These APIs can be called from http://localhost:8000 only in the development build of the learnmate-admin website.

_Please see [api_helper.rb](https://github.com/learnmatetutors/learnmate-rails/blob/master/app/helpers/api_helper.rb) for further details on CORS policies_


#### Note on all tutor api endpoints

A new field has been added to the tutor models `compiled` which is a boolean. This is a flag to indicate if gatsby has compiled a unique URL for this tutor. With each endpoint that pulls a list of tutors, you can use a new parameter that can be used - `?compiled_only=true`. Setting this parameter will filter results that only have `compiled = true`. This is needed to ensure our search engine does not return results that do not yet have a compiled URL.

#### Search For Tutors Online and Offline

**Used On:**

- Learnmate search results page
- Learnmate online search results page

```
post '/search/tutors'
```

| Attribute    | Type    | Required | Description                                              | Example  |
| ------------ | ------- | -------- | -------------------------------------------------------- | -------- |
| `limit`      | int     | yes      | The maximum number of tutors the response should contain | 100      |
| `online`     | boolean | yes      | Is this an online search query?                          | 1     |
| `subject_id` | string  | yes      | Comma separated string of subject ids for search         | 40,50    |
| `lat`        | int     | no       | Latitude of the center of searched suburb                | -37.8136 |
| `lng`        | int     | no       | Latitude of the center of searched suburb                | 144.9630 |

Notes from Gatsby Devs: It appears lat and lng are required, but must be set to 0 if you want to ignore them

Response example:

```json
[
    {
        tutor: {
        id: 3037,
        name: "Kimia G",
        site_link: "meet-our-tutors/kimia-g/",
        email: "Kim8ped6fifi27@gmail.com",
        phone: "+61469789388",
        description:
            "Hi there! I'm Kimia, a Primary School, Years 7 - 10 Science, Maths, VCE Biology, Chemistry, English and Maths Methods Tutor. It is a great pleasure for me as a highly motivated and dedicated student with passion, to help you achieve your academic goals.",
        responded: true,
        avatar_link: "wp-content/uploads/rails/Kimia G_3037_1579188020.JPG",
        subjects: [
            {
            id: 5,
            subject_name: "Biology",
            curriculum: "VCE",
            legacy_id: 5,
            active: true,
            },
            {
            id: 7,
            subject_name: "Chemistry",
            curriculum: "VCE",
            legacy_id: 7,
            active: true,
            },
            {
            id: 11,
            subject_name: "English",
            curriculum: "VCE",
            legacy_id: 11,
            active: true,
            },
            {
            id: 40,
            subject_name: "Maths Methods",
            curriculum: "VCE",
            legacy_id: 40,
            active: true,
            },
            {
            id: 206,
            subject_name: "Science",
            curriculum: "VIC710",
            legacy_id: 221,
            active: true,
            },
        ],
        },
        location: "Melbourne Victoria, Australia",
        lat: "-37.8136276",
        lng: "144.9630576",
        dist: 0.0,
    }
];
```

#### Tutor ID from Name Search

**Used on:**

- Update profile page (https://learnmate.com.au/update/update-profile/)

Shows tutors id and site link for tutors who have name which matches search string

```
post '/search/tutorname'
```

| Attribute   | Type   | Required | Description                                                     | Example |
| ----------- | ------ | -------- | --------------------------------------------------------------- | ------- |
| `tutorname` | string | yes      | A string to search for matching names in the learnmate database | deep    |

Response Example:

```json
[
  {
    "id": 189,
    "name": "Deep Bhattacharyya",
    "site_link": "meet-our-tutors/deep/"
  }
]
```

#### Tutor ID Details

**Used on:**

- Update profile page (https://learnmate.com.au/update/update-profile/)

Shows the profile details for a tutor (addresses, subjects, description)

```
post '/search/tutorid'
```

| Attribute | Type | Required | Description                                                         | Example |
| --------- | ---- | -------- | ------------------------------------------------------------------- | ------- |
| `tutorid` | int  | yes      | The ID of the tutor retrieved from the `/search/tutorname` endpoint | deep    |

Response Example:

```json
{
  "id": 189,
  "name": "Deep Bhattacharyya",
  "description": "Hi there! I'm Deep, a VCE Physics tutor (47) and I'm committed to creating engaging and interactive learning environments to help you succeed!",
  "responded": false,
  "created_at": "2018-02-06T08:34:16.778+11:00",
  "updated_at": "2020-07-31T02:01:35.007+10:00",
  "site_link": "meet-our-tutors/deep/",
  "avatar_link": "wp-content/uploads/2017/04/deep.jpg",
  "active": true,
  "base64": null,
  "teachworks_id": 28768,
  "long_desc": null,
  "phone": "+61434100056",
  "email": "bhattacharyya.deep@gmail.com",
  "subjects": ["Physics, VCE"],
  "addresses": ["Melbourne, Victoria, Australia"]
}
```

#### Enquire Form Submission

**Used on:**

- First time student customer form
- First time parent customer form
- Returning customer parent form
- Returning customer student form

Enquire form that takes customer details and sends it to heroku portal.

Done after verification of credit card details are correct.

```
post '/enquire'
```

**Student Customer**

| Attribute                   | Type   | Required | Description                                                                                                                             | Example                               |
| --------------------------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| `customer[first_name]`      | string | yes      | First Name of student                                                                                                                   | Deep                                  |
| `customer[last_name]`       | string | yes      | Last Name of student                                                                                                                    | Bhattacharyya                         |
| `customer[email]`           | string | yes      | Email of student                                                                                                                        | deep@gmail.com                        |
| `customer[mobile_phone]`    | string | yes      | Ph Number of student                                                                                                                    | 0434100056                            |
| `customer[postcode]`        | string | yes      | Postcode of student                                                                                                                     | 3170                                  |
| `customer[year_level]`      | string | yes      | Year Level of student                                                                                                                   | 11                                    |
| `customer[subjects]`        | string | yes      | Subject that the student would like tutoring for.                                                                                       | Science                               |
| `customer[lesson_location]` | string | yes      | Location of lessons                                                                                                                     | Mulgrave                              |
| `customer[other_details]`   | string | yes      | Other details textbox                                                                                                                   | Tutors own location preferred         |
| `customer[tutors]`          | string | yes      | See example below. If selected using search select2 plugin, tutors will follow this structure 1. Otherwise, it will follow structure 2. | **See structure 1 and 2**             |
| `stripeToken`               | string | yes      | Stripe token retrieved from stripe SDK if credit card information was correct. Will be `returning-customer` if returning customer form. | `returning-customer` or random string |

**Parent Customer**

| Attribute                       | Type   | Required | Description                                                                                                                             | Example                               |
| ------------------------------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| `customer[parent_first_name]`   | string | yes      | First Name of parent                                                                                                                    | Deep                                  |
| `customer[parent_last_name]`    | string | yes      | Last Name of parent                                                                                                                     | Bhattacharyya                         |
| `customer[parent_email]`        | string | yes      | Email of parent                                                                                                                         | deep@gmail.com                        |
| `customer[parent_mobile_phone]` | string | yes      | Ph Number of parent                                                                                                                     | 0434100056                            |
| `customer[parent_postcode]`     | string | yes      | Postcode of parent                                                                                                                      | 3170                                  |
| `customer[year_level]`          | string | yes      | Year Level of parent                                                                                                                    | 11                                    |
| `customer[subjects]`            | string | yes      | Subject that the parent would like tutoring for.                                                                                        | Science                               |
| `customer[lesson_location]`     | string | yes      | Location of lessons                                                                                                                     | Mulgrave                              |
| `customer[other_details]`       | string | yes      | Other details textbox                                                                                                                   | Tutors own house preferred            |
| `customer[tutors]`              | string | yes      | See example below. If selected using search select2 plugin, tutors will follow this structure 1. Otherwise, it will follow structure 2. | **See structure 1 and 2**             |
| `stripeToken`                   | string | yes      | Stripe token retrieved from stripe SDK if credit card information was correct. Will be `returning-customer` if returning customer form. | `returning-customer` or random string |
| `students`                      | string | yes      | See Example Below                                                                                                                       | _Object_                              |
| `students.customer[first_name]` | string | yes      | First name of student                                                                                                                   | Juniordeep                            |
| `students.customer[last_name]`  | string | yes      | Last name of student                                                                                                                    | Juniorbhattacharyya                   |
| `students.customer[year_level]` | string | yes      | Year level of student                                                                                                                   | 11                                    |
| `students.customer[subjects]`   | string | yes      | Subjects                                                                                                                                | Science                               |

_Example for student customer_

```json
{
  "customer[first_name]": "TEST",
  "customer[last_name]": "TEST",
  "customer[email]": "bhattacharyya.deep@gmail.com",
  "customer[mobile_phone]": "0434100056",
  "customer[postcode]": "3170",
  "customer[year_level]": "12",
  "customer[subjects]": "All",
  "customer[lesson_location]": "Home",
  "customer[other_details]": "Other Info",
  "customer[tutors]": **See Structure 1/2 Below**,
  "stripeToken": "returning-customer"
}
```

_Example post for parent customer_

```json
{
  "customer[parent_first_name]": "TEST",
  "customer[parent_last_name]": "TEST",
  "customer[parent_email]": "TEST@TEST.com",
  "customer[parent_mobile_phone]": "+61434100056",
  "customer[parent_postcode]": "3170",
  "customer[lesson_location]": "",
  "customer[other_details]": "",
  "customer[tutors]": **See Structure 1/2 Below**,
  "customer[year_level]": "TEST",
  "customer[subjects]": "TEST",
  "stripeToken": "returning-customer",
  "students": [
    {
      "customer[first_name]": "TEST",
      "customer[last_name]": "TEST",
      "customer[year_level]": "TEST",
      "customer[subjects]": "TEST"
    }
  ]
}
```

_Structure 1 `customer[tutors]` - when tutors are selected via search engine_

```json
{
  "customer[tutors]": [
    {
      "selected": true,
      "disabled": false,
      "text": "Svenja Z.",
      "id": "0",
      "val": "meet-our-tutors/svenja-z/"
    }
  ]
}
```

_Structure 2 `customer[tutors]` - when freeform text entered due to customer going directly to the enquire form_

```json
{
  "customer[tutors]": [{ "text": "Atutor Namegoeshere", "val": null }]
}
```

Response Example:

```json
{ "success": "true" }
```

#### Mailchimp Enquire Form

**Used on:**

- All enquire forms

Email address is sent to mailchimp 5 seconds after the user types it into the form

```
post '/mailchimpEnquireFormController'
```

| Attribute       | Type   | Required | Description                      | Example          |
| --------------- | ------ | -------- | -------------------------------- | ---------------- |
| `email_address` | string | yes      | Email address of student         | deep@live.com.au |
| `email_type`    | string | yes      | Type of email. Keep it as `text` | text             |
| `status`        | string | yes      | `subscribed` or `unsubscribed`   | subscribed       |

#### Delete Email from Mailchimp

**Used on:**

- All enquire forms

Email address which was subscribed will be unsubscribed when user submits the enquiry

```
post '/mailchimpEnquireFormControllerDelete'
```

| Attribute | Type   | Required | Description                                   | Example               |
| --------- | ------ | -------- | --------------------------------------------- | --------------------- |
| `user_id` | string | yes      | User ID of email just subscribed to mailchimp | hd392bd392ydb9d302pej |

Example Response:

```json
{
  "response": {
    "type": "http://developer.mailchimp.com/documentation/mailchimp/guides/error-glossary/",
    "title": "Method Not Allowed",
    "status": 405,
    "detail": "The requested method and resource are not compatible. See the Allow header for this resource's available methods.",
    "instance": "c02db902-1ba3-40cf-9511-4c5ff3c2e591"
  }
}
```

#### Check Unavailability

**Used on:**

- All tutor pages

```
post '/checkUnavailability'
```

| Attribute  | Type   | Required | Description                                                                                                                 | Example                |
| ---------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| `checkLoc` | string | yes      | The web address of the tutor to check on learnmate backend systems to see if the tutor has marked themselves as unavailable | `meet-our-tutors/deep` |

Example Response:

```json
{ "tutorUnavailable": false }
```

#### Get All Subjects

**Used on:**

- Search box for tutors

```
get '/subjects/all'
```

Example Response:

```json
[
  {
    "TASPRIM": [
      {
        "id": 754,
        "subject_name": "Chinese",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 576,
        "subject_name": "English",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 870,
        "subject_name": "Geography",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 871,
        "subject_name": "History",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 860,
        "subject_name": "Italian",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 583,
        "subject_name": "Maths",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      },
      {
        "id": 780,
        "subject_name": "Science",
        "curriculum": "TASPRIM",
        "legacy_id": null,
        "active": true
      }
    ]
  },
  ...
]
```

### New Endpoints added during Gatsby Build

#### Get All Tutors

**Used on:**

- Gatsby Compilation when generating unqiue URLS for tutors

```
get '/tutors'
```

Example Response:

```json
[
    {
        tutor: {
        id: 3037,
        name: "Kimia G",
        site_link: "meet-our-tutors/kimia-g/",
        email: "Kim8ped6fifi27@gmail.com",
        phone: "+61469789388",
        description:
            "Hi there! I'm Kimia, a Primary School, Years 7 - 10 Science, Maths, VCE Biology, Chemistry, English and Maths Methods Tutor. It is a great pleasure for me as a highly motivated and dedicated student with passion, to help you achieve your academic goals.",
        responded: true,
        avatar_link: "wp-content/uploads/rails/Kimia G_3037_1579188020.JPG",
        subjects: [
            {
            id: 5,
            subject_name: "Biology",
            curriculum: "VCE",
            legacy_id: 5,
            active: true,
            },
            {
            id: 7,
            subject_name: "Chemistry",
            curriculum: "VCE",
            legacy_id: 7,
            active: true,
            },
            {
            id: 11,
            subject_name: "English",
            curriculum: "VCE",
            legacy_id: 11,
            active: true,
            },
            {
            id: 40,
            subject_name: "Maths Methods",
            curriculum: "VCE",
            legacy_id: 40,
            active: true,
            },
            {
            id: 206,
            subject_name: "Science",
            curriculum: "VIC710",
            legacy_id: 221,
            active: true,
            },
        ],
        },
        location: "Melbourne Victoria, Australia",
        lat: "-37.8136276",
        lng: "144.9630576",
        dist: 0.0,
    }
];
```


#### Get All Subjects (Not organised into category)

**Used on:**

- Gatsby Compilation when generating unqiue fields for our search form

```
get '/subjects'
```

Example Response:

```json
[
    {
        "id": 277,
        "subject_name": "Australian Politics",
        "curriculum": "VCE"
    },
    {
        "id": 3,
        "subject_name": "Art",
        "curriculum": "VCE"
    },
  ...
]
```

#### Update tutor compiled field

**Used on:**

- Gatsby Compilation, marks all compiled tutors as 'compiled', which allows them to be searched via the search fron on the gatsby site.

```
put '/tutors/update_compiled'
```


| Attribute   | Type   | Required | Description                                                     | Example |
| ----------- | ------ | -------- | --------------------------------------------------------------- | ------- |
| `tutor_ids` | [id]   | yes      | An array of IDS to mark as compoled                             | [3360]  |


Example Post Object:

```json
{
   "tutor_ids": [3360]
}
```


#### Get All Curriculums - _Ignore_

**Used on:**

- Not used anywhere - curriculums exist in the javascript on the website to minimise load time.

```
get '/curriculums/all'
```

#### Availability Registration

**Used on:**

- When sms is sent out, the messages link to this address.

Tutor is marked as available/unavailable based on the link they click in the SMS. If the tutor clicks no, they are redirected to the register unavailability page on the learnmate website.

```
get '/availability/yes'
```

```
get '/availability/no'
```

## Admin Interface APIs

The Admin interface APIs have been documented in the [doc/ folder](https://github.com/learnmatetutors/learnmate-rails/tree/master/doc) in the root directory of the learnmate-rails git repository.

### Enquire Form Processing

**How the list of tutors sent from the enquire form is processed**

The list of tutors is sent from the enquire form goes through 3 distinct functions:

1. The Stripe user id is stored in teachworks.
   - The stripe js sdk is responsible for creating this user id using the information entered into the enquire form
   - Credit Card details are not sent to the server or to teachworks.
2. The account in teachworks is created with the relevant information (see [Enquire Form Submission](#enquire-form-submission))\
   - Teachworks API docs can be found [here](https://documenter.getpostman.com/view/10096149/SWTABydD?version=114c35a6-1ae2-4ae5-b72a-de25d005563f)
3. An entry is made into the portal with the teachworks and stripe id of the new submission (http://learnmate-admin.herokuapp.com/#/new_customers) and the automation process is started.

Any errors during this stage are shown as a popup in the form. It is important to note that these steps occur in order. If there is an error in a certain step, the rest are not executed.

### Creating New Tutors

The actual APIs used for this can be found in the [doc/ folder](https://github.com/learnmatetutors/learnmate-rails/tree/master/doc) mentioned above in tutor.rb.

Also see [the schema](https://github.com/learnmatetutors/learnmate-rails/blob/master/db/schema.rb) for more details about relationships between tutors and their attributes.

### Export Button

- Goes through all tutors in the database
- If the tutor is available, adds tutor to a csv file
- Downloads this CSV file.

### Adding and Removing Subjects and Addresses

A tutor can have 1 or more subjects (see [the schema](https://github.com/learnmatetutors/learnmate-rails/blob/master/db/schema.rb) which makes it possible to add and remove subjects and addresses.

### Changing Heroku Portal Username and Password

In [this file](https://github.com/learnmatetutors/learnmate-rails/blob/master/app/commands/authorize_admin_request.rb) there is a way to change the password for the user account.

The method that I use to do this is as follows:

- Uncomment `change_user_password_authentication`
- change the `@user.password` to the new password
- Deploy changes on heroku
- Log in using the old password
- Log out, you should now be able to log in using the new password
- Comment out `change_user_password_authentication` in [authorize_admin_request.rb](https://github.com/learnmatetutors/learnmate-rails/blob/master/app/commands/authorize_admin_request.rb) and redeploy to heroku

```
def change_user_password_authentication
    @user.password = "new password"
    @user.password_confirmation = "new password"
    @user.save!
end
```

### Automation system

Documentation for the functions of the automation system can be found in the [doc/ folder](https://github.com/learnmatetutors/learnmate-rails/tree/master/doc).

As mentioned in [enquirenow_helper.rb](https://github.com/learnmatetutors/learnmate-rails/blob/master/app/helpers/api/enquirenow_helper.rb), the main functions are:

1. Deduct \$15.95 from stripe
2. Automate the activating of student/parent teachworks account and sending email to tutor via teachworks
3. Automate sending of SMS messages to tutors

This uses selenium to do this. There are errors which can be retrieved in the process

!> Ask Dmitri for trainig document for a list of errors.

There are errors for:

- Stripe failures due to preexisting account
- Tutor unable to be found in teachworks
- Tutor profile unable to be activiated since another tutor exists with the same name
- A different issue with the tutor profile
- Student unable to be activated in teachworks
- Teachworks server failure
- SMS account out of credit

#### BurstSMS API

See BurstSMS API: https://burstsms.com.au/sms-api

After the tutor has been emailed, we also send the tutor a text message to them to confirm. This is done after the automation has activated the student's account above. 

### Diagnosing Errors in Automation System and Enquire Form

These are the regular steps I follow when new errors are brought to my attention:

- Navigate to Heroku portal for this application
- Navigate to the resources tab
- From here, click the link to the logs platform PaperTrail
- Search for the email address of the person who experienced a bug or a problem submitting their enquiry
- See any errors outputted and navigate to this line in code.
- If there is a form submission, prettify this and identify any errors in the submissin

Sometimes, errors are outputted to the users which they do not understand clearly. The common ones are:

- A parent has submitted an extra blank student in their form
- A parent has submitted two students with the same name
- Teachworks or Stripe were unreachable during the transaction due to other running processes.

## Connection To TeachWorks

### Form Submission Connection To TeachWorks

When the form is submitted:

- Teachworks API is called to submit the details that were filled out in the form
- If student form submitted:
  - New student account is created in teachworks
  - Each field is sent to teachworks
- If parent form submitted:
  - New parent account created in teachworks
  - Parent fields are sent to teachworks
  - Create an account for each student
  - Populate relevant fields for each student
- If there are any errors in this process, see [common errors](#diagnosing-errors-in-automation-system-and-enquire-form)

### Returning Students

When a returning student/parent (someone who has enquired at learnmate before) submits the returning student/parent form (https://learnmate.com.au/returning-customer/form/) they have a stripe ID of `returning-customer`. In order to verify the details that they have submitted into the form (without any credit card details), we still create a teachworks account for them (without activating it).

This teachworks account is ignored and the admins assign the tutor the tutor that they have asked for to the original teachworks account for the returning customer.

The automation does not run for the returning student form.

## Heroku Scheduler Tasks

| Task                              | Time                 | Description                                                                                                                                                                                                |
| --------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `check_teachworks_unavailability` | Daily at 2:30 PM UTC | For each tutor in teachworks, if the tutor has marked themselves as unavailable on teachworks create a new `unavailability` in the table with a `from_time` and `to_time` and associate it with the tutor. |
| `check_tutor_availability`        | Daily at 4:00 PM UTC | When a tutor has an unavailability with `from_time` <= current date and `to_time` >= current date, mark the tutor as inactive.                                                                             |
| `get_phone_and_email`             | Daily at 4:30 PM UTC | For each tutor, fetch their phone number and email address from teachworks and store this in the heroku portal.                                                                                            |

The other functions in `scheduler.rb` support these main scheduled tasks.

# Getting Started With Codebase Development

## What is this codebase?

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

## Dependencies:

- PostgreSQL v10.3
- Heroku CLI
- Ruby v2.5.0 (Use Rbenv or RVM to install locally to a directory)
- Bundler
- Yarn
- NodeJS
- PostGIS extension

## Setting Up Ruby on Rails Application

1. Get NodeJS (8.0) and Ruby 2.5.0 on a linux or mac distribution. If you run windows, you must run a linux subsystem. There is currently no support for deploying on Windows. Use rvm (not rbenv) to install ruby
   NOTE: If you wish to have a local database, please install postgresql as well. You will also need the POSTGIS Extension for postgresql. If you dont wish to go down this route, feel free to make a hosted copy of the production database and use that!
   Install rvm using brew/homebrew. Remember its ruby 2.5.0!
2. Clone the repo from heroku or github.
3. Run `yarn install` and `bundle install` to install the dependencies. Make sure you run these in the root folder of the project
4. Modify the database details in the database.yml file.
5. Run the server!

For development on ruby on rails:

1. In one command window, run `rails s`, which will host the main server on localhost:3000
2. Then in another command window, run `webpack-dev-server` which will run the development server in the background with hot module reloading enabled. You may then modify any of the react code and if you visit localhost:3000 in the browser, it will auto refresh on saving!
3. For the database, download a backup of the database from heroku, then use a program like DBeaver to import it into your local machine.

## To Deploy to Heroku:

1. `git push heroku master`
2. If you have made new migrations in the database, be sure to use `heroku run rails db:migrate`
3. Please go here: https://chromedriver.chromium.org/. Copy the latest version number for chrome's latest STABLE release (eg. 81.0.4044.69).
4. Go into settings in the heroku dashboard and update the config var `CHROMEDRIVER_VERSION` to the version on the ChromeDriver google site.

## To Insert a New Field

1. Run `rails generate migration add_field_to_model field:type`
2. `rails db:migrate`
3. Make sure you run `heroku run rails db:migrate` when you deploy to rails

## Locally trying automation functionality

1. Follow this guide if on windows: https://gist.github.com/danwhitston/5cea26ae0861ce1520695cff3c2c3315
2. Install webdriver and chromedriver (make sure it's in the PATH variable)
3. Launch the selenium server using `java -jar .\selenium-server-standalone-3.0.1.jar -port 4445`

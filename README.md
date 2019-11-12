### PT. Skyshi Digital Indonesia
##### Jl. Tampomas No. 9, Trihanggo, Kec. Gamping,
##### Kab. Sleman, Daerah Istimewa Yogyakarta, Indonesia, 55291.
##### Phone : (0274) 4547428, Email : hello@skyshi.com 

# Backend Guideline

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [What is Backend](#what-is-backend)
- [Requirement Tools](#requirement-tools)
- [API Convention](#api-convention)
    - [Endpoint Naming Principles](#endpoint-naming-principles)
    - [Common Conventions](#common-conventions)
- [API Flow](#api-flow)
- [API Input](#api-input)
- [HTTP Codes](#http-codes)
- [Git Convention](#git-convention)
- [Coding Standard](#coding-standard)
- [Database Convention](#database-convention)
- [Database Flow](#database-flow)
- [Application Flow](#application-flow)
- [Promise Flow](#promise-flow)
- [Authentication](#authentication)
- [Image Processing](#image-processing)
- [Multi Currency](#multi-currency)
- [Multi Language](#multi-language)
- [Social Login](#social-login)
- [Loging](#loging)
- [Monitoring](#monitoring)
- [Alert](#alert)
- [Error Response](#error-response)
- [Timezone](#timezone)
- [Scaling Architecture (Experimental)](#scaling-architecture-(Experimental))
- [Debugging Step](#debugging-step)
- [Useful References](#useful-references)

## Introduction
**Skyshi Procedure Documentation** is the standard documentation for each division in managing the works to ease the bureaucratic process in carrying out the given task. It is expected that by using this documentation the work between divisions becomes easier and more efficient.

## What is Backend
Backend responsibility

## Requirement Tools
- Visual Studio Code
- Postman
- NVM
- Sequel Pro
- iTerm
- Git

## API Convention

#### Endpoint Naming Principles
Try separating API by logical resources (hint: usually things written as nouns) names such as `users`, `coupons`, `transactions`, or `movies`. Once resources defined, identify what actions can be applied to it. Map those actions using HTTP methods available.

There are at least three common cases we found when naming an endpoint :

- **Common CRUD**

    For common CRUD API actions, follow the usual. Example: a `ticket` resource could have CRUD actions mapped to HTTP methods like these :
    - GET `/tickets` - Retrieves a list of tickets
    - GET `/tickets/12` - Retrieves a specific ticket with id of 12
    - POST `/tickets` - Creates a new ticket
    - PUT `/tickets/12` - Updates ticket #12
    - PATCH `/tickets/12` - Partially updates ticket #12
    - DELETE `/tickets/12` - Deletes ticket #12

- **Resources relations**
    
    If a resource is related to another resource, and the relation can only exist within that resource, then logically map those resources to parent resource. Example between `posts` and `comments`:
    - GET `/posts/12/comments` - Retrieves list of comments for posts #12
    - GET `/posts/12/comments/5` - Retrieves comments #5 for posts #12
    - POST `/posts/12/comments` - Creates a new comments in posts #12
    - PUT `/posts/12/comments/5` - Updates comments #5 for posts #12
    - PATCH `/posts/12/comments/5` - Partially updates comments #5 for posts #12
    - DELETE `/posts/12/comments/5` - Deletes comments #5 for posts #12

- **Actions not fitting in CRUD scope**

    There will be some cases that we need to do some action to a resource that can’t really be semantically mapped to HTTP methods. Such as : hide/unhide a comment resource, clear server’s cached data, or resending a registration email for a user. There are a number of approaches:
    - Restructure the action to appear like a field of a resource. This works if the action doesn't take parameters. For example an activate action could be mapped to a boolean activated field and updated via a PATCH to the resource.
    - Treat it like a sub-resource with RESTful principles. For example, GitHub's API lets you star a gist with PUT `/gists/:id/star` and unstar with DELETE `/gists/:id/star`.
    - Sometimes you really have no way to map the action to a sensible RESTful structure. For example, to search across multiple resources doesn't really make sense to be applied to a specific resource's endpoint. In this case, /search would make the most sense even though it isn't a resource or a noun. This is OK - just do what's right from the perspective of the API consumer, and if there are some thing that can’t be semantically understood above all, **make sure it's documented clearly** to avoid confusion.

    There are core principle that could rules out everything : 
    > An API is a developer's UI, it's important to ensure the user's experience is thought out carefully.
    
    *So, use these (or general web) standards where they make sense for consumer’s usages. Try to make everything understandable on consumer’s first guess by following common standard, if that’s impossible and unavoidable, make sure they understand by writing it in the API documentations.*


#### Common Conventions
- All resource names should use nouns in plural form.
- For names that has more than one word on it, use hyphens (-) to preserve the url readability. Try to avoid using underscores ( _ ) for endpoint’s url at all cost.
    ```
    https://api.example.com/inventory-management/managed-entities/{id}/install-script-location
    ```
- Do not use underscores ( _ )
- Use lowercase letters in every URI representation
- Use nouns and no verbs to represent resources
    ```
    https://api.example.com/device-management/managed-devices 
    https://api.example.com/device-management/managed-devices/{device-id} 
    https://api.example.com/user-management/users/
    https://api.example.com/user-management/users/{id}
    ```
- Use query parameters (colored blue) to apply filter on resources. For query names containing more than one word, **use underscores ( _ )** as delimiter.
    ```
    http://api.example.com/device-managements/managed-devices?region_area=USA&brand=XYZ&sort=installation-date
    ```
- Some proxies support only POST and GET methods. To support a RESTful API with these limitations, the API needs a way to override the HTTP method.
 
    *Use the custom HTTP Header X-HTTP-Method-Override to override the POST Method.*

## API Flow
- Recommended response time is about **200-300ms**.
- Multiple data should returned as **array**, and single data should returned as **object**.
- Error data should return the field and **message validation**. Don’t forget to **add HTTP code**.
- Output should have **same** field as input field.

## API Input
- All data submitted must be using :
    ```
    Content-Type: application/json
    ```
- All data submitted must aware with CORS issue. Please use [cors](https://www.npmjs.com/package/cors#simple-usage-enable-all-cors-requests).
- All data submitted to backend MUST passed validation, put validation on middleware level. Please use [validate.js](https://www.npmjs.com/package/validate-js)

    - formatAlphabetsAndSpaces : `accepts alphabets and spaces only.`
    - orderStatus : within `in-cart`, `checkout`, `payment-pending`
    - token : MUST presence

## HTTP Codes
- Data errors
    - `400` for when the requested information is incomplete or malformed.
    - `422` for when the requested information is okay, but invalid.
    - `404` for when everything is okay, but the resource doesn’t exist.
    - `409` for when a conflict of data exists, even with valid information.
- Auth errors
    - `401` for when an access token isn’t provided, or is invalid.
    - `403` for when an access token is valid, but requires more privileges.
- Success statuses
    - `200` for when everything is okay.
    - `201` for when data success created.
    - `204` for when everything is okay, but there’s no content to return.
- Standard statuses
    - `500` for when the server throws an error, completely unexpected.

## Git Convention
- Public image (not assets) directory, node_modules directory, config file, .env, etc should included in `.gitignore` file.
- Should have branch master (as production), development, and staging.
- Should push branch as feature, for example : `feature/social-login`.
- Hotfix must use master as source, never use development as source.
- Write proper and explanatory git commit message.
    
    A subject which contains the title of the git commit. This should be limited to 50 characters and should be separated from the rest of the commit with a blank line. Explanatory text explaining what has been changed and why the change was necessary. Write in the imperative mood e.g. `Fix bug causing outage` rather than `Fixed bug causing outage`.

    More detailed explanatory text, if necessary.  Wrap it to about 72 characters or so.  In some contexts, the first line is treated as the subject of an email and the rest of the text as the body.  The blank line separating the summary from the body is critical (unless you omit the body entirely); tools like rebase can get confused if you run the two together, [reference link.](https://medium.com/@steveamaza/how-to-write-a-proper-git-commit-message-e028865e5791)

- It’s good practice to follow these naming convention, [reference link](https://gist.github.com/digitaljhelms/4287848) :

    Instance | Branch | Description |
    --- |:---:|---
    Stable | stable | Accepts merges from Working and Hotfixes
    Working | master | Accepts merges from Features/Issues and Hotfixes
    Features/Issues | topic-* | Always branch off HEAD of Working
    Hotfix | hotfix-* | Always branch off Stable

    ![Git Flow](./assets/images/gitflow.png)

## Coding Standard
- Use linter (ESLint etc) to maintain the code standard
- Use camelCase for JS, and snakeCase for PHP
- Use cons if variable is static
- Write same process in function, so it can be reuse
- Function naming prefix :
    - **can** is function that **returns a boolean**.
    - **has** is function that **returns a boolean**.
    - **is** is function that **returns a boolean**.
    - **get** is function that **returns a non boolean**.
    - **set** is function that is used to **save a value**.
- Try add comment to help explanation block of code, single line comment is recommended.
- Try add TODO comment whenever you found out something needed to be done/fixed, yet you’re still having another thing to finish. Try reading a bit about broken window theory in links section.
- Try your best to write an elegant code. Imagine your codes a poetry you write for the world to read.  ;)
> Good code should be clean, simple, and **easy to understand** first of all.

## Database Convention
- Table name should have plural name.
- Both tables and columns should use PascalCase   Both tables and columns should use lower case :), with underscores. These are very readable and cross platform. 
- Each table in the database must have a timestamp column for created_at and/or updated_at, deleted_at
- Foreign key should have table name as prefix

#### Tables convention
- Use a collective name or, less ideally, a plural form. For example (in order of preference) staff and employees.
- Do not prefix with tbl or any other such descriptive prefix or Hungarian notation.
- Never give a table the same name as one of its columns and vice versa.
- Avoid, where possible, concatenating two table names together to create the name of a relationship table. Rather than cars_mechanics prefer services.

#### Columns convention
- Always use the singular name.
- Where possible avoid simply using id as the primary identifier for the table.
- Do not add a column with the same name as its table and vice versa.
- Always use lowercase except where it may make sense not to such as proper nouns.

## Database Flow
- EXPLAIN Your SELECT Queries.
    ```
    Using the EXPLAIN keyword can give you insight on what MySQL is doing to execute your query.
    ```
- Things that need attention in query that may slowing the performance.
- Add Index to key columns.
- Recommend to use multiple small query and join it in application level rather than one single bigquery. Small query easier to cached.
- Count data should implemented in database level, rather than application level.
- Insert query should use TRANSACTION, so if one of the process is failed, all insert process can be rollback. 
- Should make sure to use production environment when do migration and seed and make sure the seed process won't be truncate the table data 
- Make sure developer has the right access so the developer won't drop database or truncate it
- Choose proper Database Engine :
    - If you develop an application that reads data more often than writing (ex: search engine), choose MyISAM storage engine.
    - If you develop an application that writes data more often than reading (ex: real-time bank transactions), choose INNODB storage engine.
- LIMIT 1 When Getting a Unique Row.
    ```
    In such cases, adding LIMIT 1 to your query can increase performance. This way the database engine will stop scanning for records after it finds just 1, instead of going thru the whole table or index
    ```
- Split the Big DELETE or INSERT Queries.
    ```
    If you end up locking your tables for any extended period of time (like 30 seconds or more), on a high traffic web site, you will cause a process and query pileup, which might take a long time to clear or even crash your web server.

    If you have some kind of maintenance script that needs to delete large numbers of rows, just use the LIMIT clause to do it in smaller batches to avoid this congestion.
    ```
- Always check query generated by ORM. There are many cases that query is not optimal by ORM. Please check with EXPLAIN. Watch EXECUTION TIME properly.
- **WARNING** using associations query generated by ORM may impact performance. ALWAYS check execution time query that using associations.
- May Always wrap your query inside CACHE. Cache 1 min will help performance.

## Application Flow
- Cron is separated from main app.
- Main app can be run as cluster mode, and cron app should run as fork mode.
- Use config file (.env or .json or .js).
- Do not use static files on Express JS.
    ```
    app.use(express.static(config.publicPath, { maxAge: c.ONE_YEAR }))
    ```
- use CDN instead. Static file will slow down the app.
- If API response is too big, recommend to split it into several API.
- Date response format is **YYYY-MM-DD** (H:i if needed) or use UNIX timestamp.
- Notification (Email, push notification, etc) should use template, not hardcoded inside controller.
- Credential must be different for dev and production (FB key secret, App ID, domain, etc).
- Use cache to optimize the performance, can be cache per route or cache per flow.
- App should strip out input emoticon or reject it.
- Should add try catch as function wrapper to wrap the controller.
- Write down the functions that are widely used in other modules on the helper.
- Put the validation on middleware, so the controller will be ready to process the data without revalidate.
- Should have **automation testing** (Using Jest, Mocha, etc).
- Should implement the ACL for multi role/level user.

## Promise Flow
- NEVER EVER USE SEVERAL await in one time. Use async and promise.all instead.
- Model should return promise, it’s not recommended to use async await in model.
- Implement `async/await` in controller level.
- Wrap all independent promise in Promise.all().
- Add promise timeout if possible.
- Avoid callbackhell in promise.
    ```
    Promise should be chained, not just another callbackhell/nested or pyramid shape.
    ```
- Promise catch must be implemented.
    ```
    Don’t define rejection handlers as the second argument for then calls, always use an extra catch.
    ```
- Always return something in promise chain.
    ```
    Your .then handlers should return something specifically before call another promise in next chain.
    ```
- Use Promise.all(), Promise.spread() and other methods to control the flow.
- To limit concurrency use Promise.map() or similar.

## Authentication
- Use library [passport](https://www.npmjs.com/package/passport) to validate user login and JWT token :
    ```javascript
    app.post('/login', 
        passport.authenticate('local', { failureRedirect: '/login' }),
        function(req, res) {
            res.redirect('/');
        }
    );
    ```
- Use Header to send Authorization data `Authorization: <type> <credentials>`
    ```
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiYWJjZC0xMjM0LXF3ZXItNTY3OC1hc2RmIn0.Y7MsvX03-plHlzP_v7FMQRA6-YYP3r_mp2X5t6onaxo
    ```
- Use JWT to create token. Secret will be used by JWT to sign the payload. Secret is hardcoded on configuration level.
    ```javascript
    req.user = {
        ..req.user.toJSON(),
        token: jwt.sign({id:req.user.id}, jwtOptions.secretOrKey)
    };
    ```
- Use payload user_id and created_at. 

    ```user_id``` : user ID of current logged in user.

    ```iat``` : to get information when this token created. In case backend want to invalidate old token.
    ```javascript
    const payload = {id : user.id, created_at : craeted_at };
    const token = jwt.sign(payload, jwtOptions.secretOrkey);
    ```
    Optional, reserved keyword payload, [reference link](https://scotch.io/tutorials/the-anatomy-of-a-json-web-token) : 
    
    Claims that are not mandatory whose names are reserved for us. These include :
    - `iss` : The issuer of the token
    - `sub` : The subject of the token
    - `aud` : The audience of the token
    - `exp` : This will probably be the registered claim most often used. This will define the expiration in NumericDate value. The expiration MUST be after the current date/time.
    - `nbf` : Defines the time before which the JWT MUST NOT be accepted for processing
    - `iat` : The time the JWT was issued. Can be used to determine the age of the JWT
    - `jti` : Unique identifier for the JWT. Can be used to prevent the JWT from being replayed. This is helpful for a one time use token.
- Secret must be different for production and development. If not, problem may exist when switching between staging and live version of apps.

## Image Processing
- Saved image should save in each folder, for example `/userXX/entityYY`.
- Avoid Base64, use form-data instead.
- Recommended to use cloud CDN (AWS, Minio, etc).
- Recommended to resize the image (for example to optimize mobile).

## Multi Currency
- Cron will update the currencies once a day.
- App should have 1 base currency, as the default currency.
- Use 3rd Party to get today currencies.

## Multi Language
- Return message code and message string.
- Use Header Content-Language to request the header, especially for notification (email, push notif, etc).
- Use XLS / Spreadsheet file as source the columns will be: category, description, key, en, id, (language 3), (language 4), etc.
- Backend will provide .json file as translation source file to frontend.

## Social Login
- Backend should revalidate the auth token whether the user is valid or not, and get the email, image, etc.
- App ID, secret, etc should be different between production and development environment.

## Logging
- Enable newrelic with code `require(‘newrelic’)`  at the first line
- Logging request client and response server with execution time
- Enable loggly or logentries
- Log request client, response server, and execution time. Its recommend to use cloud logging, so it will not slow down the server and performance

## Monitoring
Install 3rd party to monitor system status (like status cake, etc)
- Database monitor
- App monitor
- Server monitor
- Crash monitoring and error tracking

## Alert
- Enable alert at newrelic for send email if :
    - Error percentage 100% for at leasr 5 minutes.
    - APdex 0.5 for at least 10 minutes.
    - Response time 0.6 seconds for at least 5 minutes.
- Enable legacy integration telegram at sentry for sending tekegram message if has new event.

## Error Response
Error messages, validation messages should use template it can be configured in .env file or other config file  so the return will be same and consistent

## Timezone
Make sure Timezone configured using `UTC`. Use below suggestion :
```
process.env.TZ = 'UTC';
```
OR
```
moment.tz.setDefault(‘UTC’);
```
OR

If you are using pm2, please add TZ on `.env`
```javascript
env: {
    NODE_ENV: 'production',
    TZ: 'UTC'
}
```

## Scaling Architecture (Experimental)
This section created for apps how to handle high traffic.

![Flow System](./assets/images/flow-scalling-architecture.png)

Always consider to use cache on query to 3rd party such as external API, database query.

## Debugging Step
- Check source of the error.
    ```
    Source of the error can be related to other platform (mobile, API, web, etc). If the error is specific in web and/or mobile, the error is in client side.
    ```
- Check the error message.
    ```
    By checking the error message, we can get the explanation the related info.
    ```
- Check the time of the issue and check the log.
    ```
    By checking the log, we can know what parameters or action was happened.
    ```
- Check the related error in other project.
    ```
    We can find the solution by checking the related issue in other project.
    ```
- Checking the environment.
    ```
    If the error also happened in development environment, we can reproduce the issue and fix it to production.
    If it related to the third party library:
    - check the third party version
    - check the current main project version
    ```
- Disable another library if possible, make the error is in small box.
    ```
    If possible, we can try to disable another current active library, By make it in small scope, we can more investigate the issue.
    ```
- Return the action as soon as possible when error happen, and output the error.
    ```
    By minimize the scope, we can know which line of code cause the error.
    ```
- Check the fields of the data.
    ```
    We can try to output the data and check the fields, Sometimes the field is null or even not exist, and it cause the error.
    ```
- Check the input of the actions.
    ```
    Variant of input, for example emoticon or special character can cause the error.
    ```
- Check the validation system.
    ```
    If the validation is not good enough, then it will cause the error in main process.
    ```
- For image or video, check the file size, main app limit configuration and check the web server limit configuration.
- Check related hook or middleware of the action.
- Check the server issue.
    ```
    Hardware issue can be storage/RAM is full, or folder permission.
    ```
- For image file, issue can be related to EXIF meta-data.
- If we use third party, check the expiration of the services, credential, and basic info.
    ```
    We should check the third party information, for example: 
    - expired date, 
    - credential key-secret, 
    - base url of services, 
    - parameter, etc.
- Check the permission/role of the user.
- Check process timeout.
- Check the cache data.
    ```
    Sometimes data is not updated because of cache is not updated.
    ```
- Check the cron process.
- Check node worker.
    ```
    If node app is set to cluster mode, then there is possibility process is redundant.
    ```

## Useful References
These are some useful references can be used to help designing a better backend system’s quality :
- [Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
- [How do you define elegant code?](https://softwareengineering.stackexchange.com/questions/97912/how-do-you-define-elegant-code)
- [Software Development and the Broken Windows Theory](https://www.rtuin.nl/2012/08/software-development-and-the-broken-windows-theory/)
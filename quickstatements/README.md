# Quickstatements docker image

Quickstatements2 as seen at  [https://tools.wmflabs.org/quickstatements/](https://tools.wmflabs.org/quickstatements/)

### Tags
Image name                            | Parent image                                    | Quickstatements version
-------------------------------       | ------------------------                        | --------------        
`wikibase/quickstatements` : `latest` | [php:5.6-apache](https://hub.docker.com/_/php/) | master                

### Environment variables

Variable                   | Default  | Description                                            
-------------------------- | -------- | -----------                                            
`WIKIBASE_HOST`            | NONE     | Host of wikibase instance as seen by QS container      
`WB_PUBLIC_HOST_AND_PORT`  | NONE     | Host and port of wikibase as seen by the user's browser
`QS_PUBLIC_HOST_AND_PORT`  | NONE     | Host and port of QS as seen by the user's browser      
`OAUTH_CONSUMER_KEY`       | NONE     | OAuth consumer key (obtained from wikibase)            
`OAUTH_CONSUMER_SECRET`    | NONE     | OAuth consumer key (obtained from wikibase)            
`PHP_TIMEZONE`             | UTC      | setting of php.ini date.timezone                       

### Filesystem layout

Directory                                   | Description                                                                   
---------------------------------           | ------------------------------------------------------------------------------
`/var/www/html/quickstatements`             | Base quickstatements directory                                                
`/var/www/html/quickstatements/public_html` | The Apache Root folder                                                        
`/var/www/html/magnustools`                 | Base magnustools directory                                                    

File                      | Description                                                                                                                              
------------------------- | ------------------------------------------------------------------------------                                                           
`/templates/config.json`  | Template for Quickstatements' config.json (substituted to `/var/www/html/quickstatements/public_html/config.json` at runtime)            
`/templates/oauth.ini`    | Template for Quickstatements' oauth.ini (substituted to `/var/www/html/quickstatements/oauth.ini` at runtime)                            
`/templates/php.ini`      | php config (default provided sets date.timezone to prevent php complaining substituted to `/usr/local/etc/php/conf.d/php.ini` at runtime)


### How to setup and use

#### Make a consumer on wikibase
To make a consumer you will need an account that has an email address
If you're using the wikibase/wikibase docker image and haven't set up email then
use the maintenance script resetUserEmail.php and don't forget --no-reset-password

Now log in as the admin user

Now navigate to http://\<your wikibase\>/Special:OAuthConsumerRegistration

Select request a token for a new consumer

Register a consumer by filling in the form:
1. Application name (e.g. 'Quickstatements)
2. Version (you can leave this as 1.0)
3. Description (e.g. 'Quickstatements')
4. callback URL:  http://<your quickstatements host>/api.php
5. Check "Allow consumer to specify a callback in requests"
6. Request permission for:
    * High-volume editing
    * Edit existing pages
    * Create, edit, and move pages
7. Check agree and click "propose consumer"


Make a note of the details on the following page

#### Set up quickstatements
In order for quickstatements to communicate with wikibase it needs to know where your instance is and how it can find it.
This must be done by setting the ENV variable WIKIBASE_HOST. n.b. This should reflect how this container when running
sees the wikibase container. For example the docker container alias like wikibase.svc.

The user's browser will also be redirected to the Wikibase instance and finally back to quickstatements. The address
the user sees for the Wikibase may be different from how the running container sees it. For example: it may be running
on localhost on a specific port. e.g. http://localhost:8181. This should be passed to the quickstatements container as
WB_PUBLIC_HOST_AND_PORT

One must also know how this container will be visible to the user as well so it can ask the wikibase to redirect the
user back here. This should be passed as QS_PUBLIC_HOST_AND_PORT

You need to pass the consumer and secret token you got from the wikibase to this container as the environment variables
 OAUTH_CONSUMER_KEY and OAUTH_CONSUMER_SECRET

You can now test it works by navigating to http://\<your quickstatements host\> and logging in using the button top right.

You should be redirected to the wiki where you can authorize this Quickstatements to act on your behalf

Finally you should be redirected back to Quickstatements and you should appear logged in.

Use Quickstatements as normal with the Run button. Currently "Run in background" is not supported by this image.

#### Approve the consumer for other users
So other users can also use quickstatements you also need to approve it as a consumer.

Navigate to http://\<your wikibase\>/Special:OAuthManageConsumers/proposed

Click 'review/manage' on your new consumer

Then fill in any reason to approve the consumer (e.g. made by admin)

and click 'Update consumer status'

#### Troubleshooting
If you see an error such as mw-oauth exception when trying to log in check that you have passed the right consumer token
and secret token to quickstatements.

If you have changed the value of $wgSecretKey $wgOAuthSecretKey since you made the consumer you'll need to make another new consumer or
reissue the secret token for the old one.

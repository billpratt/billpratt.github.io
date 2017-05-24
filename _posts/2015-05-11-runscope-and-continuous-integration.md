---
layout: post
title: Runscope and continuous integration
date: 2015-05-11 21:39:04.000000000 -04:00
permalink: /runscope-and-continuous-integration/
---
At work we use JetBrains <a href="https://www.jetbrains.com/teamcity/" target="_blank">TeamCity</a> to handle the deployment of all of our websites, both continuous integration and production. We have unit testing setup and it is part of the build steps in TeamCity. Since a lot of our sites have APIs, we were lacking real world API testing. Sure, you can have your unit testing project fire up an in-memory server and test APIs, but wouldn't it be nice to test from external sources and different regions around the world. Run real world tests and experience the same results as your users would. 

I introduce to you <a href="https://www.runscope.com" target="_blank">Runscope.com</a>. Runscope offers automated API testing and monitoring from locations around the world. You can write assertions for your tests to ensure your APIs are on their best behavior. I won't go into the full details of all that Runscope offers but you can read all about it on their <a href="https://www.runscope.com/docs" target="_blank">documentation pages</a>. The purpose of this blog is to show you how to integrate Runscope tests into your continuous integration process.

I've created a simple ASP.NET web api app for this post, created a bucket in Runscope and added a few tests.

![alt](/assets/images/2015/05/runscope_web_api_site.png)

![alt](/assets/images/2015/05/superhero_runscope_api_tests.png)

The first test will assert that the GET all superhero call returns results with a 200 OK status code.

![alt](/assets/images/2015/05/runscope_test_get_all_superheroes-1.png)
![](/assets/images/2015/05/runscope_test_get_all_assertions.png)

The next test will POST a new superhero, assert that a 201 Created status code is returned, and will capture the newly created superhero's Id as a Runscope variable to be used in later tests.

![](/assets/images/2015/05/runscope_test_post_superhero.png)

![](/assets/images/2015/05/runscope_tests_post_superhero_assertions.png)
![](/assets/images/2015/05/runscope_tests_post_superhero_variables.png)

The third test uses the SuperheroId variable that was captured in the POST test and asserts the new superhero is returned in the GET all api call. Runscope offers the ability to add JavaScript snippets to handle more advanced variable extraction and assertion scenarios. Very cool!

![](/assets/images/2015/05/runscope_tests_get_all_after_post.png)

![](/assets/images/2015/05/runscope_tests_get_all_after_post_scripts.png)

Finally, I'll delete the superhero and assert a 200 status code is returned.

![](/assets/images/2015/05/runscope_tests_delete-1.png)

![](/assets/images/2015/05/assert_delete_200.png)

A nice little feature Runscope offers when adding variables to your tests is auto-completion! A small detail that impressed me the first time I saw it. This really comes in handy when dealing with a large amount of variables.

![](/assets/images/2015/05/runscope_tests_autocomplete-1.png)

In a continuous integration environment, I want to run these tests anytime changes are checked into source control to ensure the APIs still function as expected. Runscope offers <a href="https://www.runscope.com/docs/radar/integrations" target="_blank">trigger URLs</a> to trigger a test run via a GET or POST request. You can either trigger an individual test or all tests within a bucket. In TeamCity, the quickest way to setup a Runscope trigger is to add a Powershell runner build step (Windows) or a command line runner (Linux) and invoke a web request to the test trigger.

**Powershell (Windows)**

*Individual test trigger*
```
Invoke-RestMethod https://api.runscope.com/radar/:trigger_id/trigger
```

*Bucket trigger*
```
Invoke-RestMethod https://api.runscope.com/radar/bucket/:trigger_id/trigger
```

**Command Line (Linux)**

*Individual test trigger*
```
curl https://api.runscope.com/radar/:trigger_id/trigger
```

*Bucket trigger*
```
curl https://api.runscope.com/radar/bucket/:trigger_id/trigger
```

 In a normal TeamCity CI build configuration, I would set up version control settings to check my GitHub branch for changes. If changes are detected, the process would go as followed:

* Build project source
* Run unit tests
* Deploy site to CI web server
* Trigger Runscope tests

For the purpose of this demo, I only set up the Runscope trigger build step.
Since my TeamCity server is Windows, I went with the Powershell runner.

![](/assets/images/2015/05/teamcity_runscope-1.png)

Start the TeamCity build and looking at the log we can see that the Runscope trigger was called successful.

![](/assets/images/2015/05/runscope_teamcity_build_log.png)

Heading over to Runscope, I can see that test was triggered via the Trigger URL

![](/assets/images/2015/05/runscope_test_triggered.png)

I set up these Runscope tests to email me when they are finished as well as post a message in a <a href="https://slack.com/" target="_blank">Slack</a> channel to inform the entire team of the test results.

![](/assets/images/2015/05/runscope_slack.png)

Integrating Runscope into our CI environment has been fun and very useful. I would encourage everyone to check them out. For your efforts they will hook you up with a <a href="https://www.runscope.com/free-tshirt-offer" target="_blank">free t-shirt</a>!


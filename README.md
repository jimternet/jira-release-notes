# jira-release-notes

# What is this?

Its a pretty simple gradle/groovy application. It pulls some information from JIRA and formats into a nicely formatted HTML template. The HTML template contains a bunch of inline CSS so that it can be viewed in Gmail and most other emails.

You can a blog post about it here:
http://tech.secretescapes.com/2015/10/continuous-delivery/automating-release-notes/

# How does it work?

You pass in three parameters to the app and it produces a release-notes.html file.
1. Jira URL: e.g. http://myjira.atlassian.net
2. Base 64 encoded username and password in the format of <username password> - more info @ https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-basic-authentication
3. Release version number - e.g. 2.81

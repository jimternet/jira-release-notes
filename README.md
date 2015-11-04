# Automated Jira Release Notes

# What is this?

Its a pretty simple gradle/groovy application. It pulls some information from JIRA and formats into a nicely formatted HTML template. The HTML template contains a bunch of inline CSS so that it can be viewed in Gmail and most other emails.

You can a blog post about it here:
http://tech.secretescapes.com/2015/10/continuous-delivery/automating-release-notes/

# How does it work?

You pass in three parameters to the app and it produces a release-notes.html file.
* Jira URL: e.g. http://myjira.atlassian.net

* Base 64 encoded username and password in the format of <username password> - more info at https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-basic-authentication

* Release version number - e.g. 2.81

# How can I run it?

There are two main ways to run it.

1. Run it through the gradle run command and pass in the three arguments like:

```
gradle run -PappArgs="['https://myjira.atlassian.net','JK41hbi532SdGVSA12GW==', '2.80']"
```

2. You can compile it into a jar as you would any other Gradle project and then just run it like you would any other other jar like:
 
``` 
java -jar jirareleasenotes-1.0.jar jiraurl base64encodedlogin 2.81
```

# Requirements

* Java 7_80
* Groovy 2.3.11
* Gradle 2.2

*Please note that it is very likely this will work with other versions, but has only been tested with the above.*

# What can I do with this?

Once you get the generated HTML file you can do anything you want with this. Maybe host it on a server so people can view it as a URL, or just email it with an HTML Email client to people. 

What this was initially designed for however was integrating it with Jenkins so you can send release notes out via email with one click!

# What plugins do we use for Jenkins integration?

We used a few plugins:
* Email-ext - Used to send the email with the HTML template - https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin

* Workflow - Used to send the email to an approver first before sending off to everyone else - https://wiki.jenkins-ci.org/display/JENKINS/Workflow+Plugin
 
* Gradle - Used to run the app with the gradle run command - https://wiki.jenkins-ci.org/display/JENKINS/Gradle+Plugin

* Git - Used to pull app from Git repo - https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin 

# How did we integrate Jenkins?

1. We created a job that took in the 3 required params 

2. Pulled the app from the git repo using the Git plugin 

3. Used the gradle plugin to run it. Just filled the Tasks box with:
```
run -PappArgs="['$JiraURL', '$JiraEncoded64Login', '$ReleaseVersion']"

```

4. Lastly, sent the email to the team using the Email-ext plugin. We did this by going to Add post-build action -> Editable Email Notification. Then go to Advanced.. -> and fill in the Pre-send Script box with:
```
def reportPath = build.getWorkspace().child("release-notes.html")
msg.setContent(reportPath.readToString(), "text/html");
```

Optional - If you want to use the workflow plugin with the manual verification step, this is what our script looks like:
```
//1. Send email off to just one person to test it looks good
build job: 'Jira Automated Release Notes', parameters: [[$class: 'StringParameterValue', name: 'ReleaseVersion', value: "${ReleaseVersion}"], [$class: 'StringParameterValue', name: 'ReplyToList', value: "${InitialEmail}"], [$class: 'StringParameterValue', name: 'RecipientList', value: "${InitialEmail}"]]

//2. Wait for manual acceptance
input 'Check the release notes sent to your email? Does everything look ok? Once you hit proceed the notes will get sent to everyone!'

//3. Send off to everyone
build job: 'Jira Automated Release Notes', parameters: [[$class: 'StringParameterValue', name: 'ReleaseVersion', value: "${ReleaseVersion}"], [$class: 'StringParameterValue', name: 'RecipientList', value: 'bcc:everyone@example.com'], [$class: 'StringParameterValue', name: 'ReplyToList', value: 'tech@example.com, product@example.com']]
```

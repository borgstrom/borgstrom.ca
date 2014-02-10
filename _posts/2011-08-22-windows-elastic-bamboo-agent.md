---
layout: post
title: Building a Windows Elastic Bamboo Agent
date: 2011-08-22 08:36
disqus_identifier: 9249207490
---

This week we’ve been working with a new client to help them automate and streamline their build & packaging process and are using [Bamboo](http://www.atlassian.com/software/bamboo/) and elastic agents running on the [Amazon EC2](http://aws.amazon.com/ec2/) platform to do so.

The build process needs to happen on a Windows host so the [default Atlassian image](http://confluence.atlassian.com/display/BAMBOO/default+image) just isn’t going to cut it here. There are a couple resources out there, notably a [ticket with Atlassian](https://jira.atlassian.com/browse/BAM-5206) and a [blog post](http://consultingblogs.emc.com/gracemollison/archive/2011/01/18/how-to-set-up-a-windows-ami-to-use-with-elastic-bamboo.aspx), that attempt to detail the process of setting up a Windows host but nothing that lists a complete step by step process that includes explanations for getting the host up and running ready to be used by Bamboo.

During the setup there was a lot of requisite knowledge of both Bamboo and EC2 required that the above mentioned resources leave out, so this post will address the entire process.

### Getting Started

The first thing you need to have is an EC2 account.

Next we’re going to prepare a Windows server that we will turn into a EBS backed AMI. You must make sure that the “Region” is set to “US East” as this is hard coded into Bamboo.

Choose “Launch Instance” from within the EC2 control panel, choose the “Community AMI” list, select “Amazon Images” and search for windows. You will get a list of all the Amazon images that are based off of Windows Server. Select the version you require.

Follow the wizard through the rest of the prompts. The only change you need to make is when prompted for a security group do not choose “Default”, instead create a new service group and allow inbound RDP traffic (so you can connect to the machine once its launched). You do not need to worry about any other ports now as Bamboo will create its own security group when you launch your first elastic agent, so this is essentially a throw away security group used only while we’re setting up.

Once your machine is up and running use the private key you generated during creation to get the windows password, which you can do by right clicking the instance in the AWS console and choosing “Get Windows Password”. Next right click the instance and choose “Connect”. This will display a dialog that will display the hostname and allow you to download a RDP shortcut to open up Remote Desktop Connection.

Now that the machine is up and running we need to prepare it to run the Elastic Agent Assembly, which is essential a java server that runs when the image is started by Bamboo that creates a tunnel between your machine instance and Bamboo and allows for Bamboo to control the instance and run build jobs.

### Installing Prerequisites

In this stage we’ll install the prerequisites for the Elastic Agent Assembly, but this is also when you should install anything else that is required for you to build your software. Your own requirements are (obviously) beyond the scope of this article so all we’ll cover is what’s needed for the Elastic Agent Assembly, which is version 6 of the Sun/Oracle JDK and the EC2 API Tools.

### Java JDK

Head on over the [Oracle Java Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html) and choose to download the x86 (32-bit) version 6 JDK (at the time of this writing it’s Java SE Development Kit 6 Update 27).

Once you’ve downloaded and installed the JDK we need to setup some environment variables. This is different for different versions of Windows, see the documentation for your version to determine the correct method.

Here’s what you need to set:

* JAVA_HOME — This should point to where Java was installed to, ie. C:\Program Files (x86)\Java\jdk1.6.0_27
* Path — You need to update this variable and add the following to the BEGINNING: %JAVA_HOME%\bin

To verify that everything is working properly, open up a command prompt and run:

    java -version -server

You should get back output showing you what the version of Java you have installed is. If you get an error go back and re-install Java and then double check your environment variables.

### EC2 API tools

As Bamboo needs to control the EC2 environment we need to install the [EC2 API Tools](http://aws.amazon.com/developertools/351).

Download the zip file from above and expand it to a suitable place. I like to create a `c:\apps` folder and stick thinks like this in there.

Now you’ll need to go to your [AWS account page](http://aws.amazon.com/account/) and choose “Security Credentials”, then “X.509 Certificates” and finally “Create a new certificate”. Download both the public and private key files and copy them to your server into the `c:\bamboo\home\` directory.

Finally we need to setup some environment variables:

* EC2_HOME should point to where you put the tools (ie. C:\apps\ec2-api-tools-x.x.x)
* EC2_CERT should point to the public certificate in the bamboo home directory
* EC2_PRIVATE_KEY should point to the private key in the bamboo home directory

With this setup open a command prompt and run:

    ec2-describe-instances

You should get back a bunch of output showing you your current reservations and instances that are running in your EC2 account.

Also, at this point you should be able to manually build your software on this machine.

### Installing the Elastic Agent Assembly

With the JDK, EC2 API tools & your own build requirements installed and working we can now move onto the agent assembly. The version you will install needs to match the version of Bamboo you are running. You can find your current version of Bamboo in the Administration section under System, in the System Information item.

Once identified head over to the [Atlassian Bamboo Downloads](http://www.atlassian.com/software/bamboo/BambooDownloadCenter.jspa).

If the version of Bamboo you’re running is matches the newest version then you’ll need to click the “Show All” link, on the right hand side of the page. This will show you all the downloads and you’ll find the agent assembly at the bottom.

If you’re not running the newest version of Bamboo you’ll need to click the “Downloads Archive” link that’s towards the bottom of the page.

*NOTE: At the time of this article the version of Bamboo deployed on JIRA Studio is 3.0.4-studio4, but you wont find a matching version of the agent assembly on the archive page. Instead use version 3.0.3 as it worked fine on studio version 3.0.4-studio4.*

Once you’ve located and downloaded the agent assembly we can setup our paths for Bamboo.

Create a folder named `bamboo` in the root of your `C:\` drive. Inside that folder add two more folders named `home` & `agent`.

Copy the expanded agent archive into the `C:\bamboo\agent` folder, so the full path should look like: `C:\bamboo\agent\bamboo-elastic-agent-3.0.3`

We need to update a config file that is located at `c:\bamboo\agent\bamboo-elastic-agent-3.0.3\ebs-resources\bamboo-agent.cfg.xml`.

    <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <configuration>
        <buildWorkingDirectory>C:\bamboo\home\build-dir</buildWorkingDirectory>
        <buildArtifactDirectory>C:\bamboo\home\artifacts</buildArtifactDirectory>
    </configuration>

Next, create a new file named `run.bat` inside the `C:\bamboo\agent` folder. The contents of which should look like this:

{% highlight bat %}
@echo off
rem
rem This is a script to launch the bambo elastic agent
rem

setlocal enabledelayedexpansion

rem This is the base path everything lives under
set AGENT_PATH=C:\bamboo\agent

rem This is the name of the folder for the current agent
rem When you need to update the agent assembly you will
rem need to also change this variable to point to the new version
set AGENT=bamboo-elastic-agent-3.0.3

cd %AGENT_PATH%\%AGENT%

:start

rem We need to pause here to ensure that the system is ready
rem Use ping as a hackish way to sleep for 10 seconds
ping -n 10 127.0.0.1>nul

@echo on
java -server -Xms32m -Xmx512m -XX:MaxPermSize=256m -classpath "lib\*" -Dbamboo.home=C:\Bamboo\Home com.atlassian.bamboo.agent.elastic.client.ElasticAgentBootstrap 2>&1 > "%AGENT_PATH%\bamboo-elastic-agent.out"

@echo off
rem There should be no reason we ever get here except for an error
rem Just go back and try again
goto start

endlocal
{% endhighlight %}

At this point things should be working, however please take note: **When you run this script by hand right now it will error out with an error message similar to below**.

    java.io.FileNotFoundException: http://169.254.169.254/2008-02-01/user-data
        at sun.net.www.protocol.http.HttpURLConnection.getInputStream(Unknown Source)
        at java.net.URL.openStream(Unknown Source)
        at com.atlassian.urlfetcher.URLFetcherImpl.fetchSerializedObject(URLFetcherImpl.java:16)
        at com.atlassian.aws.ec2.EC2Utils.getUserData(EC2Utils.java:69)
        at com.atlassian.bamboo.agent.elastic.client.ElasticAgentBootstrap.main(ElasticAgentBootstrap.java:41)

This is both expected and normal right now as we have not started our EC2 instance from Bamboo. When we do launch this instance from Bamboo the execution environment will be changed and the script will not error out.

Now we need to schedule that script to startup when the system does.

Do so by creating a schedule task (in Control Panel) that uses this new run.bat file we created and is set to run when the system starts.

We’re done building the image. The only time you will need to come back to this step is when your version of Bamboo changes as you’ll need to update the version of the assembly you have installed. (Also remember to update the paths in the run.bat script and the xml config file)

### Making our system an AMI image

With the system now in place we can turn it into a reusable image that we will have Bamboo launch when we need a build resource.

Shutdown your machine from within it (ie. Start->Shutdown). **It must be shutdown cleanly**, otherwise it may require intervention when started and then the Bamboo agent (obviously) won’t run properly. This will put your machine into a stopped state in your EC2 control panel. Once there right click the instance and choose “Create Image (EBS AMI)” and follow the prompts.

This process will take awhile, between 15 and 20 minutes seems to be the average, during this time you can find the image, in a a pending status, under Images->AMIs on the left menu of the AWS console. When completed its status will change to available and you can provide its AMI ID to your instance of Bamboo.

You should now be able to start up an instance of your AMI ID from within Bamboo and after about 5 minutes you should see the Elastic Agent on the instance enter into Idle state.

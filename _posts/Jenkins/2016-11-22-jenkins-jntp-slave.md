---
layout: post
title:  "Headless Browser Testing With Xvfb"
categories: Jenkins
tags: Jenkins jntp
---


#### Connect slave to Jenkins one of these ways:

*    Launch agent Launch agent from browser on slave

*    Run from slave command line:

		```
		# javaws http://$jenkins-url/computer/Ethel-Web-Testing/slave-agent.jnlp
		```

*    Or if the slave is headless:

		```
		# java -jar slave.jar -jnlpUrl http://$jenkins-url/computer/Ethel-Web-Testing/slave-agent.jnlp
		```

		```
		# java -jar slave.jar -jnlpUrl http://$jenkins-url/computer/Ethel-Web-Testing/slave-agent.jnlp -secret a132ae64a7e1a85c4592455af8ff9d9181ef4939f5f522a0a3a06a0c83682937
		```

    


#### Browser never opens when running Robot Framework tests from Jenkins
[Source Link](http://stackoverflow.com/questions/24842837/browser-never-opens-when-running-robot-framework-tests-from-jenkins)

If your Jenkins job needs to run something that displays a GUI, you cannot run that build in Jenkins which runs as a background service (whether on Windows, Mac or Linux).

(In Linux you can play tricks with Xvnc or similar fake X servers and there are even Jenkins plugins that make it simpler.)

Your alternatives are either:

*    Log in using the GUI session and run Jenkins in a terminal window by typing java -jar /path/to/jenkins.war. When Jenkins is started in the GUI context, any processes started by Jenkins are able to talk to the GUI system and draw windows.
*    Or you can set up a JNLP slave in Jenkins, then log in using the GUI session, open a web browser to access your Jenkins and start the JNLP slave that connects to Jenkins master and now that the slave is running in GUI context, you can configure the job to execute on the slave. Processes that execute in the slave will be able to talk to the GUI system and draw windows.



## Headless Browser Testing With Xvfb
[Source Link](http://tobyho.com/2015/01/09/headless-browser-testing-xvfb/)

*    install xvfb

     Download rpm package, then install - [xorg-x11-server-Xvfb-1.15.0-7.el7.x86_64.rpm](http://rpm.pbone.net/index.php3/stat/4/idpl/26648255/dir/centos_7/com/xorg-x11-server-Xvfb-1.15.0-7.el7.x86_64.rpm.html)

*    install nodejs

    yum -y install nodejs

*    install npm

    curl -L https://www.npmjs.com/install.sh | sh


*    install testem

    npm install testem -g

*    Install Chrome on RHEL7

```
# vim /etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub

# yum install google-chrome-stable
```

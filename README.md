# zap-automation
This scala library provides an abstraction above the [ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project) API which allows for simple configurable execution of spider and active scans.  The zap-automation library also produces a report summarising the alerts captured during scans, and can be tuned to fail your test run depending on the severity of the vulnerabilities found.

## Implementing a test using zap-automation

### Prerequisites
Prior to launching an active/spider scan with the zap-automation library, you will need to set up the following prerequisites:
- generate a ZAP session with an automated UI or API test suite (see below)
- start OWASP Zap (referencing the ZAP session)
- start the Services you wish to test

### Key Terminologies

#### Policy 
Policy defines the rules required for active scan. The library sets up a policy with the scanners to be used during an
active scan. More information about Zap Policy can be found [here](https://github.com/zaproxy/zap-core-help/wiki/HelpStartConceptsScanpolicy).
 
#### Context
A Zap context limits the scope of the test to the domain and technologies provided and excludes any routes that needs to be ignored.
The library makes uses of pre-defined parameters in application.conf to create a Zap context. 
More information about Context can be found [here](https://github.com/zaproxy/zap-core-help/wiki/HelpStartConceptsContexts) 
 
#### Passive Scan
A non invasive scan by Zap that checks for security vulnerabilities by just analysing the requests and responses that are proxied through Zap.
This is done automatically, no additional configuration required. 
More about passive scan can be found [here](https://github.com/zaproxy/zap-core-help/wiki/HelpStartConceptsPscan).

#### Spider
Zap uses spider to discover new resources (URLs) on a particular site. During this process, 
Zap also requests the resource and analyze the requests and response. The library uses the testUrl provided in the
 application.conf as a seed for the spider. More about spider [here](https://github.com/zaproxy/zap-core-help/wiki/HelpStartConceptsSpider). 

#### Active Scan
Active scan is an invasive attack. During active scan ZAP manipulates the requests and attacks the provided testUrl to 
 find potential vulnerabilities. Because of this nature, active scan should be run only against your own services. 
  By default Active Scan is turned off in zap-automation library. Teams interested in running an active scan using this library should set
  activeScan:true in their application.conf. See [active scan](https://github.com/zaproxy/zap-core-help/wiki/HelpStartConceptsAscan) for more information.

#### Zap Session
Todo

### In your SBT build add:

```scala
resolvers += Resolver.bintrayRepo("hmrc", "releases")

libraryDependencies += "uk.gov.hmrc" %% "zap-automation" % "x.x.x"
```

### Create the configuration
In your test suite's application.conf create a zap-config object as shown in the example [here](examples/singleConfigExample/resources/singleConfigExampleApplication.conf). If you have multiple 
security tests as part of the same suite and use different config for each of them, then refer to this example [here](examples/multipleConfigExample/resources/multipleConfigExampleApplication.conf).
  
###  Create the test
Create a runner in your test suite by extending the ZapTest trait of the zap-automation library. The runner is required 
to extend a testSuite. This can be done by extending any of ScalaTest's [testing styles](http://www.scalatest.org/user_guide/selecting_a_style). 
The Zap runner is also expected to override the ZapConfiguration and call the triggerZapScan() method to trigger the scan.
A detailed example is available [here](examples/singleConfigExample/SingleConfigExampleRunner.scala).

### Execute the test
* Start your application locally
* Changing the directory to where ZAP is installed (default Mac installation is in the root Applications directory: /Applications)
* Run this command: ZAP\ 2.7.0.app/Contents/Java/zap.sh -daemon -config api.disablekey=true -port 11000
* Run your acceptance tests pointing at your new ZAP browser profile

You need to make sure you run enough UI tests to hit all the urls that you want to run your ZAP tests on. This may be all of your tests or a subset, it’s up to you.
Once the acceptance tests are completed, run the penetration tests (using your new ZapRunner file). If the ZapRunner is under
utils.Support package, then the command to run the Zap tests will looks like this:
```sbt "testOnly utils.Support.ZapRunner"```

### Create the ZAP Session
#### Configure your tests to proxy requests via ZAP proxy
For Zap to check security vulnerabilities of your application, it needs to know the various endpoints and flow of the application. 
This can be achieved by using Zap as a proxying tool. When the Application Under Test uses Zap to proxy its requests, 
Zap performs a non invasive passive scan checking for vulnerabilities. To achieve this configure your test suite to proxy via Zap
by creating a new browser profile for ZAP in your test suite and specifying zap proxy details. This can be done
as below for firefox and chrome respectively:
   
   ```scala
       val profile: FirefoxProfile = new FirefoxPrfile
       profile.setAcceptUntrustedCertificates(true)
       profile.setPreference("network.proxy.type", 1)
       profile.setPreference("network.proxy.http", "localhost")
       profile.setPreference("network.proxy.http_port", 11000)
       profile.setPreference("network.proxy.share_proxy_settings", true)
       profile.setPreference("network.proxy.no_proxies_on", "")
       
       var options = new ChromeOptions()
       options.addArguments("test-type")
       options.addArguments("--proxy-server=http://localhost:11000")
       
//       TODO: Add proxy details for API
   ``` 


#### How do we read the output of the tests?
An html report is created at /target/zap-reports/ZapReport.html irrespective of the test outcome. Even when there are no alerts found, a report is generated indicating the summary of scans ran and the failureThreshold set. If you are surprised about getting a green build, it may be that you need to adjust the variables you are passing to the library, or you may not have run enough UI tests proxying through ZAP. 

| Key        | Description           | 
| ------------- |:-------------| 
| Low (Medium)  | Low is the Risk Code  and Medium is the Confidence Level. Risk Code is the risk of each type of vulnerability found. Confidence represents ZAP's "sureness" about the finding.| 
| URL      | The Url in which the alert was identified      |  
| Scanner ID | Id of the scanner. The passive and active scanners for your zap installation can be found at http://localhost:11000/HTML/pscan/view/scanners/ and http://localhost:11000/HTML/ascan/view/scanners/       |   
| CWE Id| [Common Weakness Enumeration (CWE™) Id](https://cwe.mitre.org/about/faq.html).      |   
| Method| HTTP method      |   
| Parameter| Parameter used for the test      |   
| Evidence| Evidence for the alert      |   
| Description| Description of the alert      |   
| Solution| Solution for the alert      |   
| Reference(s)| Future use      |   
| Internal References(s)| Future use      |   

### Supported browsers
We have tested the library using Chrome and Firefox.

## Development
### Run the unit tests for the library
```scala
sbt test
```

### Debugging
The library provides various debug flags for testing locally .....

### Issues
Please raise issues/feedback here

### License
This code is open source software licensed under the [Apache 2.0 License]("http://www.apache.org/licenses/LICENSE-2.0.html").
    

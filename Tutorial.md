# Tutorial #

**snuck** is an automated tool that can definitely help in finding XSS vulnerabilities in web applications. It is based on Selenium and supports Mozilla Firefox, Google Chrome and Internet Explorer.
The approach, it adopts, is based on the inspection of the injection's reflection context and relies on a set of specialized and obfuscated attack vectors for filter evasion. In addition, XSS testing is performed in-browser, a real web browser is driven in reproducing the attacker and possibly the victim's behavior.

# Description #

**snuck** is quite different from typical web security scanners, it basically tries to break a given XSS filter by specializing the injections in order to increase the success rate. The attack vectors are selected on the basis of the reflection context, that is the exact point where the injection falls in the reflection web page's DOM. Having access to the pages' DOM is possible through Selenium Web Driver, which is an automation framework, that allows to replicate operations in web browsers.
Since many steps could be involved before an XSS filter is "activated", an XML configuration file should be filled in order to make _snuck_ aware of the steps it needs to perform with respect to the tested web application. Practically speaking, the approach is similar to the [iSTAR](http://www.korscheck.de/diploma-thesis.pdf)'s one, but it focuses on one particular XSS filter.

### Download and first run ###

**snuck** is an open-source software written in Java, released under the Apache 2.0 license, you can download the sources by using svn.

```
svn checkout http://snuck.googlecode.com/svn/trunk/ snuck
```

Once checked out, you can use the build.xml file for asking Ant to compile the source files and generate the jar file.

```
cd snuck
ant jar
```

This will generate an executable jar file that is ready to run!

You can also directly download a ready-to-run executable jar from [here](http://code.google.com/p/snuck/downloads/detail?name=snuck-0.1.zip).

Note: No particular prerequisites are required, in particular you just need a working JVM and Firefox installed. Furthermore, if you want to run a test with Google Chrome/Chromium, you should download the appropriate server, which is a bridge between the web browser and the driver - refer to http://code.google.com/p/chromedriver/downloads/list. A similar procedure is required for Internet Explorer too, refer to http://code.google.com/p/selenium/downloads/list. The tool has been tested with IE9 and has proven to work successfully; some issues could possibly appear with older versions of IE, but we are working to make snuck compatible with these too.
Obviously since the tool is written in Java, you can run it in any platform.

Once you downloaded/generated the jar file, you will need to become familiar with the command line options, here follow the available arguments and the correspondent description.
```
> java -jar snuck.jar
Usage: snuck [-start xmlconfigfile ] -config xmlconfigfile -report htmlreportfile [-d # ms_delay] 
[-proxy IP:port] [-chrome chromedriver ] [-ie iedriver] [-remotevectors URL] [-stop-first]
[-reflected targetURL -p parameter_toTest] [-no-multi]

Options :

  -start         path to login use case (XML file)
  -config        path to injection use case (XML file)
  -report        report file name (html extension is required)
  -d             delay (ms) between each injection
  -proxy         proxy server (IP: port)
  -chrome        perform a test with Google Chrome, instead of Firefox. It needs the path to the chromedriver
  -ie            perform a test with Internet Explorer, instead of Firefox.
                 Disable the built in XSS filter in advance
  -remotevectors use an up-to-date online attack vectors source instead of the local one
  -stop-first    stop the test upon a successful vector is detected
  -no-multi      deactivate multithreading for the reverse engineering process - a sequential approach will be adopted
  -reflected     perform a reflected XSS test (without writing the XML config file)
  -p             HTTP GET parameter to inject (useful if -reflected is set)
  -help          show this help menu 
```

### XSS Attack Vectors ###

The tool keeps a set of XSS vectors, that you can find in the directory named _payloads_; this latter contains four files:

  * **html\_payloads**. it stores HTML tags whose purpose is to generate an alert dialog window. Placeholders could be used within this set of vectors; for instance, if we have `<script src=data:,%alert%></script>`, then the tool will pick a javascript alert from the following attack vector set at random to be the substitute of `%alert%`. Something like `<svg onload=%uri%>` will be treated similarly, obviously the drawing will happen among the URIs vectors (see below).
  * **js\_alert** payloads it stores many javascript approaches to trigger an alert dialog window, such as alert(1) or eval(alert(2)).
  * **uri\_payloads** it stores malicious URIs, such as `javascript:alert(1)`.
  * **expression\_alert\_payloads** it stores malicious expression payloads, such as `expression(URL=0)`; in this case it is mandatory to produce a redirect to a new URL ending with "0" in order to catch whether a vulnerability exists. Unfortunately `expression(alert(1))` would flood the web browser (IE), while `expression(write(1))` makes the browser freeze, finally `expression(alert(URL=1))` produces multiple alert dialogs and this is annoying from the web driver's perspective.

Obviously the tester is allowed to add vectors in these sets by just adding a new line. Furthermore, it is possible to employ a remote attack vectors repository instead of the local one, this can be done by starting the tool with the `-remotevectors` argument. The remote repository should be a URL whose content is the directory called `payloads` - for instance if the repository is reachable at http://www.example.com/repository/, then the tool will look for the four payload files in http://www.example.com/repository/payloads/.

### Howto and sample scenarios ###

In the following section you'll find many typical scenarios, you might be interested to, in order to become familiar with the tool and especially with the use cases - XML configuration files - design.

  * **Scenario I: Stored XSS** Let us assume to have an e-commerce web site, which allows sellers to modify the description of their items. Items' descriptions are publicly accessible, thus leading to stored XSS in the case of XSS filter bugs. Testing the XSS filter needs three steps to be performed before landing in the reflection page. Since the injection is reflected within a P html element, the tool will reverse the filter and report the allowed tags and attributes in the final report. Obviously detected XSS are reported as well. See the following image for better understanding the context.
> > ![http://www.sneaked.net/snuck/images/tutorial_I.png](http://www.sneaked.net/snuck/images/tutorial_I.png)

> In the aforementioned case _snuck_ need to know which operations are required to connect the injection page, that is the web page in which the malicious payload is supplied, to the reflection one, that is the web page in which the payload is reflected. This task can be done by passing an XML configuration file, such as the following. In particular every `<command>` reports a tuple in the form of Selenium commands, refer to Selenium selectors for further information.
```
<?xml version="1.0" encoding="UTF-8"?>
<root>
  <post>
        <commands>
            <command>
                <name>open</name>
                <target>http://wtfbay.com/modify.php?id=90</target>
                <value></value>
            </command>
            <command>
                <name>type</name>
                <target>name=name</target>
                <value>${RANDOM}</value>
          </command>                                                                     
          <command>
                <name>type</name>
                <target>id=description</target>
                <value>${INJECTION}</value>
           </command>
           <command>
                <name>click</name>
                <target>name=submit</target>
                <value></value>
           </command>
          <command>
                <name>select</name>
                <target>id=cat</target>
                <value>Bike</value>
            </command>
            <command>
                <name>click</name>
                <target>name=submit</target>
                <value></value>
            </command>
        </commands>
    </post>
</root>
```

> In addition, since it is obvious that a login operation is required for managing our items, then another XML configuration file is needed.
```
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <post>
        <commands>
            <command>
                <name>open</name>
                <target>http://wtfbay.com/login.php</target>
                <value></value>
            </command>
            <command>
                <name>type</name>
                <target>name=user</target>
                <value>admin</value>
           </command>                                                                                                  
           <command>
                <name>type</name>
                <target>name=pwd</target>
                <value>admin</value>
           </command>
           <command>
                <name>click</name>
                <target>name=submit</target>
                <value></value>
           </command>
        </commands>
    </post>
</root>
```

> So far so good, now, how can we start the tool? By assuming that we want an HTML report named report.html and the aforepresented XML files are named usecase.xml and login.xml, then:
```
> java -jar snuck.jar -config usecase.xml -report report.html -start login.xml
```

> What will the tool do? Basically the tool will firstly reverse engineer the HTML filter used against the user-supplied description, then it will start injecting multiple attack vectors and check whether an XSS is triggered - the important point here is that alert dialog windows are grabbed in order to decide whether an injection is successful; this means that false positive rate is equal to 0 as a vulnerability is reported just in case an alert dialog window is grabbed by the used web browser (note that this is automatically performed).

  * **Scenario II: Stored XSS** Let us assume to have a blog, which allows visitors to leave comments. As in the previous case, our goal is to bypass the filter the web application is using against user generated content. Since the tool just need to fill some field and perform the attack, this is actually comparable to a fuzzy approach. See the following image for better understanding the context.
> > ![http://www.sneaked.net/snuck/images/tutorial_II.png](http://www.sneaked.net/snuck/images/tutorial_II.png)

> The XML configuration file (use case) and the necessary parameters for starting _snuck_ are reported below.
```
<?xml version="1.0" encoding="UTF-8"?>
<root>               
    <post>                                                                                                      
         <parameters>
            <parameter>
                <name>username</name>
                <value>hal9001</value>
            </parameter>
        </parameters>
        <commands>
            <command>
                <name>open</name>
                <target>http://examplecom/post.php?id=90</target>
                <value></value>
            </command>
            <command>
                <name>type</name>
                <target>name=name</target>
                <value>${USERNAME}</value>
           </command>                                                                                      
            <command>
                <name>type</name>
                <target>id=comment</target>
                <value>${INJECTION}</value>
           </command>
           <command>
                <name>click</name>
                <target>name=submit</target>
                <value></value>
           </command>
        </commands>
    </post>
</root>
```
```
> java -jar snuck.jar -config usecase.xml -report report.html
```
> The remarkable points here are that variables can be defined through the `<parameter>` tags and referenced by using strings in the form of `${variable_name}`. In addition you can use `${RANDOM}` or `${RANDOM_EMAIL} `within `<value>` tags for asking the tool to supply a random alphanumeric string or a random email.

  * **Scenario III: Reflected XSS** Here we present a basic scenario in which the value of an HTTP GET parameter is reflected in a web page, in particular in the attribute `value` of an `input` element, whose `type` is setted to `hidden`. As usual, we want to test whether injecting this parameter could result in a reflected XSS vulnerability - see below for the XML configuration file and how it is possible to start _snuck_ - in this case we have two different opportunities.
> ![http://www.sneaked.net/snuck/images/tutorial_III.png](http://www.sneaked.net/snuck/images/tutorial_III.png)
```
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <get>
        <parameters>
            <targeturl>http://example.com/xss.php</targeturl>
            <reflectionurl></reflectionurl>
            <paramtoinject>param</paramtoinject>
            <parameter>foo=bar</parameter>
        </parameters>
    </get>
</root>
```
```
> java -jar snuck.jar -config config.xml -report report.html 
```
```
# faster mode (no XML file required)
> java -jar snuck.jar -report report.html -reflected "http://example.com/xss.php?foo=bar" -p param
```
> Note that the `<reflectionurl>` can be used for asking the tool to make the injections at `<targeturl>` and to look for reflections at the URL defined with `<reflectionurl>`.

**Further information** Sometimes it might be useful to discover the first successful injection without realizing a complete test. This task could be achieved through the argument `-stop-first`, which makes the tool stop at the first positive, it will find. In addition, you could obviously stop the test through CTRL+C, if so, then the tool will print in stdout the successful injections that it found so far.
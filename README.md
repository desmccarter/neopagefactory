# NEO PageFactory

(*Authored by Des McCarter*)

## Introduction

> Note: If you are familiar with Page Object patterns then you can skip this introduction and jump to [Install and Run](#installandrun).

**NEO PageFactory is an extension to the** [SeleniumHQ PageFactoy](https://github.com/SeleniumHQ/selenium/wiki/PageFactory) **library**. 

### What is SeleniumHQ PageFactory?

PageFactory enables testers to implement web / mobile based tests based on the [Page Object Pattern](http://toolsqa.com/selenium-cucumber-framework/page-object-design-pattern-with-selenium-pagefactory-in-cucumber/). In short, each page under test is mapped to an object (java, C#, ... class) with getters and setters of that class representing the *setting* and *getting* (and *clicking* etc) of page elements. **NEO PageFactory goes one step further and *implements* page object pattern based classes for you**. It scans HTML documents (via HTTP or a local HTML text file) and, based on elements found on all pages, generates page class files containing getters and setters etc representing the elements found.

> The **Page Object Pattern** is a method of representing web pages being tested as a collection of classes/objects individually. Each class represents a page and all pages have getter / setter methods representing the elements of that page.  

> **To find out more about the Page Object Pattern click** [here](http://toolsqa.com/selenium-cucumber-framework/page-object-design-pattern-with-selenium-pagefactory-in-cucumber/).

#### SeleniumHQ PageFactory Example

Below is an example using the Page Object Pattern. There is a class named *GoogleSearchPage*. This class represents the actual [Google Search Page](https://wwww.google.com) with a method named *searchFor* representing both the search box and *q* submit button of www.google.com. This class was *hand crafted*, meaning that it was (or would have been) implemented by hand.

##### Manually coded page class using standard SeleniumQA PageFactory

```java
package org.openqa.selenium.example;

import org.openqa.selenium.WebElement;

// A manually crasfted class representing
// the Google Search page ...

public class GoogleSearchPage {
    // Here's the element
    private WebElement q;

    public void searchFor(String text) {
        // And here we use it. Note that it looks like we've
        // not properly instantiated it yet....
        q.sendKeys(text);
        q.submit();
    }
}
```

*GoogleSearchPage* is then used in a test below (example test: search for the text *BJSS Location*):


```java
package org.openqa.selenium.example;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;
import org.openqa.selenium.support.PageFactory;

public class UsingGoogleSearchPage {
    public static void main(String[] args) {
        // Create a new instance of a driver
        WebDriver driver = new HtmlUnitDriver();

        // Navigate to the right place
        driver.get("http://www.google.com/");

        // Create a new instance of the search page class
        // and initialise any WebElement fields in it.
        GoogleSearchPage page = PageFactory.initElements(driver, GoogleSearchPage.class);

        // And now do the search.
        page.searchFor("BJSS Location");
    }
}
```

#### NEO PageFactory Example

As seen in the **previous example, page classes are *manually coded*** by the tester/developer testing the page. **NEO Page Factory** goes one step further by ***dynamically generating* all classes for pages under test**.

##### Dynamically generated page class using NEO PageFactory

```java
package com.dmcc.sample.pages.google;

import com.dmcc.pagegen.exceptions.PageException;
import com.dmcc.sample.pages.google.GoogleField;
import com.dmcc.pagegen.page.mccarterp.McCarterPage;

// Dynamically generated page class: generated from https://www.google.com ...

public class GooglePage extends McCarterPage{

private final String url="https://www.google.com";
private final String rRoot="../pgenexamples/src/test/resources";

	public GooglePage navigate()throws PageException {
		this.setResourcesRoot(rRoot);
		this.navigate(url);
		return this;
	}

	public void setQ(final String value) throws PageException{
		this.setValue(GoogleField.Q, value);
	}

	public void clickQ()throws PageException{
		this.click(GoogleField.Q);
	}

	public void clickGoogleSearch()throws PageException{
		this.click(GoogleField.GoogleSearch);
	}

	public void clickIMFeelingLucky()throws PageException{
		this.click(GoogleField.IMFeelingLucky);
	}

}
```

We can then use this class (*GooglePage*) within a test case, as can be seen below:

```java
package com.dmcc.pgenexamples.google;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import com.dmcc.pagegen.exceptions.PageException;
import com.dmcc.pgenexamples.TestAbstract;
import com.dmcc.sample.pages.google.GooglePage;

public class GoogleTests extends TestAbstract {
	
	@Before
	public void setup(){
		init();
	}
	
	@After
	public void teardown(){
		deinit();
	}
	
	@Test
	public void test01ImFeelingLucky() throws PageException{
		GooglePage page = new GooglePage().navigate();
		
		page.setQ("BJSS Location");
		page.clickIMFeelingLucky();
	}
	
	@Test
	public void test02Search() throws PageException{
		GooglePage page = new GooglePage().navigate();
		
		page.setQ("BJSS Location");
		page.clickGoogleSearch();
	}
}
```

#### Advantages os NEO PageFactory

Note the difference between the the standard PageFactory and NEO PageFactory test implementations:

##### 1. No need to manually code your page classes

**All elements found on the Google Search page have methods generated** (*Q* represents the search *text box*, *ImFeelingLucky* represents the *I'm Feeling Lucky button* (which renders the first page found in its search) and *GoogleSearch* represents the *standard Google Search button*.

###### Generated methods

```java
// Methods generated for Google Search page
// web elements ...

	public void setQ(final String value) throws PageException{
		this.setValue(GoogleField.Q, value);
	}

	public void clickQ()throws PageException{
		this.click(GoogleField.Q);
	}

	public void clickGoogleSearch()throws PageException{
		this.click(GoogleField.GoogleSearch);
	}

	public void clickIMFeelingLucky()throws PageException{
		this.click(GoogleField.IMFeelingLucky);
	}
```

This is a simple example but the efficiency is more predominant for pages *richer* in web elements.

##### 2. Low level webdriver (etc) initialisation implied (hidden)

Web Driver initialisation is handled *outside* the actual test case by **implementing the *TestAbstract* class and *calling init/deinit*** (before and after test execution repectively). The page is then navigated to by calling the page objects `navigate()` method:


```java
	@Before
	public void setup(){
		// *** set-up webdriver (etc) ...
		init();
	}
	
	@After
	public void teardown(){
		// *** de-init webdriver (etc) ...
		deinit();
	}

...
// and page navigation ...
// (a. Get an instance of GooglePage b. fire up *a* browser and navigate to it ...)
...

		GooglePage page = new GooglePage().navigate();
```

The type of webdriver is supplied as a maven argument on test execution.

##### 3. Test case is a lot clearer as to what it is doing

Test steps are now a lot clearer as to their intentions:

```java
...
	// Fire up a google page and render the first search page found by
	// searching for text (BJSS Location) ...

	@Test
	public void test01ImFeelingLucky() throws PageException{
		GooglePage page = new GooglePage().navigate();
		
		page.setQ("BJSS Location");
		page.clickIMFeelingLucky();
	}
	
	// Fire up a google page and list all pages found by 
	// searching for text (BJSS Location) ...

	@Test
	public void test02Search() throws PageException{
		GooglePage page = new GooglePage().navigate();
		
		page.setQ("BJSS Location");
		page.clickGoogleSearch();
	}
```

## Requirements

### Software Requirements

|Requirement|Version|
|--|--|
|Java SDK|1.8.0_144|
|Maven|3.5.0+|
|Bash|ANY|

> NOTE: Remember also to set-up your JAVA_HOME and MAVEN_HOME environment variables, along with supplying their *bin* locations in the *PATH* environment variable.

## Install and Run NEO PageFactory

<p name="installandrun"></p>

### Install

1. Open a bash instance

2. Go to a folder where you wish to install this project (I am using ~/projects in this example)
```bash
cd ~/projects
```

3. Clone [this](https://github.com/desmccarter/neopagefactory) project in bash.
```bash
git clone https://github.com/desmccarter/neopagefactory
```

4. Build and install this project 
```bash
cd neopagefactory
mvn clean install
```

### Run

> NOTE: this exmaple uses https://www.google.com but you can generate page classes with *any* URL

1. Go to your workspace folder (I am using. *~/projects/myproject* as an example).
```bash
cd ~/projects/myproject
```

2. Generate your page classes
```bash
~/projects/neopagefactory/generatepages.sh -url https://www.google.com
```

3. Classes will be generated by default in *your current location*/src/test/java/com/dmcc/sample/pages/*google*
> NOTE: The folder name *google* is taken from your *-url* argument.

```bash
-rw-r--r-- 1 Des.McCarter 1049089 317 Aug 13 08:56 GoogleField.java
-rw-r--r-- 1 Des.McCarter 1049089 862 Aug 13 08:56 GooglePage.java
```
> NOTE: GooglePage.java represents your page class. GoogleField.java represents the fields (web elements) that have been discovered on https://www.google.com.

## Generated pages: Examples

Here is an example POP (*Page Object Pattern*) classes generated using this utility.


### Google.com

```java
package com.dmcc.sample.pages.google;

import com.dmcc.pagegen.exceptions.PageException;
import com.dmcc.sample.pages.google.GoogleField;
import com.dmcc.pagegen.page.mccarterp.McCarterPage;

public class GooglePage extends McCarterPage{

private final String url="https://www.google.com";
private final String rRoot="../pgenexamples/src/test/resources";

	public GooglePage navigate()throws PageException {
		this.setResourcesRoot(rRoot);
		this.navigate(url);
		return this;
	}

	public void setQ(final String value) throws PageException{
		this.setValue(GoogleField.Q, value);
	}

	public void clickQ()throws PageException{
		this.click(GoogleField.Q);
	}

	public void clickGoogleSearch()throws PageException{
		this.click(GoogleField.GoogleSearch);
	}

	public void clickIMFeelingLucky()throws PageException{
		this.click(GoogleField.IMFeelingLucky);
	}

}
```

### Expedia.co.uk

```java
package com.dmcc.sample.pages.expedia;

import com.dmcc.pagegen.exceptions.PageException;
import com.dmcc.sample.pages.expedia.ExpediaField;
import com.dmcc.pagegen.page.mccarterp.McCarterPage;

public class ExpediaPage extends McCarterPage{

private final String url="https://www.expedia.co.uk";
private final String rRoot="../pgenexamples/src/test/resources";

	public ExpediaPage navigate()throws PageException {
		this.setResourcesRoot(rRoot);
		this.navigate(url);
		return this;
	}

	public void clickNbsp()throws PageException{
		this.click(ExpediaField.Nbsp);
	}

	public void setGssSignupPassword(final String value) throws PageException{
		this.setValue(ExpediaField.GssSignupPassword, value);
	}

	public void clickGssSignupPassword()throws PageException{
		this.click(ExpediaField.GssSignupPassword);
	}

	public void clickGssJoinProgramCheck()throws PageException{
		this.click(ExpediaField.GssJoinProgramCheck);
	}

	public void clickGssMarketingEmailOptInCheck()throws PageException{
		this.click(ExpediaField.GssMarketingEmailOptInCheck);
	}

	public void clickGssMarketingEmailOptDefaultValue()throws PageException{
		this.click(ExpediaField.GssMarketingEmailOptDefaultValue);
	}

	public void setGssSigninPassword(final String value) throws PageException{
		this.setValue(ExpediaField.GssSigninPassword, value);
	}

	public void clickGssSigninPassword()throws PageException{
		this.click(ExpediaField.GssSigninPassword);
	}

	public void setFlightOriginHpFlight(final String value) throws PageException{
		this.setValue(ExpediaField.FlightOriginHpFlight, value);
	}

	public void clickFlightOriginHpFlight()throws PageException{
		this.click(ExpediaField.FlightOriginHpFlight);
	}

	public void setFlightDestinationHpFlight(final String value) throws PageException{
		this.setValue(ExpediaField.FlightDestinationHpFlight, value);
	}

	public void clickFlightDestinationHpFlight()throws PageException{
		this.click(ExpediaField.FlightDestinationHpFlight);
	}

	public void setFlight2OriginHpFlight(final String value) throws PageException{
		this.setValue(ExpediaField.Flight2OriginHpFlight, value);
	}

	public void clickFlight2OriginHpFlight()throws PageException{
		this.click(ExpediaField.Flight2OriginHpFlight);
	}
...
```

### Direct Line.com

```java
public class DirectlinePage extends McCarterPage{

private final String url="https://www.directline.com";
private final String rRoot="C:/Users/des.mccarter/projects/pgenexamples/src/test/resources";

	public DirectlinePage navigate()throws PageException {
		this.setResourcesRoot(rRoot);
		this.navigate(url);
		return this;
	}

	public void setSkipToContent(final String value) throws PageException{
		this.setValue(DirectlineField.SkipToContent, value);
	}

	public void clickSkipToContent()throws PageException{
		this.click(DirectlineField.SkipToContent);
	}

	public void setMenu(final String value) throws PageException{
		this.setValue(DirectlineField.Menu, value);
	}

	public void clickMenu()throws PageException{
		this.click(DirectlineField.Menu);
	}

	public void setDirectLineForBusiness(final String value) throws PageException{
		this.setValue(DirectlineField.DirectLineForBusiness, value);
	}

	public void clickDirectLineForBusiness()throws PageException{
		this.click(DirectlineField.DirectLineForBusiness);
	}

	public void setContactUs(final String value) throws PageException{
		this.setValue(DirectlineField.ContactUs, value);
	}

	public void clickContactUs()throws PageException{
		this.click(DirectlineField.ContactUs);
	}

	public void setCarInsurance(final String value) throws PageException{
		this.setValue(DirectlineField.CarInsurance, value);
	}

	public void clickCarInsurance()throws PageException{
		this.click(DirectlineField.CarInsurance);
	}

	public void setHomeInsurance(final String value) throws PageException{
		this.setValue(DirectlineField.HomeInsurance, value);
	}

	public void clickHomeInsurance()throws PageException{
		this.click(DirectlineField.HomeInsurance);
	}

...
```

### Facebook.com

```java
package com.dmcc.sample.pages.facebook;

import com.dmcc.pagegen.exceptions.PageException;
import com.dmcc.sample.pages.facebook.FacebookField;
import com.dmcc.pagegen.page.mccarterp.McCarterPage;

public class FacebookPage extends McCarterPage{

private final String url="https://www.facebook.com";
private final String rRoot="../pgenexamples/src/test/resources";

	public FacebookPage navigate()throws PageException {
		this.setResourcesRoot(rRoot);
		this.navigate(url);
		return this;
	}

	public void setEmail(final String value) throws PageException{
		this.setValue(FacebookField.Email, value);
	}

	public void clickEmail()throws PageException{
		this.click(FacebookField.Email);
	}

	public void setPass(final String value) throws PageException{
		this.setValue(FacebookField.Pass, value);
	}

	public void clickPass()throws PageException{
		this.click(FacebookField.Pass);
	}

	public void clickLogIn()throws PageException{
		this.click(FacebookField.LogIn);
	}

	public void setFirstname(final String value) throws PageException{
		this.setValue(FacebookField.Firstname, value);
	}

	public void clickFirstname()throws PageException{
		this.click(FacebookField.Firstname);
	}

	public void setLastname(final String value) throws PageException{
		this.setValue(FacebookField.Lastname, value);
	}

	public void clickLastname()throws PageException{
		this.click(FacebookField.Lastname);
	}

	public void setRegEmail(final String value) throws PageException{
		this.setValue(FacebookField.RegEmail, value);
	}

	public void clickRegEmail()throws PageException{
		this.click(FacebookField.RegEmail);
	}
...
```

Project is framework which will allow to write testcases for selenium using XML files.

# Project structure and artifacts.
```
xml2Selenium                         : root project for handling projects-modules
    xml2Selenium-annotations         : project which contains some manually created annotations (temporary workaround for jaxb issue).
    xml2Selenium-core                : main project which contains core of framework functionality.
        assembly                     : folder contains assembly instructions required during building of project.
	documents                    : here place for documents :)
	src
            main
                java                 : Classes of framework.
                resources            : Configuration and other stuff.
                    Routines.xsd     : Main model for this framework. Contains description of all items which could be used in testcases. It's base for codegeneration in framework.
                    jaxbBindings.xjb : All jaxb instructions for codegenration which wasn't included into Routines.xsd directly.
            test
                java                 : Unit tests of framework classes. Also place for small POCs, experiments, etc..
                resources
                    examples/xml     : some examples of testcases which could be eaten by framework.
                    html             : simplest html pages for testing framework capabilities locally
```

# First steps.
```
  git clone git://github.com/apa/xml2selenium.git
  cd xml2selenium
  mvn clean install
  mvn eclipse:eclipse
```

For now there is only one entry point for framework execution: org.jazzteam.xml2selenium.core.ConsoleRunner.java. It contains main method witch takes parameters "`**-f** \<path_to_testcase_file>`". To run\debug framework use "`-f src/test/resources/examples/xml/basic/TestCase.xml`".

# Test case basics.

Check for examples in xml2Selenium-core/src/test/resources/examples/xml.

* testcase is xml document with enclosed tag `\<TestCase>`

* every testcase at least must be valid agaist schema (`\<xsi:schemaLocation="http://www.jazzteam.org/Routines Routines.xsd">`).

* could be 0 or more imports used in testcase (`\<import resource="src/test/resources/examples/xml/basic/CommonFrames.xml" name="appFrames">`). Imports makes items available in specified resource known in current testcase under specified name (think about names as about scopes).

* testcase could contain 1 or more `\<Test>` sections. 

* every item under Test tag could contain `"ref"` attribute. It allows to referrence this item as in testcase as from outside. If element has no `"ref"` attribute or it's empty there is no possibility to use it as parent for other element. In one `\<Test>` or `\<Imports>` couldn't be more than one direct child element with the same `"ref"` (except case with empty or not specified ref).

* every item under `\<Test>` tag could contain `"extends"` attribute. Value of this attribute could be `"importName:parentItemRef"` (if we referrence item from external resource), `":parentItemRef"` or `"parentItemRef"` if we are refference item from current testcase (..context, file. It's also true for elements in external resource, however they are not under `\<Test>` tag, but under `\<Imports>` tag). 

* Inheritance rules are very simple: if attribute of child is not empty it will remain in child. **So, parent's attributes will be inherited only in case the same attributes wasn't filled by child.** 

* There are special items could be in testcase -- containers (for example `\<Frame>`, `\<for>`, etc..). All of them are extends from `\<ContainerType>` type in Routines.xsd. For them there are some special rules, related to elements inheritance. Only attributes of containers itself follow inheritance rules as usual in elements. For childs of containers (subelements) there are restrictions: 
1. elements in container could have non empty `"extends"` attribute only if container doesn't have a parent. 
2. Elements in containers which have parent could only "overrides" elements from parent container. It could be done by specifying of the same `"ref"` attribute like in element which should be "overriden".

* elements inside containers must have unique ref only in this container. Other containers could reuse the same elements refs.


# Architectrue.
The base of framework is Routines.xsd. It contains hierarchical structures of abstract and real types\elements which reflects web-, custom-, special- elements in flow in future testcase. For now Routines.xsd doesn't contain all required types\and elements, but only basic ones (web elements like `<Field>`, `<Button>`, element for expanding with custom logic, "containers" -- `<Frame>`, flow control -- `<for>`, etc..). For sure in nearest future it will be expanded\refactored with new ones. Also naming of elements could be easily refactored as soon as we have new ideas about this. 
Based on this Routines.xsd JAXB generates classes (in xml2Selenium-core/target/generated-sources/xjc/). It's controlled by instructions which placed as in Routines.xsd (almost all for now), as in jaxbBindings.xjb.
JAXB instructions related to inheritace injects generated classes into hierachy with manually created classes. See how it looks like:

```
manual   : ElementsParent            : something very basic for all types of elements in testcase -- scope, initialization
generated:  BaseElementType          : introduces attributes "ref", "extends", "action"
manual   :   BaseElementTypeImpl     : implementation of automatic initialization of inheritance from parent beans
generated:    WebElementType         : introduces attributes which common for all web elements: id, tagName, cssSelector, xpath, etc..
manual   :     WebElementTypeImpl    : implementation of functions basic for all web elements (based on tags appeared in parent)
generated:      Input                : Abstract class for all input elements.
generated:       Button              : Class generated from non abstract <Button> tag which could be used in testcases. Represents web element - button with all specific of this tag.
manual   :        ButtonImpl         : Implementation of specific behavior of button.
and so on..
```


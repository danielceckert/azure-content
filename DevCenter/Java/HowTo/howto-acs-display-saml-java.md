<properties linkid="dev-java-how-to-acs-display-saml-java" urldisplayname="Access Control" headerexpose="" pagetitle="How to view SAML retruned by the Windows Azure Access Control Service" metakeywords="Azure Access Control Service SAML Java" footerexpose="" metadescription="" umbraconavihide="0" disquscomments="1"></properties>

# How to view SAML returned by the Windows Azure Access Control Service

This guide will show you how to view the underlying Security Assertion Markup Language (SAML) returned to your application by the Winodws Azure Access Control Service (ACS). The guide builds on the [How to Authenticate Web Users with Windows Azure Access Control Service Using Eclipse topic][], by providing code that displays the SAML information. The completed application will look similar to the following.

![Example SAML output][saml_output]

For more information on ACS, see the [Next steps](#next_steps) section.

<div class="dev-callout"> 
<b>Note</b> 
<p>The Windows Azure Access Services Control Filter (by Microsoft Open Technologies) is a community technology preview. As pre-release software, it is not formally supported by Microsoft Open Technologies, Inc. nor Microsoft.</p> 
</div>

## Table of Contents

-   [Prerequisites][]
-   [Modify the JSP file to display SAML][]
-   [Add the JspWriter library to your build path and deployment assembly][]
-   [Run the application][]
-   [Next steps][]


## <a name="pre">Prerequisites</a>

To complete the tasks in this guide, complete the sample at [How to Authenticate Web Users with Windows Azure Access Control Service Using Eclipse topic][] and use it as the starting point for this tutorial.

## <a name="modify_jsp">Modify the JSP file to display SAML</a>

Modify **index.jsp** to use the following code.

	<%@ page language="java" contentType="text/html; charset=UTF-8"
	    pageEncoding="UTF-8"%>
	    <%@ page import="javax.xml.parsers.*"
	             import="javax.xml.transform.*"
	             import="org.w3c.dom.*"
	             import="java.io.*"
	             import="javax.xml.transform.stream.*"
	             import="javax.xml.transform.dom.*"
	             import="javax.xml.xpath.*"
	             import="javax.servlet.jsp.JspWriter" %>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Sample ACS Filter</title>
	</head>
	<body>
		<h3>SAML information from sample ACS program</h3>
		<%!
	    void displaySAMLInfo(Node node, String parent, JspWriter out)
	    {
	    
		    try
		    {
				String nodeName;
			    int nChild, i;
			    
			    nodeName = node.getNodeName();
			    out.println("<br>");
			    out.println("<u>Examining <b>" + parent + nodeName + "</b></u><br>");
			       
			       // Attributes.
			       NamedNodeMap attribsMap = node.getAttributes();
			       if (null != attribsMap)
			       {
	                     for (i=0; i < attribsMap.getLength(); i++)
	                     {
	                            Node attrib = attribsMap.item(i);
	                            out.println("Attribute: <b>" + attrib.getNodeName() + "</b>: " + attrib.getNodeValue()  + "<br>");
	                     }
			       }
			       
			       // Child nodes.
			       NodeList list = node.getChildNodes();
			       if (null != list)
	 		       {
			              nChild = list.getLength();
			              if (nChild > 0)
			              {                    
	
				                 // If it is a text node, just print the text.
				                 if (list.item(0).getNodeName() == "#text")
				                 {
	                                 out.println("Text value: <b>" + list.item(0).getTextContent() + "</b><br>");
				                 }
				                 else
				                 {
				                	 // Print out the child node names.
				                	 out.print("Contains " + nChild + " child node(s): ");   
		   		                     for (i=0; i < nChild; i++)
				                     {
					                    Node temp = list.item(i);
					                    
					                    out.print("<b>" + temp.getNodeName() + "</b>");
					                    if (i < nChild - 1)
					                    {
					                    	// Separate the names.
					                    	out.print(", ");
					                    }
					                    else
					                    {
					                    	// Finish the sentence.
					                    	out.print(".");
					                    }
					                    	
				                     }
					                 out.println("<br>");
					                 
					                 // Process the child nodes.
					                 for (i=0; i < nChild; i++)
				                     {
					                    Node temp = list.item(i);
					                    displaySAMLInfo(temp, parent + nodeName + "\\", out);
				                     }
				               }
			              }
			          }
			      }
			    catch (Exception e)
			    {
			    	System.out.println("Exception encountered.");
			    	e.printStackTrace();	    	
			    }
		    }
	    %>
	
	    <%
	    try 
	    {
		    String data  = (String) request.getAttribute("ACSSAML");
		    
		    DocumentBuilder docBuilder;
			Document doc = null;
			DocumentBuilderFactory docBuilderFactory = DocumentBuilderFactory.newInstance();
			docBuilderFactory.setIgnoringElementContentWhitespace(true);
			docBuilder = docBuilderFactory.newDocumentBuilder();
			byte[] xmlDATA = data.getBytes();
			
			ByteArrayInputStream in = new ByteArrayInputStream(xmlDATA); 
			doc = docBuilder.parse(in);
			doc.getDocumentElement().normalize();
			
			// Iterate the child nodes of the doc.
	        NodeList list = doc.getChildNodes();
	
	        for (int i=0; i < list.getLength(); i++)
	        {
	        	displaySAMLInfo(list.item(i), "", out);
	        }
		}
	    catch (Exception e) 
	    {
	    	out.println("Exception encountered.");
	    	e.printStackTrace();
		}
	    
	    %>
	</body>
	</html>

## <a name="add_library">Add the JspWriter library to your build path and deployment assembly</a>

Add the library that contains the **javax.servlet.jsp.JspWriter** class to your build path and deployment assembly. If you are using Tomcat, the library is **jsp-api.jar**, which is located in the Apache **lib** folder.

1. In Eclipse's Project Explorer, right-click **MyACSHelloWorld**, click **Java Build Path**, click **Configure Build Path**, click the **Libraries** tab, and click **Add External JARs**.
2. In the **JAR Selection** dialog, navigate to the necessary JAR, select it, and click **Open**.
3. With the **Properties for MyACSHelloWorld** dialog still open, click **Deployment Assembly**.
4. In the **Web Deployment Assembly** assembly, click **Add**.
5. In the **New Assembly Directive** dialog, click **Java Build Path Entries** and click **Next**.
6. Select the appropriate library and click **Finish**.
7. Click **OK** to close the **Properties for MyACSHelloWorld** dialog.

## <a name="run_application">Run the application</a>

1. Run your application in the computer emulator or deploy to Windows Azure, using the steps documented at [How to Authenticate Web Users with Windows Azure Access Control Service Using Eclipse topic][].
2. Launch a browser and open your web application. After you log on to your application, you'll see SAML information, including the security assertion provided by the identity provider.

## <a name="next_steps">Next steps</a>

To further explore ACS's functionality and to experiment with more sophisticated scenarios, see [Access Control Service 2.0][].

[Prerequisites]: #pre
[Modify the JSP file to display SAML]: #modify_jsp
[Add the JspWriter library to your build path and deployment assembly]: #add_library
[Run the application]: #run_application
[Next steps]: #next_steps
[Access Control Service 2.0]: http://go.microsoft.com/fwlink/?LinkID=212360
[How to Authenticate Web Users with Windows Azure Access Control Service Using Eclipse topic]: howto-acs-java.md
[saml_output]: ../media/SAML_Output.png
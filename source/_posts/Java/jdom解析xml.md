---
title: jdom解析xml
date: 2008-08-16 22:21:00
category: [Java]
tags: [xml]
toc: false
---
添加依赖包jdom.jar和jaxen.jar，源代码如下
<!-- more -->
```java
package upload;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import org.jdom.Element;
import org.jdom.JDOMException;
import org.jdom.input.SAXBuilder;
import org.jdom.output.Format;
import org.jdom.output.XMLOutputter;
import org.jdom.xpath.XPath;
import org.xml.sax.ErrorHandler;
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;
/**
* @author lijm
* @date 2008-08-16 given an instance of xml parsede by jdom 1\\.0. three main
*       function. 1. open an xml document and parse it.here customize an
*       ErrorHandler which described the location of error when parse xml file,
*       2.deal document include add/remove element. 3.write document to a file
*       with a format.
*/
public class JDOMParse {
/**
  * root element of an XML document.
  */
static Element root;
static {
  SAXBuilder builder = new SAXBuilder();
  builder.setErrorHandler(new ErrorHandler() {
   public void error(SAXParseException exception) throws SAXException {
    print(exception);
   }
   public void fatalError(SAXParseException exception)
     throws SAXException {
    print(exception);
   }
   public void warning(SAXParseException exception)
     throws SAXException {
    print(exception);
   }
   private void print(SAXParseException e) {
    String str = e.getSystemId();
    str += "\r\n" + e.getLineNumber() + e.getColumnNumber();
    str += "\r\n" + e.getMessage();
    System.out.println(str);
   }
  });
  try {
   root = builder.build(new File("D:/ver.xml")).getRootElement();
  } catch (JDOMException e) {
   e.printStackTrace();
  } catch (IOException e) {
   e.printStackTrace();
  }
}
static Element addDownload(Element e, Download d) {
  Element download = new Element("download").setAttribute("desc", d.desc)
    .addContent(d.url);
  return e.getChild("downloads") == null ? e.addContent(new Element(
    "downloads").addContent(download)) : e.getChild("downloads")
    .addContent(download);
}
static Element removeDownload(Element e, Download d) {
  Element downloads = e.getChild("downloads");
  try {
   downloads.removeContent((Element) XPath.selectSingleNode(downloads,
     "./download[@desc='" + d.desc + "']" + "[text()='" + d.url
       + "']"));
  } catch (JDOMException e1) {
   e1.printStackTrace();
  }
  return e;
}
/**
  * @param args
  * @throws FileNotFoundException
  */
public static void main(String[] args) throws IOException {
  Download d = new Download("local", http://apache.org);
  JDOMParse.removeDownload(JDOMParse.root, d);
  XMLWriter.write(new File("D:/ver.xml"));
  BufferedReader br = new BufferedReader(new InputStreamReader(
    new FileInputStream("D://ver.xml")));
  String str = null;
  do {
   str = br.readLine();
   System.out.println(str);
  } while (str != null);
}
}
class Download {
public String desc = "local";
public String url = "";
Download(String desc, String url) {
  this.desc = desc;
  this.url = url;
}
}
class XMLWriter {
static XMLOutputter out;
static Format format;
static {
  out = new XMLOutputter();
  format = Format.getPrettyFormat()
    .setTextMode(Format.TextMode.NORMALIZE);
  format.setEncoding("GBK");
  out.setFormat(format);
}
static void write(File f) {
  try {
   out.output(JDOMParse.root, new FileOutputStream(f));
  } catch (FileNotFoundException e) {
   e.printStackTrace();
  } catch (IOException e) {
   e.printStackTrace();
  }
}
}
```

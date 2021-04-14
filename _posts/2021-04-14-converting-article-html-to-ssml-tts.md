---
layout: single
classes: wide
title: "Automatic Conversion of Article HTML to SSML For Text To Speech"
description: "Examples of using Kotlin to convert articles into SSML For Text-To-Speech in a new Podcast web app I'm working on."
categories:
  - Projects
---

As a side project I'm creating a small web application that takes RSS feeds of long form articles, performs text-to-speech on them and then exposes the audio as a podcast RSS feed. This involves a fairly hairy multi-step process. This post covers an improvement I recently made to enhance one step of the flow.

The application uses [Google Cloud's Text to Speech](https://cloud.google.com/text-to-speech) to convert the text extracted from articles into audio. GCP TTS accepts two different ways to encode the text:
  1. You can provide it with raw text.
  2. You can mark up the text with [Speech Synthesis Markup Language (SSML)](https://cloud.google.com/text-to-speech/docs/ssml)

SSML lets you add context to the text like pauses, emphasis, and boundaries around sections like paragraphs and sentences. This is particularly useful for manually annotating written text to provide context. I wanted to investigate automatically wrapping the text in SSML tags using the semantics of the html tags.

## Current state of the project

The previous flow for the project looked like this:

  1. Article links from the website's RSS are collected and saved regularly as the feed is refreshed.
  2. Each article becomes an episode in the podcast RSS feed. Audio for each episode is generated on demand as it is requested.
  3. Many sites with articles try to prevent bots from scraping their pages. One shortcut to get around this is to use the Wayback Machine's copy of the article. I use the [Wayback Machine's API](https://archive.org/help/wayback_api.php) to get the link for the most recent crawl of the page. 
  4. Postlight used to offer an API for extracting Article text from a webpage but they shut the API down. It is now available as an [open source project](https://postlight.com/insights/mercury-goes-open-source). Additionally the community has bundled this up into a [Docker container](https://hub.docker.com/r/wangqiru/mercury-parser-api) which I deploy onto [Google Cloud Run](https://cloud.google.com/run).
  5. The article text returned from Mercury retains some of the markup from the webpage. I use jsoup to [extract the readable text](https://jsoup.org/cookbook/extracting-data/attributes-text-html) from the html fragment.
  6. GCP TTS has a [limit](https://cloud.google.com/text-to-speech/quotas) of 5000 total characters per request (include SSML tags). This means the text has to be divided into sections under that limit. We could naively divide the text at exactly 5000 characters but that could mean splitting the text mid sentence or even mid word. This would lead to less natural sounding audio. Instead I want to split the text on word boundaries. Currently I'm using Apache OpenNLP's [sentence detector](https://opennlp.apache.org/docs/1.7.1/apidocs/opennlp-tools/opennlp/tools/sentdetect/SentenceDetectorME.html) to handle splitting the sentences. I then group sentences into sections of less then 5000 characters.
  7. Each section is submitted to GCP TTS. The audio for all the sections are then merged into a single audio file and finally uploaded to GCP Storage where they can be served to podcast players.

### A note on GCP TTS vs other options

Amazon Web Services' competitor to GCP TTS is AWS Polly. Its text length limit is smaller than GCP TTS. However it also started offering a [Asynchronous Synthesis](https://aws.amazon.com/blogs/aws/amazon-polly-update-time-driven-prosody-and-asynchronous-synthesis/#Asynchronous%20Synthesis) option in 2018. You can submit text up to 100,000 characters and an additional 100,000 characters in SSML markup. The catch is the the output will be automatically stored in S3 rather than returned by the API call. This is not a drop in replacement for all TTS applications but is actually perfect for my needs since I would be serving the audio out of S3 anyways. A simpler implementation and lower network costs sounds like a win-win. Why am I not using it?

Basically its because Google's WaveNet voices sound _really_ good and they added [WaveNet voices](https://cloud.google.com/blog/products/ai-machine-learning/cloud-text-to-speech-expands-its-number-of-voices-now-covering-33-languages-and-variants) to GCP TTS in 2019. So the tradeoff I'm making is added complexity and costs in order to improve the listening experience. Since right now I'm implementing this for my own needs and not as a product to sell I'm OK with this tradeoff. If I were expand this as a service to others I might move over to AWS since it seems all around cheaper for my needs.

## What changed

The steps above that need to change are 6 and 7 where I remove the tags from the article text and split the text on sentence boundaries. Instead of stripping all the tags I want to use them to generate SSML for the text.

Lets take this [article](https://www.bloomberg.com/opinion/articles/2021-04-08/archegos-got-too-big-for-its-banks) from Matt Levine's weekday Money Stuff* Opinion Column as an example.

Here is some of the HTML fragment that Mercury parser sends back:
{% highlight html %}
{% raw %}
<div class=\"body-copy-v2 fence-body\"> <p><strong>Programming note:</strong> <em>Money Stuff will be off tomorrow, back
    on Monday.</em></p><h2>Archegos fallout</h2>
<p>Generally speaking, in the U.S., if you want to <a
        href=\"https://web.archive.org/web/20210408195204/https://www.wsj.com/articles/investors-big-and-small-are-driving-stock-gains-with-borrowed-money-11617799940\">
    <meta>
    borrow money from your broker to buy stocks</a>, you are capped at 2-to-1 leverage. If you have $100, you can buy
    $200 worth of stock. Back in the olden days, you could have bought&#xA0;$300 or $500 or $1,000 of stock with your
    $100, borrowing the rest from your broker, but then a Great Depression happened and regulators clamped down on
    margin lending.
</p>
<aside class=\"postr-recirc postr-recirc--opinion\"> <a class=\"postr-recirc__index\"
                                                        href=\"/web/20210408195204/https://www.bloomberg.com/opinion?in_source=postr_index\"> </a> </aside>
{% endraw %}
{% endhighlight %}

Some tags provide useful information. For example the strong and em tags are highlighting important details. We may decide that we want to convert those into the SSML [emphasis tag](https://cloud.google.com/text-to-speech/docs/ssml#emphasis).

Some tags like anchors don't convey information about how the text should be said. They do however have text embedded in them that we need to extract. For these tags we emit plain text.

Lastly we need to handle block tags. Divs can be discarded. P tags can be retained and mapped into SSML [p tags](https://cloud.google.com/text-to-speech/docs/ssml#p,s) as they both represent paragraphs.

Here is some example SSML output for the HTML fragment:
{% highlight xml %}
{% raw %}
<speak>
    <p>
        <emphasis level="moderate">Programming note:</emphasis> <emphasis level="moderate">Money Stuff will be off tomorrow, back on Monday.</emphasis>
    </p>
    <emphasis level="moderate">Archegos fallout</emphasis>
    <p>Generally speaking, in the U.S., if you want to   borrow money from your broker to buy stocks, you are capped at 2-to-1 leverage. If you have $100, you can buy $200 worth of stock. Back in the olden days, you could have bought $300 or $500 or $1,000 of stock with your $100, borrowing the rest from your broker, but then a Great Depression happened and regulators clamped down on margin lending. </p> 
{% endraw %}
{% endhighlight %}

\* Money Stuff is actually the main reason I started this project! I really enjoy Matt Levine's writing. 

## Implementation

At a high level the plan is to walk the DOM of the parsed HTML fragment. As its traversed we need to emit SSML tags and text.

When you parse an HTML fragment with jsoup you can get the parsed HTML as an [Element](https://jsoup.org/apidocs/org/jsoup/nodes/Element.html) object. Element has a traverse() method that lets you walk the tree depth first. The traverse() method takes a [NodeVisitor](https://jsoup.org/apidocs/org/jsoup/select/class-use/NodeVisitor.html) that defines two methods:
 - head​(Node node, int depth): is called when a node is first visited.
 - tail(Node node, int depth): is called when a node is last visited after all its descendants have been visited.

One distinction to make is that the NodeVisitor works on Nodes not Elements. Elements are tags with their attributes and are a type of Nodes. There are other types of nodes like TextNode which represent just text.

Here is some output demonstrating the order the nodes are visited in the HTML fragment from above:

{% raw markdown %}
Starting <body>
	Starting <div>
		Found TextNode:   
		End TextNode
		Starting <p>
			Starting <strong>
				Found TextNode:  Programming note:
				End TextNode
			Ending </strong>
			Found TextNode:   
			End TextNode
			Starting <em>
				Found TextNode:  Money Stuff will be off tomorrow, back on Monday.
				End TextNode
			Ending </em>
		Ending </p>
		Starting <h2>
			Found TextNode:  Archegos fallout
			End TextNode
		Ending </h2>
		Found TextNode:   
		End TextNode
		Starting <p>
			Found TextNode:  Generally speaking, in the U.S., if you want to 
			End TextNode
			Starting <a>
				Found TextNode:   
				End TextNode
				Starting <meta>
				Found Element with unhandled tag: meta
				Ending </meta>
				Found Element with unhandled tag: <meta>
				Found TextNode:   borrow money from your broker to buy stocks
				End TextNode
			Ending </a>
			Found TextNode:  , you are capped at 2-to-1 leverage. If you have $100, you can buy $200 worth of stock. Back in the olden days, you could have bought $300 or $500 or $1,000 of stock with your $100, borrowing the rest from your broker, but then a Great Depression happened and regulators clamped down on margin lending. 
			End TextNode
		Ending </p>
		Found TextNode:   
		End TextNode
		Starting <aside>
		Found Element with unhandled tag: aside
			Found TextNode:   
			End TextNode
			Starting <a>
				Found TextNode:   
				End TextNode
			Ending </a>
			Found TextNode:   
			End TextNode
		Ending </aside>
{% endraw %}

You will notice that TextNodes are visited after we first enter an Element and before we leave the eElement. This means that we can:
 - Emit the start of an XML node when we enter an Element that maps to a SSML tag. For example creating a p tag when we enter a p tag.
 - Write out any text we encounter in a text node.
 - Emit the end of an XML node when we leave an Element that we emitted an XML start for.

There are many ways to work with XML on the JVM. The standard library alone has four different ways. I'll defer to [Baeldung's comparison](https://www.baeldung.com/java-xml-libraries) of the libraries. I went with the streaming approach to XML [StAX](https://docs.oracle.com/javase/tutorial/jaxp/stax/why.html). It lacks some features of the other libraries but those features are mainly on the XML parsing side and I'm only concerned with XML generation.

The following is a simplified implementation of the NodeVisitor. It only handles the tags in the example HTML fragment above. It also creates the log seen previously:
{% highlight kotlin %}
{% raw %}
class Visitor : NodeVisitor {
    private val logger = LoggerFactory.getLogger(javaClass)
    private val xmlStreamWriter: XMLStreamWriter
    private val stringWriter: StringWriter = StringWriter()
    private val noOpTags = setOf("a", "body", "div")
    private val emphasisTags = setOf("em", "h2", "strong")

    init {
        val xMLOutputFactory = XMLOutputFactory.newInstance()
        xmlStreamWriter = xMLOutputFactory.createXMLStreamWriter(stringWriter)
        xmlStreamWriter.writeStartElement("speak")
    }

    override fun head(node: Node?, depth: Int) {
        val indent = "\t".repeat(depth)

        when (node) {
            is Element -> {
                logger.info(indent + "Starting <${node.tagName()}>")
                when (node.tagName()) {
                    in noOpTags -> {
                        // Do Nothing
                    }
                    in emphasisTags -> {
                        xmlStreamWriter.writeStartElement("emphasis")
                        xmlStreamWriter.writeAttribute("level", "moderate")
                    }
                    "p" -> {
                        xmlStreamWriter.writeStartElement("p")
                    }
                    else -> {
                        logger.error(indent + "Found Element with unhandled tag: ${node.tagName()}")
                    }
                }
            }
            is TextNode -> {
                logger.error(indent + "Found TextNode:  ${node.text()}")
                xmlStreamWriter.writeCharacters(node.text())
            }
            else -> {
                logger.error(indent + "Found node with unhandled type: ${node?.javaClass?.simpleName}")
            }
        }
    }

    override fun tail(node: Node?, depth: Int) {
        val indent = "\t".repeat(depth)

        when (node) {
            is Element -> {
                logger.info(indent + "Ending </${node.tagName()}>")
                when (node.tagName()) {
                    in noOpTags -> {
                        // Do Nothing
                    }
                    in emphasisTags -> {
                        xmlStreamWriter.writeEndElement()
                    }
                    "p" -> {
                        xmlStreamWriter.writeEndElement()
                    }
                    else -> {
                        logger.error(indent+ "Found Element with unhandled tag: <${node.tagName()}>")
                    }
                }
            }
            is TextNode -> {
                logger.info(indent + "End TextNode")
            }
            else -> {
                logger.error(indent + "Found node with unhandled type: ${node?.javaClass?.simpleName}")
            }
        }
    }

    fun finalize(): String {
        xmlStreamWriter.writeEndElement()
        xmlStreamWriter.flush()
        xmlStreamWriter.close()
        val xmlString: String = stringWriter.buffer.toString()
        stringWriter.close()
        return xmlString
    }
}
{% endraw %}
{% endhighlight %}

Invoking the NodeVisitor is as simple as:
{% highlight kotlin %}
{% raw %}
fun convertToSsml(rawHtml: String): String {
    val soup = Jsoup.parseBodyFragment(rawHtml)

    val visitor = Visitor()

    soup.body().traverse(visitor)

    return visitor.finalize()
}
{% endraw %}
{% endhighlight %}

## Next steps

Now this only handles step 6 from the outline above where the HTML fragment is converted into an input for GCP TTS. It does not however take into account how to split the text so its less then the 5000 character limit. One approach to this would be to inspect the size of the internal string buffer before adding a node. If its larger then the limit you would write it out and start a new SSML document. The trick parts to this are:
  - You still want to split on sentence boundaries to keep the audio sounding natural. When writing out a TextNode that runs over the limit you can try to split it on sentences.
  - If you run out of space in the middle of a TextNode and that text sits inside something like emphasis you want to make sure the text that goes into the next section is also inside an emphasis.
  - When calculating whether adding the next node would put you over the limit you need to consider the length of the tags for the SSML node.
  - You need to reduce the limit by the end "speak" tag so you have enough space to include it.

Implementing all of this correctly takes a lot of care and will be the next phase of the work before I can deploy this live.

I will likely implement this as a second pass over the generated XML that splits it as needed rather than working with raw strings. Should be fun.
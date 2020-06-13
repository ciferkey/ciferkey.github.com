---
layout: single
classes: wide
title: "Labeling Web Page Category with Mechanical Turk"
description: "Setting up a project on Mechanical Turk to label categories of web pages."
categories:
  - Projects
---


# Why use Mechanical Turk?
I wanted to experiment with classifying whether a web page is an article or not. The categories I want to label are:
 - News Article
 - Blog Post
 - Press Release
 - Other

My goal is too specific to find an existing public dataset I could use. This meant that I would have to construct my own. One option is collecting unlabeled data and hand labeling with one of the [many data labeling tools](https://github.com/heartexlabs/awesome-data-labeling) available. However all of the labels would be categorized based on my own personal perspective. To get around this problem I would need many people to all work together labeling the data. It would be even better if each individual person only labeled a small number of pages. This is where services like [Mechanical Turk](https://www.mturk.com/) are helpful. They allow you to create tasks that a large number of people can be paid to complete.

# Structuring a Mechanical Turk HIT for web page categorization
In order to use MT you need to structure your problem in the context of [MT's concepts](https://docs.aws.amazon.com/AWSMechTurk/latest/RequesterUI/mechanical-turk-concepts.html). At a high level you have:

Human Intelligence Tasks (HITs) - "a single, self-contained task a Requester creates on Mechanical Turk". My HIT is to label a single URL as one of the categories.

Assignment - "You can assign many Workers to work on the same HIT, which is an effective way to achieve consensus on a subject when many workers provide the same answer." While assigning more workers per HIT can improve the quality of your results it also increases your costs. See the cost calculation below for more information. I chose to go with three assignees per HIT. 

Qualification type - "It is important to note that anyone can register to work in the Mechanical Turk Marketplace. To control who can work on your HITs, you can require that Workers have specific qualifications before they can work on your HITs." For this first pass I did not add any qualification requirements. I may add them in the future depending on the quality of the results. For example if I am working with only English web pages I may want to require the workers know English.

# Creating the project in Mechanical Turk
The AWS MechanicalTurk documentation points towards a guide for [categorizing web pages](https://blog.mturk.com/tutorial-categorizing-names-with-the-requester-website-eef620ae8aa5) that I used as a base.

I set the categories that the worker selects by updating the crowd-classifier tag:

{% highlight xml %}
<crowd-classifier 
  categories="['News Article', 'Blog Post', 'PR Piece', 'Other']"
  name="category"
  header="Is this webpage an article?"
>
{% endhighlight %}

And I provided context for the HIT by filling out a short and long description: 

{% highlight html %}
{% raw %}
<short-instructions>
Choose the appropriate category that best suits the page:
<ul>
  <li>A 'New Article' about recent news.</li>
  <li>A 'Blog Post' is a written piece but may not be about recent news.</li>
  <li>A 'PR Piece' is a news piece that is a Press Release written for a company.</li>
  <li>'Other' is anything else. This includes media like pages where the content is primarily video.</li>
</ul>
</short-instructions>

<full-instructions header="Website Classification Instructions">
<p>Read the website carefully. Not all pages on a site with articles are articles.</p>
<p>Choose the appropriate category that best suits the particular page.</p>
<p>Pages where the content is primarily media like a video should be labeled as other.</p>

<p>Examples:</p>
<ul>
  <li>A 'New Article' about recent news. Eg <a href="https://www.npr.org/sections/coronavirus-live-updates/2020/06/13/876544822/beijing-in-wartime-emergency-mode-amid-fresh-cluster-of-coronavirus-cases">Beijing In 'Wartime Emergency Mode' Amid Fresh Cluster Of Coronavirus Cases</a></li>
  <li>A 'Blog Post' is a written piece but may not be about recent news.Eg <a href="https://blog.rust-lang.org/2020/06/04/Rust-1.44.0.html">Announcing Rust 1.44.0</a></li>
  <li>A 'PR Piece' is a news piece that is a Press Release written for a company. Eg <a href="https://www.prnewswire.com/news-releases/taco-bell-debuts-its-longest-shell-ever-with-the-triplelupa-the-ultimate-reinvention-of-the-fan-favorite-chalupa-301020190.html">Taco Bell® Debuts Its Longest Shell Ever With The Triplelupa</a></li>
  <li>'Other' is anything else. Eg <a href="https://www.bbc.com/news/av/world-us-canada-53023704/celebrity-chef-jos-andrs-we-are-a-food-planet" target="_blank">Celebrity chef José Andrés: 'We are a food planet'</a> is on BBC but is a video rather than a News Article.</li>
</ul>
</full-instructions>
{% endraw %}
{% endhighlight %}

Here is what the HIT will look like in Mechanical Turk:

![Preview of Mechanical Turk HIT](/images/mechanical-turk-hit-preview.png)


# Running a batch
Once the project is configured you need to create a batch of data to submit. I pulled URLs from my crawler's database and dumped them into a CSV. The CSV needs to have one column for every variable your HIT uses. My template on MechanicalTurk has only the URL variable so the input CSV only needs a single column:


{% highlight csv %}
url
https://www.theguardian.com/us-news/2020/jun/09/protests-us-america-cars-weapons
https://www.cnn.com/2020/06/09/us/dancing-black-man-police-trnd/index.html
https://www.nbcnews.com/news/us-news/body-cam-footage-shows-officer-punching-alabama-store-owner-who-n1228006
https://abcnews.go.com/US/virginia-indiana-joining-taking-confederate-monuments/story?id=71066712
...
{% endhighlight %}

I started off with a first batch of 500 HITS and submitted it to Mechanical Turk for completion.

# Cost and time

You can use a [Mechanical Turk Cost Calculator](https://morninj.github.io/mechanical-turk-cost-calculator/) before running a batch to figure out how much it will cost. In my case a batch of 500 HITs at 1 cent per HIT and 3 assignees per HIT comes out to $30. Note that half of that cost is fees to Mechanical Turk. The workers only get $15.

The whole process was quite fast. It took about 2 hours and 15 minutes. I could even watch the progress in real time:
![Mechanical Turk Batch Progress](/images/mechanical-turk-batch-progress.png)

There was an interesting long tail where it went quite fast in the beginning and then slowed as the completion percentage went up. I had 90% complete after 30 minutes so the remaining 10% took up the 105 minutes. This makes sense as workers can accept HITs and not complete them. When that happens the HITs get kicked back into the system so other workers can complete them. Mechanical Turk recommends giving workers a generous amount of time to complete HITs. I went with the default max time of one hour. Its also interesting to note that average time per assignment started at 15 seconds and ended up at 1 minutes 36 seconds. 

# Results of batch

The Mechanical Turk documentation has a guide on [getting results](https://blog.mturk.com/tutorial-managing-results-with-the-requester-website-219f6c809e47) from the requester website. In the end you have a CSV file of the results. Each row will represent an _assignment_ so you will have multiple rows per HIT if you have multiple assignments per HIT. Each row will contain information about the worker, the inputs to the HIT and the answers the worker chose. Here are some of the columns:

![Mechanical Turk Results](/images/mechanical-turk-results.png)

In my case I had 505 HITs in the batch so there are 1,515 assignment results.

The results I got were about what I expected. There are plenty of mislabeled pages. This is mostly due to how I structured the problem. I think better clarity with the categories would improve the results. For example workers seemed to use the PR category frequently despite it having a low occurrence.

As a quick experiment I pulled the results into Python. For each URL I found the proportion of labels assigned to that HIT. For example the url "https://www.npr.org/2020/06/12/876351501/zoom-acknowledges-it-suspended-activists-accounts-at-china-s-request" was given labels ['Other', 'News Article', 'News Article'] so the proportions for each label is {'Other': 0.3333333333333333, 'News Article': 0.6666666666666666}.

I then averaged the proportions to get a rough idea for the relative agreement workers had for each category:
PR Piece: 38%
News Article: 64%
Blog Post: 44%
Other: 47%

We can see that PR Pieces had the lowest agreement as expected but the average agreement for other categories are also quite low.

When result quality is low you have a chance to approve/reject the individual results of the batch. If you do not manually approve them they will auto approve after the approval interval you set with creating the project (which defaults to 3 days). owever its important to note tat rejecting results will have an impact on both _your_ Requester score and the worker score.

# Closing thoughts
This really highlights (1) how it important it is to correctly structure your HITs to make them easy and clear to answer and (2) labeling is an inherently noisy task.

I plan to continue with future batches in Mechanical Turk after I clean up my HITs. I may also experiment with other category types. I expect things will go better with binary classification (Article or not) rather than multicategory classification.

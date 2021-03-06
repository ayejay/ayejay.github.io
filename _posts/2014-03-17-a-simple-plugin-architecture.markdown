---
layout: post
title: A simple plugin architecture
subtitle: One rule to live by, KISS
date: 2014-03-17 21:29:57.000000000 +05:30
categories:
- tech
header-img: "img/post-bg-04.jpg"
---
<p>While designing an application, it is often desirable that this application can be easily extensible, even by a third party plugin. Now, you might be  pondering about the benefits of an application using third party plugins. There a more than a few advantages of using such an architecture.<!--more--> A few most important of them are</p>
<ul>
<li>A user can customize the application as he likes if relevant plugins are available (or he can develop one).</li>
<li>A developer does not have to write every functionality imaginable all by himself. He can easily let other people develop features as they want. It is a win-win situation for both of the parties.</li>
<li>An application can be packaged with minimum bloat as the user gets the choice to pick up his own tools.</li>
</ul>
<p>After being convinced about the merits of such an architecture, we should focus on how to do this. There might be several ways of doing that, depending on the design philosophy and other constraints. However, we would have a look at one of the simplest and most common ways of enabling third-party plugins. In our this simple scheme of things, we would have to lay down few rules first. (We would later see why is that necessary)</p>
<ul>
<li>A plugin has to follow the API that developer has exposed to the outer world.</li>
<li>A plugin should have an action or event specified with it (actions or events are specified by the developer like UPDATE_ACCOUNT etc) which would describe the trigger point for it.</li>
</ul>
<p>Now, you must be wondering that this is getting way too abstract and going all over your head. That is why we will try to understand that with an example. </p>
<p>Suppose, you have a framework for developing your application. You have specified that plugin should extend base class <code>BasePlugin</code></p>
<pre lang="php">
abstract class BasePlugin {
      abstract public function execute($info);
}
</pre>
<p>Now, we can define a plugin. Let us call it <code>EscapeOutputPlugin</code> and what it does is escape the output being sent. </p>
<pre lang="php">
class EscapeOutputPlugin extends BasePlugin {
      public function execute($output) {
             return htmlentities($output, ENT_COMPAT, 'UTF-8');
      }
}
</pre>
<p>As the responsible developers in this example, we have also defined an event called <code>PREPARE_OUTPUT</code> which is triggered before the output is being sent. We can then tell our framework to register EscapeOutputPlugin on <code>PREPARE_OUTPUT</code> (via some <code>registerPlugin</code> method or adding it to some config). In our application, when we receive an event we can trigger the plugins attached on it. Thus, when <code>PREPARE_OUTPUT</code> event is triggered our plugin is executed along with others if any registered on the same event.</p>
<p>We also need to look at safety concerns. We should be cautious about not leaking unwanted info to external plugins otherwise consequences can be pretty darn. Like every safe society, our application should also have well defined rules. We do not want to give access to sensitive data or grant for admin action to any third party plugin so we should put checks in place to ensure that. </p>
<p>Now that we have learnt the ancient art of enabling plugins for our applications, let us make world a better place.</p>

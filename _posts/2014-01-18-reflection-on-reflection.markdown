---
layout: post
title: Reflection on Reflection
subtitle: Way too much meta!
date: 2014-01-18 10:58:45.000000000 +05:30
categories:
- tech
header-img: "img/post-bg-04.jpg"
---
<p>Reflection is a very powerful concept in programming domain. It is widely harnessed for various purposes too. You must have noticed quite a few important application of reflection while programming, using frameworks, using IDEs etc. Infact, there are more than few places where we can use reflection for some good.<!--more--> Reflection as per wikipedia, is</p>
<blockquote><p>the ability of a computer program to examine (see type introspection) and modify the structure and behavior (specifically the values, meta-data, properties and functions) of an object at runtime.</p></blockquote>
<p>It is a fairly straightforward definition and easy to understand. Let us have a look at some applications of reflection that could give us some hint about the ubiquity and power of reflection in modern languages.</p>
<p>You must have faced some situation where you have to make some decisions during run time and then take appropriate actions. You might use reflection for achieving the same. For example, we have a class InputProcesser which processes the input for us and has following structure.</p>
<pre lang="php">
class InputProcesser {
    public function processString($input) {
    }

    public function processInteger($input) {
    }
}
</pre>
<p>It has two functions defined, one for string processing and other for integer processing. We can not identify the input datatype until run time. In order to call the apt method at run time, we might use reflection as following.</p>
<pre lang="php">
if (is_int($input)) {
    $methodName = 'processInteger';
} else {
    $methodName = 'processString';
}

$method = new ReflectionMethod('InputProcesser', $methodName);
$method->invoke(new InputProcesser(), $input);
 </pre>
<p>or some people prefer other common way using call_user_func. Either way, they represent the idea of reflection.</p>
<p>If you use some IDE who does pretty awesome stuff based on the annotations or comments, you can again see reflection being used. For example, I use type hinting in PHPStorm to tell the IDE about the type of variable I am using</p>
<pre lang="php">
/** @var $processer InputProcesser */
</pre>
<p>like <code>$processer</code> is an instance of <code>InputProcesser</code> so that IDE can make my life with features like auto complete etc. This is just another example of annotation usage where metadata is read from the file using reflection and in some cases, it is part of program execution. Data provider in PHPUnit is a fine example of it, PHPDoc is another one in the list and you will find many more.</p>
<p>You can in fact operate on your code like a doctor using reflection or in other words you change your program using reflection. For example, you may alter the visibility of class members(for testing a piece of code or applying some hack).</p>
<pre lang="php">
.
.
private function cantCallMe() {
}
.
.
</pre>
<p>You can not call private method from outside class scope but you can change the accessibility of this method like this</p>
<pre lang="php">
$methodName = 'cantCallMe';

$method = new ReflectionMethod('InputProcesser', $methodName);
$method->setAccessible(true);
</pre>
<p>You can also analyze code for various purposes. Many frameworks who expect user defined classes to follow their interfaces might use reflection to verify the desired behaviour. XUnit frameworks analyzes classes for creating their mocks. Static code analyzing tools like pDepend also do the same.</p>
<p>In fact, there are also lots of cool things that you can experiment with reflection.</p>
<p>Although, reflection is a useful concept but you should not mindlessly start changing the program behaviour at run time. Changing program behaviour is a kind of dirty hack, instead of doing that you need to dig deep and find why you need to change it. It might take you to some design flaw at a certain level. Reflection should not be the first choice if you can do without it. Using reflection is slower than normal program execution because of reading and analyzing code. So, use it where it makes sense.</p>

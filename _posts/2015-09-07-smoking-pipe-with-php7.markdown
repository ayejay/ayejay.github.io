---
layout: post
title: Smoking Pipe with PHP7 
subtitle: Y'all got any more of them, syntactic sugar? 
date: 2015-09-07 07:29:57.000000000 +05:30
categories:
- tech
header-img: "img/post-bg-05.jpg"
---
<p>With the release of PHP7 coming near, I tried to check out all the new cool features incorporated in it. With lower memory footprint, speed boosts, improved type-hinting, better error handling, PHP7 has a lot to offer. All the developers using PHP knew these were all long due. I personally liked the <code>SPACESHIP(<=>)</code> operator. So, I decided to experiment with my own operator <code>SMOKEPIPE(~!)</code>. We would use this operator to solve the biggest problem known to the mankind. We would use this operator to find the meaning of life.</p>

<p>So, here the adventure begins.</p>

<p>First, let us check out the latest code from <a href="https://github.com/php/php-src/tree/PHP-7.0.0"  target="_blank">PHP source</a>. Like any other language, PHP also has a parser and scanner. The scanner, as the name suggests scans your code and looks if it satisfies the grammar you have defined for your language, converts the code into list of tokens. The parser, goes through the language tokens (defined in PHP like T_{TOKEN_NAME}) and create <a href="https://en.wikipedia.org/wiki/Abstract_syntax_tree" target="_blank">AST(abstract syntax tree)</a> for further processing. Languages have tokens defined for internal representation of language constructs, like for <code>||</code>, token <code>T_BOOLEAN_OR</code> is defined in PHP. So, we will first start with defining our token <code>T_SMOKEPIPE</code>. </p>

<p>We add following code to <i>language_scanner</i> to convert <code>~!</code> to <code>T_SMOKEPIPE</code> token 
<pre>
&lt;ST_IN_SCRIPTING&gt;"~!" {
	RETURN_TOKEN(T_SMOKEPIPE);
}
</pre>
</p>
<p> and following to <i>language_parser</i> to parse the SMOKEPIPE token into AST
<pre>
%right T_SMOKEPIPE //Right associative operator placed as per the precedence level
.........
%token T_SMOKEPIPE "~! (T_SMOKEPIPE)" 
.........
expr T_SMOKEPIPE expr   { $$ = zend_ast_create_binary_op(ZEND_ADD, $1, $3); } //Create tree node $1<-ADD->$3
</pre>
</p>

<p>If you are still paying attention, you must have noticed that we are using ZEND_ADD(Zend engine's internal representation for Add operator). For now, our SMOKEPIPE operator would act like alias of addition.
</p>
<p>Now, time to compile and run things. After you are done with compilation, run <code>php -r "echo 7 ~! 5;"</code> from command line, and you would get <code>12</code> printed on your screen.
</p>
<br>
<p>Wait, but have we not decided to solve the biggest problem known to the mankind? Yes, we did. Now, let us move to more fun part of this adventure.</p>
<br>
<p>
Let us change the AST creation change, we did in last step and change <code>ZEND_ADD</code> to <code>ZEND_SMOKEPIPE</code>. Zend VM has a <a href="http://php.net/manual/en/internals2.opcodes.php" target="_blank">list of OPCODES</a> and our opcode is not listed there. So, we will have to add our new opcode along with others. We would have to define the VM Handlers for the opcodes. There are different combinations of these handlers for optimization purposes. I would not go into details of that but if you are still curious, <a href="http://jpauli.github.io/2015/02/05/zend-vm-executor.html" target="_blank">here</a> is a very good article about that. 
<br>
There is an easy way to generate all these different handlers. Let us add following code to <i>vm_def</i>.
<pre>
ZEND_VM_HANDLER(173, ZEND_SMOKEPIPE, CONST|TMPVAR|CV, CONST|TMPVAR|CV) //173 is last opcode+1
{
        USE_OPLINE
        zend_free_op free_op1, free_op2;
        zval *op1, *op2;

        SAVE_OPLINE();
        op1 = GET_OP1_ZVAL_PTR(BP_VAR_R);
        op2 = GET_OP2_ZVAL_PTR(BP_VAR_R);
        smokepipe_function(EX_VAR(opline->result.var), op1, op2);
        FREE_OP1();
        FREE_OP2();
        ZEND_VM_NEXT_OPCODE_CHECK_EXCEPTION();
}
</pre>
This works as a template for the generation of specialized handlers and definition of opcodes. When you run <i>vm_gen.php</i>, you would see new handlers and opcode definitions.
</p>

<p>
We have added a <code>smokepipe_function</code> for getting us the results when we use this operator. Let us define it.
<pre>
ZEND_API int ZEND_FASTCALL smokepipe_function(zval *result, zval *op1, zval *op2)
{
        ZVAL_LONG(result, 42); //Store the meaning of life in result
        return SUCCESS;
}
</pre>
</p>

<p>
Now, compile PHP; create a file with following code and run it.
<pre>
?php
$a = 5;
$b = 6;
echo  $a~!$b;
</pre>
This would work but the following would not
<pre>
echo 5 ~! 6;
</pre>
Strange!!
</p>

<p>
If you use second version of code, PHP would try to directly evaluate the expression because we are using constants. So, we would have to add code at one more place, for opcode evaluation.
<pre>
case ZEND_SMOKEPIPE:
	return (binary_op_type) smokepipe_function;
</pre>
</p>

<p>Phew!! After all the hardships and obstacles, we have finally found the tool which would lead us to the true meaning of life.</p>

<p>After fresh compilation, run <code>php -r "echo 7 ~! 5;"</code> from command line and you would see meaning of life, right on your screen.
</p>

<p>While this would not be practically very useful, we can make PHP more fun with more sugar. The way things are moving in PHP world, future looks good.</p>

<p style="font-size:11px;">This was a little experiment meant only for fun, it might not conform to some of the best coding practices. A few of the unimportant details are not mentioned here. If you want to compile and run this, checkout the <a href="https://github.com/ayejay/php7-smokepipe" target="_blank">repo</a></p>

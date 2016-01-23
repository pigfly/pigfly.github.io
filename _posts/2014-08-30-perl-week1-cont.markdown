---
layout: post
title: "Perl Cheatsheet 2"
date: 2014-08-30 02:59:53 +1000
comments: true
header-img: "img/post-bg-05.jpg"
categories:  [perl]
---

## SLP week1: A review

### Design Choice

- Perl, is eager to give you the answer, and try its best to guess an answer for you!
- No reserved word, prefix with var distinguish data from build-ins (so developers can add more build-ins later on without conflicts)
<!--more-->
- in Perl, values are all scalars, we don't have data type explicitly stating numerical value and string, **perl will take longest path to legal values, evaluate  it, then quit**
	- e.g. scalar conversion `(13+4)."love" = "17love"`, `"12god" / 3 = 4`
	- e.g. `"recursion" x 3.3` = `recursionrecursionrecursion`



-----------

- `=~` stands for searching for pattern e.g. `=~ /a*/`
- `use warning;` will warn you dangerous thing, but will still execute all code without stopping

- Syntax
	- every statement ends with a semi-colon
	- make sure block statement wrapped with `{` and `}`
	- case sensitive
	- reference: `$ref = \$whence;` the `\` is to get the address of $var

### Variable

- Variable Define
	- check if defined: ` defined($var)` the var is defined but has empty value is different from var isn't defined  (like C)
	- e.g. `$line = <STDIN>; if (defined($line)) {…}` good practice
	- e.g. `chomp( $line = <STDIN> )` to get rid of new line symbol good practice
	- to release a var, garbage collect a var, use ` undef($var)`

- Scalar var
	- `_` in numerical var is ignored e.g. `$money = 2020_22_11`
	- V string, list ASCII value e.g. `$lang = [v]80.101.22.11` `[v]`:optional

- Scope of var
	- `my`: the inner block, within the `{` and `}`
	- `local`: a copy of var within the `{` and `}`
	- `our`: global
	- once var get out of scope, garbage collected, totally garbage collected var evaluated to 0 !

### Condition & Loop

- True and False
	- False: `"0"`, empty string, undefined var, and 0

- Operators in Conditions
	- operator set the context, again, guess an answer for you
	- `<=>` and `cpm` are comparison, with signed result

- Condition
	- `if/unless (x > 50) { print; }`
		- even only one line statement needs braces, to avoid *C programming dangle else trap*
	- `print if/unless (x > 50);`
		- e.g. for printing `print $a, " is $b", $c;`

- Loop
	- `for (my $x = 0; $x <= 0; $sp++) {`
	- `for/foreach $x (@array) {`
	- switch: `SWITCH: { if (/perl/) { $perl = 1; last SWITCH;`


### TUT

- last index of list: `$# array`, #elements ` scalar(@array) `
- make sure var stated within scope e.g. `my $var`
- `my $min = $array[0]` and `for my $var (@array)`

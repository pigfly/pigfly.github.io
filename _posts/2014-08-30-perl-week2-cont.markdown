---
layout: post
title: "Perl Cheatsheet 3"
date: 2014-08-30 17:28:42 +1000
comments: true
header-img: "img/post-bg-03.jpg"
categories: [perl]
---

## RegEx cont.

----------

### Variables

- Match vars.
	- if regex have parens i.e. groups
	- if group got successful match
		- then the content of matched pattern stored in `$1`, meaning first match against first group
		- e.g. `/(\S+)(\S+),(\S+)/` corresponding matched content is `print $1 $2 $3`
	- **classical match idiom**: `if(xx =~ xx) do sth else die`
	- three special vars.
		- a regex matching a string divides the string into 3 pars
		- `$&`: the part actually matched
		- `$\backslash`: the part before matched
		- `$'`: the part after matched

<!--more-->

- Noncapturing parens
	- sometimes, we use parens only for precedence, however, it still trigger matching variable.
	- how can we escape from that, to tell perl not to trigger matching var in parens ?
	- to avoid triggering memory, change `(…)` to `(?:…)`
	- e.g. `/(\S+)(?:and|or)(\S+)/` the middle parens won't be counted as $2, instead, the last parens is $2.

- Named captures
	- even better way, to tell which parens I want to capture, and give name to that matching var.
	- e.g. `/(?<name1>\S+)(lorem)/` now `print "$+{name1}"` will print the content of matching \S+ in the named capture you create.

----------

### Substitution

- `s/old/new/`
	- old is **RegEx**
	- **new is treated as double quoted string**, so if there is `$var` inside, will be replaced as its value. **never think of putting RegEx in new!**
	- RegEx triggers back reference var: `s/with(\w+)/without$1/` like said before, $1 is special var in double quoted string context, will be replaced by its value
	- idiom: `if (s/old/new/) {…}`
- Alternate delimiter
	- e.g. `#^https://#http://#` again, don't put `^`in http, it's merely treated as "string"!
- More Modes
	- **Global** matching, `s/\s+/ /g` transform to canonical whitespace
	- case-insensitive, `s/old/new/i`
	- `.` including new lines, `s/old/new/s`

-------------

- **List-context match**
	- you can store **parens** matched result in a list
	- e.g. `my($first, $second, $third) = /(\S)(\S),(\S)/;`
	- e.g. `my @words = /(\w+)/g`
- **Non-greedy quantifier
	- quantifier normally go long, then back off
	- we can tell perl go as short as you can !
	- **Append `?` to *quantifier* to make it lazy**
- Split function
	- `my @fields = split /separator/, $string`
	- separator here is RegEx
	- `my @ip = split /\./, "192.168.1.1";`   
- Case shifting
	- `\U`: take every var following `\U`, make them uppercase
	- `\L`: take every var following `\L`, make them lowercase
	- `\u`: take first char of followed, make it uppercase
	- `\l`: take first char of followed, make it lowercase
	- `\E`: where case shifting stops
	- N.B. all these happen in "string"!
	- e.g. `s/(\w+)/\U$1/`
	- e.g. `s/(\w{2})/\u\L$1/ig` **initial cap**

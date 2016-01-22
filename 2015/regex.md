Title: Tackling Regex
Category: Tutorials
Date: 2015
Keywords:
Description:
Template: YourMom




# Tackling Regex

I really enjoy creating regular expressions. They allow you to build a complex parser which does exactly what you want in minutes. Despite this, others often find building regular expressions to be difficult even when they're familiar with the syntax and rules. I recently designed a regular expression to match IRC messages as defined in section 2.3.1 of [RFC 2812 - IRC: Client Protocol](http://tools.ietf.org/html/rfc2812). The specification for the message format is provided in an Augmented BNF format (Don't worry, you don't need to know BNF). I'd like to explain the process of converting the spec into a working regular expression and putting it into use.

Before we begin, I'll mention that I'm only expecting a very basic knowledge of regular expressions, and I will explain each component of the regular expression exactly once. These explanations will not be repeated, however, in an effort to keep the interest of a more advanced reader. I also recommend following along in your favorite regex helper, I personally use [Regex101](https://regex101.com/).

The message format in BNF, for reference:

````
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Message given as: <prefix> <command> <params>
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
message    =  [ ":" prefix SPACE ] command [ params ] crlf
prefix     =  servername / ( nickname [ [ "!" user ] "@" host ] )
command    =  1*letter / 3digit
params     =  *14( SPACE middle ) [ SPACE ":" trailing ]
           =/ 14( SPACE middle ) [ SPACE [ ":" ] trailing ]

nospcrlfcl =  %x01-09 / %x0B-0C / %x0E-1F / %x21-39 / %x3B-FF
                ; any octet except NUL, CR, LF, " " and ":"
middle     =  nospcrlfcl *( ":" / nospcrlfcl )
trailing   =  *( ":" / " " / nospcrlfcl )

SPACE      =  %x20        ; space character
crlf       =  %x0D %x0A   ; "carriage return" "linefeed"

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Target Definitions
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
target     =  nickname / servername
msgtarget  =  msgto *( "," msgto )
msgto      =  channel / ( user [ "%" host ] "@" servername )
msgto      =/ ( user "%" host ) / targetmask
msgto      =/ nickname / ( nickname "!" user "@" host )
channel    =  ( "#" / "+" / ( "!" channelid ) / "&" ) chanstring
              [ ":" chanstring ]
servername =  hostname
host       =  hostname / hostaddr
hostname   =  shortname *( "." shortname )
shortname  =  ( letter / digit ) [ *( letter / digit / "-" ) ( letter / digit )]
hostaddr   =  ip4addr / ip6addr
ip4addr    =  1*3digit "." 1*3digit "." 1*3digit "." 1*3digit
ip6addr    =  1*hexdigit 7( ":" 1*hexdigit )
ip6addr    =/ "0:0:0:0:0:" ( "0" / "FFFF" ) ":" ip4addr
nickname   =  ( letter / special ) *8( letter / digit / special / "-" )
targetmask =  ( "$" / "#" ) mask
                ; see details on allowed masks in section 3.3.1
chanstring =  *49(%x01-06 / %x08-09 / %x0B-0C / %x0E-1F / %x21-2B /
             %x2D-39 / %x3B-FF)
channelid  = 5( %x41-5A / digit )   ; 5( A-Z / 0-9 )

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Additional parameter definitions
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
user       =  1*( %x01-09 / %x0B-0C / %x0E-1F / %x21-3F / %x41-FF )
              ; any octet except NUL, CR, LF, " " and "@"
key        =  1*23( %x01-05 / %x07-08 / %x0C / %x0E-1F / %x21-7F )
              ; any 7-bit US_ASCII character,
              ; except NUL, CR, LF, FF, h/v TABs, and " "
letter     =  %x41-5A / %x61-7A       ; A-Z / a-z
digit      =  %x30-39                 ; 0-9
hexdigit   =  digit / "A" / "B" / "C" / "D" / "E" / "F"
special    =  %x5B-60 / %x7B-7D
               ; "[", "]", "\", "`", "_", "^", "{", "|", "}"
````

Note that I have applied the errata for the rfc as listed in [RFC Errata - 2812](https://www.rfc-editor.org/errata_search.php?rfc=2812). As mentioned, do not panic! I will walk through and simplify the above.

Let's take look at the final resulting [Perl Compatible Regular Expression (PCRE)](http://www.pcre.org/) so as to squash any surprises:

````
^(?::(?P<prefix>(?P<servername>[\w](?:[\w-]*[\w])*(?:\.[\w][\w-]*[\w]*)*)|(?:(?P<nickname>[a-z\[\]\\`_\^\{\|\}][\w\[\]\\`_\^\{\|\}-]*)(?:(?:!(?P<user>[^\s@]*)){0,1}@(?P<host>(?P<hostaddr>(?P<ip4addr>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})|(?P<ip6addr>(?:[a-f\d]{1,4}(?::[A-Fa-f\d]{1,4}){7})|(?:0:0:0:0:0:(?:0|FFFF)):(?:\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})))|(?P<hostname>[\w](?:[\w-]*[\w])*(?:\.[\w][\w-]*[\w]*)*)|(?P<hostcloak>[\S]*))){0,1}))\s){0,1}(?P<command>(?:\d{1,3})|(?:[a-z]*))\s?(?P<params>(?P<middle>(?:\s?(?:[^\s:](?::|[^\s:])*))*)(?:\s?:(?P<trailing>(?::|\s|[^\s:])*)){0,1})$
````

Ugly right? Hopefully as we dive deep into this madness it will begin to appear more structured, perhaps even reasonable.


## Break It into Parts
The first step to realizing a regular expression is splitting the pattern you'd like to match into obvious components. Thankfully, the specification largely does that for us. Our messages are defined as having a prefix, a command, and a list of parameters. So those will be our parts. How do they chain together?

````
message    =  [ ":" prefix SPACE ] command [ params ] crlf
````
This means that a message *may* start with prefix, *must* then be followed by a command, *may* have parameters, and then *must* have a line ending. The brackets `[ ]` in BNF specify that something is optional.

Converting that much into a regex pattern is easy:

````
^
(:(?P<prefixTODO>)\ )?
(?P<commandTODO>)
(?P<paramsTODO>)?
$
````
There's definitely a few things to talk about here. First of all, you may notice that I've chosen to split my regex into multiple lines. I can do this if I specify the `x` parameter in my regular expression, which says that newline characters will be ignored. I will also be using the `i` parameter, which makes the regular expression case insensitive. We begin the regex with `^` and end with `$`, and make the assumption that we will only be parsing a *single* line at a time. This means that the input stream will need to be split by new line characters before being fed into the regex engine.

I've placed the colon outside of the prefix grouping. This is to not accidentally mistake the colon as being a part of the prefix.

If you aren't familiar with PCRE, you may be thrown off by the `(?P<name>)` syntax. This is a named capture, and in most engines it will allow me to reference the captured content via a dictionary such as `captures["prefix"]` after the regex has been run against my message. You'll notice that I've suffixed each of my named captures with "TODO". This is a reminder to myself that I have not completed this component of the regex and need to return to it later.

Finally, for prefix and params we end with a `?` to designate that there can be either zero, or one those groups. The literal space character is only included after prefix and not after the other components simply because that's what the specification calls for.

Peeking at the BNF, the command component is by far the most trivial, and so it will be the first that we attempt to implement.


### The Command
Let's see the command component of the BNF:

````
command    =  1*letter / 3digit
````

Well that's simple. `*` means variable repetition of the form m*nRule, where the m is the minimum number of elements, n is the maximum, and Rule is the definition of the elements. `/` means "or". This means a command is either a string of letters, or precisely 3 digits. Let's go to the regex:

````
(?P<command>
  (\d{3})
  |
  ([a-z]+)
)
````
Here we use the `{ }` quantifier to specify that we'd like a exactly 3 digits. Then an OR `|` is used to supply another pattern in the event that the first is not matched. Our other pattern is one or more elements of the set of all English letters. Remember, we're case insensitive.

And that's it. Let's add that back to our original expression of a message:

````
^
(:(?P<prefixTODO>)\ )?
(?P<command>
  (\d{3})
  |
  ([a-z]+)
)
(?P<paramsTODO>)?
$
````
Note the careful white space applied in attempt to make the regex more readable. I have also removed the TODO reminder from command because it is now finished and no longer needs consideration.

I won't be putting each of the components together again until the end, but I want to give you an idea of where we're going with this.


### The Prefix
We started off easy with command, but it's time to really dive in. Prefix is significantly more complicated, but we can do it! Try not to get discouraged when working with these patterns. Approach it one step at a time.



### The Parameters

## Finalizing Our Expression




````
^(?::
			  (?P<prefix>
				(?P<servername>[\w](?:[\w-]*[\w])*(?:\.[\w][\w-]*[\w]*)*)
				|
				(?:
				  (?P<nickname>[a-z\[\]\\`_\^\{\|\}][\w\[\]\\`_\^\{\|\}-]*)
				  (?:
					(?:!(?P<user>[^\s@]*)){0,1}@
					(?P<host>
					  (?P<hostaddr>
						(?P<ip4addr>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})
						|
						(?P<ip6addr>
						  (?:[a-f\d]{1,4}(?::[A-Fa-f\d]{1,4}){7})
						  |
						  (?:0:0:0:0:0:(?:0|FFFF)):
						  (?:\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})
						)
					  )
					  |
					  (?P<hostname>[\w](?:[\w-]*[\w])*(?:\.[\w][\w-]*[\w]*)*)
					  |
					  (?P<hostcloak>[\S]*)
					)
				  ){0,1}
				)
			  )
			  \s
			){0,1}
			(?P<command>
			  (?:\d{1,3})
			  |
			  (?:[a-z]*)
			)
			\s?
			(?P<params>
			  (?P<middle>
				(?:
				  \s?
				  (?:[^\s:](?::|[^\s:])*)
				)*
			  )
			  (?:
				\s?:
				(?P<trailing>
				  (?::|\s|[^\s:])*
				)
			  ){0,1}
			)$
````

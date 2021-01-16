# bash_option_parser

<h2>Capabilities</h2>

<ul>
  <li>
    Supports sub option parsing
  </li>
  <li>
    Supports alias names for options
  </li>
  <li>
    Supports optional arguments
  </li>
	<li>
    Supports variable arguments
  </li>
	<li>
		Prints usage and error messages.
  </li>
  <li>
    Easy to use
  </li>
</ul>

<h2>Requirements</h2>
<ul>
	<li>
		Needs at least bash version 4.
	</li>
</ul>

<h2>Index</h2>
<ul>
	<li>
		<a href="#example">Example</a>
	</li>
	<li>
		<a href="#schema">Schema</a>
	</li>
	<li>
		<a href="#parse_options">parse_options()</a>
	</li>
	<li>
		<a href="#option_parser_error_msg">option_parser_error_msg()</a>
	</li>
	<li>
		<a href="#print_usage">print_usage()</a>
	</li>
	<li>
		<a href="#parse_options_detailed">parse_options_detailed()</a>
	</li>
	<li>
		<a href="#option_parser_error_msg_detailed">option_parser_error_msg_detailed()</a>
	</li>
	<li>
		<a href="#print_usage_detailed">print_usage_detailed()</a>
	</li>
</ul>


<h2>Example</h2>

This is a simple example to show basic functionailty. Checkout <a href="example">example</a> for a better understanding.

Consider a sample command named `sample`.
It accepts options as per the following usage format:

```
sample <name1> <name2> [<name3>]
    -s|--search|--find arg1 arg2 [arg3]
    -m|--make|--create
```

To tell the parser that `sample` needs 2 arguments and 1 optional argument i.e. `<name1>`, `<name2>` and `[<name3>]`,
we pass a string like `'1 1 -1'`.
	
In `'1 1 -1'`, `1` denotes a mandatory arg and `-1` denotes an optional arg. This is called schema in this parser.
Similarly, `...`, 'S' and `0` are for variable args, suboption and no args respectively. See complete [explaination](#schema) for
more details.

We can parse options as per the above criteria as follows:

```bash
#! /usr/bin/env bash

source ./option_parser

parse_options \
	'sample'                               '1 1 -1'     \
	'-s'        , '--search' , '--find'    '1 1 -1'     \
	'-m'        , '--make'   , '--create'  '0'          \
	';' \
	"$@"

retval=$?

if [ $retval -ne 0 ]; then
	# will print a meaningful message on error
	option_parser_error_msg "$retval"
	# will print usage of command
	print_usage 'sample'
	exit 1
fi

#
# Now if some options are passed,
# they are stored in OPTIONS[] as the key.
# So let's check if '-s' was passed
#

if [ -n "${OPTIONS[-s]}" ]; then

	echo "-s option was passed"

	# cnt_args_passed contains the number of args the option received
	cnt_args_passed=${ARG_CNT[-s]:-0}

	# print the first arg:
	echo "${ARGS[-s,0]}"

	# print the second arg:
	echo "${ARGS[-s,1]}"

	#
	# As 3rd arg is optional, we can check if
	# cnt_args_passed is equal to 3
	#

	if [ $cnt_args_passed -eq 3 ]; then
		# print the third arg:
		echo "${ARGS[-s,2]}"
	fi

fi
```

<h4>Schema</h4>

Corresponding to every option, a schema is passed which determines
the patten in which the option receives arguments.
There are 4 characters available to define the schema:
<br>
&nbsp;&nbsp;&nbsp;&nbsp;<b> 1 </b>: Means argument is necessary                                          <br>
&nbsp;&nbsp;&nbsp;&nbsp;<b>-1 </b>: Means argument is optional (this can only be last element in schema) <br>
&nbsp;&nbsp;&nbsp;&nbsp;<b>...</b>: Means variable length args (this can only be last element in schema) <br>
&nbsp;&nbsp;&nbsp;&nbsp;<b> 0 </b>: Means doesn't receive any argument                                   <br>
&nbsp;&nbsp;&nbsp;&nbsp;<b> S </b>: Is a sub-command                                                     <br>
<br>
So, schema `'1 1 1 -1'` would mean the option needs 3 args and 4th is optional.
<br>
So let's say we have 3 options, `-a`, `-b` and `-c`.
<br>
`-a` needs at least one arg and can receive a second arg optionally.   <br>
`-b` doesn't need any args                                             <br>
`-c` is a sub-command                                                  <br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;Schema for `-a` = `'1 -1'` <br>
&nbsp;&nbsp;&nbsp;&nbsp;Schema for `-b` = `'0'`    <br>
&nbsp;&nbsp;&nbsp;&nbsp;Schema for `-c` = `'S'`    <br>

<h2>parse_options()</h2>

After this functions is called, 3 associative arrays are set to hold information about args. These are
`OPTIONS`, `ARG_CNT` and `ARGS`. These names can be changed by using [parse_options_detailed()](#parse_options_detailed) instead.

We will use the [above example](#example) to explain their usage:

```bash
parse_options \
	'sample'                               '1 1 -1'     \
	'-s'        , '--search' , '--find'    '1 1 -1'     \
	'-m'        , '--make'   , '--create'  '0'          \
	';' \
	"$@"
```

Here the format goes as follows:

```text
parse_options \
	     '-s'        ,        '--search'          ,        '--find'         '1 1 -1'
	<option-name> <comma> <alternative-name-1> <comma> <alternative-name-2> <schema>
	<semicolon>
	<args passed by user i.e. $@>
```

In the above setup, we pass options in order like:

```
parse_options <option-1> <schema> <option-2> <schema> ; $@
```

First option, i.e. `<option-1>` __must__ always be the key name that you want to use to denote the program itself.
Like in the above example it is `sample`.

<b>Comma(,)</b> is used to separate alternative names. <b>Semicolon(;)</b> is used to mark the end of options. After
semicolon, `$@` is passed.

After the call, the following associative arrays are set:

<h4>OPTIONS</h4>
If an option is passed, key corresponding to it is set in `OPTIONS[]` and it stores the shift count needed to
reach that arg.

For example, if we called the `sample` command as:

```
sample 1 2 -m -s 1 2 3
```

It will set `OPTIONS` array as:

```
OPTIONS[sample] = 0
OPTIONS[-m] = 2
OPTIONS[-s] = 3
```

They are generally used just to check if the particular option was passed or not. Like to check if `-s` is passed, we do
`[ -n "${OPTIONS[-s]}" ]`. Their value which contains the number of `shift` needed to reach them only comes handy in case of
suboptions. To see how this can be used in suboptions, check the code in <a href="example">this example</a>.

One special key, `error_opt` is used to store any errors that occured while parsing user args. This is used by 
[option_parser_error_msg](#option_parser_error_msg) to print a relevant error message.

<h4>ARG_CNT</h4>
If the passed option received arguments, this array stores the number of args received corresponding to that options.

For example, if we called the `sample` command as:

```
sample 1 2 -m -s 1 2 3
```

It will set `ARG_CNT` array as:

```
ARG_CNT[sample] = 2
ARG_CNT[-m] = # NOT SET BECAUSE IT DIDN'T RECEIVE ANY ARGUMENTS
ARG_CNT[-s] = 3
```

<h4>ARGS</h4>
This array is set in a 2D-array-like format to store the args received.

For example, if we called the `sample` command as:

```
sample 11 22 -m -s 1 2 3
```

It will set `ARGS` array as:

```
ARGS[sample,0] = 11
ARGS[sample,1] = 22

ARGS[-s,0] = 1
ARGS[-s,1] = 2
ARGS[-s,2] = 3
```

`parse_options()` is defined as follows:

```bash
parse_options() {
	parse_options_detailed ',' 'OPTIONS' 'ARG_CNT' 'ARGS' '0' ';' '--' 'error_opt' \
	    'OPTION_DATA' 'ALIAS_DATA' "$@"
}
```

<h2>option_parser_error_msg()</h2>

This can be used for printing error messages in case the args passed by the user are in wrong format. This function
takes the exit status of [parse_options](#parse_options) as argument and prints the corresponding error message.
This uses `OPTIONS[error_opt]` to check if any error occured.

`option_parser_error_msg()` is defined as follows:

```bash
option_parser_error_msg() {
	option_parser_error_msg_detailed "$1" 'OPTIONS' 'ARG_CNT' 'ARGS' 'error_opt'
}
```

<h2>print_usage()</h2>

This prints the usage of the command. It takes one argument which is the name of the program. This should
be same as what you passed as first option to `parse_options()`.
Like in the [above example](#example), if command is used incorrectly, following code will be called:

```bash
print_usage 'sample'
```

This will print:

```text
sample <arg1> <arg2> [<arg3>] 
    -m|--make|--create 
    -s|--search|--find <arg1> <arg2> [<arg3>] 
```

`print_usage()` is defined as follows:

```bash
print_usage() {                                                
	print_usage_detailed "$1" 'OPTION_DATA' 'ALIAS_DATA'
}
```

<hr>

Arguments accepted by `parse_options_detailed()`, `option_parser_error_msg_detailed()` and `print_usage_detailed()`
are explained below. These can be used if you want to change the separator, associative array name, etc.

<h2>parse_options_detailed()</h2>

<h4>param1: Alias Separator</h4>

As you can see in the above example, alias names of args are separated by a ','.
param1 tells what separator you want to use to separate alias names.

<h4>param2: Associative array name for storing passed options</h4>

<ol>
	<li>
		This is the name of the array that would be declared global and associative by the function.
		All the passed options are stored as keys in it and can be checked by <code>-n</code> or <code>-z</code> in if condition.
	</li>
	<li>
		If a suboption is encountered, it stores the number of shifts needed to reach the args of suboption
		as the value corresponding to name of the option as the key.
	</li>
</ol>

<h4>param3: Associative array name for storing argument count of passed options</h4>

<ol>
	<li>
		This is the name of the array that would be declared global and associative by the function.
		All options that received args have the the arg count stored as value in this array
		corresponding to name of the option as the key.
	</li>
	<li>
		This can be used with options that receive optional args and the arg count to be received is not certain.
	</li>
</ol>

<h4>param4: Associative array name for storing arguments of passed options</h4>

This is the name of the array that would be declared global and associative by the function.
All options that received arguments store them in it in a 2D-array-like format.
e.g., if some option -s received 2 args, then the args will be stored as the following keys:<br>
<pre>
[-s,0]=arg1
[-s,1]=arg2
</pre>
	
<h4>param5: shift count</h4>

<ol>
	<li>
   	This is the number of shifts made before parsing args.
   	e.g., If cmdline args were like:	
<pre>
command -v --new "file" create hello world lalala
</pre>
	and you want to parse args after 'create', so nshift should be 4.
   	If would shift initial 4 args and start parsing after create.
	</li>
	<li>
   	This is useful for parsing subcommands. We get the number of shifts
   	required from value of OPTIONS[] as discussed in param2.
	</li>
</ol>

<h4>param6: terminator</h4>

This is used to mark the end of the options. Like in the above <a href="#example">example</a>, ';' is used as the terminator.

<h4>param7: No option indicator</h4>

This tells to treat following arg as a normal arg even if it's
an option.<br>
e.g., Let's say it is '--'. <br>
If there are 2 options -n and -m where -n takes an arg, <br>
it can be that arg received by -n has the name "-m", <br>
then on cmdline, it can be done like: <br>
<pre>
command -n -- -m
</pre>
"-m" will be treated as arg to -n <br>
If the arg itself was '--', then it could have been <br>
achieved like: <br>
<pre>
command -n -- --
</pre>

<h4>param8: error opt</h4>

This is the key that would be used in param2 array
to store the name of the option which caused the error.

<h4>param9: option_data_array_name</h4>

This is the "name" of the associative array that would
store all the valid options along with their schema.
This is meant to be passed to parse_options_detailed()
so that it can use option data to print the usage.
e.g.,
	 
<pre>
parse_options \
  'self'                         '1 1 -1' \
  'c'    , 'create'              'S'      \
  '-e'   , '--eat'               '1 1 -1' \
  '-n'   , '--new'               '1'      \
  '-v'   , '--version' , '--ver' '0'      \
  ';'                                     \
  "$@"
</pre>
As parse_options() uses the name OPTION_DATA by default, after calling
this OPTION_DATA would look like:
<pre>
OPTION_DATA[self] = '1 1 -1'
OPTION_DATA[c] = 'S'
OPTION_DATA[-e] = '1 1 -1'
OPTION_DATA[-n] = '1'
OPTION_DATA[-v] = '0'
</pre>	 

<h4>param10: alias_data_array_name</h4>

This is the "name" of the associative array that would
store all the alias names corresponding to the option in a 2D array like format.
This is meant to be passed to parse_options_detailed()
so that it can use alias data to print the usage.
e.g.,
<pre>
parse_options \
  'self'                         '1 1 -1' \
  'c'    , 'create'              'S'      \
  '-e'   , '--eat'               '1 1 -1' \
  '-n'   , '--new'               '1'      \
  '-v'   , '--version' , '--ver' '0'      \
  ';'                                     \
  "$@"
</pre>
As parse_options() uses the name ALIAS_DATA by default, after calling
this ALIAS_DATA would look like:
<pre>
ALIAS_DATA[self,-1] = 0

ALIAS_DATA[c,-1] = 1 # -1 index stores alias count
ALIAS_DATA[c,0] = create

ALIAS_DATA[-e,-1] = 1
ALIAS_DATA[-e,0] = --eat

ALIAS_DATA[-n,-1] = 1
ALIAS_DATA[-n,0] = --new

ALIAS_DATA[-v,-1] = 2
ALIAS_DATA[-v,0] = --version
ALIAS_DATA[-v,1] = --ver
</pre>

<h4>self key name</h4>

<ol>
	<li>
		The first key that is passed to parse_options is for main command itself.
	</li>
	<li>
		If the options passed don't belong to a particular option and instead are arg to main command itself,
		details corresponding to these are stored in param2, param3 and param4.
	</li>
	<li>
		For example, if this arg was 'self', then in arrays param2, param3 and param4, 'self' would be used
		as the key to store args to the main arg that don't belong to any particular option.
	</li>
</ol>

<h4>retval</h4>

<ol>
	<li>
		SUCCESS: Returns 0
	</li>
	<li>
		FAILURE: It sets the key param8 in param1 array to the option name for which the error occured
		and returns:
		<ul>
			<li>
				1 : insufficient args are supplied for some particular option
			</li>
			<li>
				2 : invalid schema passed to function
			</li>
			<li>
				3 : extra arg supplied
			</li>
		</ul>
	</li>
</ol>

<h2>option_parser_error_msg_detailed()</h2>

<h4>param1: error_code</h4>

Error code returned by `parse_options_detailed()`

<h4>param2: options_array_name</h4>

This is the same options_array_name that was passed to `parse_options_detailed()`

<h4>param3: args_cnt_array_name</h4>

This is the same args_cnt_array_name that was passed to `parse_options_detailed()`

<h4>param4 : args_array_name</h4>

This is the same args_array_name that was passed to `parse_options_detailed()`

<h2>print_usage_detailed()</h2>

Prints usage as per option data and alias data.

<h4>param1: self_key_name</h4>

It is the key used to address the main command.
This is the first option passed to parse_options_detailed()

<h4>param2: option_data_array_name</h4>

This is the same name that is passed to parse_options_detailed().
Array with this name is populated by parse_options_detailed() and
then can be passed to this function

<h4>param3: alias_data_array_name</h4>

This is the same name that is passed to parse_options_detailed().
Array with this name is populated by parse_options_detailed() and
then can be passed to this function

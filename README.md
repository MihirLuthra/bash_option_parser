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
    Easy to use
  </li>
</ul>


<h2>Example</h2>

Consider a sample command named `sample`.
It accepts options as per the following usage format:

```
sample 
    -s|--search|--find arg1 arg2 [arg3]
    -m|--make|--create
```

We can parse options as per the above criteria as follows:

```bash
source ./option_parser

parse_options \
		',' 'OPTIONS' 'ARG_CNT' 'ARGS' '0' ';' '--'         \
		'-s'        , '--search' , '--find'    '1 1 -1'     \
		'-m'        , '--make'   , '--create'  '0'          \
		';' \
		"$@"
    
#
# Now if some options are passed,
# they are stored in OPTIONS[] as the key.
# So let's check if '-s' was passed
#

if [ -n "${OPTIONS[-s]}" ]; then
  
  echo "-s option was passed"

  # cnt_args_passed contains the number of args the option received
  cnt_args_passed=${ARG_CNT[-s]}

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

<h2>Params Explaination</h2>

Before looking into params, let's understand what is schema for options.

<h4>Schema</h4>

Corresponding to every option, a schema is passed which determines
the patten in which the option receives arguments.
There are 4 characters available to define the schema:
<br>
&nbsp;&nbsp;&nbsp;&nbsp;<b> 1</b> : Means argument is necessary                                          <br>
&nbsp;&nbsp;&nbsp;&nbsp;<b>-1</b> : Means argument is optional (this can only be last element in schema) <br>
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

<ol>
	<li>
		This is the name of the array that would be declared global and associative by the function.
		All options that received arguments store them in it in a 2D-array-like format.
		e.g., if some option -s received 2 args, then the args will be stored as the following keys:<br>
<pre>
[-s,0]=arg1
[-s,1]=arg2
</pre>
	</li>
</ol>
		
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

<h4>retval</h4>

<ol>
	<li>
		SUCCESS: Returns 0
	</li>
	<li>
		FAILURE: It sets the key "error_opt" in param1 array to the option name for which the error occured
		and returns:
		<ul>
			<li>
				-1 : Invalid schema passed to function
			</li>
			<li>
				1 : insufficient args are supplied for some particular option
			</li>
			<li>
				2 : extra agrs supplied
			</li>
		</ul>
	</li>
</ol>

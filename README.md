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



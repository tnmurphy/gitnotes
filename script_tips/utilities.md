# some scripting functions and utilities.

```bash
string_join() {
    local join=$1; shift
    local result=$1; shift
    for p in "$@"; do
        result="${result}${join}${p}"
    done
    echo -n "$result"
    set +x
}
```

# Add and remove paths from the path and dynamic library path

```bash
ld_add() {
    if [[ -v LD_LIBRARY_PATH ]]; then
        LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$(string_join ':' "$@")
        export LD_LIBRARY_PATH
    else
        LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$(string_join ':' "$@")
        export LD_LIBRARY_PATH
    fi
}

ld_prepend() {
    if [[ -v LD_LIBRARY_PATH ]]; then
        LD_LIBRARY_PATH=$(string_join ':' "$@"):$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
    else
        LD_LIBRARY_PATH=$(string_join ':' "$@"):$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
    fi
}

path_add() {
    path_stack_push
    export PATH=$PATH:$(string_join ':' $@)
}

path_prepend() {
    path_stack_push
    PATH=$(string_join ':' "$@"):$PATH
    export PATH
}
```


# Remove elements from the path. Takes a grep regexp as input
```bash
path_remove() {
    local item=$1; shift
    local new_path="$(echo -e "${PATH//:/\n}" |  /usr/bin/grep -v \"$item\")"
    path_stack_push
    export PATH="${new_path// /;}" 
}
```

# ripgrep with colour piped to less
```bash
vgrep() {
    rg --color=always "$@" | less -R
}
```


# Git diff to less with colour
```bash
vdiff() {
local objects=$1; shift
git -c color.ui=always diff "${objects}" | less -REX
}
```

# Run a command with a message about whether it returned failure status code
```bash
do_with_log() {
  # arguments are $1 $2 $3 $4 $5 . . . . 
  local whattodo=$1; shift
  # "$@" is all the (unshifted) arguments so you can pass them onwards:
  ${whattodo} "$@" 
   
  if [ "$?" -gt 0 ]; then
      echo "ERROR: $whattodo failed"
  else
      echo "OK: $whattodo"
  fi
  return 1 
}
```

# Searching 
```bash
vrg() {
rg --color always "$@"
}
```

# encryption
```bash
# cat an encrypted file - convenience method
decf() {
	local file="$1"; shift
	openssl enc -d  -aes-256-cbc -pbkdf2  -in "$file"
}

# encrypted a file - convenience method
encf() {
	local file="$1"; shift
	openssl enc  -aes-256-cbc -pbkdf2  -in "$file"
}

pkcs12topem() {
    pkcs12_file="$1"; shift
    newfile="$1"; shift
    
    openssl pkcs12 -in "$pkcs12_file" -out "${newfile}.crt.pem" -clcerts -nokeys &&
    openssl pkcs12 -in "$pkcs12_file" -out "${newfile}.key.pem" -nocerts -nodes
}
```


# view a json file with formatting in less
```bash
jv() { jq . "$1" -C | less -R; }
```

# Prompts and window titles
```bash
# get the name of the directory containing the current git repository
# There are other ways to do this too by running git itself of course.
get_project_name() {
    local dir="$PWD"
    while [[ ! -d $dir/.git  && $dir != '/' ]]; do
        dir=$(dirname "$dir")
    done
    if [[ $dir == '/' ]]; then
        title=$(basename "$PWD")
    
        title=$(basename "$dir")
    fi

    echo -n "$title"
}


# the terminal title - defaults to the project name
termtitle() {
    local title="$*"
    if [[ -z $title ]]; then
        title=$(get_project_name)
    fi
    echo -ne "\033]0;"$title"\007"
}

set_basic_prompt() {
	PS1='\[\033[1;32;40m\]\h\[\033[0;37;40m\]:\[\033[31;40m\][\[\033[1;34;40m\]\u\[\033[0;31;40m\]]\[\033[0;37;40m\]:\[\033[35;40m\]\w\[\033[1;33;40m\]$\[\033[0m\] 
}

set_computed_prompt() {
	export PROMPT_COMMAND=([0]="printf \"\\033]0;%s@%s:%s\\007\" \"\${USER}\" \"\${HOSTNAME%%.*}\" \"\${PWD/#\$HOME/\\~}\"")
}

```

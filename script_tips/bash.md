## Finding the absolute path to a sourced script:

```
if [[ ${BASH_SOURCE%/*} = "${BASH_SOURCE[0]}" ]]; then
    THISDIR=$PWD
else
    THISDIR=$(cd "${BASH_SOURCE%/*}" && echo "$PWD")
fi
export REPO_ROOT
```

## Finding th absolute path to a script:

```
export REPO_ROOT=$(cd "${BASH_SOURCE%/*}" && echo $PWD)
```


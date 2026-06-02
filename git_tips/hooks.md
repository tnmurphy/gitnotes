# Git Tips

## Hooks


### Pre commit message hook to add 
```
#!/bin/bash
# This is a githook that puts the branch name as a comment on top of your
# commit message.  You only have to uncomment it to include it in your commit 
message.

# tell git to look in the hooks directory (post 2.9)
#     git config --local core.hooksPath .githooks


f=$(mktemp)
echo "$(git status | grep "On branch " | sed 's/On branch //')" >$f &&
cat $1 >> $f && cp $f $1
```

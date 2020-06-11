# Shell Note

## xargs后面跟多条命令

```
    #这里执行了grep和echo两条命令
    find ./ -iname "*pusch*.csv" -print0 | xargs -0 -I {} bash -c "grep 'numOfDlHarqSubframesPacked' {}; echo {}"
    #其中bash可以改成sh:
    find ./ -iname "*pusch*.csv" -print0 | xargs -0 -I {} sh -c "grep 'numOfDlHarqSubframesPacked' {}; echo {}"
```
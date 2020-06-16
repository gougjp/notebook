# Shell Parameter Expansion -- Shell 参数扩展

- **<font color='red'>${parameter:−word}</font>**

如果parameter没有设置或者为空，则替换为word；否则替换为parameter的值。

```Shell
#varB没有设置
[jgou@localhost ~]$ varA=123
[jgou@localhost ~]$ unset varB; echo $varB

[jgou@localhost ~]$ echo ${varB:-${varA}}
123
[jgou@localhost ~]$ echo $varB

[jgou@localhost ~]$

#varB设置为空
[jgou@localhost ~]$ varA=123
[jgou@localhost ~]$ unset varB; varB=""
[jgou@localhost ~]$ echo ${varB:-${varA}}
123
[jgou@localhost ~]$ echo ${varB}

[jgou@localhost ~]$

#varB为456
[jgou@localhost ~]$ varA=123
[jgou@localhost ~]$ varB=456
[jgou@localhost ~]$ echo ${varB:-${varA}}
456
[jgou@localhost ~]$ echo ${varB}
456

# 举例
[jgou@localhost ~]$ unset x;y="abc def"; echo "/${x:-'XYZ'}/${y:-'XYZ'}/$x/$y/"
/'XYZ'/abc def//abc def/
```


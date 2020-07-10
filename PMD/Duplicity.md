# 使用PMD工具统计代码重复度

## 下载PMD工具

- 下载链接https://pmd.github.io/

- 解压后添加环境变量或者使用绝对路径执行/pmd-bin-6.25.0/bin/run.sh脚本

## 在Linux下执行重复度检查

- 进入要检查的代码目录

- 执行命令

```Shell
$ /opt/pmd-bin-6.25.0/bin/run.sh cpd --minimum-tokens 100 --files ./ --language cpp --format xml > report_cpd.xml
$ /opt/pmd-bin-6.25.0/bin/run.sh cpd --minimum-tokens 100 --files ./ --language cpp --format csv > report_cpd.csv
```

## ANT解析报告

- build_report.xml

```Shell
$ cat build_report.xml
<project name="CPD check" default="code_duplex">
<!--properties used by targets -->
	<property environment="env" />
  <property name="output_dir" value="./" />
  <property name="cpd_xlst" value="./cpdhtml.xslt" />

	<target name="code_duplex">
		<xslt in="${output_dir}/report_cpd.xml" style="${cpd_xlst}" out="${output_dir}/report_cpd.html" />							
	</target>	
	
</project>
```

- cpdhtml.xslt

```Shell
$ cat cpdhtml.xslt
<?xml version="1.0" encoding="UTF-8"?>
<!-- Stylesheet to turn the XML output of CPD into a nice-looking HTML page -->
<!-- $Id: cpdhtml.xslt 4492 2006-08-14 14:09:56Z tomcopeland $ -->
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method="html" encoding="UTF-8" doctype-public="-//W3C//DTD HTML 4.01 Transitional//EN" 
	doctype-system="http://www.w3.org/TR/html4/loose.dtd" indent="yes"/>

<xsl:template match="pmd-cpd">
<html>
	<head>
		<script type="text/javascript">
			function toggleCodeSection(btn, id)
			{
				area = document.getElementById(id);
				if (area.style.display == 'none')
					{
					btn.innerHTML = '-';
					area.style.display = 'inline';
					}
				else
					{
					btn.innerHTML = '+';
					area.style.display = 'none';
					}
			}
		</script>
		<style>
			.SummaryTitle  { }
			.SummaryNumber { background-color:#DDDDDD; text-align: center; }
			.ItemNumber    { background-color: #DDDDDD; }
			.CodeFragment  { background-color: #BBBBBB; display:none; font:normal normal normal 9pt Courier; }
			.ExpandButton  { background-color: #FFFFFF; font-size: 8pt; width: 20px; height: 20px; margin:0px; }
		</style>
	</head>
<body>
    <h2>Summary of duplicated code</h2>
    This page summarizes the code fragments that have been found to be replicated in the code.
    Only those fragments longer than 100 tokens of code are shown.
    <p/>
    <table border="1" class="summary" cellpadding="2">
      <tr style="background-color:#CCCCCC;">
        <th># duplications</th>
        <th>Total lines</th>
        <th>Total tokens</th>
        <th>Approx # bytes</th>
      </tr>
      <tr>
        <td class="SummaryNumber"><xsl:value-of select="count(//duplication[@tokens>=100])"/></td>
        <td class="SummaryNumber"><xsl:value-of select="sum(//duplication[@tokens>=100]/@lines)"/></td>
        <td class="SummaryNumber"><xsl:value-of select="sum(//duplication[@tokens>=100]/@tokens)"/></td>
        <td class="SummaryNumber"><xsl:value-of select="sum(//duplication[@tokens>=100]/@tokens) * 4"/></td>
      </tr>
    </table>
    <p/>
    You expand and collapse the code fragments using the + buttons. You can also navigate to the source code by clicking
    on the file names.
    <p/>
    <table>
    	<tr style="background-color: #444444; color: #DDDDDD;"><td>ID</td><td>Files</td><td>Tokens</td><td>Lines</td></tr>
    <xsl:for-each select="//duplication[@tokens>=100]">
        <xsl:sort data-type="number" order="descending" select="@lines"/>
        <tr>
        	<td class="ItemNumber"><xsl:value-of select="position()"/></td>
        	<td>
        		<table>
        			<xsl:for-each select="file">
        				<tr><td bgcolor="#DCDCDC"><xsl:value-of select="@path"/></td><td>     #line <xsl:value-of select="@line"/></td></tr>
        			</xsl:for-each>
        		</table>
        	</td>
			<td><xsl:value-of select="@tokens"/></td>
        	<td># lines : <xsl:value-of select="@lines"/></td>
        </tr>
        <tr>
        	<td> </td>
        	<td colspan="2" valign="top">
        		<table><tr>
        			<td valign="top">
        				<button class="ExpandButton" ><xsl:attribute name="onclick">blur(); toggleCodeSection(this, 'frag_<xsl:value-of select="position()"/>')</xsl:attribute>+</button>
        			</td>
        			<td>
        				<textarea cols="150" rows="30" wrap="off" class='CodeFragment' style='display:none;'>
        					<xsl:attribute name="id">frag_<xsl:value-of select="position()"/></xsl:attribute>
        					<xsl:value-of select="codefragment"/>
        				</textarea>
        			</td>
        		</tr></table>
        	</td>
        </tr>
        <tr><td colspan="2"><hr/></td></tr>
    </xsl:for-each>
    </table>
    
    
</body>
</html>
</xsl:template>

</xsl:stylesheet>
```

- 日志文件名report_cpd.xml, 且跟上面两个脚本放入同一个目录下; 然后执行以下命令就会生成report_cpd.html文件

```Shell
$ ant -f build_report.xml
```



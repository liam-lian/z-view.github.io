# Vim终极操作指南

vim作为最古老的编辑工具之一，一直在江湖上占据着重要的地位，对于我而言vim只是能用的境界，希望中国这篇博客，让自己多多了解vim上的那些黑科技

### 常用的命令
<table>
    <tr><td>快捷键</td><td>作用</td></tr>
    <tr><td>[n]dd/yy</td><td>整行的删除、复制</td></tr>
    <tr><td>G</td><td>移动到最后一行</td></tr>
    <tr><td>nG</td><td>移动到第n行</td></tr>
    <tr><td>gg</td><td>移动到第一行</td></tr>
    <tr><td>n[Enter]</td><td>光标向下移动n行</td></tr>
    <tr><td>x/X</td><td>向后/向前剪切一个字</td></tr>
    <tr><td>nx</td><td>向后删除n个字符</td></tr>
    <tr><td>d0/d$</td><td>删除从光标处到行首/尾的字符</td></tr>
    <tr><td>d1G/dG</td><td>删除从光标处到文件的第一行/最后一行</td></tr>
    <tr><td>y1G/yG/y0/y$</td><td>和d的意义一样</td></tr>
    <tr><td>p/P</td><td>下一行/上一行粘贴</td></tr>
</table>
<table>    
    <tr><td></td><td>查找和替换</td></tr>
    <tr><td>/word</td><td>查找</td></tr>
    <tr><td>n</td><td>下一个查找的位置</td></tr>
    <tr><td>N</td><td>上一个查找的位置</td></tr>
    <tr><td>n1,n2s/word1/word2/g</td><td>替换</td></tr>
    <tr><td>/1,$s/word1/word2/g</td><td>全文件替换</td></tr>
    <tr><td>/1,$s/word1/word2/gc</td><td>全文件替换,但是替换之前需要用户确认</td></tr>
</table>


<table>    
    <tr><td></td><td>基础操作</td></tr>
    <tr><td>/word</td><td>查找</td></tr>
    <tr><td>n</td><td>下一个查找的位置</td></tr>
    <tr><td>N</td><td>上一个查找的位置</td></tr>
    <tr><td>n1,n2s/word1/word2/g</td><td>替换</td></tr>
    <tr><td>/1,$s/word1/word2/g</td><td>全文件替换</td></tr>
    <tr><td>/1,$s/word1/word2/gc</td><td>全文件替换,但是替换之前需要用户确认</td></tr>
</table>


<table>    
    <tr><td></td><td>进入编辑模式</td></tr>
    <tr><td>i/I</td><td>i-从光标前一个位置插入/I行首第一个非空位置插入</td></tr>
    <tr><td>a/A</td><td>a-从光标后一个位置插入/I行尾插</td></tr>
    <tr><td>o/O</td><td>在当前行的后面/前面新开一行</td></tr>
    <tr><td>r/R</td><td>替换</td></tr>
</table>

   


### 块选择

| - | 操作的意义|
|- |-  |
|v|  按字符选择 |
|V|  按行选择   |
|ctrl+v| 按照块进行选择 |
|y| 复制选中的区域 |
|d| 删除选中的区域 |


|id|name|
|:-|:-|
|1|A1|
|2|A2|
|3|A3|


### 多文件编辑
vim file1 file1    
:n 编辑下一个文件  
:N 编辑上一个文件  
:files 列出当前打开的所有的文件

### 多窗口模式
：sp [filename] ===如果不写filename就会直接在新的窗口中打开当前文件  
ctrl+w+Up  切换到上面的窗口  
ctrl+w+Down 切换到下面的窗口  
ctr+w+q   推出当前的窗口

### 全局的配置
- :set nu  
- :set nonu  
- :set backup 自动进行备份，每次修改一个文件的时候，都会把原来的文件备份一下
- :set nobackup

### 一些奇葩的操作
#### vim如何实现[全选]?
ggVG   
首先gg来到第一行，V以行为单位进行选择，一直到G(最后一行)

### Idea的vim插件
IdeaVim插件可以很好的实现vim，操作非常的方便  
vim模式和普通模式切换的插件是【ctrl+alt+v】








# vi 命令

## 新建文件
`vi newFile` 进入编辑环境并打开（新建）文件

## 退出文件
`:q!` 放弃修改退出vi编辑
`:wq!` 保存修改并退出vi编辑

## vi 内操作命令

<table style="border:1px" rules="all">
  <tr>
    <th>命令</th>
    <th>功能</th>
  </tr>

  <tr>
    <td>i</td>
    <td>在当前光标处进入插入状态</td>
  </tr>
  <tr>
    <td>a</td>
    <td>在当前光标后进入插入状态</td>
  </tr>
  <tr>
    <td>A</td>
    <td>将光标移到到当前行的行末，进入插入状态</td>
  </tr>
  <tr>
    <td>o</td>
    <td>在当前行的下面插入新行，光标移到到新行的行首，进入插入状态</td>
  </tr>
  <tr>
    <td>O</td>
    <td>在当前行上面插入新行，光标移到新行的行首，进入插入状态</td>
  </tr>
  <tr>
    <td>cw</td>
    <td>删除当前光标到所在单词尾部的字符，并进入插入状态</td>
  </tr>
  <tr>
    <td>c$</td>
    <td>删除当前光标到行尾的字符，并进入插入状态</td>
  </tr>
  <tr>
    <td>c^</td>
    <td>删除当前光标之前到行尾的字符，并进入插入状态</td>
  </tr>
</table>

<br/>

<table style="border:1px" rules="all">
  <tr>
    <th>操作类型</th>
    <th>光标操作</th>
    <th>功能</th>
  </tr>
  <tr>
    <td rowspan="4">光标移动</td>
    <td>h</td>
    <td>左</td>
  </tr>
  <tr>
    <td>l</td>
    <td>右</td>
  </tr>
  <tr>
    <td>j</td>
    <td>下</td>
  </tr>
  <tr>
    <td>k</td>
    <td>上</td>
  </tr>
  <tr>
    <td rowspan="4">翻页</td>
    <td>ctrl + f</td>
    <td>向下翻页</td>
  </tr>
  <tr>
    <td>ctrl + b</td>
    <td>向上翻页</td>
  </tr>
  <tr>
    <td>ctrl + u</td>
    <td>向下翻半页</td>
  </tr>
  <tr>
    <td>ctrl + d</td>
    <td>向上翻半页</td>
  </tr>
</table>


<table style="border:1px" rules="all">
  <tr>
    <th>命令</th>
    <th>功能</th>
  </tr>
  <tr>
    <td>shift + 4</td>
    <td>跳到行末尾</td>
  </tr>
  <tr>
    <td>shift + 6</td>
    <td>跳到行首</td>
  </tr>
  <tr>
    <td>:set nu</td>
    <td>在编辑器中显示行号</td>
  </tr>
  <tr>
    <td>:set nonu</td>
    <td>取消显示编辑器中行号</td>
  </tr>
  <tr>
    <td>#G</td>
    <td>跳转到文件的第#行</td>
  </tr>
  <tr>
    <td>G</td>
    <td>跳转到文件尾部</td>
  </tr>
  <tr>
    <td>gg</td>
    <td>跳转到文件顶部</td>
  </tr>
</table>


<table style="border:1px" rules="all">
  <tr>
    <th>命令</th>
    <th>功能</th>
  </tr>
  <tr>
    <td>x</td>
    <td>删除当前光标所在字符</td>
  </tr>
  <tr>
    <td>dd</td>
    <td>删除整行/剪切</td>
  </tr>
  <tr>
    <td>yy</td>
    <td>复制当前行</td>
  </tr>
  <tr>
    <td>p</td>
    <td>粘贴</td>
  </tr>
  <tr>
    <td>u</td>
    <td>撤销当前操作</td>
  </tr>
</table>
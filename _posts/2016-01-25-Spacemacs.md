---
layout: post
title: Spacemacs简介
tags: emacs spacemacs
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline4">1. Why Spacemacs</a>
<ul>
<li><a href="#orgheadline1">1.1. Compare to Emacs?</a></li>
<li><a href="#orgheadline2">1.2. Compare to Vim?</a></li>
<li><a href="#orgheadline3">1.3. Compaire to other IDE?</a></li>
</ul>
</li>
<li><a href="#orgheadline8">2. Spacemacs introduction</a>
<ul>
<li><a href="#orgheadline5">2.1. 官网</a></li>
<li><a href="#orgheadline6">2.2. 项目结构</a></li>
<li><a href="#orgheadline7">2.3. 目录结构</a></li>
</ul>
</li>
<li><a href="#orgheadline12">3. Using spacemacs</a>
<ul>
<li><a href="#orgheadline9">3.1. 安装(Mac OS X)</a></li>
<li><a href="#orgheadline10">3.2. 配置.spacemacs</a></li>
<li><a href="#orgheadline11">3.3. 开始使用</a></li>
</ul>
</li>
</ul>
</div>
</div>

# Why Spacemacs<a id="orgheadline4"></a>

## Compare to Emacs?<a id="orgheadline1"></a>

Spacemacs作为Emacs的一个扩展,Spacemacs相对于其他的Emacs配置来说，文档的详细程度绝对完胜其他配置。从头开始折腾Emacs是不太明智的行为，而采用各路大神的配置又会有些摸不清头脑不知道如何入手。
这时候，Spacemacs详细的文档和清晰的分层可以作为想转向Emacs的人入门非常好的版本。

## Compare to Vim?<a id="orgheadline2"></a>

Spacemacs最突出的特点就是与Vim极高的相似性，以空格键作为leader键，甚至文档中专门为有兴趣转入Emacs的Vim党提供了专门的内容。安装过程中选择Vim布局，可以完全采用Vim的编辑方式享受Emacs的各种优势。

## Compaire to other IDE?<a id="orgheadline3"></a>

如果写Java，目前可能没有工具能超过Intellij Idea，但是做轻量级一些的编码工作，一个庞大的IDE可能有些太过重量级。而且Emacs高度的可扩展性给了Geek们无限的空间，其写文档、Latex的能力也是其他IDE不能相比的。

# Spacemacs introduction<a id="orgheadline8"></a>

## 官网<a id="orgheadline5"></a>

[Spacemacs.org](http://spacemacs.org)

## 项目结构<a id="orgheadline6"></a>

Spacemacs 以layer来进行功能的分割，通过.spacemacs文件来进行配置管理，具体内容在仍然在.emacs.d目录下。某个layer的具体配置可以到layer目录下查看文档。

## 目录结构<a id="orgheadline7"></a>

-   assets: 图标等内容,不需要关注。
-   core: Spacemacs 配置的核心内容，最好不要修改其内容。
-   doc: Spacemacs 的各类相关文档。
-   layers: 其他开发者贡献的代码，用于扩展Spacemacs
-   private: 本地个人目录，用于存放自己写的一些EL和keybinding等
-   tests: 测试目录，不需要关注。

# Using spacemacs<a id="orgheadline12"></a>

## 安装(Mac OS X)<a id="orgheadline9"></a>

-   安装Emacs24.4以上的版本
    
        brew tap railwaycat/homebrew-emacsmacport
        brew install emacs-mac --with-spacemacs-icon
        brew linkapps
-   下载配置文件
    
        git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
-   以非安全模式启动emacs以保证成功下载插件
    
        emacs --insecure
-   选择编辑模式、待安装完成后重启

## 配置.spacemacs<a id="orgheadline10"></a>

-   打开 *~.spacemacs*
-   在layers中打开需要用的注释掉的layer，并添加osx以使emacs能正常使用Max OS x的快捷键。
-   最下面 `(defun dotspacemacs/user-config ()` 中添加自己的配置。(若配置有明显分类，应参考官方文档创建自己的layer并进行引用管理）
-   `SPC f e R` 刷新配置

## 开始使用<a id="orgheadline11"></a>

-   与Emacs不同，采用SPC作为Leader键可以减少很多组合型快捷键的使用，在Normal Mode（Vim基本常识）中按下空格键会有大量功能键提示,多看文档多尝试。
    
    <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
    
    
    <colgroup>
    <col  class="org-left" />
    
    <col  class="org-left" />
    </colgroup>
    <tbody>
    <tr>
    <td class="org-left">b</td>
    <td class="org-left">buffer</td>
    </tr>
    
    
    <tr>
    <td class="org-left">f</td>
    <td class="org-left">file</td>
    </tr>
    
    
    <tr>
    <td class="org-left">w</td>
    <td class="org-left">window</td>
    </tr>
    
    
    <tr>
    <td class="org-left">m</td>
    <td class="org-left">mode</td>
    </tr>
    
    
    <tr>
    <td class="org-left">s</td>
    <td class="org-left">search</td>
    </tr>
    
    
    <tr>
    <td class="org-left">h</td>
    <td class="org-left">help</td>
    </tr>
    
    
    <tr>
    <td class="org-left">c</td>
    <td class="org-left">comment/compile</td>
    </tr>
    
    
    <tr>
    <td class="org-left">p</td>
    <td class="org-left">project</td>
    </tr>
    
    
    <tr>
    <td class="org-left">l</td>
    <td class="org-left">layout</td>
    </tr>
    </tbody>
    </table>
-   通过查看layout的文档、根据个人需求定制自己的Emacs

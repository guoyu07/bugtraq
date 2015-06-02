Git Bugtraq 配置规范
=======================================
版本 0.3, 2013-11-25


1. 介绍
---------------

Git的提交动作经常是与issue或bug相关的。Git Bugtraq配置允许你把Git的提交和相应的issue ID关联起来。Git客户端就可以用这种关联关系提供额外的功能，如在提交的消息中issue ID处显示超链接、链到合适的issue跟踪Web页面。

Git Bugtraq配置与Subversion bugtrq属性[1]相似，并包含了Gerrit commentlinks[2]中的概念。


2. 配置选项
------------------------

配置的主要名空间为“bugtraq”。

* bugtraq.url (必填)

声明bug跟踪系统的URL。其必须是进行了适当URI编码的，且必须包含 %BUGID% 占位符。%BUGID%会被Git客户端替换成具体的issue ID。


* bugtraq.logregex (必填),
  bugtraq.loglinkregex (选填) 与
  bugtraq.logfilterregex (选填)

声明Perl兼容的正则表达式[3]，其将被用于提交消息中提取issue ID。

logregex必须是正好只含有一个匹配组，用于提取BUGID。

如果匹配上，loglinkregex会在logregex之前被应用。它也是必须只包含一个匹配组，其从提交消息中提取应该显示为一个链接的部分内容，然后logregex被应用到所有这种链接上提取出实际的BUGID。如设置了loglinkregex，logregex必须刚好提取出一个BUGID。

If present, logfilterregex will be applied before logregex (or
loglinkregex, resp.). It must contain exactly one matching group which
extracts arbitrary parts of the commit message that will be used as
input for logregex (or loglinkregex, resp.).

Every of these regular expressions may contain additional non-matching
groups ('(?:') and matches case-sensitive unless explicitly set to
case-insensitive ('(?i)'). 全部的提取流程如下：

  commit message -> logfilterregex -> loglinkregex -> logregex -> BUGID

例 logfilterregex设为:

  [Ii]ssues?:?((\s*(,|and)?\s*#\d+)+)

loglinkregex设为:

  #\d+
  
logregex设为:

  \d+

提交消息n内容像:

  Issues #3, #4 and #5: Git Bugtraq Configuration options (see rule #12)

logfilterregex 会取到 "Issues #3, #4 and #5", loglinkregex 会取到"#3", "#4", "#5"，logregex 会取到 "3", "4" 和 "5"。

注意: 在Git-Config-like文件中，反斜杠需要被转义。


* bugtraq.loglinktext (可选)

声明用于显示由logregex 与 loglinkregex提取出的issue链接的替换文本。loglinktext中必须包含%BUGID%占位符，以便替换成具体的issue ID。

例  logregex 设为 "#?(\d+)" 且 loglinktext 设为 "#%BUGID%", 提交消息形式

  Issue #3, 4, 5: message
  
将会被替换成：

  Issue #3, #4, #5: message

其中 "#3", "#4", "#5" 变成相应问题Web页面的链接。


* bugtraq.enabled (可选)

声明Bugtraq配置是否启用。默认为'true'.


3. 多配置
--------------------------

There can be multiple configurations, either to support multiple issue
trackers or alternative configurations for the same issue tracker. In
the latter case, probably only one of these configurations will be
enabled.

When using multiple configurations, the bugtraq namespace is separated
into multiple, disjoint sub-namespaces 'bugtraq.<name>', one for each
configuration.

Example:

  bugtraq.tracker1.url=...
  bugtraq.tracker1.logregex=...
  bugtraq.tracker2.url=...
  bugtraq.tracker2.logregex=...

For a single commit message, all configurations will be applied,
possibly giving multiple issue links to different issue trackers for a
single commit message.

3.1 Handling of intersecting issues IDs

If two issue IDs are intersecting (due to intersecting configurations),
only the issue ID with the lower starting position will be displayed.
The other issue ID will be ignored.


4. Configuration files
----------------------

配置选项可以在两个地方声明：

* 在仓库根目录 .gitbugtraq 文件中。该文件使用默认的Git配置文件布局。

* 在 $GIT_DIR/config 文件中

$GIT_DIR/config中声明的选项会覆盖.gitbugtraq.

.gitbugtraq (注意反斜杠需转义)内容举例:

  [bugtraq]
    url = https://host/browse/SG-%BUGID%
    loglinkregex = SG-\\d+
    logregex = \\d+
    
Exactly the same lines could be added as an additional section to
$GIT_DIR/config as well.


5. logregex 举例
--------------------

* From messages like "Fix: #1" or "fixes:  #1, #2 and #3",
  the "1", "2" and "3" should be extracted and the numbers including
  hash-sign (#) should show up as links

  logfilterregex =  "(?i)fix(?:es)?\\: ((\\s*(,|and)?\\s*#\\d+)+)"
  loglinkregex = #\\d+
  logregex = \\d+

* From messages like "Bug: #1" or "Bug IDs: #1; #2; #3" or
  "Cases: #1, #2" the "1", "2" and "3" should be extracted and only the
  numbers itself should show up as links

  logfilterregex =
    "(?i)(?:Bug[zs]?\\s*IDs?\\s*|Cases?)[#:; ]+((\\d+[ ,:;#]*)+)"
  logregex = \\d+
  

References
----------

[1] http://tortoisesvn.net/docs/release/TortoiseSVN_en/
    tsvn-dug-bugtracker.html

[2] https://gerrit-review.googlesource.com/Documentation/
    config-gerrit.html#_a_id_commentlink_a_section_commentlink

[3] http://perldoc.perl.org/perlre.html


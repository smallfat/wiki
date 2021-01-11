---
title: clickhouse
tags: clickhouse 剖析
renderNumberedHeading: true
grammar_cjkRuby: true
---
# 文以载道
clickhouse只做一件事: 让查询更快。有人问我，为什么ch查询会更快。虽然我看了这么久的代码，也在ch上面做过一些feature的二次开发，我仍然很难准确地回答这个问题。
本文希望通过庖丁解牛的方式，剖析主要组件，理清运行脉络，关注性能指标，以量化代码来回答上述问题。

# 
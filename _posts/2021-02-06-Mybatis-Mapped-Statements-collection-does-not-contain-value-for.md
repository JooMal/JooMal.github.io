---
layout: post
title: "Mybatis Mapped Statements collection does not contain value for"
date: 2021-02-06
tags: [TIL]
category : [TIL]
comments: false
---



# Trace

org.apache.ibatis.exceptions.PersistenceException: 

Error querying database.  Cause: java.lang.IllegalArgumentException: Mapped Statements collection does not contain value for mapper.member.findById

Cause: java.lang.IllegalArgumentException: Mapped Statements collection does not contain value for mapper.member.findById



# 시도

1. mapper id가 다른 경우 : mapper 파일의 id(findById)를 잘못 사용하고 있는 경우 -> 체크할 곳 : DAO Impl  => 정상 사용중
2. 파라미터와 빈 필드명이 다른 경우 : member_id로 정상적으로 들어가고 있는 것 확인함





# Ref

[마이바티스 에러](http://blog.naver.com/PostView.nhn?blogId=javaking75&logNo=220315971085)
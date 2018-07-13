---
title: mybatis 原理分析 —— SqlSession 查询分析
date: 2018-07-13  16:26:53
tags: mybatis
category: mybatis
--- 

## SqlSession 的 selectList
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        try {
          MappedStatement ms = configuration.getMappedStatement(statement);
          return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception e) {
          throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
        } finally {
          ErrorContext.instance().reset();
        }
      }


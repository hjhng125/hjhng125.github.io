---
title:  "Spring Batch Multi Thread"
excerpt: "Spring Batch Multi Thread 방식에 대하여"

categories:
  - spring
tags:
  - Spring Batch

last_modified_at: 2021-05-06
---


미완
Spring Batch 어플리케이션을 개발하며 TaskExecutor 인터페이스와 Partitioner 인터페이스를 자주 사용하였는데 이 둘의 차이를 명확히 인지하지 못하고 있어 이번 기회에 정리해볼 것이다.

### TaskExecutor
청크 단위를 다루는 read, processor, writer마다 다른 쓰레드를 사용하여 각 구현체를 사용할 때 멀티쓰레드의 지원여부를 확인해야만 한다.


### Partitioner

스텝 자체를 멀티쓰레드로 분리하여 read, processor, writer의 멀티쓰레드 지원 여부와 상관없이 안전하게 개발가능
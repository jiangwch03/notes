# Fast-Paxos 笔记（分布式共识算法）

## 是什么

Fast-Paxos 是 Paxos 的一个优化版本，主要解决 Multi-Paxos 中 Leader 可能带来的延迟问题。

## 核心思想

- Multi-Paxos：Client → Leader → Acceptor
- Fast-Paxos：Client 直接 → Acceptor

允许 Client 绕过 Leader 直接提案，**一轮消息**就能达成一致。

## 限制

- 多个 Client 同时提案不同值时，需要 Leader 介入协调
- 冲突频繁时，性能反而不如 Multi-Paxos

## 应用

实际应用不多。主流实践仍是 Raft / Multi-Paxos / Zab。

## 一句话总结

> **Fast-Paxos 让 Client 直接提案以减少延迟，但冲突处理复杂，实际应用较少。**

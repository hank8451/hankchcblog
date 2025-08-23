+++
title = 'Process, Thread, Goroutine'
date = 2025-08-23T22:58:10+08:00
draft = false
tags = ['Operating System', 'Golang']
+++

## 前言
Goroutine是Golang提供的輕量級協程，不由OS調度，而是由Go的runtime裡的scheduler來控制。在正確使用它之前，需要在先複習process跟thread的差別。

## Process & Thread
Process(進程)是作業系統資源分配的基本單位，當我們啟動一個應用程式時，OS會創建一個Process，分配獨立的虛擬地址空間、檔案描述符、信號處理器等等。

    Process A 記憶體空間          Process B 記憶體空間
    ┌──────────────┐            ┌──────────────┐
    │   Stack      │            │   Stack      │
    ├──────────────┤            ├──────────────┤
    │   Heap       │            │   Heap       │
    ├──────────────┤            ├──────────────┤
    │   Data       │            │   Data       │
    ├──────────────┤            ├──────────────┤
    │   Text       │            │   Text       │
    └──────────────┘            └──────────────┘
        完全隔離                     完全隔離


Thread(線程)則是CPU調度的基本單位，同一個Process內的所有Thread共享該Process的記憶體空間等資源。但是每個thread有自己的Stack跟Program Counter。

    同一個 Process 內的 Threads：

    Thread 1        Thread 2        Thread 3
         ↓               ↓               ↓
    [Stack 1]       [Stack 2]       [Stack 3]  ← 各自獨立
         ↓               ↓               ↓
    ┌────────────────────────────────────┐
    │         Heap (共享)                 │  ← 共享
    │         Data (共享)                 │  ← 共享  
    │         Text (共享)                 │  ← 共享
    └────────────────────────────────────┘


所以更準確地說，Process是OS提供的抽象概念，用於資源隔離和管理，而CPU執行的是Thread，而不是Process，不要搞混。

比方在一個四核心的機器上，同一時刻最多可以並行執行4個Thread，OS的Scheduler負責決定哪個Thread在哪個CPU上執行，通過Context Switch來實現多工。

           t0      t1      t2      t3      t4
    CPU0: [P1.T1] [P1.T2] [P2.T1] [P1.T1] [P3.T1]
    CPU1: [P2.T1] [P2.T2] [P1.T3] [P2.T1] [P1.T2]
    CPU2: [P3.T1] [P3.T1] [P3.T1] [P3.T2] [P2.T2]
    CPU3: [P4.T1] [P1.T4] [P4.T1] [P4.T2] [P4.T1]
    P = Process, T = Thread

Process 和 Thread 的主要區別在於資源隔離程度。Process 擁有獨立的虛擬地址空間，包括獨立的 Heap、Data、Text 段，Process 間通訊需要 IPC 機制。Thread 則共享所在 Process 的地址空間，只有 Stack 和暫存器是獨立的，可以直接訪問共享記憶體但需要同步機制防止 race condition。
在 Context Switch 方面，Process 切換需要更換頁表、flush TLB，成本約 1000-5000 cycles；Thread 切換只需保存/恢復暫存器和 Stack 指針，成本約 100-1000 cycles。

## Goroutine
Go 的 Goroutine 則實現了 M:N 的線程模型，Context Switch 成本更低（約 100ns），Stack 初始只有 2KB 並可動態增長，這使得可以創建成千上萬個 Goroutine 而不會耗盡資源。

    應用層（User mode）：
    ┌─────────────────────────────────────┐
    │    G1  G2  G3  G4  G5  G6  G7  G8   │ ← Goroutines
    │         ↓ Go Scheduler ↓            │ ← Go Runtime 調度
    │      在這層做 Context Switch！        │
    └─────────────────────────────────────┘
              ↓ 綁定
    OS層（Kernal mode）：
    ┌─────────────────────────────────────┐
    │     M0      M1      M2      M3      │ ← OS Threads
    │         ↓ OS Scheduler ↓            │ ← Kernel 調度
    └─────────────────────────────────────┘
              ↓
    硬體層：
    ┌─────────────────────────────────────┐
    │    CPU0    CPU1    CPU2    CPU3     │
    └─────────────────────────────────────┘
從Context Switch上來考量，Process的切換需要：
1. CPU 暫存器（registers）
2. 程式計數器（PC） 
3. 堆疊指針（SP）
4. 頁表（Page Table）← 最昂貴！
5. TLB flush
6. Cache 失效


Thread：
1. CPU 暫存器
2. 程式計數器
3. 堆疊指針

因為Thread共享同一個地址空間，因此不用切換頁表

## 小節
原本只是想整理一下面試被問Process、Thread的差別，一不小心篇幅太長，下次待續
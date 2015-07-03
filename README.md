# Introduction

After almost 10 years from "The Free Lunch Is Over" article, where the need to parallelize programs started to be a real and mainstream issue, a lot of stuffs did happened:

- Processor manufacturers are reaching the physical limits with most of their approaches to boosting CPU performance, and are instead turning to hyperthreading and multicore architectures;
- Applications are increasingly need to support concurrency;
- Programming languages and systems are increasingly forced to deal well with concurrency.

The article concluded by saying that we desperately need an higher-level programming model for concurrency than languages offer today.

This thesis is an attempt to propose an overview of a paradigm that aims to properly abstract the problem of propagating data changes: **Reactive Programming** (RP). This paradigm propose an **asynchronous non-blocking** approach to concurrency and computations, abstracting from the low-level concurrency mechanisms.

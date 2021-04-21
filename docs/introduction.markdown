---
layout: page
title: Introduction
permalink: /introduction/
---

This page is about the topic Coding Dojo. Individual Coding Katas are presented and solved step by step. This should give the reader the opportunity to learn and practice programming.

I try to keep the following points in mind:

---
Step by step.

Depending on the task, it is necessary to split the task into several subtasks. According to the Roman principle "Divide and Conquer" you do not try to solve the whole task immediately. Instead, you solve the resulting subtasks.

The subtasks must be technically testable. It must be possible to determine by a written test whether the requirement is fulfilled. The size of the created packages should be chosen in a way that they can be completed within one day without any problems.

---
Test-driven development.

* Translate requirement into a test
* Run and fail the test
* An initial implementation
* Refactoring
* All tests
* Check documentation

---
Move forward securely.

A once made progress should not be lost again. That's why every step is followed by a commit with the help of Git.

I do not write this down extra. But the reader should do this by himself. This prevents lengthy searches and frustration.

---
Documentation

This part is the most annoying for most people. The problem is called forgetfulness. After a few months at the latest, no one knows what the code does anymore. And then changes become a gamble.

The documentation here should consist of several parts. The first part is the test. This should list all requirements as meaningful as possible and show with which parameters these requirements were tested.

The productive code is additionally provided with comments at all externally accessible places.

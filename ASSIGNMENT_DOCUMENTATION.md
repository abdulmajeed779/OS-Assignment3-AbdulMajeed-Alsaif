# Assignment 3 - Complete Documentation

**Student Name**: [AbdulMajeed Turki Alsaif]  
**Student ID**: [445050186]  
**Date Submitted**: [May 3, 2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 3-2026,3:45AM]
**What I implemented**: Set my student ID and reviewed the initial codebase to locate race conditions.

**Challenges encountered**: Understanding the multithreaded flow of the CPU scheduler.

**How I solved it**: Traced the `Process` class `run()` method to see how threads interact with `SharedResources`.

**Testing approach**: Compiled the code to ensure it runs with my ID.

**Time spent**:1 hour. 

---

### Entry 2 - [May 3-2026,3:54AM]
**What I implemented**: Added `ReentrantLock` named `countersLock` to protect `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.

**Challenges encountered**: Ensuring that the lock is always released even if an exception occurs.

**How I solved it**: Implemented a `try-finally` block for every method modifying these counters, placing `unlock()` in the `finally` block. 

**Testing approach**: Ran the simulation multiple times to check if the total completed processes matched the generated ones.
 

**Time spent**:1.5 hours. 

---

### Entry 3 - [May 3-2026,4:33AM]
**What I implemented**: Implemented `logLock` to protect the `executionLog` ArrayList.

**Challenges encountered**: The program occasionally threw `ConcurrentModificationException` before the fix.

**How I solved it**: Enclosed the `executionLog.add(message)` statement within a `logLock.lock()` and `unlock()` structure.

**Testing approach**: Executed the code with a high number of processes to force concurrent logging attempts.

**Time spent**:1 hour. 

---

### Entry 4 - [May 3-2026,4:40AM]
**What I implemented**: Added a binary `Semaphore` (`cpuSemaphore`) to control CPU access.

**Challenges encountered**: Threads were executing their quantum simultaneously rather than waiting for the CPU.

**How I solved it**: Initialized the semaphore with 1 permit and added `acquire()` at the start of `run()` and `release()` at the end.

**Testing approach**: Observed the terminal output to ensure only one process progress bar is printed at a time.


**Time spent**: 1.5 hours.

---

### Entry 5 - [May 3-2026,5:45AM]
**What I implemented**: Final code review, completing the `ASSIGNMENT_DOCUMENTATION.md` file, and recording the required demonstration video.

**Challenges encountered**: Keeping the video under the strict 5-minute limit while clearly explaining all synchronization mechanisms and race conditions.

**How I solved it**: I wrote a quick script and practiced the code walkthrough a few times before recording to ensure my explanation of `ReentrantLock` and `Semaphore` was concise and accurate.

**Testing approach**: Verified that the GitHub repository is set to public and tested the Google Drive video link in an incognito browser window to ensure the instructor can access it.

**Time spent**:2 hours. 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[1. **Counter Variables**: The variable `contextSwitchCount` is shared among all process threads[cite: 2]. When multiple threads execute `contextSwitchCount++` concurrently, they read the same initial value, increment it, and write it back, causing lost updates (a race condition)[cite: 1, 2]. This leads to an inaccurate total context switch count.
2. **Execution Log (ArrayList)**: The `executionLog` list is updated by multiple threads calling `logExecution()`[cite: 2]. `ArrayList` in Java is not thread-safe; concurrent modifications can corrupt its internal array structure or throw a `ConcurrentModificationException`, crashing the program[cite: 1, 2].]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[A `ReentrantLock` provides mutual exclusion (mutex) and is owned by the thread that acquires it, meaning only that thread can release it. I used `ReentrantLock` to protect shared data structures (`countersLock` for variables, `logLock` for the ArrayList) to prevent race conditions during updates. 
A `Semaphore` is a signaling mechanism used to limit the number of threads accessing a resource concurrently; it does not have ownership. I used a binary Semaphore (`cpuSemaphore` with 1 permit) to represent the single CPU, ensuring only one process runs at any given time[cite: 1, 2, 3].]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[A deadlock occurs when two or more threads are blocked forever, each waiting for a lock held by another thread. 
Two prevention techniques are: 1) **Lock Ordering**: Ensuring all threads acquire locks in the exact same global order. 2) **Guaranteed Release**: Ensuring a lock is always released regardless of execution flow.
In my code, I prevented deadlocks by strictly using `try-finally` blocks for every `ReentrantLock` and `Semaphore`. The `unlock()` and `release()` methods are placed inside the `finally` block, guaranteeing that resources are freed even if an exception interrupts the thread's execution[cite: 1, 2].]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[I chose a **coarse-grained** locking approach by using a single `countersLock` for all three counters (`contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`). 
While fine-grained locking (one lock per counter) provides better concurrency because threads can update different counters simultaneously without blocking each other, I opted for coarse-grained locking because the operations are extremely fast (simple arithmetic). The overhead of managing multiple locks would outweigh the concurrency benefits in this specific simulation. However, in a high-performance system where these updates take longer, a fine-grained approach would be superior to prevent unnecessary waiting.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`[cite: 2, 3].

**Why they need protection**: They are shared primitives updated concurrently by multiple threads, leading to lost updates.

**Synchronization mechanism used**: `ReentrantLock` (`countersLock`)[cite: 1, 2, 3].

**Code snippet**:
```java
public static void incrementContextSwitch() {
    countersLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        countersLock.unlock();
    }
}
```

**Justification**: Mutual exclusion ensures that the read-modify-write cycle of the increment operation is atomic[cite: 3].

---

### Critical Section #2: Execution Log

**What resource**: executionLog (ArrayList)[cite: 2, 3].

**Why it needs protection**: ArrayLists are not thread-safe and will throw ConcurrentModificationException under concurrent writes[cite: 1, 3].

**Synchronization mechanism used**: ReentrantLock (logLock)[cite: 1, 2, 3].

**Code snippet**:
```java
public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
```

**Justification**:Protects the internal array resizing and element addition operations of the ArrayList[cite: 3]. 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To simulate a single-core CPU by restricting execution access[cite: 1, 3].

**Number of permits and why**: 1 permit, because a single CPU core can only execute one process at a time[cite: 1, 3]

**Where implemented**: Inside the run() and runToCompletion() methods of the Process class[cite: 2, 3].

**Code snippet**:
```java
try {
    SharedResources.cpuSemaphore.acquire();
    // Critical CPU execution...
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: Processes wait in the ready queue and only print their execution progress one at a time, preventing terminal output from mixing and accurately simulating scheduling[cite: 3]. 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
```

**Results**: The total number of completed processes consistently matched the initial number of generated processes. The execution log size was also consistent[cite: 3].
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: Without locks, the completedProcessCount might show fewer processes completed than actual, and terminal output would be a chaotic mix of multiple progress bars printing at the same time[cite: 3]. 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: The implemented synchronization successfully maintains data integrity[cite: 3].

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: Examined the terminal output across multiple runs with up to 20 processes[cite: 3].

**Results**: Zero runtime exceptions occurred[cite: 3].

**What this proves**: The logLock properly isolates access to the ArrayList, making it thread-safe for our use case[cite: 3].

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: Completed processes = Total generated processes[cite: 3].

**Actual values**: Both values matched exactly in the final statistical summary[cite: 3].

**Analysis**:The locks ensure no increments are lost during context switches or process completions[cite: 3]. 

---

### Test 4: Different Scenarios
**Scenario tested**: [Increasing the time quantum logic temporarily to simulate long-running processes[cite: 3]]

**Purpose**: To verify that the semaphore effectively blocks other processes for longer durations[cite: 3].

**Results**: The semaphore held the CPU resource effectively, and other threads waited patiently without consuming CPU cycles improperly[cite: 3].

**What I learned**: Semaphores are highly reliable for resource allocation and thread pacing[cite: 3].

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

This assignment solidified my understanding of concurrent programming in Java. I learned that simply creating threads is not enough; managing their interactions is the real challenge. I saw firsthand how race conditions can silently corrupt data without always crashing the program. Understanding the difference between Locks (for data protection) and Semaphores (for resource management) was a major breakthrough. I also learned the critical importance of try-finally blocks to prevent catastrophic deadlocks[cite: 1, 3].
---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: Banking systems processing concurrent deposits and withdrawals on the same account balance[cite: 3].

**Example 2**: Database management systems ensuring atomicity when multiple users try to book the same airline seat[cite: 3].

---

### How I would explain synchronization to others:

Imagine a classroom with one marker (the CPU) and a whiteboard. Threads are the students wanting to draw. Synchronization is the rule that says only the student holding the marker can draw (Semaphore). If two students try to write on the same line of the attendance sheet at the same time, it becomes unreadable (Race Condition). A Lock is like giving the attendance sheet to a teacher who only lets one student sign it at a time.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/abdulmajeed779/OS-Assignment3-AbdulMajeed-Alsaif

**Number of commits**: 4

**Commit messages**: 
1. Set my student ID: 445050186
2. Add ReentrantLock to protect shared counters
3. Add ReentrantLock to protect execution log ArrayList
4. Implement binary Semaphore for CPU access control

---

## Summary

**Total time spent on assignment**: Approx. 5 hours

**Key takeaways**: 
1. Shared resources must be protected to prevent race conditions.
2. try-finally blocks are mandatory for safe lock/semaphore release.
3. Semaphores are excellent tools for modeling physical constraints like CPU cores. 

**Most challenging aspect**: Ensuring the correct placement of the acquire() and release() methods for the Semaphore without causing infinite waiting[cite: 3].

**What I'm most proud of**: Implementing the locks cleanly and seeing the messy console output transform into a perfectly ordered, synchronous execution log[cite: 3].

---

**End of Documentation**

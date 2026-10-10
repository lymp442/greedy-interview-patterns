# greedy algorithm interview questions: how to spot the pattern, prove the choice is safe, and know when to switch to DP

You can solve activity selection in your sleep. Then the interviewer asks "why is picking the earliest finishing interval actually safe?" and the room gets quiet.

That's the thing about greedy questions. They almost never show up as "explain a greedy algorithm." They show up dressed as meeting rooms, gas stations, balloon bursts, and cooldowns — and the real test isn't whether you can write the loop. It's whether you can commit to a local decision, defend it, and recognize the moment when that decision is wrong and you need dynamic programming instead.

This is a walkthrough of the greedy questions that actually get asked, the families they fall into, the answer structure that holds up under follow-up questions, and how to practice it without grinding a 3,000-problem bank.

## Why greedy feels harder than trees or binary search

Every other topic gives you a shape to recognize. Binary search means sorted input. Graphs mean nodes and edges. Greedy gives you nothing visual — it's a decision, and the decision is only correct if the problem has two specific properties:

- **Greedy choice property** — a locally optimal choice is always part of some globally optimal solution. You never have to take a choice back.
- **Optimal substructure** — after you make that choice, what's left is a smaller version of the same problem.

When those hold, greedy is both correct and fast. When they don't, greedy produces a plausible wrong answer that passes small examples and dies on a hidden test case.

The 0/1 knapsack versus fractional knapsack split is the cleanest demonstration. Fractional knapsack — where you can take any fraction of an item — is solved optimally by sorting on value-to-weight ratio and taking greedily. Remove the ability to take fractions and the same rule breaks immediately. Take a capacity of 10 with items (weight 6, value 6), (weight 5, value 5), (weight 5, value 5). Greedy by ratio grabs the first item and stops with value 6. The optimal answer is 10. Same rule, same sorting key, different problem structure.

Coin change is the other classic. Greedy "always take the largest coin" is optimal for US denominations because the coin system is canonical. It fails for the set {1, 20, 30} with a target of 40: greedy gives 30 + ten 1s = 11 coins, while 20 + 20 = 2 coins. Nothing about the greedy *strategy* changed. The input structure did.

That distinction is what interviewers are fishing for.

## The two things a greedy question is really testing

**1. Can you attack your own rule?**

Before committing, you want to hunt for a counterexample with tiny inputs — three or four elements, small numbers, a tie. If you find one in thirty seconds, you just saved yourself the rest of the interview. If you can't find one after genuinely trying, that's not proof, but it's a reasonable basis for proceeding and saying so out loud.

**2. Can you justify the choice in one or two sentences?**

You don't need a formal proof in a 45-minute loop. You need the exchange argument's core idea: suppose an optimal solution makes a different first choice; show you can swap in the greedy choice without making the result worse; repeat until the optimal solution *is* the greedy solution.

For interval scheduling that sentence reads: "The interval that finishes earliest leaves at least as much room for everything after it as any other choice, so replacing the first interval in any optimal solution with it can't hurt." That's it. That's the proof, compressed.

Compare the two ways to open:

> **Weak:** "I think greedy works here."
>
> **Strong:** "Since picking the earliest-finishing interval leaves the maximum remaining space, this local choice is safe. Let me sort by end time and sweep."

Same algorithm. Different signal to the interviewer.

## The greedy questions that keep showing up

Grouped by the decision structure they share — because the grouping is what makes them memorable.

### Interval scheduling and overlap

The largest family by far, and the one where choosing the wrong sorting key loses the round.

| Problem | LeetCode | The greedy key |
| --- | --- | --- |
| Meeting Rooms II | 253 | Sort by start; min-heap of end times counts concurrent rooms |
| Non-overlapping Intervals | 435 | Sort by end time; keep the earlier finisher on conflict |
| Minimum Arrows to Burst Balloons | 452 | Sort by end coordinate; shoot at the first balloon's end |
| Merge Intervals | 56 | Sort by start, extend the current interval's end |
| Insert Interval | 57 | Walk and merge without a full sort |
| Maximum Length of Pair Chain | 646 | Sort by second element, then sweep |

Sorting by start time when the decision hinges on end time is the single most common way this family gets failed. In 435 and 452 the comparison operator (`<` versus `<=`) also flips whether touching intervals count as overlapping — a detail that has cost real candidates real offers.

### Reachability and circular routes

No sorting, one pass, O(n). These are the ones that look like they need DP and don't.

- **Jump Game (55)** — track the farthest reachable index; if your current position ever exceeds it, return false.
- **Jump Game II (45)** — track the current jump's range and the farthest index reachable from inside it. When you hit the end of the range, you're forced to jump; increment the count and extend the range.
- **Gas Station (134)** — if total gas is at least total cost, a valid start exists. The start is the station right after the last point where the running tank went negative.

The interviewer's follow-up on all three is the same: "why don't you need to consider backtracking?" Which is your cue to explain the safety of the local choice.

### Sort, then match

Straightforward once you accept that sorting is the preprocessing step, not a hint that you're overengineering.

- **Assign Cookies (455)** — sort both arrays, two pointers, smallest cookie to least hungry child.
- **Boats to Save People (881)** — sort weights, pair the heaviest with the lightest if they fit, otherwise send the heaviest alone.
- **Two City Scheduling (1029)** — sort by the *difference* between the two city costs, not by either cost.
- **Queue Reconstruction by Height (406)** — sort by height descending, then by position ascending, and insert.
- **Maximum Number of Coins You Can Get (1561)** — sort, take the second-largest of each group of three.

Two City Scheduling is the interesting one, because the non-obvious part isn't the greedy logic — it's identifying what to sort on.

### Frequency, heaps, and cooldowns

Here the greedy choice is made repeatedly over a changing set, which is the signal for a priority queue.

- **Task Scheduler (621)** — the most frequent task is the bottleneck. The answer is `max(len(tasks), (max_count - 1) * (n + 1) + tasks_with_max_count)`.
- **Huffman Coding** — repeatedly merge the two least frequent nodes using a min-heap; total O(n log n).
- **Course Schedule III (630)** — iterate by deadline, keep a max-heap of durations, and drop the longest course whenever the running total exceeds the current deadline.
- **Minimum Number of Refueling Stops (871)** — a max-heap of fuel amounts you've passed, tapped only when you run out.
- **Reorganize String (767)** — always place the character with the highest remaining count that isn't the previous one.

### Stack-based greedy

The monotonic stack pattern, which is greedy reasoning wearing a data structure costume.

- **Remove K Digits (402)** — pop larger digits while removals remain.
- **Remove Duplicate Letters (316)** — track last occurrence, pop when a smaller character is still available later.
- **Best Time to Buy and Sell Stock II (122)** — sum every positive daily gain.
- **Maximum Subarray (53)** — Kadane's algorithm: restart the running sum whenever it goes negative.

Kadane's shows up on both greedy and DP lists, which is a fair reflection of reality — the same recurrence can be described either way, and candidates being asked about it should be comfortable with both framings.

### Greedy inside graph algorithms

Dijkstra's shortest path, Prim's and Kruskal's minimum spanning trees, and Huffman coding are all greedy algorithms. Interviewers rarely ask you to re-derive them, but "which parts of Dijkstra are greedy?" is a fair question, and the honest answer involves both the greedy selection of the minimum-distance unvisited vertex and the DP-flavored distance relaxation that follows.

## A short answer script that survives follow-ups

1. **Restate the optimization target.** "We're maximizing the number of non-overlapping intervals."
2. **Look for recognition signals.** Words like *maximum*, *minimum*, *schedule*, *select*, *assign*, plus any hint that sorting would help.
3. **Name the local decision and its sorting key.** "I'll process intervals in order of finish time and take each one that doesn't conflict."
4. **Attack it.** Try three or four small inputs, including one with ties and one with touching endpoints.
5. **Justify in one or two sentences.** Exchange argument or optimal substructure.
6. **State complexity and note the fallback.** "O(n log n) for the sort, O(n) for the sweep. If intervals carried weights, greedy would break and I'd go to DP."

Step 6 is where strong candidates separate themselves. Mentioning the weighted version of the problem — where each interval has a profit and the answer requires DP over sorted intervals with binary search for the previous compatible one — shows you understand *why* greedy works rather than just *that* it works.

## Telling greedy from DP in the first five minutes

| Signal in the problem | Reach for | Because |
| --- | --- | --- |
| Count or group non-overlapping intervals | Greedy | Earliest-finish choice is provably safe |
| Intervals with weights or profits | DP | Local finish time says nothing about total value |
| Sorted input + one pass, no backtracking | Greedy | No overlapping subproblems |
| "At most k" or "exactly k" selections | Usually DP | The k constraint ties future choices to past ones |
| Fractional items | Greedy | Ratio sorting is optimal |
| 0/1 items with capacity | DP | Local ratio blocks better combinations later |
| Canonical coin system | Greedy | Denominations guarantee the greedy choice property |
| Arbitrary denominations | DP | Counterexamples exist, as {1, 20, 30} → 40 shows |
| Repeatedly picking min/max from a changing set | Greedy + heap | The choice is local, the set is dynamic |
| Circular feasibility (gas station) | Greedy | Total feasibility plus reset-on-negative is sufficient |

The practical heuristic: if you can state your local rule in one sentence *and* you can't break it in two minutes of trying, write it, explain it, and move on. If you find a counterexample, say so out loud and pivot. Interviewers reward the pivot. Stubbornly coding a broken greedy solution for twenty minutes is the actual failure mode.

## What AlgoMonster covers here, and where it stops

AlgoMonster is a pattern-based interview prep course built by ex-Google engineers, and greedy sits inside its broader pattern curriculum rather than as a standalone video series. The approach is text plus diagrams — around 700 interactive illustrations across 48 patterns — with a decision flowchart that asks questions like "is the input sorted?", "does a greedy solution apply?", and routes you to the likely technique. That flowchart is the part most comparable to the recognition problem described above.

What the platform gives you for greedy specifically: pattern-by-pattern lessons with code templates, a curated problem set (roughly 231 problems across all patterns rather than thousands), Speedrun mode for timed practice, and lessons written by competitive programmers. The curriculum is designed to be finishable in weeks rather than months.

The honest limitations: there are no video walkthroughs at all, which matters if you learn by watching someone code. Problem volume is far smaller than LeetCode's, so pattern recognition is covered but raw volume practice probably isn't. And the system design and behavioral content is introductory — fine as a bonus, not sufficient for senior-level system design rounds. There's also no live coaching or mock interviews with real engineers.

### Plans and current pricing

AlgoMonster runs sales constantly, so treat these as observed values rather than permanent prices. There is no monthly plan and no team or enterprise tier — annual or lifetime, that's it.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | — | [Start with the free tier](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems, unlimited AI assistant, unlimited Speedrun, 500+ lessons across 6 courses, member discount on live bootcamps | roughly $69.99–$99/year depending on the active sale | Billed once annually | [See the current yearly price](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan plus permanent access and lifetime updates | roughly $89–$189 one-time depending on the active sale | One-time payment | [Check the lifetime deal](https://algo.monster?fpr=li69) |

Subscriptions also include bonus courses at no extra cost, currently a CS fundamentals course and a build-an-LLM course.

Which one makes sense depends on your timeline. If you have interviews in the next two to three months and you'll stop studying after, yearly is the cheaper commitment. If you expect to interview again in two or three years — which is the normal pattern for engineers targeting promotions or external offers — the lifetime plan stops being a stretch, since it's a one-time payment and yearly renewals are not.

## Questions people ask before they start

**Do I need to be able to write formal proofs?** No. Interviewers want one or two sentences of reasoning, not a theorem. The exchange argument is a mental tool for building confidence, not a deliverable.

**Are greedy questions actually common in interviews?** Yes, but almost always disguised as an interval, scheduling, or reachability problem. Being asked "define a greedy algorithm" is rare; being asked about Meeting Rooms II or Jump Game II is not.

**How many greedy problems should I solve?** Around twenty to twenty-five, as long as they span all the families above. Solving ten variations of interval scheduling teaches you interval scheduling.

**Is Huffman coding tested?** Not in full implementation form usually, but the "merge the two smallest with a heap" logic, the complexity, and why the choice is safe come up.

**Can I always just use DP instead?** Sometimes — Jump Game and Maximum Subarray both have DP solutions. But DP costs you O(n²) or worse in places where greedy gives O(n), and using DP where greedy works often signals that you didn't recognize the structure. Knowing both, and being able to say why you picked one, is the strongest position.

The pattern to internalize: name the local decision, try to break it, justify it in a sentence, and know the DP fallback exists. Do that consistently and greedy questions stop being the ones you hope don't come up. 👉 [Browse the pattern curriculum and current pricing](https://algo.monster?fpr=li69)

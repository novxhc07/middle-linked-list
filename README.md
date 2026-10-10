# middle of the linked list: solve LeetCode 876 in one pass with fast and slow pointers (Python, Java, C++, Go)

Most people meet this problem in their first week of linked-list practice, get it accepted in two traversals, and move on. Then an interviewer says "one pass, constant space," and the solution they rehearsed quietly stops being the answer. Below is what the problem actually asks, why the two-pointer version works, the exact guard condition that breaks it, and the follow-up problems where finding the middle is only step one.

## What the problem actually asks

You get the head of a singly linked list. Return the middle node. If there are two middle nodes, return the **second** one. That clause is doing real work — it's the difference between a correct answer and a wrong one on half the test cases.

| Input | Node returned | What the judge serializes |
| --- | --- | --- |
| `[1,2,3,4,5]` | node 3 | `[3,4,5]` |
| `[1,2,3,4,5,6]` | node 4 (the second of 3 and 4) | `[4,5,6]` |
| `[1]` | node 1 | `[1]` |

The constraints are tiny: between 1 and 100 nodes, values between 1 and 100. Small enough that brute force passes the judge and still fails the interview. The constraints aren't the target; the technique is.

One detail worth knowing if you're asked a variant: the official constraints guarantee a non-empty list, so `head = null` never reaches the judge here. The two-pointer code handles it anyway, because the loop simply never starts. That's a free robustness win, not a requirement.

## The obvious solution, and why it isn't the answer

Walk the list once to count the nodes, compute `n // 2`, then walk again from the head that many steps and return where you land:

python
def middleNode(head):
    n, cur = 0, head
    while cur:
        n += 1
        cur = cur.next
    cur = head
    for _ in range(n // 2):
        cur = cur.next
    return cur


This is O(n) time and O(1) space. It's correct, and on a real codebase nobody would complain. It's also the version interviewers use as a baseline before pushing further.

A second common approach dumps every node into an array and indexes the middle — O(n) time, O(n) space. Fine as a first instinct in an interview, since you can state it and then improve it, but it throws away the entire reason this problem exists: you're supposed to find the middle without knowing the length in advance.

The real gain from the fast/slow version is the single traversal. If the list arrives as a stream you can't rewind, or you're told the length is unknown, the counting approach is dead on arrival. That's the case the interviewer is checking for.

## The fast and slow pointer trick

Two pointers start at the head. Each iteration, `slow` moves one node and `fast` moves two. When `fast` runs off the end, `slow` is standing on the middle, because `fast` covers exactly twice the ground.

Traced on the odd-length list `1 → 2 → 3 → 4 → 5`:

| Step | `slow` | `fast` | Loop check |
| --- | --- | --- | --- |
| Start | 1 | 1 | `fast` exists, `fast.next` exists → continue |
| After iteration 1 | 2 | 3 | `fast` exists, `fast.next` exists → continue |
| After iteration 2 | 3 | 5 | `fast` exists, `fast.next` is null → stop |

Return `slow` → node 3.

Traced on the even-length list `1 → 2 → 3 → 4 → 5 → 6`:

| Step | `slow` | `fast` | Loop check |
| --- | --- | --- | --- |
| Start | 1 | 1 | continue |
| After iteration 1 | 2 | 3 | continue |
| After iteration 2 | 3 | 5 | `fast.next` = 6, exists → continue |
| After iteration 3 | 4 | null | `fast` is null → stop |

Return `slow` → node 4. The second-middle rule falls out of the mechanics for free: `fast` steps past the last node entirely, so `slow` lands one node later than the first middle. No branches, no special cases.

> If you remember one thing about this problem, make it this: the loop guard belongs to `fast`, not `slow`. Almost every wrong submission here is a null-pointer crash, not a logic error.

## The loop condition is where people break it

The guard is `while fast != null && fast.next != null`, and both checks are load-bearing.

`slow` never travels further than `fast`, so checking `slow` proves nothing about whether `fast.next.next` exists. Drop the `fast.next != null` check and your second-to-last iteration dereferences null on every even-length list. Reorder the two checks — test `fast.next` before `fast` — and you crash on odd-length lists instead. Java, C++ and JavaScript all evaluate the conjunction left to right, which is exactly why the order in `fast != null && fast.next != null` matters, while Python's `while fast and fast.next` reads the same way without the syntax noise.

There's a subtler failure that doesn't crash anything: writing the loop so it exits one iteration early or late, which silently gives you the first middle node instead of the second. It passes `[1,2,3]` and fails `[1,2,3,4]`. Always trace your loop on both an odd and an even input before saying you're done.

## Working code in four languages

Python:

python
def middleNode(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow


Java:

java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}


C++:

cpp
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}


Go:

go
func middleNode(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil && fast.next != nil {
        slow = slow.next
        fast = fast.next.next
    }
    return slow
}


Four loops, four identical shapes. That's the point of pattern-based prep — the language changes, the pointer skeleton doesn't. If you'd rather see the pointer positions printed iteration by iteration before you write anything yourself, AlgoMonster keeps [a walkthrough of this exact problem with every step drawn out](https://algo.monster/liteproblems/876?fpr=li69), including the even-length trace that usually gets skipped.

## If the interviewer wants the first middle node

Some variants and some real interviewers ask for the lower middle — for `[1,2,3,4]`, node 2 rather than node 3. You change one line, the guard:

python
while fast.next and fast.next.next:
    slow = slow.next
    fast = fast.next.next


Now `fast` stops before it can leave the list, so `slow` stays one node earlier on even-length inputs. Trace it: on `[1,2,3,4]` you get one iteration (`slow` = 2, `fast` = 3), then `fast.next.next` is null and the loop ends, returning 2. On `[1,2,3,4,5]` you get two iterations and return 3. Odd lengths behave identically under both guards — which is why the lower-middle bug survives every odd test case you throw at it. Confirm which convention is wanted before writing code, and say the convention out loud so the interviewer can correct you.

## Complexity, and what "one pass" buys you

| Approach | Time | Extra space | Traversals |
| --- | --- | --- | --- |
| Copy to array | O(n) | O(n) | 1 |
| Count, then walk | O(n) | O(1) | 2 |
| Fast and slow pointers | O(n) | O(1) | 1 |

The loop body runs about `n/2` times with constant work inside, so O(n) time. Two pointers is O(1) auxiliary space regardless of list length. You cannot do better than O(n) in time: any correct solution has to touch at least the middle node, and on a singly linked list reaching it requires walking from the head.

## Why this small problem keeps showing up

Its Easy rating is misleading in one specific way: the technique is a component of harder problems, and interviewers use it as a cheap check that you can handle pointer movement before they trust you with anything bigger. Finding the middle is a subroutine in several problems that are not Easy.

| Follow-up problem | Where the middle comes in |
| --- | --- |
| Palindrome Linked List (234) | Find the middle, reverse the second half, compare |
| Reorder List (143) | Split at the middle, reverse the back half, merge alternately |
| Delete the Middle Node of a Linked List (2095) | Find the middle, unlink it via the previous node |
| Linked List Cycle (141) | Same two-pointer skeleton, different stop condition (pointers meet) |
| Remove Nth Node From End (19) | Two pointers offset by n, same speed — mirror image of this trick |
| Sort List (148) | Split at the middle for a merge-sort recursion |

Notice how often the fast/slow skeleton is reused: the movement is the same, only the stopping rule and the follow-up action change. That's the argument for learning the pattern instead of the solution to problem 876. Memorizing "return slow" gets you one accepted submission. Recognizing "I need the midpoint of a list I can't measure" gets you the split step in three of the problems above.

## Getting the pattern into muscle memory

If you're solving this with LeetCode open and a solution tab on standby, the gap isn't this problem — it's that the pattern behind it doesn't stick from one exposure. AlgoMonster's curriculum is organized around roughly 48 patterns, with two pointers among them, plus template code in six languages (Python, Java, JavaScript, C++, Go, Haskell), around 230 lessons and problems, and a Speedrun mode for reviewing patterns shortly before an interview. It's text-and-diagram based rather than video, which suits people who'd rather read a loop trace than watch someone narrate it, and the trade-off is real: reviewers consistently note there's no live mock-interview environment and that system design coverage is thin, so it complements a system design resource rather than replacing one.

Worth being straight about the pricing too, since third-party write-ups disagree with each other and the numbers move with whatever sale is running:

| Plan | What you get | Price | Billing period | Access |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | No payment, no card | [Browse the free tier](https://algo.monster?fpr=li69) |
| Pro - Yearly | Full pattern curriculum, all lessons and problems, Speedrun review, company question banks, updates during the term | $99/year (about $8.25/month, billed annually) | Annual, auto-renews | [Start Pro Yearly](https://algo.monster/subscribe?fpr=li69) |
| Pro - Lifetime | Everything in Yearly, plus lifetime updates | $189 one-time | Single payment, no renewal | [Get Pro Lifetime](https://algo.monster/subscribe?fpr=li69) |

Prices above reflect current promotional pricing. Older reviews quote a lifetime price as high as $459, which should tell you how often the promotion changes — check the number on the checkout page before paying, and don't budget off a blog post.

The honest break-even math: if you have an interview loop in the next two or three months, the yearly plan costs less than a year of the usual premium problem bank and gives you a structured order to study in. If you've already interviewed twice in your career and expect to again, the one-time payment stops mattering after the second use. If you only want volume practice on a problem bank you already understand, the free tier plus free problem sets will do, and spending $189 here would be money badly allocated.

## Quick answers to the questions that come up

**Does it work when the list has one node?** Yes. The guard fails immediately, `slow` is still the head, and you return it.

**Do I need a special case for an empty list?** The judge never sends one, but the loop handles it — `fast` is null, the loop is skipped, `slow` returns null.

**Is the array approach acceptable?** For the judge, yes. In an interview, state it as a first instinct and then offer the O(1) space version; presenting it as your final answer signals you didn't notice the space cost.

**Can this be done recursively?** You can recurse to the end and count backward, but that costs O(n) stack space and buys nothing over two pointers.

**What if the interviewer asks for the first middle node instead?** Change the guard to `fast.next and fast.next.next`, as above, and say which convention you're implementing.

## What to do next

Write the four-line loop from memory in your main language, twice, without looking. Then trace it by hand on `[1,2,3,4]` and `[1,2,3,4,5]` and check that the even one returns the second middle. Once that's automatic, move to Palindrome Linked List (234) and Reorder List (143) — both start with the exact midpoint you just learned to find, and both will tell you whether you learned the pattern or just this problem.

If you'd rather follow a curriculum than assemble one problem at a time, 👉 [here's the pattern-based track that covers two pointers and the rest](https://algo.monster/subscribe?fpr=li69) — and the free tier is enough to judge whether the teaching style fits how you learn before you pay anything.

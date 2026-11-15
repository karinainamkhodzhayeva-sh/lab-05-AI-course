# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 ( passing tool errors back to the model) changed my results the most. Befor Layer 1, bare agent score 16/4/4 (correct/ wrong/ no andswer) crashing whenever tool rejected unknown region name. With Layer 1 enabled, the results shifted to 18/6/0. It completely eliminated all 4 "no answer" crashes and genuinely expanded what the agent could do on alias questions: the model error massege, called list_regions, and successfully recover on q23 and q24. On q21 and q22 recovering from the crash simply allowed the model to guess the wrong region and produce wrong answers instead of stopping. By contrast, Layer 2 changed nothing at all ( remaining at 18/6/0) while Layer3 only changed what the agent admits-turning 2 wrong answers (q18 and q22) into "no answer" (UNVERIFIED) to teach 18/4/2


This matched my prediction that Layer 1 would fix alias crashes while Layer 2 would change nothing, though the bare agent actually did better than my initial 10–12 guess (16/24) because it handled percentage math well.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

The full harness (Layer 3) still got q08 wring asks which region had the lowest average monthly wage in 2024. In the trace, the model correctly called compare_regions in step 1, receiving a list sorted from highest to lowest. However, it went wrong right after reading the tool output: instead of picking the last region in the list, the model replied Jambyl. In step 2 Layer 3 only rejected the reply for missing the ANSWER: prefix, so the model simly repeated ANSWER: Jambyl and passied. The grounding check could not catch this because "Jambyl" actually appeared in the tool output, even though it was not the lowest value. To fix this, I would add a post-tool sorting/extremum check in the harness for ranking questions, which would cost extra validation code and might fail if a question asks for second-highest or median values. 

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

For monthly publication asking which region is highest and lowest on one indicator in one year, I would definitely ship the fixed workflow. In Part 1, on the 15 fixed-shape questions, the workflow achieved 13/15 correct answers with 0 wrong answers (the only 2 "no answer" results were due to unnormalized alias names, which are easy to standardize in a monthly pipeline). Compared to the bare agent (9/15 correct, 2 wrong, 4 no answer), the workflow was far more reliable and much cheaper: it used only 15 model calls, 3,852 tokens, and 15.4 seconds total, whereas the agent required 26 model calls, 19,835 tokens, and 47.7 seconds (over 5 times more tokens and 3 times slower). I would only choose the agent if dala Insight needed to answer open-ended, multi-step analytical questions, such as calculating custom percentage growth rates or combining multiple indicators, that do not fit a rigid template 


This confirmed my prediction that both methods would fail on alias names like 'Nur-Sultan', though the workflow saved even more tokens than the 3–4x gap I expected (over 5 times fewer).

---

I wrote this file myself, without any AI tool.

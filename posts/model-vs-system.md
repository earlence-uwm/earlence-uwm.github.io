Title: Have prompt injections been solved at the model level?
Date: 2026-07-25
Tags: prompt-injection, systems

# Have prompt injections been solved?
### Earlence Fernandes

[Opus 5](https://www-cdn.anthropic.com/c5fbac3f0b1280a933ebd26d3cb8bb9f5bdeaf48/Claude%20Opus%205%20System%20Card.pdf#page=73) is claiming very low prompt injection success rates against human and adaptive automated testing (many times, its zero!). Are prompt injections mostly solved at the model level now and does this imply that systems-level guardrails are unnecessary given the utility and deployment costs they impose? (side note: Its fun to see Gray Swan mentioned so many times in the Anthropic post. My former student Xiaohan Fu works on the Shade tool over there, so I'm quite proud.)

## The failure modes of model-level and systems-level guardrails

You need both types of defensive layers because their failures are different. Let's examine those failure modes. A model-level defense fails when a prompt injection attacker finds an input that is out of distribution of the security training. The attacker solves a discrete search problem where the goal is to find an input that evades the alignment. There's a lot of work in this space to find such inputs. 


A systems-level defense fails in two different ways: the security policy has a gap, or the enforcement mechanism has an implementation bug. For the first failure mode, the prompt injection attacker must have knowledge of what policy is enforced and then determine whether there is any gap that will let them achieve their goals. If there is a gap, then they can search for a prompt injection to exploit the gap. For example, if we have a stateless policy system with a policy that says, ``the agent may only purchase stuff on amazon that is less than 50 dollars.'' The gap is that this policy allows multiple purchases individually less than 50 dollars but accumulate to much more than 50. The attacker can notice this and craft their prompt injection appropriately. However, if the policy system was stateful, then there is no prompt injection that would ever cause a violation. This illustrates an important point: a systems-level guardrail is dependent critically on policy expressivity and specification correctness. 

For the second failure mode, the attacker has to find and exploit an implementation bug in the systems-level defense. I do not know of a prompt injection attacker who can achieve this yet: hijack a capable model and give it the task of exploiting a vulnerability in a systems-level guardrail around it. It seems pretty complex because the attacker has to achieve prompt injection AND jailbreaking for a long-range task. Jailbreaking is necessary because most models will have some alignment that prevent the model from engaging in behavior like hacking a sandbox. But, there's another attacker here: the model itself engages in reward hacking to evade its system-level guardrails. This has happened, with the [OpenAI-HuggingFace incident](https://openai.com/index/hugging-face-model-evaluation-security-incident/). 


To summarize, we should aim to blend the model-level and systems-level guardrails in such a way that the attacker has to exploit two different failure modes to achieve full system compromise. A bunch of work needs to be done to investigate how to do this for realistic agents. I can think of one situation where this can work.

## Systems-level guardrails establish envelopes, model-level guardrails control fine-grained behavior

Consider the simple property of preventing `rm -rf /`. Model-level defenses train against this behavior, but a gap might exist. A systems-level approach watches for the filesystem operations that would destroy the tree and refuses. This shows where the systems-level guardrail shines: when you can clearly establish an envelope on what is okay and what is not, you get a guarantee that no prompt injection attacker can evade — *provided* the policy covers the action and the enforcement mechanism is correct. 

Why is this property of `rm -rf /` easy?  It is easy because it is fully-defined based on the action alone. Enforcing it does not involve considering context, provenance or information flow. Almost everything hard in agent security is context-dependent: `rm -rf` on a scratch directory is fine, sending an email is fine unless the body contains the contents of a private repo, an outbound request is fine unless it is exfiltration. Those policies quantify over where the data came from and what the task was, not over which syscall fired.

Within the systems-level envelope, we are still relying on model-level defenses to ensure the agent is generally aligned with the user's intent. Attacks are still possible, but attacks that violate the envelope are very difficult because it requires the attacker to find and exploit a hole in how the systems-level defense *implements* the policy. 

Can systems-level guardrails do better than being an envelope? How can we ensure that security policies are complete? These are big challenges. We organized a workshop at IEEE S&P earlier in 2026 to explore some of these challenges and [wrote about potential approaches](https://arxiv.org/abs/2605.18991). 

## Patching a systems-level guardrail is monotone, patching a model is not

If there is a gap discovered in a policy specification, you can patch it and that's it. The policy is monotonically improved. It won't regress if the attacker tries to rephrase their prompt injection. By contrast, once you discover a hole in model-level guardrails, you'd train the model to recognize that new pattern. This says nothing of variants of that pattern. There is no reason to believe there isn’t an attacker out there who can create attacks that are outside the distribution tested in the adversarial evaluation. Indeed, this has been the history of adversarial machine learning in the computer vision era and in the LLM era. 

Let's revisit the Amazon example above for a moment. The fix was to change the expressivity of the policy system. It wasn't a simple patch. Its a more complex patch, but it can be done, and once it's done, you get a monotonic improvement without a chance for regression under distributional shifts of the inputs to the agent. 

## You need systems-level guardrails even if there isn't a prompt injection

I am going to claim that even if there isn’t a prompt injection attacker, you will still benefit from systems-level guardrails. As evidence, let’s consider the incident where an OpenAI model escaped its sandbox and hacked HuggingFace to steal an answer to a benchmark it was being evaluated upon. The model has engaged in reward hacking here. What is the line of defense against this behavior? Alignment is one, and it keeps getting better, but again, alignment can and does fail in unpredictable ways. 

A systems-layer of defense would be valuable to have. But, in this particular incident, there was a systems-layer defense — the sandbox that the agent broke out of! Counter-intuitively, I think this is where systems-level techniques can shine. Particularly, formal verification of systems implementations has made progress in leaps and bounds — seL4, CompCert, Everest are formally verified implementations of complex software. We need a formally-verified sandboxing system. Although formal verification also relies on having complete specifications, notice that these specifications are  different than security policy specifications for agentic tasks. A sandbox specification is a fairly static thing — ensure that the sandboxed software has no way to take actions that violate a given policy. 

## Summary

Here is a table that organizes the various failure modes discussed above:

| Failure mode | Patching | Stronger attacker? | Formal verification helps? |
|---|---|---|---|
| Model defense: OOD input | Non-monotone (rephrasings regress it) | Yes | No — no formal spec exists |
| Systems: policy gap | Monotone for that gap | No, if the policy covers the action | No — spec *is* the problem |
| Systems: implementation bug | Monotone | **Yes** — bug-finding is search, and search scales with capability | **Yes** — this is the seL4 case |

So, here is what I think needs to happen:

1. We need to blend model-level and systems-level ideas, because they present different failure modes. An attacker has to solve two different kinds of problem, which raises the bar.
2. Systems-level defenses need to make progress on specifying policies correctly. That is row two, and it is the hardest of the three.
3. Systems-level enforcement mechanisms should be formally verified, and kept small enough that verifying them is possible. That is row three, and unlike row two, we know how to do it.

## So, have we solved prompt injections at the model-level?
I think this is an incorrect way of thinking about the problem. As the post explains, this issue is quite nuanced and many things need to come together to get security against prompt injections, and more broadly, misbehaving AI. 


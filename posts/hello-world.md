Title: Some Stuff We’ve Been Doing at UCSD
Date: 2025-08-31
Tags: ucsd, research, security, adversarial-ml, cycling

# Some Stuff We’ve Been Doing at UCSD

It has been little more than two years since I moved to sunny San Diego and restarted my research group (now with **Luoxi Meng, Nishit Pandya, Andrey Labunets and Xiaohan Fu**, and occasionally **Ashish Hooda** from UW Madison). I thought I’d write a short post discussing what we’ve been up to.  

This post describes the context and rationale for how and why we choose to do certain projects, and also discusses our contributions to computer security along the way.  

Also, everyone seems to be on LinkedIn?! Whatever happened to Twitter.  

---

## Cybersecurity and (Professional) Sports!

What do these two things have in common? This is a new area for me that merges my professional and personal interests. Some of you might know that I’m a cycling fanatic. When I say cycling, it’s not a 2-mile trip to the grocery store, but a **70-mile ride with 8,000 feet of climbing on dirt roads**! (This was roughly the stats for the Catalina trip above). That’s my idea of a fun ride.  

We had a paper at the new **USENIX WOOT** conference this year where we analyzed the **security of wireless gear shifting in bikes** used in professional races such as the Tour de France. We showed that an attacker can remotely manipulate the gear positions of a target rider.  

This can have real impact on race outcomes, especially in professional cycling, which has had a long and troubled history with integrity issues.  

This project was an excellent collaboration with wireless wizards **Aanjhan Ranganathan** and **Maryam Motallebi** from Northeastern.  

The project had more impact than I expected:  
- An excellent **WIRED article** by Andy Greenberg (among other press).  
- **Shimano implemented changes** to their software based on our recommendations.  
- Professional racers at the **Vuelta a España** actually raced with that software!  
- We briefed the **UCI** (governing body of pro cycling) on tech and wireless fraud.  

Out of all projects in recent years, this was the most fun. I even raced up a mountain against a colleague (an actual racer!) to record a demo video of the attack.  

**Backstory:** I had this project idea in 2019, but it didn’t take off until a conversation with **Yoshi Kohno** at USENIX Security 2023. I also met Aanjhan there and pitched it — the rest is history.  

---

## Least Privilege Authorization

*Or: how to ensure that “agents” like ChatGPT don’t read your email when they aren’t supposed to.*  

A long-running line of work in my group re-interprets the **principle of least privilege** for modern systems. It’s not the hottest thing in research, but it’s important work that makes computers secure — and has remarkable potential for today’s technologies.  

The principle (dating back to 1975) is hard to apply to modern large, complex, intelligent systems. We proposed new abstractions (a **client-supplied reference monitor** and **stateful policies**) for protocols like **OAuth**. This protects tokens from abuse by limiting their privilege while still retaining functionality.  

This was a paper at **USENIX Security** this year, supported by a **Google Research Scholar Award**.  

I also see this as fundamental to securing “AI agents.”  

For 10 years, adversarial ML research has mostly focused on model robustness in isolation — the “helpful and harmless” mindset. But our systems security experience tells us that’s not enough. You need **systems-level, end-to-end thinking** when securing ML, especially as models gain the ability to act on data and devices.  

---

## Adversarial Machine Learning for Problems That Matter

Since my first paper at the intersection of ML and computer security, I’ve looked for problems that actually matter. The first wave of adversarial ML (mid-2010s) often lacked real applications. Without systems, there’s nothing to secure beyond the model.  

Nearly a decade since Szegedy et al.’s adversarial examples, and 6 years since our “Stop Signs” paper, we finally have **realistic applications** where security matters:  
1. **LLM-based agents that use tools**.  
2. **Vision-based harmful imagery scanning systems**.  

### Imprompter Attacks

Our first result in this space: **adversarial prompts for LLM agents**.  

Example: you’re writing a job application letter and grab a “polisher” prompt from a marketplace like PromptBase. Without realizing it, you may be running an **adversarial example** that:  
- Extracts PII keywords from your session.  
- Formats them into a malicious URL (e.g., `velocity.show`).  
- Issues a hidden markdown image load to exfiltrate data.  

The user sees nothing (the image is a 1×1 transparent PNG). Data stolen.  

We call these **Imprompter Attacks** (thanks to Geoff Voelker & Stefan Savage).  

- Computed with an adapted GCG algorithm on an open-weights Mistral model.  
- Transferred to LeChat (Mistral-based).  
- Generalizes to unseen conversations, better than hand-written English prompts.  

Mistral rated it medium severity; their fix was disabling external image loads. This prevents exfiltration but also reduces agent functionality — an open research question.  

Impact: our adversarial prompt work **led to real security fixes** in production systems, plus a **WIRED article** by Matt Burgess.  

We also disclosed to **ChatGLM**, and are expanding to other chatbot agents.  

### NDSS Paper: Perceptual Hashing

Our second result: an **NDSS paper** on adversarial examples in **perceptual hashing**.  

We showed:  
- They cause **surveillance issues** in client-side scanning protocols (for harmful imagery).  
- A stronger perceptual hash = more robust surveillance of unrelated material.  

This raises important **privacy vs. safety trade-offs** for society.  

---

## Wrapping Up

My group wrote a few more papers in the past couple years, but I don’t have time to cover them all here.  

You can find them at: [earlence.com/publications](https://www.earlence.com/publications.html)  

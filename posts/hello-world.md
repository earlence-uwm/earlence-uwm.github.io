Title: Some Stuff We’ve Been Doing at UCSD
Date: 2024-10-01
Tags: ucsd, research, security, adversarial-ml, cycling

# Some Stuff We’ve Been Doing at UCSD

It has been little more than two years since I moved to sunny San Diego and restarted my research group (now with Luoxi Meng, Nishit Pandya, Andrey Labunets and Xiaohan Fu, and occasionally Ashish Hooda from UW Madison), so I thought that I'd write a short post discussing what we've been up to. The post describes the context and rationale for how and why we choose to do certain projects and also discusses the contributions to computer security we made along the way. Also, everyone seems to be on LinkedIn?! Whatever happened to Twitter. 

![Catalina Bikepacking, those are wireless shifters lol](../posts/images/oct-2024/catalina-bikepack.jpeg)

---

## Cybersecurity and (Professional) Sports!

What do these two things have in common? This is a new area for me that merges my professional and personal interests. Some of you might know that I’m a cycling fanatic. When I say cycling, its not a 2 mile trip to the grocery store, but a 70 mile ride with 8,000 feet of climbing on dirt roads! (this was roughly the stats for the Catalina trip above). That’s my idea of a fun ride. We had a paper at the new USENIX WOOT conference this year where we analyzed the security of wireless gear shifting in bikes used in professional races such as the Tour de France. We showed that an attacker can remotely manipulate the gear positions of a target rider. This can have real impact on race outcomes, especially in professional cycling that has had a long and troubled history with integrity issues. This project was an excellent collab. ft. wireless wizards Aanjhan Ranganathan and Maryam Motallebi from Northeastern. The project had more impact than I expected. In addition to an excellent article in [WIRED](https://www.wired.com/story/shimano-wireless-bicycle-shifter-jamming-replay-attacks/) by Andy Greenberg (among many [other](https://www.forbes.com/sites/daveywinder/2024/08/28/its-2024-and-now-bicycle-hackers-can-shift-your-gears/) excellent articles from different [venues](https://escapecollective.com/heres-the-real-lesson-from-that-wireless-shifting-hack/)), Shimano implemented changes to their software based on our recommendations and professional racers at the Vuelta a Espana (one of the grand tours in cycling!) actually raced with that software! We also briefed the UCI on tech and wireless fraud! (UCI is the governing body for the sport of professional cycling). Out of all the projects I’ve done in the recent years, this was the most fun! I even raced up a mountain against a colleague of mine (who is an actual bike racer) to record a [demo video](https://www.youtube.com/watch?v=7Pgd-EpLtDg&ab_channel=AanjhanRanganathan) of the attack in action!



---

## Least Privilege Authorization


A long running line of work in my group; the focus here is re-interpreting the principle of least privilege for modern systems. I know it is not the hottest thing on the research market right now, but hey, this is important stuff that actually makes computers work securely, and as I will explain shortly, it has remarkable potential to positively affect the security of some of the hot technologies out there.

As it turns out, a design principle from 1975 is a good idea in theory, but very difficult to actually apply to contemporary systems that are large, complex and intelligent (none of which existed or even predicted at the time this principle was introduced). In an attempt at a modern and more relevant interpretation of the principle, we introduced a new set of abstractions (a client-supplied reference monitor and stateful policies) for protocols like OAuth. This can protect tokens from abuse by limiting their privilege while also retaining functionality of the system. This was a paper at [USENIX security](https://www.usenix.org/conference/usenixsecurity24/presentation/cao-leo) this year and was supported by a research scholar award from [Google](https://research.google/programs-and-events/research-scholar-program/recipients/).

**This type of improvement is also fundamental to securing “AI agents” in my opinion. 

The past 10 years of research in adversarial machine learning has focused on improving the robustness of the model in isolation. This is akin to always writing software that is “helpful and harmless”. As computer security researchers, our experience tells us this is strategy does not work. You absolutely need systems-level and end-to-end thinking when it comes to dealing with the vulnerabilities of machine learning, especially now when people are building models that can actually take action on your data and devices (see the next part).


---

## Adversarial Machine Learning for Problems That Matter

Ever since my [first paper](https://arxiv.org/abs/1707.08945) at the intersection of machine learning and computer security, I’ve had an evolving interest in the space, in search of problems that actually matter. I think the first wave of adversarial machine learning didn’t really have real applications that we could use as a barometer for measuring security progress. A lot of this research happened in the context of computer vision, but at the time (early late 2016 - early 2017), there wasn’t much going on beyond some efforts around self-driving cars. In my opinion, this is also a reason why there hasn’t been much practice-relevant progress in securing ML systems — when there is no system, there isn’t anything to secure beyond the model itself. And as I explained earlier, securing the model itself isn't good enough if you want meaningful end-to-end security guarantees.  

Almost a decade since the Szegedy et al. paper on adversarial examples (and 6 years since our Stop signs paper), I think we’ve finally hit a point where there are realistic applications of machine learning. This is when and where computer security problems actually matter. It also provides a solid set of systems to work with and to reason about what security properties we actually want to build. A few examples of realistic systems are: (1) LLM-based agents that use tools; and (2) vision-based harmful imagery scanning systems. 

Our first result in this space concerns adversarial prompts for LLM Agents. Let's say that you are prepping a job application cover letter and you want to use an AI agent (such as ChatGPT or LeChat) to polish that letter. You go on a marketplace such a PromptBase to look for a prompt that will do the best job at polishing that letter. You find this:


![Example Imprompter Attack](../posts/images/oct-2024/imprompter.jpeg)

Without reading further, can you say what this prompt does? Do you think it polishes a job application letter? Would you try it in your chat session? As it turns out, this is an automatically computed adversarial example that will analyze your chat session, pull out PII keywords, format those keywords into a syntactically-valid URL whose fully-qualified domain points to an attacker's server (velocity.show in the example, can you pick it out?), and finally issue a markdown image load with that URL and then exfiltrate that PII. This is probably the first adversarial example for an LLM agent that misuses a tool (the markdown image load tool)! The user sees nothing (the loaded image is a 1x1 transparent png). And their data is stolen. Of course, now that I've told you what this prompt does, you can go back to look at it, and maybe you can decipher some of its behavior. Thanks to the naming powers of Geoff Voelker and Stefan Savage, we call these [Imprompter Attacks](https://imprompter.ai/), coz they  prompt agents to improperly use tools.

We computed these attacks on an open-weights Mistral model (using special-purpose objective functions and an adapted form of the GCG algorithm that was originally invented for jailbreaking, but we re-purpose for a prompt injection attack). It transferred in a blackbox manner to the LeChat agent from Mistral that uses some variant of the open-weights model. It generalizes to unseen conversations and performs better than the hand-written English language version! This attack style builds on existing work of the prompt injection community. In particular, these fall in the class of so-called "markdown exfiltration attacks." Prior work has used natural language malicious instructions, that can even be made invisible with unicode encoding tricks. However, chatbot agents have since started filtering out this stuff. Our work shows how an optimization-based attacker could compute a single obfuscated sequence of tokens that achieves a complex effect. 

We disclosed this attack to the Mistral security team and they rated it as a medium severity issue. Their fix was to disable external loading of images. Whether this is the best fix we can achieve is open to debate (and research!). On the one hand, it does fix the exfiltration issue, but it limits the agent's functionality -- the entire point of an LLM agent is to use tools! The team was credited in the Mistral release notes pictured below.

![Research team credited with reporting the vuln](../images/oct-2024/mistral.png)

The exciting thing for us as a research team was that this optimization-based adversarial prompt led to actual security fixes in a production AI Agent system! I am very excited about this impact, in addition to the excellent [WIRED](https://www.wired.com/story/ai-imprompter-malware-llm/) article by Matt Burgess.

(Emphasis on system here, coz that's what an agent is -- ML integrated with non-ML software so that you can actually do useful stuff).

We also disclosed this stuff to ChatGLM and it is likely we will be making disclosures to other companies soon depending on our progress in evaluating this attack vector on different chatbot agents.

Our second result on adversarial examples that matter appeared in an [NDSS](https://www.ndss-symposium.org/ndss-paper/experimental-analyses-of-the-physical-surveillance-risks-in-client-side-content-scanning/) paper this year. We showed how adversarial examples in perceptual hashing functions lead to surveillance issues in client-side scanning protocols (these protocols aim to curb the spread of harmful imagery exploiting children). We also experimentally uncovered a rather undesirable trade-off: a more robust perceptual hashing function leads to more robust surveillance of material that is not connected to harmful imagery. While I acknowledge that curbing harmful imagery about children needs some technical solutions (in addition to societal efforts), it is also important to balance that with the privacy of the communications of the general public.




---

## Wrapping Up

My group wrote a few more papers in the past couple years, but I don’t have time to cover them all here.  

You can find them at: [earlence.com/publications](https://www.earlence.com/publications.html)  

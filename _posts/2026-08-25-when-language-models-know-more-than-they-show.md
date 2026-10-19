---
layout: post
title: "When Language Models Know More Than They Show: Internal Representations and the Problem of Hallucination"
date: 2026-08-25
description: "A reflection on Orgad et al. (ICLR 2025), probing, and why model outputs may be a lossy measurement of the internal computation that produced them."
---

When a language model produces an incorrect answer, it is natural to assume that the relevant information was simply absent. In many cases that may be true, but it is not the only possible explanation. An incorrect output is ultimately an observation of model behavior, not a direct observation of the computation that produced it. A model might fail because the relevant information was never represented, because competing representations were resolved incorrectly, or because information that was available internally did not ultimately influence generation in the expected way.

This distinction is what makes Orgad et al.’s ICLR 2025 paper, *LLMs Know More Than They Show: On the Intrinsic Representation of LLM Hallucinations*, particularly interesting to me. The paper asks whether the hidden representations of language models contain information about answer correctness that is not fully reflected in their final outputs. Using linear probes across multiple models, layers, token positions, and tasks, the authors find substantial evidence that they do.

At one level, this is a paper about hallucination detection. At another, I think it raises a broader interpretability question. To what extent can observable behavior be treated as a faithful measurement of a model’s internal state? The results suggest that the relationship may be considerably less direct than output-based evaluation tends to assume.

## Locating information about correctness

Orgad et al. study Mistral-7B, Mistral-7B-Instruct, Llama-3-8B, and Llama-3-8B-Instruct across ten datasets spanning factual question answering, reasoning, mathematics, sentiment analysis, bias, and related tasks. Their primary method is linear probing. Hidden activations are extracted from different layers and token positions, and linear classifiers are trained to distinguish correct from incorrect responses.

The simplicity of this method is useful because it makes the interpretation relatively clear. If a linear probe can reliably distinguish correct from incorrect responses from an intermediate activation, then information relevant to correctness is decodable from that representation.

There is, however, an important distinction between decodability and causal use. A successful probe demonstrates that information is statistically accessible from the representation. It does not establish that the model itself relies on that feature when producing its answer. This is a familiar limitation of probing methods, but it matters especially here because the paper’s title naturally invites a stronger interpretation about what a model “knows.”

One of the authors’ most consistent findings is that correctness information is highly dependent on where the representation is measured. They pay particular attention to the “exact answer tokens,” the subset of tokens whose content directly determines whether an answer is correct. In a sentence such as “The capital of Connecticut is Hartford,” for example, the representation associated with “Hartford” is more directly tied to factual correctness than the representation associated with surrounding syntactic material.

Probes trained on these answer-bearing positions often perform substantially better than probes using more arbitrary locations such as the final token of the response. The strength of the signal also changes across layers.

I found this result important for reasons that extend beyond hallucination detection. Interpretability research often phrases questions at the level of whether a model “represents” a particular property, but representation is not necessarily spatially or temporally uniform. A property that is weakly decodable from one token or layer may be strongly expressed elsewhere.

This makes interpretability partly a problem of measurement validity. Before concluding that a feature is absent, researchers need to know whether they are observing the relevant part of the computation at the relevant time. In that sense, disagreements between probing studies may sometimes reflect differences in measurement location rather than substantive differences in what a model internally represents.

## A universal representation of truth?

The paper then examines a more ambitious possibility: whether truthfulness is represented by a common internal feature that generalizes across tasks.

The idea is attractive. If there were a relatively universal “truth direction,” a probe learned from factual questions might also detect correctness in reasoning, mathematics, or other domains. Such a representation would offer both theoretical simplicity and obvious practical value.

Orgad et al. test this through cross-dataset transfer. Some probes initially appear to generalize between tasks, suggesting that there may be shared information about correctness in representation space. However, the authors compare these results against information already available from output logits and find that much of the apparent cross-task advantage is reduced. In some cases, relationships between representation and correctness can even change direction across datasets.

Their conclusion is therefore more restrained. Truthfulness appears to be represented in a multifaceted and task-dependent manner rather than through a single universal latent direction.

To me, this is one of the paper’s most interesting findings, even though it is essentially a negative result.

Concepts such as truthfulness, confidence, uncertainty, and correctness appear semantically coherent from a human perspective. It is easy to assume that a neural network should therefore organize them into similarly coherent internal variables. But there is no reason this must be the case. A factual retrieval task and a mathematical reasoning task may both have objectively correct answers while depending on very different internal computations.

The result therefore raises a larger issue about the kinds of invariance interpretability research should expect to find. Perhaps the most useful invariant is not always a particular direction or coefficient that transfers unchanged across settings. It may instead be a reliable method for identifying which representation is relevant under a given computational structure.

That is a less elegant result, but potentially a more realistic one.

## From correctness prediction to behavioral structure

Another aspect of the paper that I found especially valuable is the move beyond binary correct-versus-incorrect labels.

The authors repeatedly sample responses to the same question and distinguish several recurring behavioral patterns. A model may answer correctly with high consistency, repeatedly produce the same incorrect answer, alternate between two competing answers, generate a wider distribution of responses, or refuse to answer.

These patterns contain information that ordinary accuracy metrics discard. Two models can both receive a score of zero on a question even if one confidently repeats the same wrong answer and another oscillates among several possibilities. Behaviorally, those are different situations, and they may reflect different internal conditions.

Orgad et al. show that hidden representations contain information that helps distinguish several of these error types. Linear probes can therefore predict not only whether a response is likely to be wrong but also something about the form that the failure is likely to take.

I think this changes the interpretation of the probing problem in an important way. Binary error detection asks whether the model is likely to fail. Distinguishing among failure types begins to resemble a much more general problem of behavioral state estimation.

This also makes the category of “hallucination” look somewhat coarse. What is grouped together at the output level may actually consist of multiple computational phenomena. Stable commitment to one incorrect answer, competition between alternatives, and diffuse instability are all failures by an accuracy criterion, but they need not have the same internal cause.

If interpretability is intended to explain model behavior rather than merely classify it, these distinctions should matter.

## When representation and behavior diverge

The experiments that most directly motivate the paper’s title examine situations in which a model strongly prefers an incorrect answer but occasionally generates the correct one.

In one of the paper’s examples, Mistral-7B-Instruct generates the same incorrect answer in 29 of 30 samples while producing the correct response only once. Despite this overwhelming behavioral preference for the wrong answer, the internal probe is able to distinguish the representation associated with the correct candidate.

This is a striking result, but I think its significance is easier to understand if the claim is stated somewhat more cautiously than the title suggests.

The result does not necessarily establish that the model “knows” the correct answer in the ordinary cognitive sense. What it shows is that information capable of distinguishing a correct candidate can be encoded in the model’s hidden representation even when the model’s generation process strongly favors an incorrect candidate.

That distinction does not make the finding weaker. In some ways, it makes the underlying problem more interesting.

If relevant information is internally available but the final behavior remains wrong, then the failure cannot automatically be attributed to a complete absence of information. There may instead be a divergence between representation and action.

Several mechanisms could produce such a divergence. Competing candidate representations may coexist, with one eventually dominating generation. Useful information may appear transiently and then be transformed at later layers. A representation may contain information correlated with correctness without playing any causal role in generation. Or the model’s later computation may simply fail to convert an informative internal state into the appropriate output.

The experiments in this paper cannot determine which of these explanations is correct. But they show why the distinction is empirically meaningful.

This is where I think the phrase “the model failed” becomes scientifically underspecified. Two models can produce exactly the same incorrect answer while differing substantially in whether the relevant information was represented internally and how that representation evolved before generation. Output-level evaluation collapses those cases together.

For evaluating task performance, that may be acceptable. For explaining the computation, it is not.

## From decodable information to computational state

The paper also left me thinking about what evidentiary standard should be required before we call a representation a meaningful internal state rather than simply a predictive correlate.

High probe accuracy is one useful criterion, but it seems insufficient on its own. A more convincing internal state variable would ideally have temporal precedence over the behavior it predicts, remain interpretable under controlled changes to the input, generalize beyond the examples used to identify it, and eventually show some evidence of causal relevance through intervention.

These criteria distinguish two different scientific claims.

One is that hidden activations contain information from which an external observer can predict behavior.

The stronger claim is that a particular internal variable participates in the computation that generates that behavior.

Orgad et al. primarily establish the first. They are appropriately cautious about the second. However, their results make the distinction between the two much clearer.

The temporal issue is especially interesting. Some of the strongest truthfulness signals in the paper occur around the exact answer tokens, when the response is already being instantiated. This is valuable for hallucination detection, but it leaves open the question of how early the relevant information becomes available.

There is a meaningful difference between detecting a failure as it emerges and identifying an internal state from which the failure can be predicted before the critical behavior occurs. The former is diagnostic. The latter begins to approach forecasting.

For practical monitoring and for mechanistic understanding, that difference could be important.

## Why I think the paper matters

I originally approached *LLMs Know More Than They Show* primarily as a paper about detecting hallucinations from internal activations. After reading it, I think its broader importance lies somewhere else.

The paper provides evidence that observable behavior may be a lossy measurement of internal computation.

A model’s final answer is produced after many transformations of internal representations. Information that appears at one point may not survive unchanged. Different representations may compete. Similar internal information may eventually produce different outputs, while superficially similar outputs may arise from different internal conditions.

This suggests that a complete science of model behavior will probably need to distinguish at least three things: what information is internally represented, what computation is performed over those representations, and what behavior is eventually produced.

Those variables are related, but Orgad et al. provide evidence that they should not be treated as interchangeable.

I also think the paper’s less tidy findings are among its most valuable. The absence of a clean universal truthfulness direction, the strong dependence on measurement location, and the gap between decodability and causal interpretation all make the scientific problem harder. But they also prevent us from adopting an overly simple picture of model internals.

One possibility is that interpretability research will gradually move away from asking only whether some human-interpretable concept can be decoded from an activation. A more demanding goal would be to identify internal variables that remain predictive under new conditions, determine what causes them to change, and test whether they have a causal relationship with downstream behavior.

That begins to resemble experimental system identification more than conventional representation probing.

Orgad et al. do not provide such a framework, nor do they claim to. But their results offer a strong motivation for pursuing one.

The title of the paper asks us to consider whether language models know more than they show. I think the more general lesson is slightly different. We should be careful not to treat what a model shows as a complete account of what is happening internally.

A wrong answer may be the end of an evaluation, but it does not have to be the end of the scientific explanation.

### Reference

Orgad, H., Toker, M., Gekhman, Z., Reichart, R., Szpektor, I., Kotek, H., & Belinkov, Y. (2025). *LLMs Know More Than They Show: On the Intrinsic Representation of LLM Hallucinations*. International Conference on Learning Representations (ICLR 2025).

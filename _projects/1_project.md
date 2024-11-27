---
layout: page
title: Distillation mitigates Hallucinations
description: On the effects of Knowledge Distillation on LLM Hallucinations
img: assets/img/work1_maxentropy.png
importance: 1
category: research
# related_publications: true
---

<h5><b>Motivation</b></h5>

In my senior year of masters, I conducted some analysis on the optimization aspect in traditional training of decoder transformers with One Hot Encoding (OHE) target distributions and found that: <b><i>by optimizing models against OHE targets of zero entropy, we train them to make assumptions</i></b> ([principle of Maximum Entropy](https://en.wikipedia.org/wiki/Principle_of_maximum_entropy#:~:text=To%20choose%20a%20distribution%20with%20lower%20entropy%20would%20be%20to%20assume%20information%20we%20do%20not%20possess) states that to choose a distribution with entropy lower than allowed by the provided information would be to assume information we do not possess).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_maxentropy.png" title="OHE targets cause hallucinations" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Given the context "The student did well in", the distribution of the next token should have high probability for subjects, sports, arts, etc. However, OHE targets force the model to choose one token with 100% probability to be "Physics", an information not provided in the context. Thus, the model learns to make assumptions and thus hallucinate.
</div>

I hypothesized that this leads to hallucination and that distribution-based Knowledge Distillation (KD), replacing OHE targets with teacher context-aware distributions, avoids assumptions and helps reduce hallucination.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_kdhelps.png" title="OHE targets cause hallucinations" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Replace OHE targets with context-aware distributions (non-zero probability for other possibilities, e.g. Maths, Arts, English) from a teacher model, the model learns to generate tokens that are more contextually relevant and less hallucinatory.
</div>

I began working with [senior PhD student Zihao He](https://zihaohe123.github.io/) at [SEA (Socially-Embedded AI) Lab](https://lermanlab.github.io/) led by Professor [Kristina Lerman](https://www.isi.edu/people/lerman/about) to study the effects of KD on LLM Hallucinations.

<h5><b>Experiments</b></h5>

To verify my hypothesis, I instruction finetuned 7B pretrained LLMs under two methods: traditional <b>Supervised Fine-Tuning (SFT)</b> as baselines and <b>KD</b> (with 13B-70B teachers), and then compare them using a robust hallucination evaluation pipeline. Advised by Cohere For AI researchers, I evaluated them with benchmarks of summarization tasks, using external metrics (rougeL, [factual consistency](https://vectara.com/blog/hhem-v2-a-new-and-improved-factual-consistency-scoring-model/) from Vectara) that compares predictions to ground truths and novel internal metrics (attention rates from [LookbackLens for hallucination detection](https://arxiv.org/abs/2407.07071)) that compares predictions to contexts.

<h5><b>Results</b></h5>

So far, we have seen decent improvements of KD models over SFT baselines across metrics and benchmarks. These results have consolidated the hypothesis, as well as the trustworthiness of mainstream metrics by its consistency with the novel internal metrics.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_rougel.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_attrates.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Boxplots of HPO experiments: SFT baseline and KD models with lambda_KD of 0.1, 1.0, 10.0 (KD is aggregate of lambda_KD of 0.1, 1.0, 10.0) for RougeL (left) and attention rates (right)
</div>

An interesting observation is that factual consistency (or support rate), while show similar patterns as RougeL and attention rates, gave very low scores for human labels. This suggests that factual consistency might not fully reflect hallucination ([as pointed out](https://github.com/vectara/hallucination-leaderboard#:~:text=Wouldn%27t%20an%20extractive%20summarizer%20model%20that%20just%20copies%20and%20pastes%20from%20the%20original%20summary%20score%20100%25%20(0%20hallucination)%20on%20this%20task) in their repository):

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_vectara.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    
</div>

Prior to this, we also got similar results for models of size 300M-1B.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_sLLM_QA.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Model (fine-tuned with Llama as the teacher on the PubMedQA dataset) Evaluation Results on Truthful QA and Hotpot QA for Q&A task
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_sLLM_summ.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Model (fine-tuned with Mistral as the teacher on the DialogSum dataset) Evaluation Results on CNN Dailymail for summarization task
</div>

As we progress, we want to verify the hypothesis with more model families like Llama3.1, Mistral, and Qwen.

<h5><b>Future works</b></h5>

There are many questions that I want to explore next:
- Does a model's tendency to generate inaccurate information from its internal knowledge (factual hallucination) lead to generation of inaccurate responses when given contexts (faithfulness hallucination)? Could addressing the former also mitigate the latter?
- Are acquired improvements on external metrics were merely due to better task fulfillment instead of hallucination-free reasoning? This is because these metrics only compare prediction text to ground truth text, while hallucination can be subtle and requires non-trivial reasoning to identify. In fact, we've already started tackling this question with internal metrics and proved the effectiveness of external metrics.
- Other factors causing hallucination: exposure bias, data imbalance, attend-to-all mechanism that distract models, etc. 
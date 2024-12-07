---
layout: page
title: Distillation mitigates Hallucinations
description: On the effects of Knowledge Distillation on LLM Hallucinations (ongoing)
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

I hypothesized that this leads to hallucination and that Sequence and Word-level Knowledge Distillation (KD) in [3], replacing OHE targets with teacher context-aware distributions, avoids assumptions and helps reduce hallucination:

$$
\mathcal{L} = (1-\alpha) \mathcal{L}_{NLL}(\theta) + \alpha \mathcal{L}_{KD}(\theta, \theta_T)
$$

,
where $$\mathcal{L}_{NLL}$$ is the negative log-likelihood loss, which is the Cross-entropy between student distributions with OHE targets, $$\mathcal{L}_{KD}$$ is the Cross-entropy between the teacher and student distributions, and $$\alpha$$ is the weight of the KD loss.

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

To verify my hypothesis, I instruction finetuned 7B pretrained LLMs under two methods: traditional <b>Supervised Fine-Tuning (SFT)</b> as baselines and <b>KD</b> (with 13B-70B teachers), and then compare them using a robust hallucination evaluation pipeline. Advised by Cohere For AI researchers, I evaluated them with benchmarks of summarization tasks, using external metrics (rougeL, [factual consistency](https://vectara.com/blog/hhem-v2-a-new-and-improved-factual-consistency-scoring-model/) from Vectara) and un-conventional internal metrics (lookup ratios from Lookback-Lens in [2]).

<h5><b>Results</b></h5>

Results from ablation experiments with search space $$lr \in \{1e-05, 5e-06\}$$ and $$bs \in \{4, 8\}$$ show that KD 7B models with $$\alpha=0.1$$ mostly outperform SFT 7B baselines for both ROUGE-L and lookup ratios across CNN-DM and XSUM benchmarks (except for lookup ratios for KD models on XSUM, which are on par with the baselines). Secondly, as $$\alpha$$ increases, KD models performance seems to drop, which sheds light on further exploration of KD with $$\alpha$$ values near $$0.1$$.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_rougel.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_attrates.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Boxplots of ROUGE-L (left) and lookup ratios (right) from ablation experiments (the higher the better): SFT 7B baselines (<i>sft</i>) and KD 7B models with <b>α</b>; of 0.1 (<i>kd01</i>), 1.0 (<i>kd1</i>), 10.0 (<i>kd10</i>); <i>KD</i> is the aggregate performance across all <b>α</b>; values. Boxplots are from entries of models trained with <b>lr ∈ {1e-05, 5e-06}</b> and <b>bs ∈ {4, 8}</b>.
</div>


As shown in the following table, factual consistency (or support rate) also demonstrates consistent improvement of KD models from SFT baselines. Interestingly, despite similar patterns as RougeL and lookup ratios, factual consistency gave very low scores for human labels. This suggests that factual consistency might not fully reflect hallucination ([as pointed out](https://github.com/vectara/hallucination-leaderboard#:~:text=Wouldn%27t%20an%20extractive%20summarizer%20model%20that%20just%20copies%20and%20pastes%20from%20the%20original%20summary%20score%20100%25%20(0%20hallucination)%20on%20this%20task) in their repository):



<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_vectara.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    
</div>
In addition to this, as shown in the above table, factual consistency of human lables for XSUM is significantly low in comparison with CNN-DM, despite being reported to have superior performance when evaluated on TRUE, SummaC Benchmark and AnyScale Ranking Test for Hallucinations. For both XSUM and CNN-DM are abstractive summarization benchmarks, this result shows that XSUM is more difficult, and also supports the claim and that factual consistency does not fully correlate with LLM hallucinations.

Prior to this, we also got similar results for models of size 300M-1B:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_sLLM_QA.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Model (fine-tuned with Llama as the teacher on the PubMedQA dataset). Evaluation Results on Truthful QA and Hotpot QA for Q&A task
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_res_sLLM_summ.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Model (fine-tuned with Mistral as the teacher on the DialogSum dataset). Evaluation Results on CNN Dailymail for summarization task
</div>

<h5><b>Conclusions</b></h5>

These results have consolidated my hypothesis, as well as the trustworthiness of mainstream metrics by its consistency with the novel internal metrics.

In abstractive summarization XSUM, while ROUGE-L indicates that KD models outperform SFT models, lookup ratios shows that KD models are just as good as the baselines. This can be accounted by two factors: either lookback ratios or ROUGE-L does not perform well estimating data under XSUM. However, lookback ratios was reported to generalize to XSUM even though it was trained from attention weights under CNN-DM. Thus, this shows that ROUGE-L may overestimate models' performance, giving false conclusions on their hallucination ability, and lookup ratios can uncover these subtle insights overlooked by ROUGE-L. This insight also highlights the importance of diverse evaluation pipeline.


As we progress, we want to verify the hypothesis with more model families like Llama3.1, Mistral, and Qwen.

<h5><b>Future works</b></h5>

There are many questions that I want to explore next:
- [1] has shown that models struggle to answer closed-book real-world questions even when they have learned the relevant details. This indicates that such behavior may be intrinsic to the models themselves. Thus, it is likely that even when provided with context, they will still hallucinate. Indeed, hallucination persists even in RAG settings. This raises a significant question: <b>is a model's tendency to generate inaccurate responses based solely on its own learned internal knowledge (factuality hallucination) the main reason it generates inaccurate responses when given external contexts (faithfulness hallucination)?</b> If so, addressing the issue of factuality hallucination might be the key to resolving the problem. These definitions have also revealed that our work has not focused on factuality hallucinations, which will be our next priority. In fact, insights on both types of hallucinations will shed light on the previously raised question.
- Are acquired improvements on external metrics were merely due to better task fulfillment instead of hallucination-free reasoning? This is because these metrics only compare prediction text to ground truth text, while hallucination can be subtle and requires non-trivial reasoning to identify. In fact, we've already started tackling this question with internal metrics and proved the effectiveness of external metrics.
- Other factors causing hallucination: exposure bias, data imbalance, attend-to-all mechanism that distract models, etc. 


<h5><b>References</b></h5>

[1] - Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong, Zhangyin Feng, Haotian Wang, Qianglong Chen, Weihua Peng, Xiaocheng Feng, Bing Qin, Ting Liu <a href="https://dl.acm.org/doi/abs/10.1145/3703155">A Survey on Hallucination in Large Language Models: Principles, Taxonomy, Challenges, and Open Questions</a>. <i>arXiv:2311.05232</i>, 2023.

[2] - Yung-Sung Chuang, Linlu Qiu, Cheng-Yu Hsieh, Ranjay Krishna, Yoon Kim, James Glass <a href="https://arxiv.org/abs/2407.07071">Lookback Lens: Detecting and Mitigating Contextual Hallucinations in Large Language Models Using Only Attention Maps</a>. <i>arXiv:2407.07071</i>, 2024.

[3] - Yoon Kim, Alexander M. Rush <a href="https://arxiv.org/abs/1606.07947">Sequence-Level Knowledge Distillation</a>. <i>arXiv:1606.07947</i>, 2016.

---
layout: page
title: Distillation mitigates Hallucinations
description: On the effects of Knowledge Distillation on LLM Hallucinations (ongoing)
img: assets/img/work1_maxentropy.png
importance: 1
category: research
# related_publications: true
---

<h3><b>Exciting news as of 02/2025</b></h3>

🎉 I'm excited to announce that my first-author research, "Smoothing Out Hallucinations: Mitigating LLM Hallucination with Smoothed Knowledge Distillation", is now available as a [preprint](https://arxiv.org/abs/2502.11306)!

🧐 In this work, we found that hard labels in cross entropy loss carry arbitrary assumptions, leading models to make assumptions when processing information and hallucinate. We used knowledge distillation (KD) to remedy this by using teacher-generated distributions as contextual soft labels. Our experiments showed decent improvements of KD models over SFT baselines in common hallucination benchmarks across internal and external metrics!

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_hallu_cnndm.png" title="External metrics performance of SFT and KD of Llama-2 and Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_hallu_cnndm.png" title="External metrics performance of SFT and KD of Llama-2 and Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Hallucination evaluation results for student models finetuned with supervised finetuning (SFT) and knowledge distillation (KD). Models are evaluated on the CNN/Daily Mail (CNNDM) and XSUM datasets using three metrics: ROUGE-L (↑, %) for n-gram overlap, factual consistency (↑, %) for context grounding, and factual rate (↑, %) for specialized hallucination detection.The results suggest that in most cases KD reduces hallucination compared to SFT, as models trained with soft labels from a teacher model demonstrate improved faithfulness.
</div>

I would like to express my deepest gratitude to Dr. [Zihao He](https://zihaohe123.github.io/) for his guidance so far throughout this research. I would also like to thank [USC ISI](https://isi.edu/) for providing the hardware infrastructure that made these experiments possible!


<h3><b>The journey</b></h3>

<h4><b>Motivation</b></h4>

In my senior year of masters, I conducted an analysis on the optimization aspect in traditional training of decoder transformers with One Hot Encoding (OHE) target distributions and found that: <b><i>by optimizing models against OHE targets of zero entropy, we train them to make assumptions</i></b> ([principle of Maximum Entropy](https://en.wikipedia.org/wiki/Principle_of_maximum_entropy#:~:text=To%20choose%20a%20distribution%20with%20lower%20entropy%20would%20be%20to%20assume%20information%20we%20do%20not%20possess) states that to choose a distribution with entropy lower than allowed by the provided information would be to assume information we do not possess).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_maxentropy.png" title="OHE targets cause hallucinations" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Given the context "The student did well in", the distribution of the next token should have high probability for subjects, sports, arts, etc. However, OHE targets force the model to choose one token with 100% probability to be "Physics", an information not provided in the context. Thus, the model learns to make assumptions and thus hallucinate.
</div>

I hypothesized that this leads to hallucination and that Sequence and Word-level Knowledge Distillation (KD) in [3], replacing ignorant OHE targets with teacher context-aware distributions, avoids assumptions and helps reduce hallucination:

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

I began working with Dr. [Zihao He](https://zihaohe123.github.io/) at [SEA (Socially-Embedded AI) Lab](https://lermanlab.github.io/) led by Professor [Kristina Lerman](https://www.isi.edu/people/lerman/about) to study the effects of KD on LLM Hallucinations.

<h4><b>Experiments</b></h4>

To verify my hypothesis, I instruction finetuned Llama-2-7B and Llama-3.1-8B pretrained LLMs under two methods: traditional <b>Supervised Fine-Tuning (SFT)</b> as baselines and <b>KD</b> (with 13B and 70B teachers, respectively), and then compare them using a robust hallucination evaluation pipeline. For KD, following the recipe in [3], I first ran SFT of teacher models, and then use these teachers to augment the training dataset with beam-search, which will then be used to train student models. Advised by Cohere For AI researchers, I evaluated models with benchmarks of summarization tasks, using external metrics (rougeL, [factual consistency](https://vectara.com/blog/hhem-v2-a-new-and-improved-factual-consistency-scoring-model/ $$\mathcal{fc}$$) from Vectara) and a non-conventional internal metric (factual rate $$\mathcal{fr}$$ from Lookback-Lens in [2]).

<h4><b>Results</b></h4>

Results from ablation experiments with search space $$lr \in \{1e-05, 5e-06\}$$ and $$bs \in \{4, 8\}$$ show that for both Llama-2 and Llama3.1, KD models with $$\alpha=0.1$$ consistently outperform SFT baselines for ROUGE-L on both CNN-DM and XSUM benchmark.
<!-- Secondly, as $$\alpha$$ increases, KD models performance seems to drop, which sheds light on further exploration of KD with $$\alpha$$ values near $$0.1$$. -->

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_hallu_cnndm.png" title="External metrics performance of SFT and KD of Llama-2 and Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_hallu_cnndm.png" title="External metrics performance of SFT and KD of Llama-2 and Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    RougeL on CNN-DM and XSUM hallucination benchmarks from ablation experiments of Llama-2 and Llama-3.1 (the higher the better): SFT baselines (<i>sft</i>) and KD models with <b>α</b>; of 0.1 (<i>kd0.1</i>), 1.0 (<i>kd1.0</i>), 10.0 (<i>kd10.0</i>), 0.002 (<i>kd0.002</i>), 0.05 (<i>kd0.05</i>). Entries are from models trained with <b>lr ∈ {1e-05, 5e-06}</b> and <b>bs ∈ {4, 8}</b>.
</div>

<!-- In addition to this, factual consistency of human lables for XSUM is significantly low in comparison with CNN-DM, despite being reported to have superior performance when evaluated on TRUE, SummaC Benchmark and AnyScale Ranking Test for Hallucinations. -->

<!-- As shown in the following table, factual support rate on Llama-2 also demonstrates improvement of KD models from SFT baselines on CNN-DM benchmark. -->

Results on the internal mertric of factual rate also shows improvements of KD over SFT Llama-2 on CNN-DM. XSUM, however, reflects no such improvements.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_llama_2_internal.png" title="example image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    
</div>

We also looked at the performance of pretrained models, and observed that finetuning, though did not necessarily yield better performance on downstream benchmarks, but under KD seems to yield reduced hallucination in comparison with SFT.

Below is the boxplots view of the results:
- Llama-2 SFT and KDs:
<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_external_metrics_rougeL.png" title="RougeL Llama-2" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_external_metrics_fs.png" title="Factual consistency Llama-2" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_internal_metrics_lookback_ratios.png" title="Factual rate Llama-2" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Boxplots of ROUGE-L (left), Factual consistency (middle), and Factual rate (right) from ablation experiments (the higher the better): SFT Llama-2-7B baselines (<i>sft</i>) and KD Llama-2-7B with <b>α</b>; of 0.1 (<i>kd01</i>), 1.0 (<i>kd1</i>), 10.0 (<i>kd10</i>); <i>KD</i> is the aggregate performance across all <b>α</b>; values. Boxplots are from entries of models trained with <b>lr ∈ {1e-05, 5e-06}</b> and <b>bs ∈ {4, 8}</b>.
</div>
- Llama-3.1 SFT and KDs:
<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_external_metrics_rougeL-llama3.1.png" title="RougeL Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work1_external_metrics_fs-llama3.1.png" title="Factual consistency Llama-3.1" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Boxplots of ROUGE-L (left) and Factual consistency (right) from ablation experiments (the higher the better): SFT Llama-3.1-8B baselines (<i>sft</i>) and KD Llama-3.1-8B with <b>α</b>; of 0.1 (<i>kd01</i>), 1.0 (<i>kd1</i>), 10.0 (<i>kd10</i>); <i>KD</i> is the aggregate performance across all <b>α</b>; values. Boxplots are from entries of models trained with <b>lr ∈ {1e-05, 5e-06}</b> and <b>bs ∈ {2, 4}</b>.
</div>


For both XSUM and CNN-DM are abstractive summarization benchmarks, these results show that:
- XSUM is more difficult than CNN-DM. Improvements on CNN-DM do not fully indicate superiority in avoiding hallucination.
- Degradation of Llama-3.1 on XSUM might be due to the degradation in its performance in general, as shown in its worse performance on all benchmarks comparing to the pretrained.
- Factual consistency does not fully correlate with LLM hallucinations (pointed out [here](https://github.com/vectara/hallucination-leaderboard#:~:text=Wouldn%27t%20an%20extractive%20summarizer%20model%20that%20just%20copies%20and%20pastes%20from%20the%20original%20summary%20score%20100%25)).

Prior to this, we also got similar improvements on hallucination benchmarking for models of size 300M-1B:

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

<h4><b>Conclusions</b></h4>

These results partly consolidated my hypothesis, as well as the trustworthiness of mainstream metrics by its consistency with the novel internal metrics.

In abstractive summarization XSUM, while ROUGE-L indicates that KD models outperform SFT models, factual rate shows that KD models are just as good as the baselines. This can be accounted by two factors: either factual rate or ROUGE-L does not perform well estimating data under XSUM. However, factual rate was reported to generalize to XSUM even though it was trained from attention weights under CNN-DM. Thus, this shows that ROUGE-L may overestimate models' performance, giving false conclusions on their hallucination ability, and factual rate can uncover these subtle insights overlooked by ROUGE-L. This insight also highlights the importance of diverse evaluation pipeline.


As we progress, we want to verify the hypothesis with more model families like Mistral, Gemma, and Qwen.

<h4><b>Future works</b></h4>

There are many questions that I want to explore next:
- [1] has shown that models struggle to answer closed-book real-world questions even when they have learned the relevant details. This indicates that such behavior may be intrinsic to the models themselves. Thus, it is likely that even when provided with context, they will still hallucinate. Indeed, hallucination persists even in RAG settings. This raises a significant question: <b>is a model's tendency to generate inaccurate responses based solely on its own learned internal knowledge (factuality hallucination) the main reason it generates inaccurate responses when given external contexts (faithfulness hallucination)?</b> If so, addressing the issue of factuality hallucination might be the key to resolving the problem. These definitions have also revealed that our work has not focused on factuality hallucinations, which will be our next priority. In fact, insights on both types of hallucinations will shed light on the previously raised question.
- Are acquired improvements on external metrics were merely due to better task fulfillment instead of hallucination-free reasoning? This is because these metrics only compare prediction text to ground truth text, while hallucination can be subtle and requires non-trivial reasoning to identify. In fact, we've already started tackling this question with internal metrics and proved the effectiveness of external metrics.
- Other factors causing hallucination: exposure bias, data imbalance, attend-to-all mechanism that distract models, etc. 


<h4><b>References</b></h4>

[1] - Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong, Zhangyin Feng, Haotian Wang, Qianglong Chen, Weihua Peng, Xiaocheng Feng, Bing Qin, Ting Liu <a href="https://dl.acm.org/doi/abs/10.1145/3703155">A Survey on Hallucination in Large Language Models: Principles, Taxonomy, Challenges, and Open Questions</a>. <i>arXiv:2311.05232</i>, 2023.

[2] - Yung-Sung Chuang, Linlu Qiu, Cheng-Yu Hsieh, Ranjay Krishna, Yoon Kim, James Glass <a href="https://arxiv.org/abs/2407.07071">Lookback Lens: Detecting and Mitigating Contextual Hallucinations in Large Language Models Using Only Attention Maps</a>. <i>arXiv:2407.07071</i>, 2024.

[3] - Yoon Kim, Alexander M. Rush <a href="https://arxiv.org/abs/1606.07947">Sequence-Level Knowledge Distillation</a>. <i>arXiv:1606.07947</i>, 2016.

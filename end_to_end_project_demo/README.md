# End to End AI Project Demo: Building, Evaluating and Iterating an AI system

## Video Walkthrough

The process of creating this project is captured in this video (also available on YouTube). And yes, the video was re-encoded with a command from the project before uploading!

[Set Up Your AI Project the Right Way: Evals, Synthetic Data and Fine Tuning](https://vimeo.com/1104945621)

## Creating a “Correctness” eval
- The most important feature of the system is that it generates the correct ffmpeg commands, so we started by building a correctness eval
- Finding a good judge: 
  - I manually labeled a golden dataset, so we could check if the LLM-as-Judge evals in Kiln actually aligned to human judgement
  - Iterated on judges, trying different prompts and models for the judge. I got a big jump in judge performance when I added the entire man page for ffmpeg to the judge prompt, and that ended up being the best judge.
  - Note: there’s probably some room to improve here. I’m really not sure my manual labels are correct. More eval data, and more iteration on the judge would be a good place to invest.

## Finding the best way to run your task

With a correctness eval in hand, it was easy to try a variety of prompts and models to see what works best. I tried about a dozen combinations, which you can see in the video and demo files.

- My one-shot prompt did better than the many-shot prompt from the WTFFmpeg project for most models, but their prompt was optimal for the Phi 4 5.6B
- GPT 4.1 (full, mini and nano) really dominated from the start. I would have expected more from Sonnet 4, but GPT 4.1 models just seem better at this task.

## Fine-Tuning Metrics

Here's a quick glance at the fine-tuning loss from the demo project. When you fine-tune your own project with Kiln, metrics are logged into your Weights and Biases account.

<img width="723" height="408" alt="Screenshot 2025-07-25 at 7 05 05 PM" src="https://github.com/user-attachments/assets/3e1dd557-a20d-4bf5-9884-f7d15f08908b" />

## Next Steps

If I wanted to really make a high-quality system, here’s where I’d focus next.

- **Improve the Correctness Judge**
  - I’m really not sure my manual labels are correct in the golden dataset (I’m not much of a ffmpeg wiz). More eval data (golden and eval_set), and more iteration on the judge would be a good place to invest.
- **More iteration trying to find the best method of running the task**
  - More iteration on thinking models
  - Try more prompts: try few-shot prompts, try adding some key knowledge to the prompt (for web videos include "-movflags +faststart", etc)
- **More Fine-Tuning: only if local or cost optimization was a goal**
  - The foundation models work pretty well. However, they are either hosted (GPT 4.1, Sonnet) or too big for the average person to run locally (Qwen 3 32B/Llama 70B). Our quick pass at fine-tuning has shown it can really improve the performance of smaller models (Llama 3B, Gemini Flash). If I wanted a system that could run locally or to really reduce price per token, fine-tuning would help.
  - Important: I probably wouldn’t invest in this if I was fine with hosted APIs; the large models seem to already have good knowledge about how to use ffmpeg. Iterating on prompting would probably be faster and more effective than fine-tuning. Fine-tuning can be great when cost/privacy are critical, but it's a lot of extra work and will slow you down.
  - It would be worthwhile to generate a bigger training set. I started with 330 examples, then added 719 more and could see a benefit. Generating a list of all ffmpeg features, then using that as top-level topics for synthetic data gen could generate a great training dataset quickly.
  - I'd try more base models like Gemma, Llama 3.1, Qwen. We've already seen that there's a wide quality gap between models (likely from their pre-training data). The right base model could really boost quality.
  - I'd try smaller models. I'd bet with the right training data, even a 1B parameter model could do well given how constrained this problem is.

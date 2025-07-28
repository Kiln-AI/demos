# AI Boilerplate: Evals, Synthetic Data Gen, Fine-Tuning, and Git

There are a ton of boilerplates out there for web development that integrate the tools you need to start a project (web framework, tests, CSS framework, auth, CI, etc). So why aren't there good boilerplates for AI projects with evals, synthetic data, routing, fine-tuning, etc?

I've been building a boilerplate for an AI project with the tools most projects need: evals, synthetic data, fine-tuning, collaboration, and more. They all work together smoothly, and you can set up a new project in a few minutes. It offers a systematic way to iterate on AI systems, moving past vibe-checks. It's called [Kiln](https://github.com/Kiln-AI/Kiln), and it's a free+open tool that runs locally. This walkthrough covers:

 - [Creating an eval](#creating-a-correctness-eval) including generating synthetic eval data, creating LLM-as-judge evals, and validating the eval with human ratings
 - [Finding the best way to run your task](#finding-the-best-way-to-run-your-task) by evaluating prompt/model pairs
 - [Fine-tuning models](#fine-tuning) including synthetic training data and evaluating tunes
 - [Iterating as project evolves](#iterating-as-project-evolves): new evals and prompts
 - [Set up collaboration with git & GitHub](#setting-up-git-collaboration)

## Video Walkthrough

<a href="https://www.youtube.com/watch?v=f1JU3wIxExE"><img width="404" height="263" alt="video screenshot" src="https://github.com/user-attachments/assets/d49a0830-3c69-49b4-bb9a-ced485c67daa" /></a>

## About the Project

 - [Kiln](https://github.com/Kiln-AI/Kiln) is the "boilerplate" which combines the tools most AI projects need in an easy to use package: evals, synthetic data, routing, and fine-tuning.
 - Some of the tools are built into Kiln (evals, synthetic data), and some are integrations with other libraries/services (fine-tuning via Fireworks/Together/OpenAI/Unsloth) and routing (LiteLLM, OpenRouter).
 - Kiln focuses on usability + integration. As an example, the synthetic data generation tool knows if you're generating data for an eval or fine-tuning, as those require very different data. The UI is designed to be easy to use.
 - Kiln is extensible: our open source python library lets you run or extend Kiln from code. You can add tools/data any way you want.
 - Kiln is local/private: it runs locally on your machine, and your project data is saved to local files. No one has access to your project files unless you share them. While designed for collaboration, you get to bring your own git server and keep things private.
 - Kiln is still growing: we're working on adding tools like RAG, agents and multi-modal models.

Learn more about Kiln on our [primary GitHub repo](https://github.com/Kiln-AI/Kiln) (this repo is for Kiln demo projects).

## Preview of Findings from this Demo Project

For the demo project we built a natural language to ffmpeg command builder. Inspired by other projects like [this one](https://github.com/scottvr/wtffmpeg). Some interesting findings were:

 - Adding the `man` page of ffmpeg to the LLM-as-judge model produced the most effective eval
 - Fine-tuning could boost performance significantly (21% over the base model), but it didn't end up being needed because...
 - GPT-4.1 outperformed all other models for this task, even Sonnet 4 and fine-tunes. Don't use LMArena-style evals to predict domain-specific performance.
 - Initial high eval scores were tempered by bugs, e.g. the model would happily delete your entire hard drive. We needed to iterate on a set of "product"/"domain" evals to make it into a better product.

## Walkthrough

To use this boilerplate:
 - To set up the same stack (evals, synth data, tuning), but for your own project/task: download the [Kiln app](https://getkiln.ai), follow along the steps below, creating your own task in step 1
 - To try the demo project from the video: download this repository, and import the `end_to_end_project_demo/project.kiln` file into the [Kiln app](https://getkiln.ai). 

### Creating our task

First we set up the task, by defining a title, initial prompt, and the input/output schema.


- [Video walkthrough of setting up task: 1m01s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=1m01s)

### Creating a “Correctness” eval

The most important feature of the system is that it generates the correct results, so we started by building a correctness eval. This process involved:

- Creating an eval with a name and prompt
- Generating synthetic eval data. 80% for the eval set, and 20% for a golden eval set.
- I manually labeled the golden dataset, so we could check if the LLM-as-Judge evals in Kiln actually aligned with human judgement. This is used to calculate the correlation between LLM-as-judge systems and human evaluators using a range of correlation scores (Kendall Tau, Spearman, MSE)
- I iterated on judges, trying different prompts and models for the judge until I found one that worked well. I got a big jump in judge performance when I added the entire man page for ffmpeg to the judge prompt, and that ended up being the best judge.

See details:
 - [Video walkthrough of eval creation: 2m32s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=2m32s)
 - [Docs and detailed demo of evals](https://docs.getkiln.ai/docs/evaluations)
 - [Ideas for next steps](#improve-the-existing-eval)

### Finding the best way to run your task

With a correctness eval in hand, it was easy to try a variety of prompts and models to see what works best. I tried about a dozen combinations, which you can see in the video and demo files.

Some insights from the experimentation:

- My one-shot prompt did better than the many-shot prompt from the WTFFmpeg project for most models, but their prompt was optimal for the Phi-3 5.6B
- GPT-4.1 (full, mini and nano) really dominated from the start. I would have expected more from Sonnet 4, but GPT-4.1 models just seem better at this task.

Details:
 - [Video walkthrough of finding the best prompt + model: 6m20s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=6m20s)
 - [Docs for comparing run methods](https://docs.getkiln.ai/docs/evaluations#comparing-run-methods)
 - [Ideas for next steps](#iteration-on-modelprompt)

### Fine-Tuning

Next we tried fine-tuning 13 models for this task. This involved building a training data set using synthetic data, then dispatching fine-tune jobs. We tried:

 - Various base models: Llama, Qwen, Gemini, GPT-4.1
 - Various fine-tuning providers: Fireworks, Together, OpenAI, Google Vertex. Note: we didn't do local, but that's also possible.
 - A range of parameters: we varied epochs, lora-rank, learning rate.

There were some promising results in the batch (16% and 21% gains in evals over the base model), but I'd experiment more before picking a winner.

Here's a quick glance at the fine-tuning loss from the demo project. When you fine-tune your own project with Kiln, metrics are logged into your Weights and Biases account:

<img width="362" height="204" alt="train/loss of fine-tuning runs" src="https://github.com/user-attachments/assets/3e1dd557-a20d-4bf5-9884-f7d15f08908b" />

Details:
 - [Video walkthrough of fine-tuning: 11m31s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=11m31s)
 - [Docs on fine-tuning](https://docs.getkiln.ai/docs/fine-tuning-guide)
 - [Ideas for next steps](#more-fine-tuning-only-if-local-or-cost-optimization-was-the-goal)

### Iterating as Project Evolves

Next we walk through some examples of iterating on the project:

 - A case of fixing a bug we discovered in the system: the model will delete every file on your hard drive if you ask it to. We create a new eval ensuring the generated commands are not destructive, and then iterate on prompts to prevent this from happening.
 - Using evals to find tradeoffs between concerns. In this demo we found instruction following went down as we got more prescriptive about things like destructive edits. This makes sense, but we'd have to confirm we want that tradeoff.
 - Adding more evals representing product goals: don't recommend competitors, technical preferences (prefer mp4 container), etc

Details:
 - [Video walkthrough of iterating after v1: 14m9s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=14m9s)
 - [Blog Post: Many Small Evals Beat One Big Eval, Every Time](https://getkiln.ai/blog/you_need_many_small_evals_for_ai_products#setup-team-evals)
 - [Ideas for next steps](#iterate-on-product-side-with-evalsprompts)

### Setting Up Git Collaboration

Next we sync the project to GitHub, allowing a team to collaborate with familiar synchronization tools and git history.

Details:
 - [Video walkthrough of Github/collaboration: 17m10s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=17m10s)
 - [Collaboration Docs](https://docs.getkiln.ai/docs/collaboration)

### Python Library

We only used the Kiln app for this project, but in the video we briefly introduce the [Kiln python library](https://pypi.org/project/kiln-ai/) which you can use to work with Kiln project in code.

### Recap, Docs and Discord

 - [Walkthrough video of the recap: 18m24s](https://www.youtube.com/watch?v=f1JU3wIxExE&t=18m24s)
 - [Kiln AI Docs](https://docs.getkiln.ai)
 - [Kiln AI Discord](https://getkiln.ai/discord)

## Next Steps

This was a quick demo of all the pieces working together. If I wanted to continue to iterate on quality, here’s where I’d focus next:

### Improve the Existing Eval

I’m really not sure my manual labels are correct in the golden dataset (I’m not much of a ffmpeg wizard as I thought). More eval data (golden and eval_set), and more iteration on the judge prompt, and better labels from an actual ffmpeg expert would be a good place to invest.

### Iteration on model+prompt

I'd iterate more trying to find the best method of running the task, including:

- Try more thinking models, and chain-of-thought on non-thinking models. Building command line parameters seems ideal for step-by-step thinking.
- Try more prompts: try few-shot prompts, try adding some key knowledge to the prompt (for web videos include "-movflags +faststart", "prefer mp4 container if not specified", etc)

### Iterate on product side with evals/prompts

Each model has its own style of output; they swap the order of the explanation/command and the formatting. I'd want to define the best version, and write some evals to confirm they follow it. 

### More Fine-Tuning: only if local or cost optimization was the goal

I probably wouldn't iterate on fine-tuning unless I really wanted a system that could run locally or to really reduce price per token.

Why? The foundation models work pretty damn well. They seem to already have good knowledge about how to use ffmpeg. Iterating on prompting would probably be faster and more effective than fine-tuning. Fine-tuning can be great when cost/privacy are critical, but it's a lot of extra work and will slow your project down.

If I wanted a system that could run locally or to really reduce price per token, fine-tuning would help. The high quality models either closed/hosted (GPT 4.1, Sonnet) or too big for the average person to run locally (Qwen 3 32B/Llama 70B). 

Our quick pass at fine-tuning has shown it can really improve the performance of smaller models; in particular Llama 3B and Gemini Flash saw big jumps. To iterate I'd try:

- Generate a larger training set. I started with 330 examples, then added 719 more and could see a benefit. To do this I'd probably manually build a list of all ffmpeg features and use that as top-level topics for synthetic data generation that would let me systematically ensure every feature was contained in the training dataset. I'd also look around for existing examples to include.
- I'd try more base models like Gemma, Llama 3.1, Qwen. We've already seen that there's a wide quality gap between models (likely from their pre-training data). The right base model could boost quality, although as the training set got better I'd imagine the gap closes.
- I'd try smaller models. I'd bet with the right training data, even a 1B parameter model could do well given how constrained this problem is.

## Using the Project

This was built as a demo project, not a real product. However I've actually found it handy. I used it to re-encode the video walkthrough before uploading it to Vimeo, and it did things I wouldn't have thought of like using a lanczos scaler to keep sharper text (since I told it that the content was a screencast). You can see the [commit here](https://github.com/Kiln-AI/demos/commit/a26492e3607fb629c04121758ab8a4981164b093). Kinda cool!

If any ffmpeg wizard wants to take a stab at improving the performance with new evals/prompts/tunes, I'd be happy to take PRs.

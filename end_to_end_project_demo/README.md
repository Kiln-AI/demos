# AI Project Boilerplate with Evals, Synthetic Data Gen, Fine-Tuning, and Git

This is a boilerplate/demo project where I setup an AI project with the a full stack of integrated tools for evals, syntehtic data, fine-tuning, and collaboration. We're using [Kiln](https://getkiln.ai), a free+open tool that runs completley locally. We cover:

 - [Creating an eval](#creating-a-correctness-eval) including generating synthetic eval data, creating LLM-as-judge evals, and validating the eval with human ratings
 - [Finding the best way to run your task](#finding-the-best-way-to-run-your-task) by evaluating prompt/model pairs
 - [Fine-tuning models](#fine-tuning) including synthetic training data and evaluating models
 - Iterating as project evolves: new evals and prompts
 - Set up collaboration with git/GitHub sync

To use this boilerplate:
 - To try this demo project: download this repository, and import the `end_to_end_project_demo/project.kiln` file into the [Kiln app](https://getkiln.ai). 
 - To set up this stack for your own project/task: download the [Kiln tool](https://getkiln.ai) and follow along the guide steps below. 

## Video Walkthrough

The process of creating this project is captured in this video:

[Set Up Your AI Project the Right Way: Evals, Synthetic Data and Fine Tuning](https://vimeo.com/1104945621)

Yes, the video was re-encoded with a command from the project before uploading!

## Walkthrough

### Creating our task

First we setup the task, by defining a title, initial prompt, and the input/output schema.

For the demo project we build a natural language to ffmpeg command builder. Inspired by other projects like [this one](https://github.com/scottvr/wtffmpeg).

- [Video walkthrough of setting up task: 1m32s](todo)

### Creating a “Correctness” eval

The most important feature of the system is that it generates the correct results, so we started by building a correctness eval. This process involved:

- Creating a eval with a name and prompt
- Generating synthetic eval data. 80% of the eval, and 20% for a golden eval set.
- I manually labeled the golden dataset, so we could check if the LLM-as-Judge evals in Kiln actually aligned to human judgement.
- Iterated on judges, trying different prompts and models for the judge until I found one that worked well. I got a big jump in judge performance when I added the entire man page for ffmpeg to the judge prompt, and that ended up being the best judge.

See details:
 - [Video walkthrough of eval creation: 2m32s](todo)
 - [Docs and detailed demo of evals](https://docs.getkiln.ai/docs/evaluations)
 - [Ideas for next steps](#improve-the-exiting-eval)

### Finding the best way to run your task

With a correctness eval in hand, it was easy to try a variety of prompts and models to see what works best. I tried about a dozen combinations, which you can see in the video and demo files.

Some insights from the experimentation:

- My one-shot prompt did better than the many-shot prompt from the WTFFmpeg project for most models, but their prompt was optimal for the Phi 4 5.6B
- GPT 4.1 (full, mini and nano) really dominated from the start. I would have expected more from Sonnet 4, but GPT 4.1 models just seem better at this task.

Details:
 - [Video walkthrough of finding the best prompt + model: 6m20s]()
 - [Docs for comparing run methods](https://docs.getkiln.ai/docs/evaluations#comparing-run-methods)
 - [Ideas for next steps](#iteration-on-modelprompt)

### Fine-Tuning

Next we tried fine-tuning 13 models for this task. This involved building a training data set using synthetic data, then dispatching fine-tune jobs. We tried:

 - Various base models: Llama, Qwen, Gemini, GPT 4.1 
 - Various fine-tuning providers: Fireworks, Together, OpenAI, Google Vertex. Note: we didn't do local, but that's also possible.
 - A range of parameters: we varried epochs, lora-rank, learning rate.

There were some promising results in the batch (15%+ gains in evals over the base model), but I'd experiment more before picking a winner.

Here's a quick glance at the fine-tuning loss from the demo project. When you fine-tune your own project with Kiln, metrics are logged into your Weights and Biases account:

<img width="362" height="204" alt="train/loss of fine-tuning runs" src="https://github.com/user-attachments/assets/3e1dd557-a20d-4bf5-9884-f7d15f08908b" />

Details:
 - [Video walkthrough of fine-tuning: 11m31s](todo)
 - [Docs on fine-tuning](https://docs.getkiln.ai/docs/fine-tuning-guide)
 - [Ideas for next steps](#more-fine-tuning-only-if-local-or-cost-optimization-was-the-goal)

### Iterating as Project Evolves

Next we walk through some examples of iterating on the project:

 - A case of fixing a bug we discovered in the system: the model will delete every file on your hard drive if you ask it to. We create a new eval ensuring the generated commands are not destructive, and then iterate on prompts to prevent this from happening.
 - Using evals to find tradeoffs between concerns: instruction following going down when we're prescriptive about things like desctivtive edits
 - Adding more evals representing product goals: don't recommend competitors, technical preferences (prefer mp4 container), etc

Details:
 - [Video walkthrough of iterating after v1: 14m9s](todo)
 - [Blog Post: Many Small Evals Beat One Big Eval, Every Time](https://getkiln.ai/blog/you_need_many_small_evals_for_ai_products#setup-team-evals)
 - [Ideas for next steps](#iterate-on-product-side-with-evalsprompts)

### Setting Up Git Collaboration

Next we sync the project to GitHub, allowing a team to collaborate with familiar syncronization tools and git history.

Details:
 - [Video walkthrough of Github/collaboration: 17m10s](todo)
 - [Collaboration Docs](https://docs.getkiln.ai/docs/collaboration)

### Python Libary

We only used the Kiln app for this project, but in the video we briefly introduce the [Kiln python library](https://pypi.org/project/kiln-ai/) which you can use to work with Kiln project in code.


### Recap, Docs and Discord

 - [Walkthrough video of the recap: 18m23s](todo)
 - [Kiln AI Docs](https://docs.getkiln.ai)
 - [Kiln AI Discord](https://getkiln.ai/discord)

## Next Steps

This was a quick demo of all the pieces working together. If I wanted to continue to iterate on quality, here’s where I’d focus next:

### Improve the Exiting Eval

I’m really not sure my manual labels are correct in the golden dataset (I’m not much of a ffmpeg wiz as I thought). More eval data (golden and eval_set), and more iteration on the judge prompt, and better labels from an actual ffmpeg expert would be a good place to invest.

### Iteration on model+prompt

I'd iterate more trying to find the best method of running the task, including:

- Try more thinking models, and chain-of-thought on non-thinking models. Building command line parameters seems ideal for step-by-step thinking.
- Try more prompts: try few-shot prompts, try adding some key knowledge to the prompt (for web videos include "-movflags +faststart", "prefer mp4 container if not specified", etc)

### Iterate on product side with evals/prompts

Each model has it's own style of output; they swap the order of the explanation/command and the formatting. I'd want to define the best version, and write some evals to confirm they follow it. 

### More Fine-Tuning: only if local or cost optimization was the goal

I probably wouldn't iterate on fine-tuning unless I really wanted a system that could run locally or to really reduce price per token.

Why? The foundation models work pretty damn well. They seem to already have good knowledge about how to use ffmpeg. Iterating on prompting would probably be faster and more effective than fine-tuning. Fine-tuning can be great when cost/privacy are critical, but it's a lot of extra work and will slow your project down.

If I wanted a system that could run locally or to really reduce price per token, fine-tuning would help. The high quality models either closed/hosted (GPT 4.1, Sonnet) or too big for the average person to run locally (Qwen 3 32B/Llama 70B). 

Our quick pass at fine-tuning has shown it can really improve the performance of smaller models; in particular Llama 3B and Gemini Flash saw big jumps. To iterate I'd try:

- Generate a larger training set. I started with 330 examples, then added 719 more and could see a benefit. To do this I'd probably manually build a list of all ffmpeg features and use that as top-level topics for synthetic data genl that would let me systematically ensure every feature was contained in the training dataset. I'd also look around for existing examples to include.
- I'd try more base models like Gemma, Llama 3.1, Qwen. We've already seen that there's a wide quality gap between models (likely from their pre-training data). The right base model could boost quality, although as the training set got better I'd imagine the gap closes.
- I'd try smaller models. I'd bet with the right training data, even a 1B parameter model could do well given how constrained this problem is.



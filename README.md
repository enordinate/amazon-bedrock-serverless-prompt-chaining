## Amazon Bedrock Serverless Prompt Chaining

This repository provides examples of using [AWS Step Functions](https://aws.amazon.com/step-functions/)
and [Amazon Bedrock](https://aws.amazon.com/bedrock/) to build complex, serverless, and highly scalable
generative AI applications with prompt chaining.

[Prompt chaining](https://docs.anthropic.com/claude/docs/prompt-chaining) is a technique for
building complex generative AI applications and accomplishing complex tasks with large language models (LLMs).
With prompt chaining, you construct a set of smaller subtasks as individual prompts. Together, these subtasks
make up your overall complex task that you would like the LLM to complete for your application.
To accomplish the overall task, your application feeds each subtask prompt to the LLM in a pre-defined order or
according to a set of defined rules.

For applications using prompt chaining, Step Functions can orchestrate complex chains of prompts and invoke foundation models in Bedrock.
Beyond simple ordered chains of prompts, Step Functions workflows can contain loops, map jobs, parallel jobs,
conditions, and input/output manipulation. Workflows can also chain together steps that invoke a foundation model in Bedrock,
steps that invoke custom code in AWS Lambda functions, and steps that interact with over 220 AWS services.
Both Bedrock and Step Functions are serverless, so you don't have to manage any infrastructure to deploy and scale up your application.

<!-- toc -->

1. [Prompt chaining examples](#prompt-chaining-examples)
    1. [Write a blog post](#write-a-blog-post)
    1. [Write a story](#write-a-story)
    1. [Plan a trip](#plan-a-trip)
    1. [Pitch a movie idea](#pitch-a-movie-idea)
    1. [Plan a meal](#plan-a-meal)
1. [Deploy the examples](#deploy-the-examples)
1. [Security](#security)
1. [License](#license)
<!-- tocstop -->

## Prompt chaining examples

This repository contains several working examples of using the prompt chaining techniques described above,
as part of a demo generative AI application. The [Streamlit-based](https://streamlit.io/) demo application
executes each example's Step Functions state machine and displays the results,
including the content generated by foundation models in Bedrock.
The Step Functions state machines are defined using [AWS CDK](https://aws.amazon.com/cdk/) in Python.

### Write a blog post

This example generates an analysis of a given novel for a literature blog.

![Screenshot](/docs/screenshots/blog_post.png)

This task is broken down into multiple subtasks to first generate individual paragraphs
focused on specific areas of analysis, then a final subtask to pull together the generated
content into a single blog post. The workflow is a simple, sequential chain of prompts.
The previous prompts and LLM responses are carried forward as context and included in
the next prompt as context for the next step in the chain.

![Visualization of the blog post workflow](/webapp/pages/workflow_images/blog_post.png)

CDK code: [stacks/blog_post_stack.py](stacks/blog_post_stack.py)

### Write a story

This example generates a short story about a given topic.

![Screenshot](/docs/screenshots/story_writer.png)

This task is broken down into multiple subtasks to first generate a list of characters for the story,
generate each character's arc for the story, and then finally generate the short story using
the character descriptions and arcs. This example illustrates using a loop in a Step Functions
state machine to process a list generated by a foundation model in Bedrock, in this case to process
the generated list of characters.

![Visualization of the blog post workflow](/webapp/pages/workflow_images/story_writer.png)

CDK code: [stacks/story_writer_stack.py](stacks/story_writer_stack.py)

### Plan a trip

This example generates an itinerary for a weekend vacation to a given destination.

![Screenshot](/docs/screenshots/trip_planner.png)

![Screenshot](/docs/screenshots/trip_planner_itinerary.png)

This task is broken down into multiple subtasks, which first generate suggestions for hotels,
activities, and restaurants and then combines that content into a single daily itinerary.
This example illustrates the ability to parallelize multiple distinct prompts in a Step Functions
state machine, in this case to generate the hotel, activity, and restaurant recommendations in
parallel.
This example also illustrates the ability to chain together prompts and custom code.
The final step in the state machine is a Lambda function that creates a PDF of the itinerary
and uploads it to S3, with no generative AI interactions.

![Visualization of the blog post workflow](/webapp/pages/workflow_images/trip_planner.png)

CDK code: [stacks/trip_planner_stack.py](stacks/trip_planner_stack.py)

### Pitch a movie idea

This example acts as an AI screenwriter pitching movie ideas to the human user acting as a movie producer.
The movie producer can greenlight the AI's movie idea to get a longer one-page pitch for the idea,
or they can reject the movie idea and the AI will generate a new idea to pitch.

![Screenshot](/docs/screenshots/movie_pitch.png)

![Screenshot](/docs/screenshots/movie_pitch_one_pager.png)

This task is broken down into multiple subtasks: first, the prompt for generating a movie idea is invoked
multiple times in parallel with three different
[temperature settings](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html#text-playground-adjust-random)
to generate three possible ideas. The next prompt in the chain chooses the best idea to pitch to the movie producer.
The chain pauses while waiting for human input from the movie producer, either "greenlight" or "pass".
If the movie producer greenlights the idea, the final prompt in the chain will generate a longer pitch using
the chosen movie idea as context. If not, the chain loops back to the beginning of the chain, and generates three new ideas.

This example illustrates the ability to parallelize the same prompt with different inference parameters,
to have multiple possible paths in the chain based on conditions, and to backtrack to a previous step in the chain.
This example also illustrates the ability to require human user input as part of the workflow, using a Step Functions
[task token](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html#connect-wait-token)
to wait for a callback containing the human user's decision.

![Visualization of the blog post workflow](/webapp/pages/workflow_images/movie_pitch.png)

CDK code: [stacks/movie_pitch_stack.py](stacks/movie_pitch_stack.py)

### Plan a meal

This example generates a recipe for the user, based on a few given ingredients they have on hand.

![Screenshot](/docs/screenshots/meal_planner.png)

This task is broken down into multiple subtasks: first, two possible meal ideas are generated in parallel, with
the two prompts acting as two different "AI chefs". The next prompt in the chain scores the two meal suggestions
on a scale of 0 to 100 for "tastiness" of each described meal.
The AI chefs receive the scores for both meal suggestions, and attempt to improve their own score in comparison
to the other chef by making another meal suggestion.
Another prompt determines whether the chefs have reached a consensus and suggested the same meal. If not,
the chain loops back to generate new scores and new meal suggestions from the chefs.
A small Lambda function with no LLM interaction chooses the highest-scoring meal,
and the final prompt in the chain generates a recipe for the winning meal.

This example illustrates how prompt chains can incorporate two distinct AI conversations, and have
two AI personas engage in a debate with each other to improve the final outcome.
This example also illustrates the ability to chain together prompts and custom code, in this case
choosing the highest meal score.

![Visualization of the blog post workflow](/webapp/pages/workflow_images/meal_planner.png)

CDK code: [stacks/meal_planner_stack.py](stacks/meal_planner_stack.py)

## Deploy the examples

See the [development guide](DEVELOP.md) for instructions on how to deploy the demo application.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
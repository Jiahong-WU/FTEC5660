# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
> 
I developed a LangChain‑based processing pipeline to extract structured information from supermarket receipt images. In the build_chain() function, I instantiate the specified multimodal foundation model deepseek‑v4‑flash‑vision‑exp. Each receipt image is encoded as a base64 data URL and wrapped into a multimodal HumanMessage, paired with a task‑specific prompt that constrains the model to output structured JSON with three target numeric fields: final_payment, subtotal, and total_discount. For better robustness against the unstructured text artifacts that vision‑language models often produce, I add a regular expression filtering step to isolate valid JSON payloads from the model’s raw output.
For the answer_queries() function, I use the built‑in chain.batch() method to run batched, parallel inference over the full set of receipt samples. To avoid floating‑point errors when summing monetary values, I perform all arithmetic operations using Python’s Decimal type. The aggregation step calculates two key metrics: summing all final_payment values gives the total actual expenditure, while adding subtotal and total_discount together reconstructs the total pre‑discount amount. I format all final outputs as HK$XX.XX strings, so each result contains exactly one monetary value and meets the assignment’s formatting rules.
One key practical limitation comes from the inherent uncertainty of multimodal models: even with carefully designed prompts, visual misrecognition of printed values on receipts can still introduce errors into the final aggregation results.
```mermaid
flowchart LR
    A[Input list of receipt image Path objects] --> B[RunnableLambda prepare_multimodal_input<br/>Convert image to data‑url multimodal HumanMessage]
    B --> C[ChatDeepSeek deepseek‑v4‑flash‑vision‑exp vision LLM]
    C --> D[RunnableLambda parse_json_output<br/>Regex extract JSON string from LLM output]
    D --> E[answer_queries: chain.batch parallel inference for all receipts]
    E --> F[Aggregate: sum final_payment; sum subtotal + total_discount with Decimal]
    F --> G[Return dict with two queries mapped to HK$XX.XX formatted strings]
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
> to students: please fill your solution description here.
This solution builds an extraction chain using the DeepSeek multimodal vision model to parse structured numerical fields from supermarket receipt images. I configure the model with zero temperature to encourage deterministic outputs and craft a targeted prompt that constrains responses to JSON containing final payment, subtotal and total discount. Each input image is converted into a base64‑encoded data URL for multimodal input. Since the model occasionally wraps JSON content inside markdown code fences, I implement simple string pre‑processing to strip these markers before deserialisation. Parsed values are aggregated following the assignment requirements: total real expenditure is accumulated from each receipt’s final‑payment value, while the pre‑discount total is computed by summing subtotal and total discount terms, deliberately omitting rounding‑related adjustments. Calculated aggregates are then formatted as standard HKD strings ready for the autograding workflow.
flowchart LR
    A[Input folder with receipt images] --> B[Iterate & load each image]
    B --> C[Encode image to base64]
    C --> D[LangChain Prompt + DeepSeek‑Flash‑Vision]
    D --> E[Model outputs structured receipt data]
    E --> F[Parse numeric values from model response]
    F --> G[Aggregate total actual payment across receipts]
    F --> H[Sum original pre‑discount amounts across receipts]
    G --> I[Return answer: total spent]
    H --> J[Return answer: total without discount]
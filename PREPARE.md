# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: npm test

## Setup

Install dependencies and prepare the evaluation environment:

```bash
npm install
```

The `prereq_command` runs the test suite (`npm test`) before each benchmark to ensure correctness. Any changes that break tests will be rejected before the performance benchmark runs.

## Run command

```bash
node -e "
const camelCase = require('./index.js').default;

// Benchmark corpus covering diverse cases
const testCases = [
  'foo-bar',
  'foo_bar_baz',
  'FooBarBaz',
  'XMLHttpRequest',
  'hello1world',
  'розовый_пушистый_единороги',
  'foo-bar-baz-qux',
  '__foo__bar__',
  'foo2bar',
  'b2b_registration_request',
  'turn_on_2sv',
  'Hello1World',
  'FooIDs',
  'A::a',
  'mGridCol6@md'
];

const iterations = 100000;
const start = Date.now();

for (let i = 0; i < iterations; i++) {
  for (const test of testCases) {
    camelCase(test);
    camelCase(test, {pascalCase: true});
    camelCase(test, {preserveConsecutiveUppercase: true});
  }
}

const elapsed = (Date.now() - start) / 1000;
const totalOps = iterations * testCases.length * 3;
const opsPerSec = Math.round(totalOps / elapsed);

console.log('ops_per_sec=' + opsPerSec);
"
```

## Output format

The benchmark must print `METRIC=<number>` or `ops_per_sec=<number>` to stdout.

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Ground truth

The baseline metric measures throughput (operations per second) on a diverse corpus of 15 test cases, each processed with 3 different option configurations (default, pascalCase, preserveConsecutiveUppercase) for 100,000 iterations. This totals 4.5 million camelCase operations.

The corpus includes:
- Simple ASCII cases (foo-bar, foo_bar)
- Complex mixed case (XMLHttpRequest, FooBarBaz)
- Unicode strings (розовый_пушистый_единороги)
- Numbers with letters (hello1world, b2b_registration_request)
- Edge cases (leading underscores, special characters, consecutive uppercase)

This benchmark ensures changes optimize real-world usage patterns, not artificial micro-benchmarks.

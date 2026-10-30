# The Outside-In Method — agent skills (moved)

The `outside-in` plugin — the skills `derive` and `review` — now lives in **<https://github.com/teob-cc/method>**, the
repository of the whole method: the specification, the book, the derivations and the skills, with the skills' step
reference generated from the specification.

**Existing installs keep working.** This marketplace stays registered and its `outside-in` entry points at the new
repository, so `/plugin marketplace update teob-method` followed by `/plugin update outside-in@teob-method` brings
the current version.

**New installs** add the method's own marketplace:

```bash
/plugin marketplace add teob-cc/method
/plugin install outside-in@method
```

Then, in any repository: `/outside-in:review` audits what exists, `/outside-in:derive` derives what doesn't yet.

Full method, worked example and eight derivations: **<https://teob.cc/method>**. MIT, published by Lambda House.

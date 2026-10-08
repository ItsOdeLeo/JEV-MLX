# JEV MLX, OpenDecider, and Laya: local decision inference

[Back to home](../README.md)

This guide places JEV MLX alongside OpenDecider, Laya, and Laya-MLX for developers
evaluating local action selection, classification, and semantic routing. The
technical connection is a shared need: turn application state and a question into
a bounded decision that host code can use.

JEV MLX is an independently maintained library for existing causal MLX-LM models.
The related projects below are separate implementations. Links describe technical
similarities and alternatives; they do not establish vendor affiliations or
checkpoint compatibility.

## Reuse existing weights without additional training

**JEV MLX adds decision scoring to a compatible MLX-LM checkpoint without training
or fine-tuning it, adding a decision head or adapter, or requiring separate
decision-model weights.** If compatible base weights are already available locally,
reuse them. Otherwise obtain the base checkpoint first; the backend loads a local
directory and does not download weights.

The full model processes the prompt, with valid prefix-cache reuse where available.
Its unchanged output head produces next-token logits at the final input position.
JEV MLX selects the logits for tokenizer-verified option codes and maps the result
to a business ID. This removes the need for a generated continuation; it does not
reduce inference to running only the final layer. See the [model path](architecture.md)
and [published scoring implementation](../src/jev_mlx/backends/mlx_lm.py).

[OpenDecider](https://github.com/manjunathshiva/opendecider/blob/b830ea46961105da8c30e30a5f11ccce211bbf43/README.md)
and [Laya](https://github.com/NandhaKishorM/laya/blob/3cf26cbcb18725dbc2d127bb8bb2c4c43243ae63/README.md)
publish trained decision-model checkpoints; Laya-MLX retains Laya's pretrained
weights. Their users can load published weights without training them themselves.
The distinction is **reusing existing compatible base-model weights versus loading
a dedicated decision-model checkpoint**, not a requirement that other users train
their own models.

## Implementation comparison

| Project | Model and runtime approach | Integration distinction |
| :--- | :--- | :--- |
| **JEV MLX** | Reuse existing compatible causal MLX-LM weights on Apple Silicon; read original option-token logits with no extra training. | Dynamic candidates with stable business IDs, `no_match` / `abstain`, state versions, and single-use execution authorization. |
| [OpenDecider](https://github.com/manjunathshiva/opendecider/blob/b830ea46961105da8c30e30a5f11ccce211bbf43/README.md) | Open-weight System 1 decision-model families, including encoder and decoder approaches. | Its own typed question contracts, serving clients, and model-specific runtimes. |
| [Laya](https://github.com/NandhaKishorM/laya/blob/3cf26cbcb18725dbc2d127bb8bb2c4c43243ae63/README.md) · Convai Innovations | Encoder-based decision models with choice, ordered-score, and yes/no outputs. | Its own question schema, decision heads, and checkpoint routing. |
| [Laya-MLX](https://github.com/mizorewww/laya-mlx/blob/ca5940aa9286dbbdfaacbecbdb7b337295ad36a2/README.md) | Independent MLX implementation of Laya's encoder and decision heads. | Local Apple Silicon inference for Laya checkpoints; a separate API from JEV MLX. |

The linked upstream READMEs are the sources for the related-project descriptions.
JEV MLX's behavior is documented in its [architecture](architecture.md) and
[Python API](python-api.md).

## What to evaluate for your application

If you already have an evaluated MLX-LM checkpoint and need a small set of allowed
application actions, JEV MLX provides that request and execution contract. If you
want a dedicated decision-model family, review OpenDecider or Laya's model cards
and APIs. For Laya models on Apple Silicon through MLX, review Laya-MLX's conversion
and inference instructions. These are starting points for evaluation, not a
ranking of model quality.

Use representative application states and allowed choices to compare semantic
accuracy, rejection behavior, latency, memory, and integration work. Keep model
startup, loaded weights, reusable prefixes, and state updates separate. Different
hardware, models, inputs, or datasets do not establish a speed or accuracy advantage.
JEV MLX's own [evaluation protocol](evaluation.md) and
[36-case results](extended-results.md) provide its evidence and limitations.

## Can JEV MLX load OpenDecider or Laya weights?

The shared decision use case is not a loading guarantee. JEV MLX currently provides
an MLX-LM backend and evaluates only the local checkpoints listed in its
[model compatibility guide](models.md). No OpenDecider or Laya adapter or checkpoint
validation is included. Similar backbone names are insufficient to establish
tokenizer, prompt, output-head, or semantic compatibility.

## Do the scores mean the same thing?

No. JEV MLX computes a restricted softmax over candidate logits and a raw-logit
margin; its scores are **not calibrated probabilities of correctness**. Upstream
OpenDecider and Laya documentation describes calibrated decision distributions.
Those descriptions are upstream claims, not JEV MLX measurements. Thresholds and
scores should not be transferred between these systems without validation on the
application's own data.

## Source snapshots

Reviewed on **2026-10-08** against these first-party README revisions:

- [OpenDecider · `b830ea4`](https://github.com/manjunathshiva/opendecider/blob/b830ea46961105da8c30e30a5f11ccce211bbf43/README.md).
- [Laya · `3cf26cb`](https://github.com/NandhaKishorM/laya/blob/3cf26cbcb18725dbc2d127bb8bb2c4c43243ae63/README.md).
- [Laya-MLX · `ca5940a`](https://github.com/mizorewww/laya-mlx/blob/ca5940aa9286dbbdfaacbecbdb7b337295ad36a2/README.md).

Upstream capabilities can change after these snapshots. This is a documentation
comparison; JEV MLX has not run a common benchmark against these projects.

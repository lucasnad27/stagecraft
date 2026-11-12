# herdr

Enabled by `herdr` under `STAGECRAFT.md` → **Integrations**.

[herdr](https://herdr.dev) is a terminal multiplexer for coding agents. It detects agent state on its own but has no phase field, so each stage marks the phase as an emoji prefix on its tab label. The sidebar then reads as a pipeline: 🔬 research, 🎨 design, 📋 plan, 🔨 implement, 🔍 review.

At the start of every stage, before anything else:

```bash
bash "<skill-dir>/scripts/herdr-phase.sh" <stage>
```

`clear` strips the marker and keeps the feature name. The script exits immediately when `HERDR_ENV` is not `1`, so it is safe in CI, plain terminals, and nested agent runs. Never edit it.

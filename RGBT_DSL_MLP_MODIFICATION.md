# RSTTrack RGBT-Language Final Modification

This branch implements the agreed core design directly on the supplied
`RSTTrack-main-uncertainty-memory-full-origin(1)` codebase.

## Implemented core

### 1. Modality-aware Language Routing (MLR)

- The previously unused `tir_text_adap` is now used by a dedicated TIR CLIP prompt path.
- RGB and TIR text features are encoded separately.
- A visual-conditioned router assigns each compact language token RGB-specific,
  TIR-specific and shared routing probabilities.
- The iTPN main stream still receives exactly one language token per modality, so the
  original temporal/template/search slicing logic is preserved.
- Multi-token routed language features are retained only for the sparse prompt branch.

Main files:
- `lib/models/rsttrack/itpn.py`
- `lib/models/layers/rgbt_dsl.py::ModalityAwareLanguageRouter`

### 2. Reliability-aware Dynamic Sparse Learning (RDSL)

At every existing cross-modal location `[5, 11, 17, 23]`, the original fixed-ratio
candidate-elimination branch is replaced (when enabled) by a reliability-aware sparse
prompt generator:

1. RGB/TIR/shared-language/current temporal context estimate per-frame modality
   reliability `r_rgb`, `r_tir`.
2. Search-token importance is obtained from the current transformer attention.
3. Spatial masks at ratios `[0.65, 0.75, 0.85, 1.0]` are generated and dynamically
   mixed per modality. Reliability biases the mixture toward denser or sparser
   retention.
4. Visual-query / language-key-value cross-attention constructs modality-specific
   multi-modal cues.
5. Channel-transposed sparse attention performs Top-K attention in channel space with
   ratios `[0.5, 2/3, 0.75, 0.8]` and mixes these branches with the learned
   reliability-conditioned channel weights.
6. The resulting RGB/TIR sparse semantic prompts are passed into the existing
   `ACE + MCI` cross-modal interaction path.

This keeps the main token sequence length unchanged and preserves the existing tracking
head, uncertainty head, dual memory and Qwen online-language update pipeline.

Main file:
- `lib/models/layers/rgbt_dsl.py::ReliabilityAwareDSL`

### 3. Dynamic Multi-Level Prompt Perception (DMLP)

- Sparse RGB/TIR prompts are used to reweight only current-frame search tokens.
- DMLP is enabled at interaction layers `[11, 17]` by default, matching the middle/late
  semantic-guidance principle rather than applying excessive guidance at all layers.
- The residual guidance scale is initialized to zero, so loading the original RSTTrack
  checkpoint begins from the baseline behavior and the new guidance is learned
  progressively.
- Per-stage RGB/TIR reliability further modulates prompt strength.

Main file:
- `lib/models/layers/rgbt_dsl.py::MultiLevelPromptModulator`

## Configuration

New config groups in `lib/config/rsttrack/config.py` and
`experiments/rsttrack/RSTTrack.yaml`:

```yaml
MODEL:
  LANGUAGE_ROUTER:
    ENABLED: true
    TOKEN_NUM: 3
    HIDDEN_DIM: 128
    DROPOUT: 0.0

  DSL:
    ENABLED: true
    NUM_HEADS: 8
    HIDDEN_DIM: 128
    DROPOUT: 0.0
    SPATIAL_RATIOS: [0.65, 0.75, 0.85, 1.0]
    CHANNEL_RATIOS: [0.5, 0.6666667, 0.75, 0.8]
    RELIABILITY_BIAS_STRENGTH: 2.0

  MLP_GUIDANCE:
    ENABLED: true
    LOC: [11, 17]
    INIT_SCALE: 0.0
```

For ablations, each component can be independently disabled with its `ENABLED` flag.
When `MODEL.DSL.ENABLED=false`, the original fixed-ratio candidate elimination remains
as a fallback.

## Training diagnostics

`lib/train/actors/rsttrack.py` now logs, without adding extra losses:

- RGB/TIR reliability at every DSL stage;
- effective spatial keep ratio;
- effective channel keep ratio;
- average RGB/TIR/shared language-routing probabilities.

This is useful for checking whether the learned reliability behaves sensibly under
illumination changes, thermal crossover, occlusion, etc.

## Additional code-safety fixes

- `candidate_elimination()` no longer hard-codes `.to('cuda')`.
- `generate_template_mask()` now follows the input tensor device.
- The frozen CLIP encoder is attached *after* the iTPN custom weight initialization,
  preventing `self.apply(self._init_weights)` from accidentally overwriting pretrained
  CLIP Linear/LayerNorm weights.

## Validation performed in this environment

- `python -m compileall -q lib RGBT_workspace tracking`: passed.
- Independent random-tensor unit test of language routing, RDSL and DMLP: passed.
- Shape check for the default two-template setting:
  - per-modality main stream: `[B, 386, 512]`;
  - sparse search prompt: `[B, 256, 512]`;
  - routed language tokens: `[B, 3, 512]`.
- DMLP zero-init check: output exactly equals its input at initialization.

A full end-to-end training/inference run was not possible inside this container because
the supplied project expects its own external CLIP package/checkpoint and several
environment dependencies. The original absolute CLIP/Qwen model paths are intentionally
left unchanged so they remain compatible with the user's existing runtime setup.

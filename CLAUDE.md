# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShinkaEvolve is an evolutionary algorithm framework that combines LLMs with evolutionary search for automated scientific code discovery. The system evolves populations of programs across generations, using LLMs as intelligent mutation operators. Developed by Sakana AI (arXiv 2509.19349).

## Build & Development Commands

```bash
# Installation (Python 3.11 recommended)
uv venv --python 3.11
source .venv/bin/activate
uv pip install -e .            # Standard install
uv pip install -e ".[dev]"     # With dev tools (pytest, black, isort, flake8)

# Running examples
shinka_launch variant=circle_packing_example
python examples/circle_packing/run_evo.py

# Tests
pytest tests/

# Visualization WebUI
shinka_visualize --port 8888 --open
```

## Architecture

### Three-Configuration Model
- **EvolutionConfig** - LLM models, patch types, generations budget
- **DatabaseConfig** - Multi-island population topology, elitism, migration
- **JobConfig** - Execution environment (local, SLURM with Docker/Conda)

### Core Module Structure (`/shinka`)

| Module | Purpose |
|--------|---------|
| `core/runner.py` | Main `EvolutionRunner` orchestrating the evolution loop |
| `core/sampler.py` | `PromptSampler` constructs LLM mutation prompts |
| `database/dbase.py` | `ProgramDatabase` - SQLite-backed population with island topology |
| `llm/llm.py` | `LLMClient` - Multi-model interface (OpenAI, Anthropic, Google) |
| `llm/dynamic_sampling.py` | Bandit algorithms for model selection |
| `edit/apply_diff.py` | Unified diff application with SEARCH/REPLACE format |
| `edit/apply_full.py` | Full code block replacement |
| `launch/scheduler.py` | Job scheduling (local processes, SLURM) |
| `prompts/` | LLM prompt templates for diff/full/cross patches |

### Execution Flow
1. **Generation 0**: LLM generates initial solution
2. **Evolution Loop**: For each generation:
   - Submit mutation jobs (diff/full/cross patches)
   - Evaluate via user's `evaluate.py`
   - Update database with results
   - Generate meta-recommendations periodically
3. **Patch Application**: LLM outputs parsed and applied via `apply_diff.py` or `apply_full.py`

### Island-Based Evolution
- Multiple independent islands evolve in parallel (default 4)
- Elite solutions migrate between islands at intervals
- Archive preserves best solutions for cross-pollination
- Parent selection strategies: `power_law`, `weighted`, `beam_search`

### EVOLVE-BLOCK Markers
User-controlled mutable code regions:
```python
# EVOLVE-BLOCK-START
def algorithm():
    # This code will be evolved by LLM
    pass
# EVOLVE-BLOCK-END
```

### Patch Types
1. **Diff** - SEARCH/REPLACE format for targeted modifications
2. **Full** - Complete EVOLVE-BLOCK rewriting
3. **Cross** - Combine inspiration from archive programs

## Configuration System (Hydra)

Uses Hydra with YAML composition. Pre-configured variants in `configs/variant/`.

```bash
# Override syntax
shinka_launch variant=circle_packing_example database=island_large evolution=large_budget
```

Key config directories:
- `configs/variant/` - Complete experiment presets
- `configs/task/` - Task-specific settings
- `configs/database/` - Island/archive configurations
- `configs/evolution/` - Budget settings (small/medium/large)

## Creating New Tasks

1. Create `evaluate.py` with:
   - `validate_fn()` - Returns `(is_valid, error_msg)`
   - `aggregate_metrics_fn()` - Returns metrics dict with `combined_score`
   - `get_kwargs()` - Provides evaluation parameters

2. Create `initial.py` with starter code containing EVOLVE-BLOCK markers

3. Add configs in `configs/task/` and `configs/variant/`

See `/examples/` for complete working implementations.

## Key Data Structures

- **Program** (dataclass): `code`, `metrics`, `generation`, `parent_id`, `archive_inspirations`
- **EvolutionConfig**: `diff_model`, `full_model`, `num_generations`, `patch_strategy`
- **DatabaseConfig**: `num_islands`, `archive_size`, `parent_selection`

## Adding LLM Models

Model pricing defined in `/shinka/llm/models/pricing.py`. Supports OpenAI, Anthropic, Google, and Azure variants.

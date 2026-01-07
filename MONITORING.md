# Experiment Monitoring Commands

## Output Directory Structure

```
results/<exp_name>/<timestamp>/
├── launch_hydra.log      # Main evolution log
├── evolution_db.sqlite   # All solutions & metrics database
├── meta_memory.json      # Meta-learning recommendations
├── best/                 # Current best solution
│   └── results/
│       ├── correct.json
│       └── metrics.json
└── gen_N/                # Per-generation outputs
    └── results/
        ├── correct.json
        └── metrics.json
```

## Quick Commands

### Watch Log in Real-Time
```bash
tail -f results/<exp_name>/<timestamp>/launch_hydra.log
```

### Check Current Generation Progress
```bash
ls results/<exp_name>/<timestamp>/ | grep gen_
```

### View Best Score
```bash
cat results/<exp_name>/<timestamp>/best/results/metrics.json
```

### View Specific Generation Metrics
```bash
cat results/<exp_name>/<timestamp>/gen_N/results/metrics.json
```

### Count Completed Generations
```bash
ls -d results/<exp_name>/<timestamp>/gen_*/ 2>/dev/null | wc -l
```

## WebUI Visualization

### Launch Dashboard
```bash
# Auto-detect latest experiment
shinka_visualize --port 8888 --open

# Specific experiment directory
shinka_visualize results/<exp_name>/<timestamp>/ --port 8888 --open

# Specific database file
shinka_visualize --db results/<exp_name>/<timestamp>/evolution_db.sqlite --port 8888 --open
```

### Remote Access (SSH Tunnel)
```bash
# On local machine
ssh -L 3000:localhost:8888 username@remote-host

# Then open http://localhost:3000
```

## Database Queries (Optional)

```bash
# List all programs with scores
sqlite3 results/<exp_name>/<timestamp>/evolution_db.sqlite \
  "SELECT id, generation, combined_score FROM programs ORDER BY combined_score DESC LIMIT 10;"

# Count programs per generation
sqlite3 results/<exp_name>/<timestamp>/evolution_db.sqlite \
  "SELECT generation, COUNT(*) FROM programs GROUP BY generation;"
```

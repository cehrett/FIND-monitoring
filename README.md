# FIND Real-Time Dashboards

Static GitHub Pages site for publishing CUSUM dashboards generated in `FIND_retro_x`. Each monitored topic gets its own directory so analysts can share a stable link such as `https://YOURUSERNAME.github.io/<topic>/`.

## Repository Layout
```
.
├── index.html               # Hub page linking to all live topics
├── topics.json              # Optional manifest consumed by index.html
└── dashboards/
    └── <topic>/
        ├── index.html
        ├── assets/         # CSS/JS/fonts copied from data/dashboards/<topic>/
        └── ...             # Any additional files required by the dashboard
```
- The root `index.html` (or another landing page) should list every topic and link to `dashboards/<topic>/`.
- Add or update shared JS/CSS assets here if topics reference global resources.

## Source of Truth
Dashboards originate in the main FIND repository:

```
/home/cehrett/Projects/Trolls/FIND_retro_x/data/dashboards/<topic>/
```

After `python -m src.realtime.dashboard_manager <topic>` runs, copy that folder into this repo under `dashboards/<topic>/`. Each run should replace the topic directory entirely so stale files are removed.

## Manual Publish Workflow
1. Activate the `find-cusum` environment and run the dashboard builder for the desired topic(s).
2. On the HPC login node (or any machine with repo access):
   ```bash
   export DASH_ROOT=/scratch/$USER/FIND/dashboards/github-pages
   cd $DASH_ROOT
   git pull --rebase
   rsync -av --delete \
       /home/cehrett/Projects/Trolls/FIND_retro_x/data/dashboards/<topic>/ \
       dashboards/<topic>/
   git add dashboards/<topic>
   git commit -m "Update dashboard for <topic>"
   git push origin main
   ```
3. Verify the live site at `https://YOURUSERNAME.github.io/<topic>/`.

Repeat for each topic. If no files changed, skip the commit.

## Automation Plan
Eventually the sbatch pipeline (`scripts/collect_and_process_youtube.sbatch`) will:
- Ensure this repo is cloned under `/scratch/$USER/FIND/dashboards/github-pages`.
- Use `rsync --delete` to mirror `data/dashboards/<topic>/` into `dashboards/<topic>/`.
- Stage, commit, and push changes automatically (skipping if `git status` is clean).
- Log failures so operators know whether the publish step succeeded.

Credential management (SSH keys or tokens) will be added when automated pushes are enabled.

## Requirements
- Git 2.30+ with SSH access to GitHub.
- The `find-cusum` Conda environment to generate dashboards.
- Write access to `/scratch/$USER/FIND/dashboards/`.

## Troubleshooting
- **Dashboard missing assets**: confirm relative paths inside `index.html` still point to files within the copied directory.
- **Site not updating**: check `git status` for uncommitted files, ensure `git push` succeeded, and wait a minute for Pages to redeploy.
- **404 on topic URL**: make sure the folder name matches the link exactly (case-sensitive) and contains an `index.html`.

Feel free to adapt the layout or landing page as the list of topics grows.

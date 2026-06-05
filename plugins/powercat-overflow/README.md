# powercat-overflow
<img width="2036" height="1080" alt="image" src="https://github.com/user-attachments/assets/a0ef4ffd-2505-465b-94a7-38e1b9f93812" />
<img width="2036" height="1080" alt="image" src="https://github.com/user-attachments/assets/b19bf17e-a022-43f5-b192-575601b1caca" />

Reviews every Power Automate cloud flow inside a Power Platform solution (`.zip`) against Microsoft''s coding guidelines and produces a `[SolutionName].findings.json` next to the uploaded solution. Then opens the hosted PowerCAT-Overflow viewer at <https://microsoft.github.io/power-cat-skills/PowerCAT-Overflow.html> and uploads both files so the user can explore the results.

## Skills

- **powercat-overflow** — Reviews every Power Automate cloud flow inside a Power Platform solution `.zip` against Microsoft''s coding guidelines, writes a single solution-level findings JSON next to the uploaded solution, then opens the hosted PowerCAT-Overflow viewer with both files loaded.

## Usage

- `powercat overflow`
- `review my solution`
- `audit Power Automate solution`
- `check this solution against guidelines`

Attach a Power Platform solution `.zip` (containing a `Workflows/` folder) and the skill will run end-to-end.

## Notes

- **Authoritative source list** is fetched at run time from `microsoft/power-cat-skills` → `Common/PowerCAT OverFlow/sources.md`. The skill hard-fails if the source list is unreachable.
- **Output schema** validates against `solution.findings.schema.json` from the same repo.
- **No local HTML rendering.** The hosted viewer (GitHub Pages) renders the diagram and overlays findings — Playwright opens it and uploads both files automatically.
- **Replaces** the previous `powercat-flow-review` plugin (single-flow JSON input + local HTML output).

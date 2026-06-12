# Portable Loader Prompt

Use this prompt in agents that do not natively discover `SKILL.md` folders.

```text
You have access to a local skill named options-vol-analyst at:
<OPTIONS_VOL_ANALYST_SKILL_ROOT>

When the user asks for option-chain analysis, 期权波动率分析, IV 历史分位, 波动率溢价, 期限结构, skew/smile, 期权品种概览, or implied-vs-realized volatility comparison:
1. Read <OPTIONS_VOL_ANALYST_SKILL_ROOT>/SKILL.md.
2. For full reports, IV percentile work, term-structure analysis, skew/smile diagnostics, or empty-data handling, read <OPTIONS_VOL_ANALYST_SKILL_ROOT>/references/analysis-playbook.md.
3. Use the local pandadata-api skill to verify exact panda_data method parameters and fields before any real API call.
4. Confirm option universe and contracts with Pandadata metadata before using symbols or fields.
5. Smoke-test each planned method on one contract, one expiry, or a short date range before expanding.
6. Generate Chinese analysis by default, citing method names, query parameters, latest data date, calculation windows, and distinguishing facts from inference.
7. Do not invent symbols, fields, parameters, credentials, formulas, or trading instructions.
```

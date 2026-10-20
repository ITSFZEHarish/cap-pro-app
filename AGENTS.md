# PRO Expenses development instructions

Follow `.agents/AGENTS.md` and the seven numbered PRO agent definitions. The unnumbered POS definitions are historical reference only and do not apply to this application.

The user-approved plan is `doc/plan/pro-expenses-plan.md` and contracts in `doc/plan/contracts.md`. Agent-01 coordinates contract changes across the 8 active services (`ExpenseService`, `OperationsService`, `WorkflowService`, `AdminService`, and the 4 `AccessControl` services). Use at most four concurrent agents including the coordinator. Preserve unrelated files. No commits or live deployment without a user request.

Use npm with the lockfile. After code changes run `npm run check`; run explicit targeted test files using `node --import tsx --test test/<file>.test.ts`. Packaging is authorized via `npm run package`. These instructions replace the inherited prohibitions on build/test commands for this project. Dependencies must use exact pins and versions at least seven days old. Install with `--ignore-scripts`; do not run remote installers.

`cds serve` / `cds watch` (`cds s`) require `tsx` when `tsconfig.json` exists. Never map `compilerOptions.paths["@sap/cds"]` to `@cap-js/cds-types/dist/cds-types.d.ts`. Point it at `./node_modules/@cap-js/cds-types` (same as `jsconfig.json` and `cds add typescript`). A `.d.ts` target makes `tsx` load the declaration file from `node_modules` and Node throws `ERR_UNSUPPORTED_NODE_MODULES_TYPE_STRIPPING`. See `doc/known-issues.md`.

Consult available SAP MCP documentation before SAP changes. When those servers are unavailable, consult installed package APIs and official SAP documentation and record this limitation. Strict TypeScript, top-level imports, decimal-safe money, server-side authorization, private attachments, i18n UI text, and `cds.log` are required. Never fabricate integration confirmations or claim unexecuted checks.

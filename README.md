# WebApp UI test automation

## 📝 Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
  - [System requirements](#system_requirements)
  - [General prerequisites](#general_prerequisites)
  - [Setup](#setup)
- [Usage](#usage)
  - [Development flow](#development_flow)
  - [CLI_flags](#cli_flags)
  - [GH Actions debug](#gh_actions_debug)
  - [Validation of export](#export_validation)
  - [Env selection (dev/staging/prod/custom)](#env_selection)
  - [AWS secrets (User roles and etc)](#aws_secrets)
- [Useful VS Code extensions](#vs_code_extensions)
- [Using Husky](#husky_usage)

## About <a id="about"></a>
This repository contains the code of end-to-end tests, written in  Cypress framework (https://docs.cypress.io/guides/getting-started/writing-your-first-test). Main pattern used for this project - is Page Object. We describe elements of pages and the way they can behave (*pages* folder). We describe actions, which we use to interact with pages (*actions* folder). And describe test specs (*integration* folder) - things/flows we want to test and verify on our pages, using actions to put the app in a required state.

There are several main folders of these project:

* .github - contains GitHub actions workflows files
* cypress - base folder, that contains the following:
  * actions - contains classes with methods which describe the interaction with pages. Contains subfolders, named by application's tabs.
  * fixtures - contains test data, named by test spec names. Data is stored in js files for convenient exporting and autocompletion.
  * integration - contains test specs
  * pages - contains classes, which describe what elements different pages have and how pages can behave. Contains subfolders, named by application's tabs. Also contains manager files for each folder, that accumulates files of folder to one manager file to prevent big amount of imports in specs.
  * plugins - contains **index.js** file, that can be used to tap into plugins, modify, or extend the internal behavior of Cypress.
  * support - contains three files:
    * **commands.js** - contains custom cypress commands.
    * **index.d.ts** - contains types definitions for custom cypress commands.
    * **index.js** - used for enabling additional modules for cypress
* utils - folder for helper functions. Contains useful functions validating format of data, working with uploading fixtures, acquiring baseUrl for current environment of test run.

## Getting started <a id="getting_started"></a>

### System requirements <a id="system_requirements"></a>
- (Windows users) [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install) (Windows Subsystem for Linux). We use *Ubuntu 20.04* and you need to [use version 2 of WSL](https://docs.microsoft.com/en-us/windows/wsl/install#upgrade-version-from-wsl-1-to-wsl-2).
- Docker. Get it for your system from [here](https://docs.docker.com/get-docker/).If you use Windows - please, [use WSL 2 based engine](https://docs.docker.com/desktop/windows/wsl/) for Docker.

### General prerequisites <a id="general_prerequisites"></a>
1. Install `nvm`([macOS/Linux](https://github.com/nvm-sh/nvm), [Windows](https://github.com/coreybutler/nvm-windows)).
2. **WARN: please, use lts version of node 16. , currently (05.05.22) - lts/gallium*** Install lts node version. Run `nvm install lts`.
3. Run `nvm use lts` to use it.

### Setup <a id="setup"></a>
1. Clone repo
2. Install packages. Run `npm i`
3. Add `cypress.env.json` file to the root of project with following format:
```json
{
  "USERNAME": "value", 
  "PASSWORD": "value"
}
```

## Usage <a id="usage"></a>

General way to run all cypress tests to run `npx cypress run` command. This command will run all existing test spec headless in electron browser at staging environment, using Api login method by default. General way to open cypress GUI is to run `npx cypress open` command.

`package.json` file in `"scripts":` property contains ready to use commands for some mostly used cases. For example `npm run cy:open` command will open cypress GUI, `npm run cy:chrome_headed_prod_api` will run all tests in chrome headed browser at production environment, using Api login method etc.

### Development flow <a id="development_flow"></a>

We don't have strict rules for our development flow. Everything is pretty standard: 
  1. You branching from master (**always push empty branch after its creation, it will be a signal that you at least started work on a ticket**)
  2. If you develop test / framework feature - name branch in next notation **feature/your_name/jira_ticket_id** (for example, feature/Ernst/QA-666)
  - If you developing hotfix -> **hotfix/your_name/ticket_name_OR_hotfix_name**
  3. Assign at least 2 reviewers for your pull request. **Get at least 2 approvals.**.
  4. If you are implementing test case with export document verification, then 'it's title **HAVE TO** contain 'Check export' string in exact this state! And spec name **HAS TO** contain .export. in it's name, for example: QA-4667.export.spec.ts 
  5. If you want to add improvements into someone's PR - branch from feature branch, make changes and create to PR into parents branch (naming: **feature/your_name/ticket_id__pr_changes**). *You can commit into someones branch*, **but you are allowed to do that in exceptional cases** (for example, PR almost merged and you need to run and apply ESLint changes) 
  6. When you got approvals - merge branch by yourself or ping someone who was a reviewer.
  7. Use the pull request template to create PR, it is **NECESSARY** to check only 1 environment checkbox to trigger pull_request_check workflow appropriately. If you choose to run your tests on custom env, paste the link to customEnvLink URL section. This will trigger to run tests from your PR to check, if they work.
      Workflow for PR will run **ONLY** if you label it with **ready_for_review** label, it won't run even if you create pull request without this label

  **NOTE**: 
  - **please, while developing anything** - run `npm run tsc:watch` command in separate terminal instance (or split terminal into two). This will make TypeScript compilier keep an eye on your files changes and alert you when you forget, for example, update methods names after merge.
  - please, when writing commit message - add something meaningful, rather than "added some code". Good commit message: "[QA-something] added new actions into module_name" / "[misc] linter fixes". Bad commit message: "upd" / "fix" 
  - (ernst): I do not force to use small commits instead of big ones, but when commit something - think what you would do with the big one if you have to revert / reset or cherry-pick it. 
     

### CLI flags <a id="cli_flags"></a>

About cypress command line and it's general flags can be read [here](https://docs.cypress.io/guides/guides/command-line).

Project's specific environment variables for `--env` flag:
1. `url=` - accepts values `dev`,`prod` or `staging` for development, production or staging environment. Example of usage: `npx cypress run --env url=prod` will run tests at production environment. If this variable was not passed, uses `staging` by default.
2. `loginMethod=` - accepts values `ui` or `api` for login by UI or Api. Example of usage: `npx cypress run --env loginMethod=ui` will launch tests with login by UI. If this variable was not passed, uses `api` by default.
Example of combining previous variables: `npx cypress run --env url=dev,loginMethod=ui` will launch tests at development environment with login by UI.
3. `customEnv=` - accepts url to specific branch environment. Example of usage: `npx cypress run --env customEnv=https://someUrl/to/env` will launch tests at this environment.

### GH Actions debug <a id="gh_actions_debug"></a>

If your task will be connected with GH Actions changes or you would like to check how your newly implemnted test can behave in GH Actions - you should use [act](https://github.com/nektos/act), rather then commit a lot of times into the repo and trigger the real pipeline.

Main flow of how we use act for this repo - described in txt file in [these notes](./.act/install_notes.txt).

### Validation of export <a id="export_validation"></a>

Since we have a lot of test cases which has validation of Report Export (it will `*.docx` file) - we had to find the way we could automate these checks somehow. 

We found a way we can somehow automate it - **we convert `docx` file into html and then open it in Cypress**. 

You can refer to [QA-4053 spec](./cypress/integration/not_full_reports/sales/value_conclusion/QA-4053.spec.ts) to see the code of such tests.

**Flow for ReportExport checks**

1. (1st `it` in `describe`) Your test creates report.
2. (1st `it` in `describe`) Your test downloads report. Report has `job_id.docx` name and stored in `cypress/download`. Inside method `downloadAndConvertDocxReport()` we call several tasks (code which executes in nodejs): wait until file showed up in filesystem -> we convert docx into html -> we rename docx file from `job_id.docx` to `QA-test_case_number.docx` -> we rename html file from `job_id.html` to `QA-test_case_number.html`
3. (2nd `it` in `describe`) **important** Before calling `cy.task("getFilePath")` to get the path of your html report - you will need explicitly change `baseUrl` in `cypress.json` by adding next line:

```ts
Cypress.config().baseUrl = null;
```

This will set baseUrl of `cypress.json` to `null` and reload browser's window, which eventually made possible to navigate to static html page in filesystem. Without it - Cypress will consider relative path of html report as path to web resource (will navigate to `http://baseUrl.com/
cypress/downloads/TestAutoReport-QA-4719 - 462 1st Avenue, Manhattan, NY 10016.docx.html`, for example).

4. (2nd `it` in `describe`) Your test opens generated html report in Cypress (Cypress *can't* (well, until [release 9.6.0](https://github.com/cypress-io/cypress/releases/tag/v9.6.0)) [visit other origin url](https://docs.cypress.io/guides/guides/web-security#Same-superdomain-per-test))
5. (2nd `it` in `describe`) Your test makes traverse and assert on generated html report. 

### Env selection (dev/staging/prod/custom) <a id="env_selection"></a>

For dynamic [`baseUrl`](https://docs.cypress.io/guides/references/configuration#e2e) set we use [Cypress env variables](https://docs.cypress.io/guides/guides/environment-variables) 
and access to `config` from [`setupNodeEvents`](https://docs.cypress.io/guides/references/configuration#setupNodeEvents) (yeah, since Cypress 10 released - a lot of changed).

Basically what we do - we reassign `baseUrl` during runtime here:
```ts
config.baseUrl = evalUrl(config);
```
If want more details on hot it works - dive into `evalUrl` function.

**How to use:**

Pass `--env url=key_of_the_env` (urls and their **keys** defined in [ENVS](./utils/env.utils.ts))

Opens Cypress GUI with dev url:
```shell
npm run cy:open -- --env url=dev
```

Opens Cypress GUI with custom url:
```shell
npm run cy:open -- --env url=custom,customUrl='https://playwright.dev'
```

**The same approach** remains for `npx cypress run` and CI runs.

In CI we have check whether `url==custom` and in that case assign `customUrl`.

Previously, we were selecting specific url to run the tests with help of [Cypress environmental variables](https://docs.cypress.io/guides/guides/environment-variables), but our `baseUrl` in `cypress.json` wasn't set. It [wasn't a good approach](https://docs.cypress.io/guides/references/best-practices#Setting-a-global-baseUrl), but it allowed us dynamically set the url to visit and url for api requests. **BUT ALSO** our `before/beforeEach` **hooks were executing two time** (two time of login through api). And if we would try to create report - we would create two report and were working in the second one.

### AWS secrets (User Roles and etc) <a id="aws_secrets"></a>

We have tests which requires login as specific user (Lead Appraiser, Appraiser user, Admin and etc), you can find them by `@permissions_roles` tag. 

We could've store these secrets in GH Actions secrets, but in that case we won't have an option to edit them (especially, if we store a pretty big `json`).

That's why we need AWS Secret Manager. We set there secret `Github/Cypress/User_Roles` which store data about User Roles (their usernames and passwords). 
If you want to edit/add secrets: 
1. Go to the AWS (use AWS SSO, which you can find in `Google apps` in your Bowery Gmail account).
2. Select `GoogleSAMLPowerUserRole` in `bowery-prod` section.
3. Find AWS Secret Manager in Search and navigate there.
4. Paste secret name you want to edit or create your own (**IMPORTANT:** Naming convention for such secrets should be next - Github/Cypress/**, like `Github/Cypress/User_Roles`).

Code of custom action can be found [here](https://github.com/Bowery-RES/action-run-e2e-tests). Flow of credentials exposing into execution environment (this is why we don't need to write or somehow add secret info into codebase):
1. Connect to AWS Secret Manager (step "Configure secrets") from **allowed GH repository** (private info, configurable by privacy policy in AWS) with **specific role** (public info).
2. Reading AWS secrets and emmiting them into environment (step "Set secrets to GH environment") That's why Cypress-specific creds need to have `CYPRESS_` prefix to be accessible with `Cypress.env` method).
3. That's it! Variables was read from AWS and set to the environment in runtime. From that moment they are accessible to our test.

## Useful VS Code extensions <a id="vs_code_extensions"></a>

WARN: if you use Prettier - **make sure to disable it**, since it has conflicts with ESLint.

List of useful extensions:
  - ESLint
  - GitGraph
  - LiveShare
  - GitLens
  - GitHub Pull Requests
  - Jira and Bitbucket (you will use only Jira integration)

## Using Husky <a id="husky_usage"></a>

Husky is a tool that allows custom scripts to be ran against your repository.
Currently husky pre-commit hook is set up in that way so 2 scripts are executed:
*lint:run* and *tsc:check*.

**Create a hook**

To add a command to a hook or create a new one, use `husky add <file> [cmd]`

`npx husky add .husky/pre-commit "npm test"`
  
`git add .husky/pre-commit`

Try to make a commit
`git commit -m "Keep calm and commit"`

If *_npm_* *_test_* command fails, your commit will be automatically aborted.

**Note**
  
Husky is currently not working with VScode UI commits, so to prevent "broken" commits get into repository
use git commands in terminal

[husky documentation](https://typicode.github.io/husky/#/)
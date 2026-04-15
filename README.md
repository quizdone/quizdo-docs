# App specification

## Description
>
> This application helps teachers and instructors to efficiently generate different versions of exams from tasks (questions) from parametrically defined properties. It's mostly opensource project, but also contains some paid features to keep it running.
>
> The main goal is to simplify and modernize the process of exam preparation and tailor it to the needs of modern education. Automatization of the process of exam preparation, translation, evaluation and sharing of the content and using LLMs (AI) to simplify (the already simple) the process of question and exams creation.
>
> Secondary goal is to provide a platform for creating and sharing tasks, exams, and other educational content, provide API for developers to use the content and the app functions in their own projects using API.
>
> Application can be found at [Quizdo.eu](https://quizdo.eu).

### Features

- Safe authentication using Auth providers
- Simple generation of multiple versions (values, languages) based a parametrically specified questions.
- Create exam from specified categories of questions, randomizing order, using selected language.
- Supports both closed and open questions.
- Context aware exam generation - different outputs based on different user needs;
  - Style of the question, answers, inputs
  - Printed exam, excercise, online interactive tests, etc.
- Generating values and performing calculations with it
- Evaluating the task and the resulting exam qualities according to paedagogic theory using AI.
  - [Taxonomy of questions in learning according to D. Tollinger (1970)](https://is.muni.cz/el/1441/podzim2012/ZS1BK_PDD/um/Tollingerova.pdf)
- User controlled translation using Deepl AI translation
- Enable user to use Anthropic, ChatGPT, Gemini to create task/exam parameters from text input
- Run live tests and log results, watch for cheating etc.
- Sharing and reusing content between users, reading and copying questions (like social networks)
- Integration with other systems using API. This enables to use the questions by developers outside the app, potentially enabling infinite number of excercises, gamification.

### Example workflows

1. Teacher creates account and logs in
2. Adds subject of interest to the account
   - Subject = category (Mathematics, Physics...)
3. Teacher creates a question (task) with parametric properties
   - Can use LLM (AI) to prepare the question (task) for him from frontend
   - Title, description, the task itself
   - Steps for solving the task correctly
   - Adds translations to the question (eg. english, german, czech, etc.)
   - Can use variables, answers, etc. to automatically generate multiple versions of the question (task)
4. Creates new exam with selected questions, categories, tags, etc.
   - Creates query for selecting questions based on selected categories, tags, available languages etc.
   - The app generates the exam with questions using data from the question (task)
   - They either downloads the exam in printable format or generates link to fill the exam online with questions generated using language they want to use
   - The task version is stored in the database and can be also used later for different purposes
   - LLM helps decide the quality of the exam using peadagogical theory and suggests improvements
5. Students take the exam (test)
   - The tasks/exams can be used without storing results in the database, students can safely excercise online and get results immediately
6. Teacher evaluates the exam (test)
   - The app can use LLM (AI) to evaluate the exam (test)
7. Everyone is happy and keeps using the app

## Documentation

- [**General components**](general/components/README.md) — User-readable meaning of domains (categories, questions, exams, tags, …); product “user story” in one place.
- [**ADRs**](adr/README.md) — Architecture Decision Records
- [**Backend**](backend/README.md) — Microservices (identity, content, collector, evaluator), layout, running.
- [**Frontend**](frontend/README.md) — GUI (Next.js/React), consumed APIs, tech.

## Troubleshooting WSL

When using WSL (Windows Subsystem for Linux), you may encounter several issues:

### Docker integration issues
- When installing or running Docker for the first time, WSL integration may fail.
- A simple fix is to restart WSL:
```bash
    wsl --shutdown
```
- After restarting, Docker should properly integrate with WSL.

### Git + SSH issues (especially with submodules)
- When using git pull over SSH inside WSL, the initial pull may work, but submodules can fail.
- This usually happens because WSL does not have access to your SSH keys from Windows.
- Fix: Copy SSH keys into WSL

```bash
    mkdir -p ~/.ssh
    cp /mnt/c/Users/<your-username>/.ssh/id_ed25519* ~/.ssh/

    chmod 600 ~/.ssh/id_ed25519
    chmod 644 ~/.ssh/id_ed25519.pub

    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
```
- Then verify the connection:
```bash
    ssh -T git@github.com
```
- Alternative
    Instead of configuring SSH in WSL, you can use `Git Bash` on Windows for cloning, pulling, and pushing changes.

### Working with submodules

- After cloning the repository, make sure to initialize and update submodules:
```bash
    git submodule update --init --recursive
```

- Then switch into each submodule directory:

```bash
    cd <submodule-directory>
```
- By default, submodules are checked out at a specific commit (detached HEAD),
- which may not include the latest changes.

- To work with the latest version, switch to the dev branch (called main in this project):

```bash
    git switch dev
    # or
    git checkout dev
```

- Then pull the latest changes:
```bash
    git pull
```

- Note: Submodules are pinned to specific commits in the parent repository.
- Switching to a branch like dev ensures you are working with the latest updates instead.


### Line endings (LF vs CRLF)

- When working on Windows, `Git` / `VS Code` may automatically convert line endings from `LF` to `CRLF`.
- This can cause issues in WSL or Linux environments where LF is expected.

- If `.gitattributes` doesn't exist, crea one with: 

`* text=auto eol=lf`

#### Fix: enforce LF using `.gitattributes`
```bash
    git add .gitattributes
    git add --renormalize .
```

- this will renormalize all line endings to LF

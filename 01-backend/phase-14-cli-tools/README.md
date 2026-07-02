# Phase 14 — CLI Tools

Node.js là nền tảng tuyệt vời để build CLI tools — có thể build scaffolding tools, dev utilities, automation scripts, và full-featured CLIs chạy trên mọi platform. Những packages cần biết: Commander (argument parsing), Inquirer (interactive prompts), Chalk (colored output).

---

## Cấu trúc CLI Project

```
my-cli/
├── package.json       ← "bin" field để register command
├── src/
│   ├── index.js       ← entry point
│   ├── commands/
│   │   ├── init.js
│   │   └── generate.js
│   └── utils/
│       ├── logger.js
│       └── files.js
```

```json
// package.json
{
  "name": "my-cli",
  "version": "1.0.0",
  "bin": {
    "mycli": "./src/index.js"   // command → file
  },
  "scripts": {
    "dev": "node src/index.js"
  }
}
```

```js
// src/index.js — shebang line (QUAN TRỌNG)
#!/usr/bin/env node
'use strict'
// ... rest of CLI code
```

```bash
# Link locally để test
npm link
mycli --help

# Unlink
npm unlink
```

---

## Commander — Argument Parsing

```bash
npm install commander
```

```js
#!/usr/bin/env node
const { Command } = require('commander')
const { version } = require('../package.json')

const program = new Command()

program
  .name('mycli')
  .description('My awesome CLI tool')
  .version(version)
  .showHelpAfterError()         // hiển thị help sau lỗi

// === Subcommands ===

// init command
program
  .command('init <project-name>')      // <required> argument
  .description('Initialize a new project')
  .option('-t, --template <type>', 'project template', 'default')
  .option('-d, --directory <path>', 'target directory', '.')
  .option('--no-git', 'skip git initialization')
  .option('--ts, --typescript', 'use TypeScript')
  .action(async (projectName, options) => {
    // options.template, options.directory, options.git, options.typescript
    const { initProject } = require('./commands/init')
    await initProject(projectName, options)
  })

// generate command với subcommands
const generate = program.command('generate').alias('g')
  .description('Generate code files')

generate
  .command('component <name>')
  .option('-p, --path <path>', 'component path', 'src/components')
  .action(async (name, options) => {
    const { generateComponent } = require('./commands/generate')
    await generateComponent(name, options)
  })

generate
  .command('service <name>')
  .action(async (name) => {
    const { generateService } = require('./commands/generate')
    await generateService(name)
  })

// list command với options
program
  .command('list [filter]')           // [optional] argument
  .description('List all items')
  .option('-f, --format <type>', 'output format (table|json|csv)', 'table')
  .option('-l, --limit <number>', 'limit results', '10')
  .action(async (filter, options) => {
    const limit = parseInt(options.limit)
    const { listItems } = require('./commands/list')
    await listItems({ filter, limit, format: options.format })
  })

// Global options
program
  .option('-v, --verbose', 'verbose output')
  .option('--config <path>', 'config file path', '.myclirc')

program.parse()

// Nếu không có subcommand, show help
if (!process.argv.slice(2).length) {
  program.outputHelp()
}
```

---

## Inquirer — Interactive Prompts

```bash
npm install @inquirer/prompts
```

```js
const {
  input, password, select, checkbox, confirm, number
} = require('@inquirer/prompts')

// Ví dụ: Init wizard
async function runInitWizard() {
  const projectName = await input({
    message: 'Project name:',
    default: 'my-project',
    validate: (value) => {
      if (!value.trim()) return 'Project name cannot be empty'
      if (!/^[a-z0-9-]+$/.test(value)) return 'Use lowercase letters, numbers, and dashes only'
      return true
    }
  })

  const template = await select({
    message: 'Select template:',
    choices: [
      { name: 'Express API', value: 'express-api', description: 'REST API with Express' },
      { name: 'Fastify API', value: 'fastify-api', description: 'REST API with Fastify' },
      { name: 'NestJS', value: 'nestjs', description: 'Enterprise Node.js framework' },
      { name: 'Empty', value: 'empty', description: 'Blank project' }
    ]
  })

  const features = await checkbox({
    message: 'Select features:',
    choices: [
      { name: 'TypeScript', value: 'typescript', checked: true },
      { name: 'ESLint', value: 'eslint', checked: true },
      { name: 'Prettier', value: 'prettier', checked: true },
      { name: 'Vitest', value: 'vitest' },
      { name: 'Docker', value: 'docker' },
      { name: 'GitHub Actions', value: 'github-actions' }
    ]
  })

  const dbType = await select({
    message: 'Database:',
    choices: [
      { name: 'PostgreSQL + Prisma', value: 'postgres-prisma' },
      { name: 'MongoDB + Mongoose', value: 'mongodb-mongoose' },
      { name: 'None', value: 'none' }
    ]
  })

  const port = await number({
    message: 'Server port:',
    default: 3000,
    validate: (v) => (v > 0 && v < 65536) || 'Invalid port'
  })

  const confirmed = await confirm({
    message: `Create "${projectName}" with ${template}?`,
    default: true
  })

  if (!confirmed) {
    console.log('Aborted.')
    process.exit(0)
  }

  return { projectName, template, features, dbType, port }
}
```

---

## Chalk — Colored Output

```bash
npm install chalk
```

```js
const chalk = require('chalk')

// Basic colors
console.log(chalk.green('Success!'))
console.log(chalk.red('Error!'))
console.log(chalk.yellow('Warning!'))
console.log(chalk.blue('Info'))
console.log(chalk.gray('Debug'))

// Styling
console.log(chalk.bold('Bold text'))
console.log(chalk.underline('Underlined'))
console.log(chalk.italic('Italic'))
console.log(chalk.strikethrough('Strikethrough'))

// Combine
console.log(chalk.bold.green('Bold green'))
console.log(chalk.bgRed.white(' ERROR '))
console.log(chalk.hex('#FF6B6B')('Custom color'))

// Template literals
const name = 'Alice'
console.log(chalk`Hello {bold ${name}}, you are {green.bold logged in}!`)

// Logger utility
const log = {
  success: (msg) => console.log(`${chalk.green('✓')} ${msg}`),
  error: (msg) => console.error(`${chalk.red('✗')} ${chalk.red(msg)}`),
  warn: (msg) => console.log(`${chalk.yellow('⚠')} ${chalk.yellow(msg)}`),
  info: (msg) => console.log(`${chalk.blue('ℹ')} ${msg}`),
  step: (n, total, msg) => console.log(`${chalk.gray(`[${n}/${total}]`)} ${msg}`)
}

log.success('Project created successfully')
log.error('Failed to connect to database')
log.step(2, 5, 'Installing dependencies...')
```

---

## Putting It Together — Full Example

```js
#!/usr/bin/env node
// src/index.js
const { Command } = require('commander')
const { select, input, confirm } = require('@inquirer/prompts')
const chalk = require('chalk')
const { mkdir, writeFile } = require('fs/promises')
const path = require('path')

const program = new Command()

program
  .name('create-node-app')
  .description('Scaffold a Node.js project')
  .version('1.0.0')

program
  .command('new <name>')
  .option('-t, --template <type>', 'template', 'express')
  .option('-y, --yes', 'skip prompts, use defaults')
  .action(async (name, options) => {
    console.log(chalk.bold.cyan('\n🚀 Create Node App\n'))

    let template = options.template
    if (!options.yes) {
      template = await select({
        message: 'Select template:',
        choices: [
          { name: 'Express', value: 'express' },
          { name: 'Fastify', value: 'fastify' }
        ]
      })
    }

    const targetDir = path.resolve(process.cwd(), name)

    log.step(1, 3, `Creating directory ${chalk.cyan(targetDir)}...`)
    await mkdir(targetDir, { recursive: true })
    await mkdir(path.join(targetDir, 'src'))

    log.step(2, 3, 'Creating files...')
    await writeFile(path.join(targetDir, 'package.json'), JSON.stringify({
      name,
      version: '1.0.0',
      main: 'src/index.js',
      scripts: { start: 'node src/index.js', dev: 'nodemon src/index.js' },
      dependencies: { [template]: 'latest' }
    }, null, 2))

    log.step(3, 3, 'Done!')
    console.log(`\n${chalk.green('✓')} Project created!`)
    console.log(chalk.gray(`\n  cd ${name}`))
    console.log(chalk.gray('  npm install'))
    console.log(chalk.gray('  npm run dev\n'))
  })

program.parse()

function log() {}
log.step = (n, total, msg) => console.log(`  ${chalk.gray(`[${n}/${total}]`)} ${msg}`)
```

---

## Progress Bars và Spinners

```bash
npm install ora cli-progress
```

```js
const ora = require('ora')
const cliProgress = require('cli-progress')

// Spinner
const spinner = ora('Installing dependencies...').start()
await installDependencies()
spinner.succeed('Dependencies installed')

spinner.fail('Failed to connect')
spinner.warn('Retrying...')

// Progress bar
const bar = new cliProgress.SingleBar({
  format: 'Processing [{bar}] {percentage}% | {value}/{total} files'
})
bar.start(100, 0)

for (let i = 0; i <= 100; i++) {
  await processItem(i)
  bar.update(i)
}

bar.stop()
```

---

## Bài tập thực hành

1. **Task tracker CLI**: CLI tool quản lý todos — `add <task>`, `list`, `done <id>`, `delete <id>`. Lưu vào `~/.tasks.json`. Dùng Commander + chalk colored output.

2. **Project scaffolder**: CLI tool tạo Node.js project structure — interactive prompts (project name, template, features), copy template files, run `npm install`.

3. **GitHub activity CLI**: Gọi GitHub API, hiển thị 10 recent events của một user — `gh-activity <username>`. Format đẹp với chalk, handle errors gracefully.

4. **Publish package**: Publish CLI tool lên npm — setup `bin` field, test với `npm link`, viết README, publish với `npm publish`.

---

## Resources

- [Commander.js docs](https://github.com/tj/commander.js) — Argument parsing
- [Inquirer.js docs](https://github.com/SBoudrias/Inquirer.js) — Interactive prompts
- [Chalk docs](https://github.com/chalk/chalk) — Terminal colors

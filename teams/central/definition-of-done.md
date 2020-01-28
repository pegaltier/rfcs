
# Definition of Done (result)

> **Collaborator vs Codeowner** <br>
The difference between these 2 personas is the level of responsibility
and understanding required.  A codeowner is expected to have a complete awareness of the entire
codebase they are owning; and is also considered a collaborator.

### Quick checklist
- [ ] **Automation -** do these changes break the automation rules?
- [ ] **Code review -** are these changes understood by other codeowners?
- [ ] **Logging -** can another collaborator debug with the assistance of logging?
- [ ] **Tests -** can another collaborator run passing tests?
- [ ] **Documenation -** can a non-collaborator start using these changes?

## Automation (CI/CD)
- `master` - must represent deployed state
  - integration tests are passing
- `develop` - must represent latest working state
  - unit tests are passing
  - e2e tests are passing

## Code review
The purpose of reviews are
- to gain concensus on implementation
- to propogate understanding
- to confirm that tests pass for other devs
- to catch new bugs or technical debt

## Logging
Logging is vital for developers with less working knowledge to assist in development.  It should be
considered mandatory for collaboration.

## Tests
Tests are intended to provide a deterministic state of compatibility.  They also represent the
truest form of documentation for a collaborator.

#### Integration tests
Are to catch API incompatibilities

- an integration test is defined as any test that verifies 2 or more projects are working together
- **all should be passing before a PR is merged into master**

#### Unit tests
Are to catch logical incompatibilities

- a unit test is defined as any test that verfies the output of code with given input(s)
- these must run locally without any other dependencies
- **all should be passing before a PR is merged into develop**

#### e2e tests
Are to catch incompatibilities in a flow of logical units

- an e2e test is defined as any test that flows through multiple units in a single project
- **all should be passing before a PR is merged into develop**


## Documentation
The purpose of documentation is to reduce the reliance on individual collaborators.  Minimal
documentation is considered to be an amount sufficient for a non-collaborator to begin using this
project.  `README.md` should be the starting point that leads to all other documentation.

### Types of documentation

#### Architecture
Should provide context and requirements (eg. link to RFC)

#### External docs
Are intended to support non-collaborators

- Reference - link to unit/e2e tests (required)
- Discussions/recipes (optional)
- Tutorials (optional)
- How-to guides (optional)

#### Internal docs
Are intended to support collaborators

- `nix-shell` configuration (required)
- `CONTRIBUTING.md`
  - run tests (required)
  - contributing rules/guidelines
  - explain external dependencies
	- (eg. nodejs devDependencies)
- code comments (optional)
  - comments in code should explain reasoning, not implementation
  - comments can be used to explain input assumptions
- `Makefile` (optional)
    - automation for steps described in README

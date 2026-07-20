# Stack Playbooks — concrete commands per detected stack

Detect the stack in Phase 1, then use the matching sections. Commands are
starting points; adapt to the repo. Anything that can't run in the current
environment: note it in the artifact as a command the USER should run, with
expected output described.

## Universal (any repo)

```bash
git ls-files | wc -l
git log --oneline | wc -l                     # age/activity
git shortlog -sn --no-merges | head           # who built this
git log --format='%ad' --date=short | sort | uniq -c | sort -rn | head  # activity eras
git log --grep='fix\|hack\|workaround\|revert\|HOTFIX' --oneline | head -30
find . -maxdepth 2 -type d | grep -vE '\.git|node_modules|dist|build|vendor|target|__pycache__'
ls .github/workflows/ 2>/dev/null; cat Jenkinsfile .gitlab-ci.yml 2>/dev/null
cat Dockerfile* docker-compose* 2>/dev/null
find . -name '*.md' -path '*doc*' -o -name 'ADR*' -o -path '*adr*' -name '*.md' | grep -v node_modules
tokei . || cloc . || (git ls-files | sed 's/.*\.//' | sort | uniq -c | sort -rn | head)  # LOC by language
grep -rn "TODO\|FIXME\|XXX\|HACK" --include='*.*' -l | head -20    # debt markers
```

History mining for the decision log:
```bash
git log --follow -p -- <file> | less          # one file's full story
git blame -w -C -C <file>                     # authorship, ignoring moves/whitespace
git log -S '<symbol>' --oneline               # when a symbol appeared/vanished
```

## JavaScript / TypeScript (Node, React, Next, etc.)

Detect: package.json. Monorepo: turbo.json / nx.json / lerna.json /
pnpm-workspace.yaml / workspaces field.

```bash
cat package.json | jq '{scripts, dependencies: (.dependencies|keys), devDependencies: (.devDependencies|keys)}'
npx madge --circular --extensions ts,tsx,js src/           # cycles
npx madge --summary src/                                   # most-depended-on = god modules
npx dependency-cruiser --no-config --output-type dot src | head -100
npx ts-prune 2>/dev/null | head -30                        # dead exports
grep -rn "createContext\|configureStore\|createStore" src/ # state roots (frontend)
grep -rn "router\|Route\|createBrowserRouter" src/ --include='*.tsx' -l  # route table
```
Entry points: main/module/exports in package.json; framework conventions
(pages/ or app/ for Next, src/index for CRA/Vite, server bootstrap for Node).
Seams: axios/fetch wrappers, prisma/typeorm/knex config, kafka/amqp clients.

## Java / Spring Boot

Detect: pom.xml / build.gradle. Monorepo: maven modules / gradle subprojects.

```bash
mvn dependency:tree -Dscope=compile 2>/dev/null | head -60   # or: gradle dependencies
grep -rn "@RestController\|@Controller" --include='*.java' -l     # HTTP surface
grep -rn "@KafkaListener\|@RabbitListener\|@SqsListener\|@JmsListener" --include='*.java'  # async surface
grep -rn "@Scheduled" --include='*.java'                          # temporal surface
grep -rn "@Transactional" --include='*.java' -l                   # tx boundaries (flow §5)
grep -rn "@Entity\|@Table" --include='*.java' -l                  # data model
find . -path '*resources*' -name 'application*.yml' -o -name 'application*.properties'
find . -path '*db/migration*' -o -path '*changelog*' | sort       # flyway/liquibase timeline
grep -rn "@Configuration\|@Bean" --include='*.java' -l | head     # DI wiring map
grep -rn "@CircuitBreaker\|@Retry\|@RateLimiter\|Resilience4j" --include='*.java'  # seam resilience
```
God-module signal: packages named common/core/util/shared; check import
frequency with `grep -rn "import com.x.common" | wc -l` per candidate.

## Go

Detect: go.mod (go.work = monorepo of modules).

```bash
go list ./... | head -30
go mod graph | head -40
go vet ./... 2>&1 | head
grep -rn "func main(" --include='*.go'                       # entry census
grep -rn "http.HandleFunc\|mux\|gin\.\|echo\.\|chi\." --include='*.go' -l  # HTTP surface
grep -rn "sarama\|kafka-go\|amqp\|nats" --include='*.go' -l  # async surface
grep -rn "go func" --include='*.go' | wc -l                  # concurrency density
grep -rn "context.WithTimeout\|WithDeadline" --include='*.go' -l  # timeout discipline at seams
govulncheck ./... 2>/dev/null | head                         # risk register input
```

## Python (Django / FastAPI / Flask)

Detect: pyproject.toml / requirements.txt / setup.py.

```bash
cat pyproject.toml 2>/dev/null || cat requirements*.txt
pydeps <package> --max-bacon 2 --show-cycles 2>/dev/null     # pip install pydeps
grep -rn "@app.route\|@router\|APIRouter\|path(" --include='*.py' -l   # HTTP surface
grep -rn "celery\|@task\|@shared_task" --include='*.py' -l   # async surface
find . -path '*migrations*' -name '*.py' | sort              # migration timeline (Django)
grep -rn "class.*models.Model\|Base):" --include='*.py' -l   # data model
grep -rn "settings\." --include='*.py' | wc -l               # config surface density
```

## Monorepo layer (any stack)

```bash
cat turbo.json nx.json pnpm-workspace.yaml lerna.json go.work 2>/dev/null
# Build graph = the REAL package dependency structure:
npx turbo run build --dry=json 2>/dev/null | jq '.tasks[].dependencies' | head
npx nx graph --file=graph.json 2>/dev/null
```
Map: which packages are apps vs libs, which libs are imported by every app
(platform layer — master these first), which packages share a datastore
(hidden coupling that the build graph won't show).

## Infra & runtime truth (any stack)

```bash
find . -name '*.tf' -o -name '*.yaml' -path '*k8s*' -o -name 'helm' -type d | head
grep -rn "image:\|replicas:\|resources:" --include='*.y*ml' k8s/ helm/ 2>/dev/null | head
grep -rn "ENV \|ARG " Dockerfile* 2>/dev/null                # config surface
```
Cross-check: does the infra reference services/queues/DBs that Phase 1's
code recon never found? That gap is either dead infra or a component you
missed — resolve before writing 01-system-map.
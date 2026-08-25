# Sessions 8–9 — Two-Day Hackathon: Build, Containerise & Automate a Two-Service Application — Trainer Script

The course capstone, run as a **hackathon**. Over two days, teams scaffold a two-microservice application (Express + SQL, models/controllers/routers, no frontend), containerise it, build a pipeline that produces Docker images on every push, and write Terraform that provisions a VM with proper networking.

There is **no new technology**. Every skill was taught in Sessions 1–7. The challenge is doing it themselves, at pace, with the tools connected.

---

## 📦 STARTER CODE — put this in the repo before training

Everything goes into **`/hackathon/starter-code`** before Day 1.

**Ship the setup, withhold the build.** The local Postgres, the reference scaffold script and the notes template are all things that would otherwise eat time without teaching anything. The services, the `Jenkinsfile` and the Terraform are the hackathon — teams write those.

<br>

**`README.md`**
```markdown
# Two-Day Hackathon — Starter Code

## The brief, in one line

Build TWO Express microservices backed by SQL, containerise them,
automate the image build in a pipeline, and provision a VM with
Terraform to run them on.

## What's here

- **docker-compose.yml** — a local Postgres, so you're not fighting
  a database install on day one. Run it and forget it.

- **reference/scaffold** — the scaffold script from Session 2.
  Use it, extend it, or write a better one. Your choice.

- **db/init.sql** — creates the two databases. Loaded automatically
  by docker-compose on first run.

- **infra/main.tf** — a starting point. YOU add the resources.

- **commands.md** — every command you'll need across two days.

- **hackathon-notes.md** — your team's decision log. Fill it in
  as you go; you'll present from it.

- **completed-code/** — CATCH-UP ONLY. Opening this on day one
  means you've skipped the entire point.

## Before we start

    docker --version
    docker compose version
    node --version
    az account show
    terraform version
```

<br>

**`docker-compose.yml`** *(local Postgres, so nobody loses a morning to a database install)*
```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: hackathon-db
    environment:
      POSTGRES_USER: hackathon
      POSTGRES_PASSWORD: hackathon
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

<br>

**`db/init.sql`** *(runs automatically the first time the container starts)*
```sql
-- Two databases, one per service.
-- Each service owns its own data. Neither reads the other's tables.

CREATE DATABASE countries_db;
CREATE DATABASE cities_db;
```

<br>

**`reference/scaffold`** — copy in the finished `scaffold` script from Session 2's solution. Teams may reuse it, extend it, or replace it.

<br>

**`infra/main.tf`** *(a skeleton — teams add the resources)*
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

variable "prefix" {
  type        = string
  description = "Your name or team name — keeps resource names unique"
  # default   = "team-alpha"
}

variable "location" {
  type    = string
  default = "uksouth"
}

# TODO: azurerm_resource_group
# TODO: azurerm_virtual_network
# TODO: azurerm_subnet
# TODO: azurerm_network_security_group  + rules for SSH (22) and your API ports
# TODO: azurerm_public_ip
# TODO: azurerm_network_interface  + NSG association
# TODO: azurerm_linux_virtual_machine
# TODO: outputs — at minimum the public IP
```

<br>

**`commands.md`**
```markdown
# Command Reference — Hackathon

## Local database

    docker compose up -d          Start Postgres in the background
    docker compose ps             Is it running?
    docker compose logs postgres  Why won't it start?
    docker compose down           Stop it
    docker compose down -v        Stop it AND delete the data

    # Connect to it directly:
    docker exec -it hackathon-db psql -U hackathon -d countries_db

## Node / Express

    npm install
    npm start
    npm test
    curl localhost:3000/countries        Test an endpoint

## Docker

    docker build -t <name>:<tag> .
    docker run -p 3000:3000 <name>:<tag>
    docker ps / docker ps -a
    docker logs <container>
    docker exec -it <container> sh
    docker push <username>/<name>:<tag>

    # Building on an Apple Silicon Mac? You MUST do this:
    docker build --platform linux/amd64 -t <name>:<tag> .

## Terraform

    terraform init / validate / fmt
    terraform plan
    terraform apply
    terraform destroy
    terraform output -raw <name>

## Azure

    az account show --query id -o tsv
    az resource list -o table
    az group list -o table
    curl ifconfig.me              Your public IP — for the SSH NSG rule

## SSH

    chmod 400 key.pem
    ssh -i key.pem azureuser@<public-ip>
```

<br>

**`hackathon-notes.md`** *(the decision log — they present from this)*
```markdown
# Team: ______________________

## Our two services

| Service | What it does | Port | Database |
|---|---|---|---|
| | | | |
| | | | |

## Decisions we made, and why

| Decision | What we chose | Why |
|---|---|---|
| Scaffold: reuse Session 2 script, extend it, or write new? | | |
| One database or two? | | |
| Do the services talk to each other? | | |
| Image registry: Docker Hub or ACR? | | |
| One pipeline or two? | | |
| Terraform: one VM or two? | | |

## Blockers we hit, and how we fixed them

| Blocker | Fix |
|---|---|
| | |

## What we'd do differently with another two days

>
```

<br>

**`completed-code/`** — a working reference implementation of both services, a `Jenkinsfile`, and finished Terraform. **Mark clearly as catch-up only**, and tell students on Day 1 that opening it early means skipping the point.

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

**Post reactively.** In a hackathon the rule is stricter than in a taught session: **post a snippet only after a team has genuinely struggled for ten minutes.** A snippet posted early removes the learning; posted late it removes only the frustration.

| # | When | What to post |
|---|---|---|
| 1 | Day 1, 09:05 | Pre-flight check |
| 2 | Day 1, 09:30 | `docker compose up` + how to check Postgres is alive |
| 3 | Day 1, ~10:00 | The `pg` connection snippet — reactive only |
| 4 | Day 1, ~11:30 | A working router/controller/model example — reactive only |
| 5 | Day 1, 14:15 | A working Dockerfile for a Node service |
| 6 | Day 1, ~15:00 | The Apple Silicon `--platform` warning — **proactive** |
| 7 | Day 1, 15:45 | Docker Hub login + push |
| 8 | Day 2, 09:15 | **The Jenkins image with Docker + Terraform** — proactive |
| 9 | Day 2, ~11:00 | A two-service `Jenkinsfile` skeleton — reactive |
| 10 | Day 2, 14:15 | Terraform VM + NSG blocks — reactive |
| 11 | Day 2, 16:45 | Teardown |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### How this document differs from Sessions 1–7

Those are **lecture scripts**, written to be read from the front. **This one isn't.** After a 20-minute kickoff you are a **facilitator**, not a lecturer.

So the recurring sections are adapted:

- **Instead of ASK/ANSWER** → **DIAGNOSTIC QUESTIONS**: what to ask a stuck team so *they* find the answer, with the answer you're steering towards
- **Instead of Solution blocks** → **UNBLOCKING SNIPPETS**: reference code to paste when a team is genuinely stuck. Not a model answer — every team's build will differ, and should
- **Instead of HANDS ON** → **CHECKPOINTS**, with what "good" looks like and what to do about teams that aren't there

**Run-from annotations still apply**, because you'll be typing alongside people at their machines.

### The brief

Teams of **2–3**. Over two days:

**Day 1 — Build it.**
1. Scaffold **two** Express services with models, controllers and routers — reusing or extending their Session 2 script
2. Back each with a **SQL database**, with a schema and seed data
3. Get both running locally and responding to `curl`
4. **Containerise** both
5. Push both images to a registry, by hand, once

**Day 2 — Automate it.**
6. A **Jenkins pipeline** that, on every push, tests and builds **both** Docker images and pushes them
7. **Terraform** provisioning a resource group, VNet, subnet, NSG, public IP, NIC and a **Linux VM**
8. **Extension:** the pipeline runs the Terraform too, with an approval gate

**No frontend.** `curl` and the Jenkins console are the only interfaces that matter.

### The application

To keep everyone building the same *shape* while leaving the interesting decisions open, the domain is fixed:

| Service | Owns | Example endpoints |
|---|---|---|
| **countries-service** | `countries` table | `GET /countries`, `GET /countries/:id`, `POST /countries` |
| **cities-service** | `cities` table | `GET /cities`, `GET /cities/:id`, `POST /cities` |

A city has a `country_id`. **Whether the two services actually talk to each other is a team decision** — see the diagnostic question in the 10:45 block.

**NOTE FOR TRAINERS — why this domain** <br>
`countries` is deliberately the same resource the Session 2 scaffold script generated. Teams that built a good script get a genuine head start, which is exactly the reward you want for having done the work properly. Teams that didn't will need to write or fix one — which is a fair consequence, and the reference script in `starter-code/reference/` stops it becoming fatal. <br>
**END OF NOTE**

### Duration & schedule

Two days, **09:00–17:00** each, with a one-hour lunch and two 15-minute breaks per day.

**Day 1**

| Time | Session |
|---|---|
| 09:00–09:20 | Kickoff: the brief, checkpoints, support model |
| 09:20–10:30 | Scaffold both services |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Make them actually work: SQL, seed data, live endpoints |
| 12:30–13:00 | **Checkpoint 1** — trainer round |
| **13:00–14:00** | **Lunch** |
| 14:00–15:15 | Containerise both services |
| **15:15–15:30** | **Break** |
| 15:30–16:30 | Push images by hand; start the Terraform |
| 16:30–17:00 | **Checkpoint 2** — trainer round and catch-up triage |

**Day 2**

| Time | Session |
|---|---|
| 09:00–09:15 | Standup: yesterday's blockers, today's plan |
| 09:15–10:30 | Jenkins: checkout, test, build both images |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Push both images; trigger on push |
| 12:30–13:00 | **Checkpoint 3** — trainer round |
| **13:00–14:00** | **Lunch** |
| 14:00–15:00 | Terraform: resource group, network, NSG, VM |
| **15:00–15:15** | **Break** |
| 15:15–15:45 | Extension: pipeline runs Terraform / peer review |
| 15:45–16:00 | **Checkpoint 4** + demo prep |
| 16:00–17:00 | **Final showcase** |

### Set-up

#### For Trainers

Plan your two days as a **facilitator**, not a presenter.

- **Prepare the kickoff** (20 minutes, hard limit). Everything after that is roaming, reviewing and unblocking
- **Schedule yourself around the four checkpoints** so you're free to see every team as they hit them
- **Prepare a shared space** — a Slack channel — for teams to log where they're stuck. **This is the single highest-value piece of prep**: it lets you spot a blocker three teams share and fix it once to the room instead of five times one-to-one
- **Have the unblocking snippets queued** in a draft message before Day 1
- **Book the showcase properly.** It matters more than it looks

**NOTE FOR TRAINERS — running the room** <br>
Two failure modes, opposite-looking, same fix. <br>
**The team that won't ask.** They'll sit quietly failing for ninety minutes. The shared channel exists mostly for them — make posting in it explicitly normal by **posting in it yourself during the kickoff**. <br>
**The team that asks immediately.** They'll route every error straight to you without reading it. Answer with a **diagnostic question**, not a fix: *"what does the error actually say?"* and *"which layer is that — app, container, network, or cloud?"* Your job across two days is to make yourself progressively less necessary. <br>
**END OF NOTE**

#### For Students

- **Direct** students to the starter code
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/hackathon/starter-code**
- **Make sure**, before Day 1 starts, every student has:
  - **Docker Desktop** installed and running
  - **Node 20+**
  - A **GitHub account**, and the ability to create a new repository
  - A **Docker Hub** account with an access token
  - An **Azure subscription** with credit, and their **Service Principal** from Session 4
  - A working **Jenkins** — their own image from Session 7, or a shared one you provide
  - Their **Session 2 scaffold script**, if they still have it

**NOTE FOR TRAINERS — pre-flight** <br>
Have teams run this at 09:05 on Day 1, before the kickoff finishes. Ten minutes here saves an hour of scattered firefighting. <br>
**END OF NOTE**

**💬 SLACK — snippet 1**, post at 09:00 Day 1:
```bash
mkdir -p ~/hackathon && cd ~/hackathon

docker --version            # Docker installed and running?
docker compose version      # compose available?
node --version              # Node 20+?
git --version
az account show             # signed in to Azure?
terraform version
export | grep ARM           # Service Principal creds present?
```

Anything blank or erroring gets fixed **before** that team starts building.

## Learning objectives

- **Apply** every skill from Sessions 1–7 on one project, independently and at pace
- **Design and justify** their own architecture and tooling decisions rather than following a prescribed path
- **Produce** two working containerised microservices backed by SQL
- **Produce** a pipeline that builds and publishes both images automatically on every push
- **Produce** Terraform provisioning a VM with appropriate networking and security rules
- **Troubleshoot** across the full stack without step-by-step instructions
- **Present** and demonstrate a finished solution clearly to peers

<br>

---

## Day 1: Build It

### 09:00–09:20 — Kickoff
*(The only part of two days you deliver from the front. Keep to 20 minutes.)*

Morning. Two days, and they work differently from everything else on this course, in one specific way: **I'm not going to teach you anything new.**

There's no new tool. Every skill you need, you already have — you wrote a scaffold script in Session 2, you built pipelines in Session 3, you wrote Terraform in Sessions 4 and 5, you've containerised things all course. **The only variable that changes today is that nobody is walking you through it.**

**Here's the brief.**

By tomorrow afternoon you'll demo, to this room:

- **Two Express microservices**, each with models, controllers and routers, each backed by a SQL database
- **Both containerised**, images published to a registry
- **A pipeline** that rebuilds and pushes both images automatically when you push code
- **Terraform** that provisions a VM with a resource group, network and security rules

No frontend. `curl` is your UI.

The domain is fixed so we're all building the same *shape*: a **countries service** and a **cities service**. A city belongs to a country. **Whether those two services talk to each other is your call** — and I'll ask you to justify it.

Three things about how this runs:

**One — the interesting choices are yours.** Reuse your Session 2 scaffold script or write a better one. One database or two. One pipeline or two. Docker Hub or ACR. I'll tell you if something won't work; I won't tell you which to pick. **In the showcase I'll ask *why*** — and *"it was simpler and we didn't need more"* is a genuinely good answer. *"It's what we did in the session"* is not.

**Two — being stuck is the work.** Not a sign it's going wrong. Two days of an unfamiliar problem with no instructions is much closer to the job than any session we've run. When you're stuck, **post in the channel** — where you are, what you expected, what actually happened. I'd rather see fifteen messages in there than fifteen people quietly failing.

**Three — four checkpoints**, around midday and end of day, both days. Not tests. Me checking in so nobody discovers at 4pm tomorrow that something broke this morning. **Being behind at a checkpoint is useful information, not a problem.**

**DIAGNOSTIC QUESTION** *(ask the room, take a few answers)* <br>
Before you touch anything — what's the **first** thing you should get working? <br>
**STEERING TOWARDS** <br>
**One service, running locally, returning JSON from a database.** Nothing else. <br>
The most common way to lose a day here is trying to debug in the cloud something that was never working on your laptop. Every layer you add — a second service, a container, a registry, a VM, a pipeline — is another place the failure could be. **Start with zero layers and add one at a time.**

Right. Get the database up, decide who's driving first, and start. I'll come round.

**💬 SLACK — snippet 2**, post immediately after the kickoff:
```bash
cd ~/hackathon/<your-repo>

# Start the local Postgres (it creates both databases on first run)
docker compose up -d
docker compose ps                 # should show 'running'
docker compose logs postgres      # if it isn't

# Prove you can reach it:
docker exec -it hackathon-db psql -U hackathon -d countries_db -c "\l"
```

<br>

### 09:20–10:30 — Scaffold Both Services

Teams confirm their repo and get **two service folders** generated.

The target structure — roughly what Session 2's script produced, twice:

```
.
├── docker-compose.yml
├── db/
│   └── init.sql
├── countries-service/
│   ├── package.json
│   ├── Dockerfile
│   └── server/
│       ├── index.js
│       ├── app.js
│       ├── controllers/countries.js
│       ├── models/Country.js
│       ├── routers/countries.js
│       └── db/
│           ├── connect.js
│           ├── countries.sql
│           └── setup.js
└── cities-service/
    └── ... same shape, for cities
```

**DIAGNOSTIC QUESTIONS — scaffolding** <br>

*"Do you still have your Session 2 script?"* → If yes, this block takes ten minutes and they should spend the rest on the SQL. If no, `reference/scaffold` in the starter repo is theirs to use. **Don't let a missing script cost a team the morning** — but do note who's benefiting from having kept theirs.

*"What does your script need to change to generate a service rather than an app?"* → Mostly nothing. It already takes a resource name and a model name. The interesting question is whether they parameterise the **port** and the **database name**, since two services can't both listen on 3000.

*"Are you running it twice, or extending it to take multiple resources?"* → Both are fine. Running it twice is faster today; extending it is the better script. Ask which they'd want if they had to do this ten more times.

**UNBLOCKING SNIPPET** — extending the scaffold to take a port. **`SLACK`**: reactive only.
```bash
# Add a third argument for the port, defaulting to 3000
resource="$1"
model="${2:-${resource^}}"
port="${3:-3000}"

# ...then in the package.json heredoc:
#   "start": "PORT=${port} node server/index.js"
# ...and in server/index.js, it already reads process.env.PORT
```

<br>

*(Take a 15 minute break — 10:30.)*

<br>

### 10:45–12:30 — Make Them Actually Work

Scaffolding produces files. This block makes them **respond to a request with data from a database**.

Three things per service:
1. A **schema and seed data** in the `.sql` file
2. A **working `connect.js`** pointing at the local Postgres
3. At least `GET /<resource>` and `GET /<resource>/:id` returning real rows

**Nothing else happens until this returns JSON:**

*(Run from `~/hackathon/<your-repo>/countries-service`)*
```bash
npm install
npm run db:setup
npm start
```

*(In a second terminal, run from anywhere)*
```bash
curl localhost:3000/countries
```

**DIAGNOSTIC QUESTIONS — "it won't connect to the database"** <br>
The big one of Day 1. **Work the layers, smallest first.** Don't let them skip to the middle.

*"Is Postgres actually running?"* → `docker compose ps`. If not, `docker compose logs postgres`.

*"Can you reach it outside your app?"* → `docker exec -it hackathon-db psql -U hackathon -d countries_db -c "\dt"`. **If this works but the app doesn't, it's a connection-string problem, not a database problem.** That single test halves the search space.

*"What's your connection string, out loud?"* → Nine times in ten it's the wrong database name (`postgres` instead of `countries_db`), or a missing password, or `localhost` versus `127.0.0.1`.

*"Did you run the setup script?"* → Empty table versus missing table are different errors. `psql -c "\dt"` tells them which.

**UNBLOCKING SNIPPET** — a working `connect.js` and connection string. **`SLACK`**: reactive, after 10 minutes.
```javascript
// server/db/connect.js
const { Pool } = require("pg");

const db = new Pool({
  connectionString:
    process.env.DATABASE_URL ||
    "postgres://hackathon:hackathon@localhost:5432/countries_db",
});

module.exports = db;
```
```bash
# The cities service uses the SAME user and password, DIFFERENT database:
#   postgres://hackathon:hackathon@localhost:5432/cities_db
```

**UNBLOCKING SNIPPET** — a working router/controller/model set. **`SLACK`**: reactive only, and only after they've attempted it.
```javascript
// server/models/Country.js
const db = require("../db/connect");

class Country {
  static async findAll() {
    const result = await db.query("SELECT * FROM countries ORDER BY id");
    return result.rows;
  }

  static async findById(id) {
    const result = await db.query("SELECT * FROM countries WHERE id = $1", [id]);
    return result.rows[0];
  }
}

module.exports = Country;
```
```javascript
// server/controllers/countries.js
const Country = require("../models/Country");

async function index(req, res) {
  const rows = await Country.findAll();
  res.json(rows);
}

async function show(req, res) {
  const row = await Country.findById(req.params.id);
  if (!row) return res.status(404).json({ error: "Not found" });
  res.json(row);
}

module.exports = { index, show };
```
```javascript
// server/routers/countries.js
const express = require("express");
const controller = require("../controllers/countries");

const router = express.Router();
router.get("/", controller.index);
router.get("/:id", controller.show);

module.exports = router;
```

**DIAGNOSTIC QUESTION — the architecture decision** <br>
Ask every team at some point in this block: *"Does your cities service need to know anything about countries?"* <br>
**STEERING TOWARDS** <br>
Three valid answers, and they should pick deliberately: <br>
**(a) No.** `cities` stores a `country_id` and returns it. The client joins. Simplest, most decoupled, and genuinely how a lot of real systems work. <br>
**(b) Yes, over HTTP.** The cities service calls `GET /countries/:id` on the countries service to enrich its response. More realistic microservices — and it introduces service discovery, timeouts and a failure mode when the other service is down. **Ask them what happens if countries-service is unavailable.** <br>
**(c) Yes, via the database.** One database, a SQL `JOIN`. **Simplest to build and the worst answer architecturally** — the two services are now coupled through a shared schema, so neither can change its tables independently. Worth letting a team choose it and then asking *"what happens when the countries team renames a column?"* <br>
No wrong answer, but a **thoughtless** answer is worth challenging.

<br>

### 12:30–13:00 — ✅ Checkpoint 1

Go round every team. Looking for:

- [ ] A GitHub repo with **both** service folders scaffolded
- [ ] Postgres running via `docker compose`, with both databases created
- [ ] **At least one** service returning real JSON from a real table via `curl`
- [ ] A written answer in `hackathon-notes.md` to "do the services talk to each other, and why?"

**What "behind" looks like, and what to do:**

| Situation | Action |
|---|---|
| Nothing scaffolded yet | Give them `reference/scaffold` and have them run it twice. Ten minutes, then move on |
| Scaffolded but no database connection | Work the layer questions with them. **10 minutes, not 40** |
| One service working, second not started | **Fine.** That's on track. Push them to finish the second over the afternoon |
| Both working already | Push them at the architecture decision, and at writing a test — they'll need `npm test` tomorrow |
| Nothing working at all | Pair them with a team that's ahead for the afternoon. **Better a strong trio than a lost pair** |

<br>

*(Lunch — 13:00–14:00.)*

<br>

### 14:00–15:15 — Containerise Both Services

Each service needs its own `Dockerfile`, and each needs to run in a container and still reach the database.

*(Run from `~/hackathon/<your-repo>/countries-service`)*
```bash
docker build -t countries-service:local .
docker run -p 3000:3000 countries-service:local
```

**And then it won't connect to the database.** That's expected, and it's the most valuable ten minutes of Day 1.

**DIAGNOSTIC QUESTIONS — "it worked outside the container and not inside"** <br>

*"Inside the container, what does `localhost` mean?"* → **The container itself.** Not your laptop. So a connection string pointing at `localhost:5432` is looking for a Postgres **inside the API container**, which doesn't exist. This catches everyone once and is worth letting them hit.

*"So how does a container reach another container?"* → Either by **Docker network and service name** (`postgres:5432`, if both are on the same compose network), or via the **host** (`host.docker.internal:5432` on Docker Desktop). Steer them towards adding their services to `docker-compose.yml` — it's the cleaner answer and it makes the whole stack startable with one command.

*"Is your app listening on 0.0.0.0 or 127.0.0.1?"* → An app bound to `127.0.0.1` inside a container is unreachable from outside it, even with correct port mapping. Express defaults to all interfaces, so this is rarer here than in other stacks — but check it if the port mapping looks right and nothing responds.

**UNBLOCKING SNIPPET** — a working Node Dockerfile. **`SLACK`**: snippet 5, post at 14:15.
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
```
# .dockerignore
node_modules
.git
```

**UNBLOCKING SNIPPET** — adding the services to compose so they share a network. **`SLACK`**: reactive.
```yaml
services:
  postgres:
    # ...as before...

  countries-service:
    build: ./countries-service
    ports:
      - "3000:3000"
    environment:
      # 'postgres' is the SERVICE NAME — Docker resolves it on the shared network
      DATABASE_URL: postgres://hackathon:hackathon@postgres:5432/countries_db
    depends_on:
      - postgres

  cities-service:
    build: ./cities-service
    ports:
      - "3001:3000"
    environment:
      DATABASE_URL: postgres://hackathon:hackathon@postgres:5432/cities_db
    depends_on:
      - postgres
```

**NOTE FOR TRAINERS** <br>
**Watch for Apple Silicon here.** An image built on an M-series Mac defaults to `linux/arm64` and will fail on a standard Azure VM with `exec format error` — which surfaces **tomorrow**, at deploy time, looking like a completely unrelated problem. **Catch it today**, proactively, before anyone pushes an image. <br>
**END OF NOTE**

**💬 SLACK — snippet 6**, post **proactively** at ~15:00 Day 1:
```bash
# Building on an Apple Silicon Mac? Build for the cloud's architecture,
# or your image will fail on an Azure VM tomorrow with "exec format error":

docker build --platform linux/amd64 -t <username>/countries-service:v1 .
```

<br>

*(Take a 15 minute break — 15:15.)*

<br>

### 15:30–16:30 — Push Images by Hand; Start the Terraform

Two things this block, deliberately in this order.

**1. Push both images to a registry, by hand, once.** Proving the whole chain works manually before automating any of it tomorrow.

*(Run from `~/hackathon/<your-repo>/countries-service`)*
```bash
docker login
docker build --platform linux/amd64 -t <username>/countries-service:v1 .
docker push <username>/countries-service:v1
```

Then the same for `cities-service`.

**2. Start the Terraform.** They won't finish it today, and that's fine — the goal is that nobody starts Day 2 with a blank `main.tf`.

**DIAGNOSTIC QUESTION — the registry decision** <br>
*"Docker Hub or Azure Container Registry?"* <br>
**STEERING TOWARDS** <br>
**Docker Hub** is faster to set up and they've used it. **ACR** is more realistic for an Azure shop, keeps images inside their subscription, and integrates with managed identity — but it's another Terraform resource and another authentication step. **Either is fine; the answer should be deliberate.** If a team picks ACR, make sure they know it's an extra thirty minutes and check they have the time.

**DIAGNOSTIC QUESTIONS — Terraform** <br>

*"What does a VM need around it before it can exist?"* → Resource group, VNet, subnet, NSG, public IP, NIC, NSG association. **Six resources before the VM.** If they can't list them, send them back to their Session 5 notes rather than giving them the answer.

*"Which ports does your NSG need open, and why?"* → 22 for SSH. Then **whichever ports their APIs listen on** — and this is the point where they realise two services on one VM need two ports, or a reverse proxy, or two VMs. Good decision to make consciously.

*"Where's your state?"* → Local is fine today. **Flag now** that if they attempt the Day 2 extension — pipeline runs Terraform — **they will need a remote backend**, and they should know that before 2pm tomorrow rather than at 3pm.

**💬 SLACK — snippet 7**, post at 15:45:
```bash
docker login

# Build for the right architecture and tag with your username
docker build --platform linux/amd64 -t <username>/countries-service:v1 .
docker push <username>/countries-service:v1

cd ../cities-service
docker build --platform linux/amd64 -t <username>/cities-service:v1 .
docker push <username>/cities-service:v1

# Confirm they're there:
#   hub.docker.com > your repositories
```

<br>

### 16:30–17:00 — ✅ Checkpoint 2 & Catch-up Triage

- [ ] **Both** services returning JSON from their databases
- [ ] **Both** services containerised and running in containers
- [ ] **Both** images pushed to a registry
- [ ] `infra/main.tf` started — at minimum a resource group and a VNet
- [ ] They can explain, **out loud**, why their containerised service couldn't reach `localhost`

That last one isn't a formality. A team that fixed it by copying a snippet without understanding will hit the same class of problem tomorrow when the VM can't reach something.

**DIAGNOSTIC QUESTION — worth asking every team** <br>
*"If I deleted your laptop right now, how much of today could you get back, and how?"* <br>
**STEERING TOWARDS** <br>
The code is in Git and the images are in a registry — so quite a lot. **What isn't recoverable is anything they did by hand and didn't write down.** It's a good frame for Day 2: **everything you did manually today becomes a stage in tomorrow's pipeline.**

**NOTE FOR TRAINERS — this is the important half hour of Day 1** <br>
Anyone significantly behind gets pulled aside **now**, not tomorrow morning. Twenty minutes here is worth far more than a whole day of them stuck. <br>
Concretely: if a team has no working containerised service by 17:00, **cut scope for them**. One service instead of two is a perfectly good Day 2 for a struggling team — **the Day 2 learning is the pipeline**, and losing that because a second service didn't containerise is the worst outcome available. <br>
**END OF NOTE**

**Costs — say this before anyone leaves.** Nothing should be running in Azure yet, but check:

*(Run from anywhere)*
```bash
az resource list -o table
```

<br>

---
## Day 2: Automate It

### 09:00–09:15 — Standup

Short and structured. Round the room, thirty seconds per team: **where you got to, what's blocking you, what you're doing first today.**

Two purposes. It surfaces overnight blockers before they cost an hour, and it lets the room hear that **other teams are stuck too** — which materially changes whether people ask for help.

Then set the day: **the pipeline is the point of today.** Everything from yesterday becomes a stage.

**DIAGNOSTIC QUESTION** *(to the room)* <br>
Yesterday you built and pushed two images by hand. **List the commands you typed, in order.** <br>
**STEERING TOWARDS** <br>
`npm install`, `npm test`, `docker build`, `docker push` — twice, once per service. **That's the pipeline.** There's nothing to design; the stages are yesterday's commands, in order, with the exit codes taken seriously. <br>
Framing it this way removes most of the intimidation, and it's true: **a `Jenkinsfile` is a list of things you already know how to type.**

<br>

### 09:15–10:30 — Jenkins: Checkout, Test, Build Both Images

The `Jenkinsfile`, committed to their repo. Built **one stage at a time**, each passing before the next is added.

Say this explicitly and repeat it: *a pipeline with five stages you built and tested one at a time is trivially debuggable; five stages written at once and pushed is not.*

Stage order for this block:
1. Checkout
2. Install and test — **both services**
3. Build both Docker images

**DIAGNOSTIC QUESTIONS — pipeline** <br>

*"Does your Jenkins have the tools it needs?"* → **The single biggest time sink of Day 2.** Stock `jenkins/jenkins:lts` has no Docker CLI and no Terraform. Every `command not found` traces here. Ask this **before** they write a single stage.

*"Where are your credentials?"* → If any secret is in the `Jenkinsfile`, **stop them immediately.** It's a success criterion, and Git history is permanent.

*"One pipeline or two?"* → Genuinely open. **One pipeline building both** is simpler and guarantees the two services are always built from the same commit. **Two pipelines** means a change to one service doesn't rebuild the other — faster, and closer to how independent microservices are usually handled. Ask them which problem they'd rather have.

*"How are you handling two services in one repo?"* → `dir('countries-service') { ... }` around each set of steps. If they've split into two repos, that's a legitimate choice with its own trade-offs — ask them to justify it.

**UNBLOCKING SNIPPET** — a Jenkins image with Docker and Terraform. **`SLACK`**: snippet 8, post **proactively** at 09:15. Nobody should lose an hour to this.
```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y curl unzip gnupg lsb-release docker.io \
    && rm -rf /var/lib/apt/lists/*
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg \
      | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
      > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform && rm -rf /var/lib/apt/lists/*
USER jenkins
```
```bash
docker build -t jenkins-devops .
docker rm -f jenkins
docker run -d --name jenkins -p 8080:8080 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root jenkins-devops

# VERIFY BEFORE WRITING ANY STAGES:
docker exec jenkins docker --version
docker exec jenkins terraform version
```

<br>

*(Take a 15 minute break — 10:30.)*

<br>

### 10:45–12:30 — Push Both Images; Trigger on Push

Continuing one stage at a time:

4. Push both images, tagged with `$BUILD_NUMBER`
5. Trigger the pipeline **from a Git push**, not a manual build

**DIAGNOSTIC QUESTIONS** <br>

*"How is each image tagged?"* → If they're all `:latest`, ask how they'd know **which build produced a running container**. Steer to `$BUILD_NUMBER`, and optionally `latest` as well.

*"Does the pipeline actually know if the tests failed?"* → For anyone whose test stage passes suspiciously fast. Have them break a test deliberately and watch the pipeline go red. **If it doesn't, their test isn't exiting non-zero** — which is the Session 2 lesson arriving late.

*"Manual build or real push?"* → A pipeline that only runs on **Build Now** isn't CI. Webhooks can't reach `localhost`, so **Poll SCM** (`H/2 * * * *`) is the reliable classroom answer.

**UNBLOCKING SNIPPET** — a two-service `Jenkinsfile` skeleton. **`SLACK`**: snippet 9, reactive only.
```groovy
pipeline {
    agent any

    environment {
        DOCKER_USER_NAME = "your-dockerhub-username"
        IMAGE_TAG        = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Test Both Services') {
            parallel {
                stage('countries') {
                    agent { docker { image 'node:20' } }
                    steps { dir('countries-service') { sh 'npm install'; sh 'npm test' } }
                }
                stage('cities') {
                    agent { docker { image 'node:20' } }
                    steps { dir('cities-service') { sh 'npm install'; sh 'npm test' } }
                }
            }
        }

        stage('Build Images') {
            steps {
                dir('countries-service') {
                    sh 'docker build -t $DOCKER_USER_NAME/countries-service:$IMAGE_TAG .'
                }
                dir('cities-service') {
                    sh 'docker build -t $DOCKER_USER_NAME/cities-service:$IMAGE_TAG .'
                }
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_USER_NAME/countries-service:$IMAGE_TAG'
                    sh 'docker push $DOCKER_USER_NAME/cities-service:$IMAGE_TAG'
                }
            }
        }
    }

    post {
        success { echo "Pushed both services, build ${BUILD_NUMBER}" }
        failure { echo "FAILED — see ${BUILD_URL}console" }
    }
}
```

<br>

### 12:30–13:00 — ✅ Checkpoint 3

- [ ] A full pipeline run triggered by a **real `git push`**, not a manual build
- [ ] Every stage passing
- [ ] **Both** images visible in their registry, tagged with the build number
- [ ] They can point at where a secret is stored, and it isn't in the repo

**The checkpoint action:** *"Change something in one of your services, push it, and let me watch."* Nothing else demonstrates it as convincingly, and it's a dry run for the showcase.

**Triage for anyone not there:**

| Situation | Action |
|---|---|
| Works on **Build Now**, not on push | Trigger config. Poll SCM `H/2 * * * *` is the reliable fallback |
| `docker: not found` in the pipeline | Custom Jenkins image, or the socket mount. Give them snippet 8 |
| One service building, not the other | Fine. Get one **fully** working — build *and* push — before adding the second |
| Nowhere near | **Cut scope.** Drop the test stage and one service. **Get checkout → build → push working for one service.** A three-stage pipeline they understand beats a seven-stage one they don't |

<br>

*(Lunch — 13:00–14:00.)*

<br>

### 14:00–15:00 — Terraform: Resource Group, Network, NSG, VM

Now the infrastructure. **Deliberately basic** — a resource group, a virtual network, a subnet, a network security group with sensible rules, a public IP, a NIC, and one Linux VM.

They ran this by hand in Session 5. Today it's from their own `infra/main.tf`, and `terraform apply` runs **from their laptop** — automating it is the extension.

The target:

| Resource | Purpose |
|---|---|
| `azurerm_resource_group` | Container for everything |
| `azurerm_virtual_network` | The private network, e.g. `10.0.0.0/16` |
| `azurerm_subnet` | A slice of it, e.g. `10.0.1.0/24` |
| `azurerm_network_security_group` | The firewall |
| `azurerm_network_security_rule` × 2+ | SSH (22), plus their API ports |
| `azurerm_public_ip` | A reachable address |
| `azurerm_network_interface` | The virtual network card |
| `..._security_group_association` | Joins the NSG to the NIC |
| `azurerm_linux_virtual_machine` | The machine itself |
| `output` | At minimum, the public IP |

**DIAGNOSTIC QUESTIONS — Terraform** <br>

*"Which ports are you opening, and why each one?"* → 22 for SSH. Then their API ports — and the moment they realise **two services can't both use 3000 on one VM**. Options: map them to different host ports, use a reverse proxy, or run two VMs. **Any is fine; the decision should be conscious.**

*"Who can reach port 22?"* → If it's `0.0.0.0/0`, ask whether that's a decision or a default. `curl ifconfig.me` gives their own IP; `/32` restricts to exactly it. **Not required, but a team that does it has understood something.**

*"What's your plan actually saying?"* → For anyone applying without reading. `1 to add, 0 to change, 1 to destroy` on day two of a hackathon is worth noticing before pressing yes.

*"Are you running the containers on this VM, or just provisioning it?"* → **Provisioning is enough for the brief.** Actually running the containers on it is a legitimate stretch — but flag that it needs Docker installed on the VM (`custom_data` or a provisioner) and pulling from their registry. **Don't let a team lose the afternoon to it.**

**UNBLOCKING SNIPPET** — VM and NSG blocks. **`SLACK`**: snippet 10, reactive.
```tf
resource "azurerm_network_security_rule" "api_countries" {
  name                        = "AllowCountriesAPI"
  priority                    = 120
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "3000"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.rg.name
  network_security_group_name = azurerm_network_security_group.nsg.name
}

resource "azurerm_linux_virtual_machine" "app_vm" {
  name                  = "${var.prefix}-vm"
  resource_group_name   = azurerm_resource_group.rg.name
  location              = azurerm_resource_group.rg.location
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/azure/azure_ssh_keys/default-vm-ssh.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
}

output "vm_public_ip" {
  value = azurerm_public_ip.pip.ip_address
}
```

<br>

*(Take a 15 minute break — 15:00.)*

<br>

### 15:15–15:45 — Extension & Peer Review

Two tracks, depending where a team is. **Split the room.**

#### Track A — teams who finished early: the extension

Add Terraform stages to the pipeline. **This is the stated extension in the brief, and it's genuinely harder than it looks.**

**DIAGNOSTIC QUESTION — the extension's real blocker** <br>
*"Where is your Terraform state?"* <br>
**STEERING TOWARDS** <br>
**If it's local, this cannot work.** The Jenkins workspace starts empty every build, so Terraform believes nothing exists and plans to create everything — duplicating resources or failing on name collisions. Then the workspace is discarded and the next build repeats it. <br>
**They need a remote backend**, exactly as in Session 5. Let them hit the wall for five minutes first — the penny drops far harder that way — then point them at their Session 5 notes.

The stages to add:
```groovy
stage('Terraform Init')     { steps { dir('infra') { sh 'terraform init' } } }
stage('Terraform Validate') { steps { dir('infra') { sh 'terraform validate' } } }
stage('Terraform Plan')     { steps { dir('infra') { sh 'terraform plan -out=tfplan' } } }
stage('Terraform Apply') {
    steps {
        input message: 'Apply this plan?', ok: 'Apply'
        dir('infra') { sh 'terraform apply -auto-approve tfplan' }
    }
}
```

Plus the four `ARM_*` credentials in the `environment` block.

**DIAGNOSTIC QUESTION** <br>
*"Should the apply gate be there?"* <br>
**STEERING TOWARDS** <br>
For a hackathon, **yes** — it's a safety net while they're still learning, and it makes the demo better because the room watches someone decide. In a mature team the review would happen on the pull request instead. **Either answer is right if they can justify it.**

#### Track B — everyone else: peer review

Pair teams with a **different** team. 15 minutes each way.

Not looking for perfection. Two things only:
1. One thing they'd have done differently
2. One thing they're going to steal for their own next project

**NOTE FOR TRAINERS** <br>
Pair teams who made **different architecture decisions** wherever possible — one that had the services talk to each other with one that didn't, or one pipeline with two. A team reading a different approach learns more about the trade-off in fifteen minutes than from any amount of you explaining it. Same-shape pairs mostly just confirm each other's choices. <br>
**END OF NOTE**

<br>

### 15:45–16:00 — ✅ Checkpoint 4 & Demo Prep

- [ ] Terraform applied successfully — resources visible in the Azure Portal
- [ ] `.tf` files committed to the repo
- [ ] They can explain, **out loud**, every resource their Terraform creates and why it's needed
- [ ] `hackathon-notes.md` complete, including the decisions table
- [ ] Peer review done, or the extension attempted

Then five minutes of demo prep. Two practical warnings:

- **Have your change ready to push before you present.** Don't write it live in front of the room
- **Know how long your pipeline takes.** If it's four minutes, talk through the architecture while it runs rather than watching a progress bar in silence

<br>

### 16:00–17:00 — Final Showcase

Each team gets **6 minutes**:

1. **60-second architecture summary** — two services, what each does, what's running where
2. **A live demo** — `curl` both services, then make a small change, push it, and let the group watch the pipeline run
3. **One decision you made and why** — from your notes table
4. **One thing that went wrong**, and how you fixed it

**NOTE FOR TRAINERS** <br>
The "what went wrong" part is **deliberately in the brief, not optional colour.** It normalises debugging as core expected work rather than personal failure, and it's frequently the most useful five minutes for the room — chances are several teams hit the identical issue and each thought they were alone. <br>
**If a demo fails live, that's a gift.** Let them debug it in front of everyone for two minutes. It's the most realistic thing that will happen across two days, and **how you react sets whether the room reads failure as normal or shameful.** <br>
**END OF NOTE**

**Questions worth asking each team** — pick one or two, keep it light:
- *"Why did your services talk / not talk to each other?"*
- *"One pipeline or two, and would you choose the same again?"*
- *"What would break first if this got 100× the traffic?"*
- *"What's still manual that you'd automate next?"*
- *"If you started again on Monday, what would you do differently?"*

<br>

---

## Success Criteria

| Requirement | Evidence |
|---|---|
| Two Express services with models, controllers and routers | The repo structure, and `curl` returning JSON from both |
| Each backed by a SQL database with schema and seed data | The `.sql` files, and real rows in the responses |
| Both containerised | A `Dockerfile` per service; both running as containers |
| Images published | Both visible in Docker Hub or ACR, tagged per build |
| **Automated image builds** | A `Jenkinsfile` in the repo, triggered by a **real `git push`** |
| Secrets handled properly | **No credentials in Git** — Jenkins credentials store used |
| Infrastructure as code | `.tf` files in the repo; `terraform apply` runs cleanly |
| Network security configured | An NSG with **justified** rules — not `0.0.0.0/0` on everything by accident |
| **Extension:** pipeline runs Terraform | Terraform stages in the `Jenkinsfile`, with a remote backend |
| They can explain their own architecture | Verbal walkthrough at the showcase, without notes |

Not a pass/fail checklist marked in isolation — it's what you're looking for at each checkpoint, and what teams should be able to point at by the showcase.

<br>

---

## Stretch Goals

For teams finishing early:

- **Run the containers on the VM** — install Docker via `custom_data`, pull both images, run them
- **Add a `docker-compose.yml` on the VM** and start the whole stack there
- **Service-to-service calls** — have cities-service enrich its response from countries-service, and handle the failure case when it's unavailable
- **Real tests** — integration tests hitting a test database, gating the pipeline properly
- **Two pipelines** — one per service, so a change to one doesn't rebuild the other
- **Azure Container Registry** instead of Docker Hub, provisioned by their own Terraform
- **`when { branch 'main' }`** and a Multibranch Pipeline, so PRs only test and build
- **A `latest` tag** alongside the build number
- **Health endpoints** on both services, and an NSG rule allowing a monitoring source only

<br>

---

## Trainer's Quick Triage Reference

The failures you will actually see, roughly in the order you'll see them. Worth having open on a second screen across both days.

| Symptom | Almost always |
|---|---|
| `ECONNREFUSED` connecting to Postgres locally | Container not running, or wrong database name in the connection string |
| `relation "countries" does not exist` | The `db:setup` script was never run, or ran against the wrong database |
| Works with `npm start`, not in a container | `localhost` inside the container means the container. Needs the compose service name or `host.docker.internal` |
| Container exits immediately | **Read `docker logs`.** It's in there |
| `exec format error` on the VM | arm64 image on amd64. Rebuild with `--platform linux/amd64` |
| `denied: requested access to the resource is denied` | Image name doesn't start with their Docker Hub username |
| `docker: not found` in Jenkins | Stock Jenkins image; needs the custom build and the socket mount |
| Pipeline green on Build Now, not on push | Webhook can't reach localhost. Use Poll SCM |
| Both services trying to use port 3000 on one VM | Genuine design decision — different host ports, a proxy, or two VMs |
| Terraform plans to recreate everything | **Local state**, and they're running it from the pipeline. Needs a remote backend |
| App unreachable at the VM's public IP | NSG rule missing for that port, or the container isn't running. `curl localhost` **on the VM** splits app problems from network problems |
| Terraform state locked | A previous build still sitting at the approval gate |

<br>

---

## Conclusion

- **Inform** students this marks the end of the programme
- **Run** the showcase as a group activity, not a private trainer review — seeing several valid approaches to the same brief is a large part of the value
- **Collect** a short written retrospective from each team: what they'd do differently, and what they're proudest of
- **Confirm** everyone has torn down their Azure infrastructure — run `az resource list -o table` and `az group list -o table` as a room before anyone leaves
- **Direct** teams towards the stretch goals as self-directed follow-up

**💬 SLACK — snippet 11**, post at 16:45 Day 2:
```bash
# Tear down Azure — nothing should be left running
cd ~/hackathon/<your-repo>/infra
terraform destroy        # type: yes

az group list -o table
az resource list -o table

# Stop the local stack
cd ..
docker compose down

# Stop Jenkins if you want your laptop back
docker stop jenkins
```

---

[Back](./README.md)

---

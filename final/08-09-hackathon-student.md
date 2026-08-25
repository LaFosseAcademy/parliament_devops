# Two-Day Hackathon — The Brief

**Build two microservices. Containerise them. Automate the build. Provision the infrastructure.**

Two days. Teams of 2–3. No frontend.

---

## The short version

By 4pm tomorrow you'll demo, to this room:

1. **Two Express microservices**, each with models, controllers and routers, each backed by a SQL database
2. **Both containerised**, with images published to a registry
3. **A pipeline** that rebuilds and pushes both images automatically when you push code
4. **Terraform** that provisions a VM with a resource group, network and security rules

**Extension:** the pipeline runs your Terraform too.

That's it. How you get there is largely up to you.

---

## There is nothing new here

This matters, so read it properly.

**Every skill you need, you already have.** You wrote a scaffold script in Session 2. You built pipelines in Session 3. You wrote Terraform in Sessions 4 and 5. You've been containerising things all course.

The only variable that changes today is that **nobody is walking you through it.**

That's uncomfortable and it's meant to be. Two days of an unfamiliar problem with no step-by-step instructions is much closer to the actual job than any session we've run.

**Being stuck is the work, not a sign it's going wrong.**

---

## What you're building

The domain is fixed, so everyone builds the same *shape* and the interesting decisions stay open.

| Service | Owns | Example endpoints |
|---|---|---|
| **countries-service** | the `countries` table | `GET /countries`, `GET /countries/:id`, `POST /countries` |
| **cities-service** | the `cities` table | `GET /cities`, `GET /cities/:id`, `POST /cities` |

A city has a `country_id`.

**No frontend.** `curl` and your pipeline's console output are the only interfaces that matter. Don't build a UI, even if you want to.

### Roughly the structure you're aiming for

```
.
├── docker-compose.yml
├── Jenkinsfile
├── db/
│   └── init.sql
├── infra/
│   └── main.tf
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

If that looks familiar, it's because it's what your Session 2 scaffold script produced — twice.

---

## Decisions you have to make

These are genuinely open. **I'll ask you to justify them at the showcase**, and *"it was simpler and we didn't need more"* is a good answer. *"It's what we did in the session"* is not.

| Decision | Things to weigh |
|---|---|
| **Reuse your Session 2 script, extend it, or write a new one?** | Reusing is fastest. Extending it to take a port and a database name makes it a better tool. Which would you want if you had to do this ten more times? |
| **One database or two?** | Two means each service owns its data and neither can break the other. One is simpler and lets you `JOIN`. What happens when one team wants to rename a column? |
| **Do the services talk to each other?** | Three valid answers: the client joins the data; cities-service calls countries-service over HTTP; or they share a database. If you pick HTTP — what happens when the other service is down? |
| **Docker Hub or Azure Container Registry?** | Hub is faster and you've used it. ACR keeps images in your subscription and is more realistic for an Azure shop — but it's another Terraform resource and another auth step. |
| **One pipeline or two?** | One guarantees both services are always built from the same commit. Two means changing one service doesn't rebuild the other. Which problem would you rather have? |
| **One VM or two?** | You'll hit this the moment you realise two services can't both listen on port 3000. Different host ports? A reverse proxy? Two VMs? |

Write your answers in `hackathon-notes.md` as you make them. You'll present from it.

---

## What's in the starter repo

**`/hackathon/starter-code`**

| File | What it's for |
|---|---|
| **`docker-compose.yml`** | A local Postgres. Run it and forget it — don't lose a morning installing a database |
| **`db/init.sql`** | Creates both databases. Loaded automatically the first time Postgres starts |
| **`reference/scaffold`** | The scaffold script from Session 2, in case you didn't keep yours |
| **`infra/main.tf`** | A skeleton with `TODO`s. You add the resources |
| **`commands.md`** | Every command you'll need across two days |
| **`hackathon-notes.md`** | Your decisions log. Fill it in as you go |
| **`completed-code/`** | **Catch-up only.** Opening this on day one means you've skipped the entire point |

### Before you start anything

*(Run from `~/`)*
```bash
mkdir -p ~/hackathon && cd ~/hackathon

docker --version            # Docker installed and running?
docker compose version
node --version              # Node 20+?
git --version
az account show             # signed in to Azure?
terraform version
export | grep ARM           # Service Principal creds present?
```

**Anything blank or erroring — flag it now**, not at 11am.

### Getting the database up

*(Run from `~/hackathon/<your-repo>`)*
```bash
docker compose up -d
docker compose ps                 # should show 'running'
docker compose logs postgres      # if it isn't

# Prove you can reach it:
docker exec -it hackathon-db psql -U hackathon -d countries_db -c "\l"
```

---

## The schedule

You're responsible for your own pace. The checkpoints are where I'll come round — **being behind at one is useful information, not a problem.**

### Day 1 — Build it

| Time | What you should be doing |
|---|---|
| 09:00–09:20 | Kickoff |
| 09:20–10:30 | Scaffold both services |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Make them actually work — SQL schema, seed data, live endpoints |
| **12:30–13:00** | **✅ Checkpoint 1** |
| **13:00–14:00** | **Lunch** |
| 14:00–15:15 | Containerise both services |
| **15:15–15:30** | **Break** |
| 15:30–16:30 | Push images by hand; start the Terraform |
| **16:30–17:00** | **✅ Checkpoint 2** |

### Day 2 — Automate it

| Time | What you should be doing |
|---|---|
| 09:00–09:15 | Standup |
| 09:15–10:30 | Pipeline: checkout, test, build both images |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Push both images; trigger on push |
| **12:30–13:00** | **✅ Checkpoint 3** |
| **13:00–14:00** | **Lunch** |
| 14:00–15:00 | Terraform: resource group, network, NSG, VM |
| **15:00–15:15** | **Break** |
| 15:15–15:45 | Extension, or peer review |
| **15:45–16:00** | **✅ Checkpoint 4** + demo prep |
| 16:00–17:00 | **Showcase** |

---

## Checkpoints

### ✅ Checkpoint 1 — Day 1, 12:30

- [ ] A GitHub repo with **both** service folders scaffolded
- [ ] Postgres running via `docker compose`, both databases created
- [ ] **At least one** service returning real JSON from a real table via `curl`
- [ ] A written answer to *"do your services talk to each other, and why?"*

### ✅ Checkpoint 2 — Day 1, 16:30

- [ ] **Both** services returning JSON from their databases
- [ ] **Both** services containerised and running in containers
- [ ] **Both** images pushed to a registry
- [ ] `infra/main.tf` started — at minimum a resource group and a VNet
- [ ] You can explain **out loud** why your containerised service couldn't reach `localhost`

### ✅ Checkpoint 3 — Day 2, 12:30

- [ ] A full pipeline run triggered by a **real `git push`**, not a manual build
- [ ] Every stage passing
- [ ] **Both** images in your registry, tagged with the build number
- [ ] You can point at where each secret is stored, and it isn't in the repo

### ✅ Checkpoint 4 — Day 2, 15:45

- [ ] Terraform applied — resources visible in the Azure Portal
- [ ] `.tf` files committed
- [ ] You can explain **every resource your Terraform creates** and why it's needed
- [ ] `hackathon-notes.md` complete, including the decisions table

---

## Success criteria

What I'm looking for by the showcase:

| Requirement | Evidence |
|---|---|
| Two Express services with models, controllers, routers | The repo structure, and `curl` returning JSON from both |
| Each backed by SQL with schema and seed data | The `.sql` files, and real rows in the responses |
| Both containerised | A `Dockerfile` per service; both running as containers |
| Images published | Both in your registry, tagged per build |
| **Automated image builds** | A `Jenkinsfile` in the repo, triggered by a **real `git push`** |
| Secrets handled properly | **No credentials in Git** — Jenkins credentials store used |
| Infrastructure as code | `.tf` files in the repo; `terraform apply` runs cleanly |
| Network security configured | An NSG with **justified** rules — not `0.0.0.0/0` on everything by accident |
| **Extension:** pipeline runs Terraform | Terraform stages in the `Jenkinsfile`, with a remote backend |
| You can explain your own architecture | A verbal walkthrough, without notes |

---

## How to get unstuck

### Start with zero layers and add one at a time

**The most common way to lose a day is trying to debug in the cloud something that was never working on your laptop.**

Every layer you add is another place the failure could be:

```
one service, locally, hitting a database        <- START HERE
  + a second service
    + containers
      + a registry
        + a pipeline
          + a VM
```

Get each one green before adding the next. **Do not** write all six pipeline stages and then push.

### Work the layers, smallest first

When something doesn't respond, don't guess. Ask, in order:

1. **Is the thing actually running?** `docker compose ps` / `docker ps` / `kubectl get pods`
2. **If not, why did it stop?** `docker logs <name>` — the answer is almost always in there, unread
3. **What port is it actually listening on?** Say the number out loud from your code, not from memory
4. **Can you reach it from the machine it's running on?** `curl localhost:3000` **on the VM**, over SSH
5. **Is something in between blocking it?** NSG rule, port mapping, firewall

**Step 4 is the one people skip and it's the most valuable.** If `curl localhost` works on the box but the public IP doesn't, you've narrowed it to **networking** — you've just halved the search space.

### Questions worth asking yourselves

- *"Which layer is this — app, container, network, or cloud?"*
- *"What did I expect, what actually happened, and what's the smallest difference between them?"*
- *"Have I actually read the error, or just the last line of it?"*
- *"Did this ever work? What changed since?"*

### When you're properly stuck

**Post in the channel.** Say:
- where you are
- what you expected
- what actually happened (paste the error)

Ten minutes of genuine effort first. After that, asking is the right call — and I'd much rather see fifteen messages in the channel than fifteen people quietly failing.

---

## Things that will catch you out

Save yourself the time — these are the ones that get almost everyone.

| Symptom | Almost always |
|---|---|
| `ECONNREFUSED` connecting to Postgres locally | Container not running, or the wrong **database name** in your connection string |
| `relation "countries" does not exist` | Your `db:setup` never ran, or ran against the wrong database |
| Works with `npm start`, fails in a container | **`localhost` inside a container means the container**, not your laptop. You need the compose service name (`postgres:5432`) or `host.docker.internal` |
| Container exits immediately | **Read `docker logs`.** It's in there |
| `exec format error` on the VM | You built an **arm64** image on an Apple Silicon Mac. Rebuild with `--platform linux/amd64` |
| `denied: requested access to the resource is denied` | Your image name doesn't start with **your** Docker Hub username |
| `docker: not found` inside Jenkins | Stock Jenkins image has no Docker CLI. You need your custom image **and** the socket mount |
| Pipeline works on **Build Now**, not on push | Trigger config. Webhooks can't reach `localhost` — use **Poll SCM** (`H/2 * * * *`) |
| Both services want port 3000 on one VM | That's a real design decision. Different host ports, a proxy, or two VMs |
| Terraform plans to recreate everything | **Local state**, run from a pipeline. You need a remote backend |
| App unreachable at the VM's public IP | NSG rule missing for that port, or the container isn't running. **`curl localhost` on the VM** tells you which |
| Terraform state locked | A previous pipeline run is still sitting at an approval gate |

### One to fix on Day 1, not Day 2

**If you're on an Apple Silicon Mac**, your images default to `linux/arm64` and **will fail on an Azure VM tomorrow** with a confusing `exec format error` that looks completely unrelated to anything you did.

Build them properly from the start:

```bash
docker build --platform linux/amd64 -t <username>/countries-service:v1 .
```

---

## The extension

If you finish the core brief, add Terraform stages to your pipeline:

```
Checkout → Test → Build Images → Push Images
        → Terraform Init → Validate → Plan → [approval] → Apply
```

**One thing will block you, and it's worth knowing in advance:** a Jenkins workspace starts **empty** on every build. If your Terraform state is a local file, Terraform will believe nothing exists and try to create everything — every single build.

You need a **remote backend**. You built one in Session 5; go and find your notes.

---

## Stretch goals

If you get everything else done:

- **Run the containers on the VM** — install Docker via `custom_data`, pull both images, run them
- **Put a `docker-compose.yml` on the VM** and start the whole stack there
- **Service-to-service calls** — have cities-service enrich its response from countries-service, and handle the case where it's unavailable
- **Real tests** — integration tests against a test database, gating the pipeline properly
- **Two pipelines**, one per service
- **Azure Container Registry**, provisioned by your own Terraform
- **`when { branch 'main' }`** and a Multibranch Pipeline, so PRs only test and build
- **Restrict SSH** to your own IP (`curl ifconfig.me`, then `/32`) rather than the whole internet
- **Health endpoints** on both services

---

## The showcase — Day 2, 16:00

**6 minutes per team.** Four things:

1. **60-second architecture summary** — two services, what each does, what's running where
2. **A live demo** — `curl` both services, then **make a small change, push it, and let us watch the pipeline run**
3. **One decision you made, and why** — straight from your notes table
4. **One thing that went wrong**, and how you fixed it

That last one is **in the brief deliberately.** Debugging is the job, not an embarrassment — and there's a good chance two other teams hit the same thing and each assumed they were the only one.

**Two practical warnings:**
- **Have your change ready to push before you present.** Don't write it live in front of the room
- **Know how long your pipeline takes.** If it's four minutes, talk through your architecture while it runs

Questions I might ask:
- *"Why did your services talk / not talk to each other?"*
- *"One pipeline or two — would you choose the same again?"*
- *"What would break first if this got 100× the traffic?"*
- *"What's still manual that you'd automate next?"*

---

## Before you leave on Day 2

**Tear everything down.** Azure charges by the hour whether you're using it or not.

*(Run from `~/hackathon/<your-repo>/infra`)*
```bash
terraform destroy        # type: yes
```

*(Run from anywhere)*
```bash
az group list -o table
az resource list -o table
```

Both should come back empty. If anything's left, sort it before you close the laptop.

*(Run from `~/hackathon/<your-repo>`)*
```bash
docker compose down      # stop the local stack
docker stop jenkins      # if you want your laptop's resources back
```

---

## Good luck

You have every skill this needs. The only new thing is doing it yourselves, at pace, with the pieces connected.

**Start small. Add one layer at a time. Read the errors. Ask early.**

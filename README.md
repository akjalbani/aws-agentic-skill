# ☁️ AWS Agent Skills

> 🚀 **Agentic Skills for AWS Development — Learner Lab Edition**
>
> Supercharge your AI coding assistant with curated, production-ready skills for Amazon Web Services.
> Built specifically for **AWS Academy Learner Lab** students. Works with:
>
> 🟣 **Claude Code** | 🔵 **GitHub Copilot** | 🟠 **Cursor** | 🟢 **Codex CLI**

---

AWS Agent Skills is a curated collection of **`SKILL.md` files** following the
[Agent Skills open standard](https://agentskills.io/). Each skill teaches your AI
coding assistant expert-level guidance on specific AWS services — including the
critical constraints of **AWS Learner Lab** that stock AI assistants don't know about.

---

## 🚀 Quick Start

### 1. Clone this repo

```bash
git clone https://github.com/akjalbani/aws-agent-skills.git
```

### 2. Copy skills to your AI assistant

| AI Assistant | Project-level | Global |
|---|---|---|
| **Claude Code** | `{your-project}/.claude/skills/` | `~/.claude/skills/` |
| **GitHub Copilot** | `{your-project}/.github/skills/` | `~/.copilot/skills/` |
| **Cursor** | `{your-project}/.cursor/skills/` | — |
| **Codex CLI** | `{your-project}/.codex/skills/` | `~/.codex/skills/` |

```bash
# Example: copy all skills to Claude Code global path
cp -r aws-agent-skills/skills/* ~/.claude/skills/
```

> ⚠️ Copy the folders **inside** `skills/` (e.g., `aws-lambda/`), NOT the `skills/` folder itself.

### 3. Start coding

Just ask naturally:

> "How do I create a Lambda function in my Learner Lab?"
>
> "Set up an S3 bucket and trigger Lambda on upload"
>
> "My credentials expired — how do I fix this?"

The skills provide accurate, Learner Lab–aware guidance automatically.

---

## 📦 Available Skills

| Skill | Description |
|-------|-------------|
| 🔬 **aws-learner-lab** | **Start here!** Critical constraints, credential setup, budget tips, and session management |
| ⚡ **aws-lambda** | Serverless functions — creation, triggers, deployment, packaging, logs |
| 🪣 **aws-s3** | Object storage — buckets, uploads, permissions, static websites, presigned URLs |
| 💻 **aws-ec2** | Virtual machines — launch, SSH, security groups, user data, cost management |
| 🔒 **aws-iam** | Identity — understanding LabRole, passing roles to services, common errors |
| 🏗️ **aws-cloudformation** | Infrastructure as Code — templates, stacks, deploy, cleanup |
| 🌐 **aws-vpc** | Networking — default VPC, security groups, avoiding expensive NAT Gateways |

---

## 🎯 Why a Learner Lab Edition?

AWS Learner Lab has specific constraints that general AWS documentation doesn't cover:

- ❌ Can't create IAM users or roles → use pre-existing `LabRole`
- ⏱️ Session credentials expire every ~4 hours → need to refresh
- 💰 Budget limit (~$50-100) → must stop resources after use
- 🚫 Some services unavailable (Route53, etc.)
- 🌎 Restricted to `us-east-1` region by default

Every skill in this collection is **Learner Lab aware** — it tells the AI what works
and what doesn't in your environment.

---

## 📖 How Skills Work

Skills use the [SKILL.md standard](https://agentskills.io/). Each skill has:

1. **YAML frontmatter** with `name` and `description` — the AI reads this to decide when to load the skill
2. **Full instructions** — loaded when you ask a relevant question
3. **Code examples** — tested patterns that work in Learner Lab
4. **Documentation links** — authoritative AWS sources

---

## ➕ Adding More Skills

To add a new AWS service skill:

```bash
mkdir skills/aws-dynamodb
cat > skills/aws-dynamodb/SKILL.md << 'EOF'
---
name: aws-dynamodb
description: >
  Use this skill for Amazon DynamoDB — creating tables, putting/getting items,
  querying, scanning, and using DynamoDB from Lambda or EC2.
---

# Amazon DynamoDB
...
EOF
```

Follow the pattern in the existing skills. Key things to include:
- ⚠️ Learner Lab constraints for this service
- CLI commands (always include `--region us-east-1`)
- Python boto3 examples
- Links to official AWS documentation

---

## 📚 Recommended Learning Path

1. Start with **aws-learner-lab** — understand your environment
2. Add **aws-iam** — understand how permissions work
3. Learn **aws-s3** — simplest storage service, free tier friendly
4. Learn **aws-lambda** — serverless, cheap, powerful
5. Add **aws-cloudformation** — manage everything as code
6. Then **aws-ec2**, **aws-vpc** — virtual machines and networking

---

## ⚖️ License

- Skills content: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- Code examples: [MIT License](https://opensource.org/licenses/MIT)

---

## 📚 Resources

- [Agent Skills Standard](https://agentskills.io/)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS CLI Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [AWS Academy](https://awsacademy.instructure.com/)

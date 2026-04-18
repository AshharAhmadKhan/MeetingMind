<div align="center">

# MeetingMind

### AI meeting intelligence — built on AWS

[![AWS](https://img.shields.io/badge/AWS-14_Services-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://react.dev)
[![Serverless](https://img.shields.io/badge/Architecture-Serverless-FD5750?logo=serverless&logoColor=white)](https://aws.amazon.com/serverless/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**[Demo Video](https://www.youtube.com/watch?v=AH7-1JtyZIk)** • **[AWS Article](https://builder.aws.com/content/3A2donG56UEkNpzuma7fRqX0LpT/aideas-meetingmind)** • **[Documentation](docs/)**

*Built for AWS AIdeas Competition 2026 — backend currently offline. Run `sam deploy` to spin it back up.*

</div>

---

## The Problem

I kept noticing the same thing in every team meeting — someone would write down action items, everyone would nod, and then half of them would just never happen. Not because people were lazy, but because there was no system. The tasks lived in someone's notes app, or a Notion page nobody checked, or just in people's heads.

I wanted to build something that actually fixes that. Not just another transcription tool, but something that tracks what happens to the commitments made in meetings over time.

---

## What It Does

You upload a meeting recording and MeetingMind handles the rest. Within a few minutes you get:

- A transcript with speaker labels
- The decisions that were made
- Action items with owners and deadlines automatically pulled from the conversation
- A risk score for each task based on how likely it is to get dropped
- Duplicate detection so the same task doesn't get assigned twice across different meetings
- A dashboard showing the financial cost of all the incomplete work sitting in your backlog

The part I'm most proud of is the Graveyard — a page that shows every action item that's been sitting untouched for over 30 days, with an AI-generated epitaph explaining why it probably died. It sounds dark but it's genuinely useful for making teams confront the work they keep avoiding.

---

## Screenshots

<div align="center">

### Dashboard - Meeting Overview
<img src="docs/screenshots/dashboard.png" alt="Dashboard" width="800"/>

*All your meetings in one place, each with an AI-generated health score*

---

### Kanban Board - Action Management
<img src="docs/screenshots/kanban-board.png" alt="Kanban Board" width="800"/>

*Drag and drop actions between columns, with risk scores and deadline countdowns*

---

### The Graveyard
<img src="docs/screenshots/graveyard.png" alt="Graveyard" width="800"/>

*Abandoned tasks, how long they've been sitting there, and a way to bring them back*

---

### Meeting Debt Dashboard
<img src="docs/screenshots/meeting-debt.png" alt="Meeting Debt" width="800"/>

*A dollar estimate of how much incomplete work is costing your team*

</div>

---

## Features

**Audio processing**
Upload MP3, MP4, WAV, M4A, or WEBM files up to 500MB. Amazon Transcribe handles speaker diarization, and Bedrock (Claude, Nova Lite, Nova Micro in fallback order) extracts the structured data. Usually takes 2-5 minutes for a 30-minute meeting.

**Action management**
A Kanban board across all your meetings. Drag items between To Do, In Progress, Blocked, and Done. Each card shows a risk score calculated from deadline urgency, whether it has an owner, how vague the task description is, and how old it is.

**Meeting debt**
Every incomplete action item has a cost attached to it — based on average hourly rates and estimated blocked time. The debt dashboard adds all of that up and shows you the trend over 30 days.

**The Graveyard**
Anything untouched for 30+ days ends up here. You can see how long it's been abandoned, read the AI-generated epitaph, and resurrect it with a new owner and deadline if it still matters.

**Pattern detection**
After you have enough meetings, the app starts identifying patterns — things like "your team keeps generating more tasks than it completes" or "one person is doing most of the work." It gives you a name for the pattern and a suggested fix.

**Teams**
Create a team, share a 6-character invite code, and everyone on the team can see all the meetings and update action items. There's a leaderboard that ranks people by completion rate.

---

## Architecture

Built entirely serverless on AWS. 18 Lambda functions, 14 services, zero servers to manage.

```
User uploads audio -> S3 -> SQS -> Lambda -> Transcribe
                                       |
                                   Bedrock AI
                                       |
                              DynamoDB + Embeddings
                                       |
                           Email notification (SES)

User opens dashboard -> CloudFront -> React app
                                          |
                                    API Gateway
                                          |
                                   Lambda functions
                                          |
                                      DynamoDB
```

**Frontend:** React 19, Vite, AWS Amplify for auth, @dnd-kit for drag and drop

**Backend:** Python 3.11, AWS SAM, API Gateway, Cognito

**AI:** Amazon Transcribe, Bedrock (Claude Haiku, Nova Lite, Nova Micro, Titan Embeddings)

**Data:** DynamoDB, S3, CloudFront

**Monitoring:** CloudWatch, X-Ray, SES, SNS, EventBridge

---

## Getting Started

You'll need an AWS account with Bedrock access, AWS CLI set up, Python 3.11+, Node.js 18+, and the SAM CLI.

**Deploy the backend:**

```bash
cd backend
sam build
sam deploy --guided
```

**Deploy the frontend:**

```bash
cd frontend
cp .env.example .env.production
# Fill in the API Gateway URL and Cognito IDs from the backend deploy output

npm install
npm run build

aws s3 sync dist/ s3://YOUR_BUCKET --delete
aws cloudfront create-invalidation --distribution-id YOUR_ID --paths "/*"
```

Then open the CloudFront URL, sign up, and upload a recording.

Full instructions in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/ARCHITECTURE.md) | How the system is designed |
| [Architecture Diagrams](docs/ARCHITECTURE_DIAGRAM.md) | Visual diagrams and flows |
| [API Reference](docs/API.md) | REST API endpoints |
| [Deployment Guide](docs/DEPLOYMENT.md) | Step-by-step deployment |
| [Development Guide](docs/DEVELOPMENT.md) | Running it locally |
| [Testing Guide](docs/TESTING.md) | How to run the tests |
| [Features](docs/FEATURES.md) | Detailed feature breakdown |

---

## Project Structure

```
meetingmind/
├── backend/
│   ├── functions/          # 18 Lambda functions
│   │   ├── process-meeting/
│   │   ├── get-all-actions/
│   │   ├── check-duplicate/
│   │   ├── get-debt-analytics/
│   │   └── ...
│   └── template.yaml       # SAM infrastructure
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── utils/
│   └── package.json
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── scripts/
│   ├── deploy/
│   ├── monitoring/
│   ├── utils/
│   └── data/
│
└── docs/
```

---

## Tests

```bash
python -m pytest tests/
python -m pytest tests/integration/
coverage run -m pytest tests/ && coverage report
```

Pre-commit hooks run the tests automatically before each commit.

---

## Author

**Ashhar Ahmad Khan** — CS student, built this for the AWS AIdeas Competition 2026

- Email: itzashhar@gmail.com
- LinkedIn: [linkedin.com/in/ashhar-ahmad-khan](https://www.linkedin.com/in/ashhar-ahmad-khan)
- GitHub: [@AshharAhmadKhan](https://github.com/AshharAhmadKhan)

---

## License

MIT — see [LICENSE](LICENSE) for details.

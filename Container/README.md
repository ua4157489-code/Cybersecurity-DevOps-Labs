<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=220&section=header&text=Podman%20Mastery%20Labs&fontSize=48&fontColor=ffffff&animation=fadeIn&fontAlignY=35&desc=From%20First%20Container%20to%20Production-Ready%20Workflows&descAlignY=55&descSize=18" width="100%"/>

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=24&duration=3000&pause=800&color=8A2BE2&center=true&vCenter=true&width=700&lines=Learn+Containers+the+Hands-On+Way;Podman+%E2%80%A2+Pods+%E2%80%A2+Volumes+%E2%80%A2+Compose;17+Labs+%7C+Zero+to+Advanced;Build.+Break.+Debug.+Repeat." alt="Typing SVG" />

<br/>

![Labs](https://img.shields.io/badge/Labs-17-8A2BE2?style=for-the-badge&logo=bookstack&logoColor=white)
![Podman](https://img.shields.io/badge/Podman-Engine-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

![Made with Love](https://img.shields.io/badge/Made%20with-%E2%9D%A4-red?style=flat-square)
![Beginner Friendly](https://img.shields.io/badge/Beginner-Friendly-success?style=flat-square)
![Last Updated](https://img.shields.io/badge/Last%20Updated-Today-orange?style=flat-square)

</div>

---

## 📖 About This Repository

Welcome to the **Podman Mastery Labs** — a hands-on, progressively structured series of labs that take you from *"what is a container?"* all the way to debugging, networking, and orchestrating multi-container apps with Compose.

Every lab lives in its own folder, has its own `README.md`, and builds directly on the skills from the one before it. No fluff — just practical exercises you run yourself.

<div align="center">
<img src="https://user-images.githubusercontent.com/74038190/212284100-561aa473-3905-4a80-b561-0d28506553ee.gif" width="100%">
</div>

---

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# Jump into Lab 1
cd 01-Introduction-to-Containers
cat README.md
```

> 💡 **Tip:** Work through the labs in order — each one assumes you've completed the previous.

---

## 🗺️ Learning Path

<div align="center">

```mermaid
flowchart LR
    A[🧱 Foundations] --> B[🖼️ Images]
    B --> C[⚙️ Runtime Config]
    C --> D[💾 Data & State]
    D --> E[🌐 Networking & Debug]
    E --> F[🧩 Compose & Orchestration]

    style A fill:#8A2BE2,color:#fff,stroke:#333
    style B fill:#892CA0,color:#fff,stroke:#333
    style C fill:#6A0DAD,color:#fff,stroke:#333
    style D fill:#4B0082,color:#fff,stroke:#333
    style E fill:#5D3FD3,color:#fff,stroke:#333
    style F fill:#483D8B,color:#fff,stroke:#333
```

</div>

---

## 📚 Lab Index

<details open>
<summary><b>🧱 Module 1 — Foundations</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 01 | [Introduction to Containers](./01-Introduction-to-Containers) | Container fundamentals & core concepts | ✅ |
| 02 | [Exploring Podman CLI](./02-Exploring-Podman-CLI) | Getting comfortable with the `podman` command | ✅ |
| 03 | [Running Containers with Podman](./03-Running-Containers-with-Podman) | Launching and managing your first containers | ✅ |
| 04 | [Creating and Managing Pods](./04-Creating-and-Managing-Pods) | Group containers into Podman pods | ✅ |

</details>

<details open>
<summary><b>🖼️ Module 2 — Images</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 05 | [Managing Container Images](./05-Managing-Container-Images) | Pulling, tagging, and inspecting images | ✅ |
| 06 | [Building Custom Container Images](./06-Building-Custom-Container-Images) | Writing Containerfiles from scratch | ✅ |
| 07 | [Layer Caching and Optimization](./07-Layer-Caching-and-Optimization) | Faster, leaner image builds | ✅ |

</details>

<details open>
<summary><b>⚙️ Module 3 — Runtime Configuration</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 08 | [Environment Variables in Images](./08-Environment-Variables-in-Images) | Configuring containers at runtime | ✅ |
| 09 | [Using ENTRYPOINT and CMD](./09-Using-ENTRYPOINT-and-CMD) | Controlling container startup behavior | ✅ |

</details>

<details open>
<summary><b>💾 Module 4 — Data & State</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 10 | [Persisting Data with Volumes](./10-Persisting-Data-with-Volumes) | Keeping data alive beyond the container | ✅ |
| 11 | [Running Stateful Containers](./11-Running-Stateful-Containers) | Databases & stateful workloads | ✅ |
| 12 | [Backup and Restore Data](./12-Backup-and-Restore-Data) | Protecting your persistent volumes | ✅ |

</details>

<details open>
<summary><b>🌐 Module 5 — Networking & Debugging</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 13 | [Troubleshooting Containers](./13-Troubleshooting-Containers) | Diagnosing common container issues | ✅ |
| 14 | [Networking in Containers](./14-Networking-in-Containers) | Container-to-container communication | ✅ |
| 15 | [Remote Debugging of Containers](./15-Remote-Debugging-of-Containers) | Attaching debuggers to running containers | ✅ |

</details>

<details open>
<summary><b>🧩 Module 6 — Compose & Orchestration</b></summary>
<br/>

| # | Lab | Description | Status |
|---|-----|-------------|:------:|
| 16 | [Compose Basics](./16-Compose-Basics) | Defining multi-container apps declaratively | ✅ |
| 17 | [Compose with Dependencies](./17-Compose-with-Dependencies) | Managing service dependencies & startup order | ✅ |

</details>

---

## 📊 Progress Overview

<div align="center">

![Progress](https://progress-bar.xyz/100/?title=Labs%20Completed&width=500&color=8a2be2&suffix=%20(17/17))

</div>

| Category | Labs | Progress |
|----------|:----:|----------|
| 🧱 Foundations | 4 | ████████████████████ 100% |
| 🖼️ Images | 3 | ████████████████████ 100% |
| ⚙️ Runtime Config | 2 | ████████████████████ 100% |
| 💾 Data & State | 3 | ████████████████████ 100% |
| 🌐 Networking & Debug | 3 | ████████████████████ 100% |
| 🧩 Compose | 2 | ████████████████████ 100% |

---

## 🛠️ Tech Stack

<div align="center">

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

---

## 📁 Repository Structure

```text
.
├── 01-Introduction-to-Containers/
├── 02-Exploring-Podman-CLI/
├── 03-Running-Containers-with-Podman/
├── 04-Creating-and-Managing-Pods/
├── 05-Managing-Container-Images/
├── 06-Building-Custom-Container-Images/
├── 07-Layer-Caching-and-Optimization/
├── 08-Environment-Variables-in-Images/
├── 09-Using-ENTRYPOINT-and-CMD/
├── 10-Persisting-Data-with-Volumes/
├── 11-Running-Stateful-Containers/
├── 12-Backup-and-Restore-Data/
├── 13-Troubleshooting-Containers/
├── 14-Networking-in-Containers/
├── 15-Remote-Debugging-of-Containers/
├── 16-Compose-Basics/
├── 17-Compose-with-Dependencies/
└── README.md
```

---

## 🤝 Contributing

Contributions, fixes, and new lab ideas are welcome!

1. Fork the repo
2. Create a feature branch (`git checkout -b lab/18-new-topic`)
3. Commit your changes
4. Open a Pull Request

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.

---

<div align="center">

### ⭐ If this helped you learn Podman, consider starring the repo!

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=120&section=footer" width="100%"/>

</div>

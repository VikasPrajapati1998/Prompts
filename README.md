# Technology Documents & Prompts Repository

A curated collection of technical guides, cheat sheets, and reusable prompts covering AI/ML infrastructure, quantum computing, security auditing, development workflows, and productivity tools.

---

## 📚 Repository Contents

| Document | Category | Description |
|----------|----------|-------------|
| [vLLM.md](vLLM.md) | LLM Infrastructure | Complete guide to vLLM — high-throughput LLM inference and serving with PagedAttention |
| [AirLLM_Guide.md](AirLLM_Guide.md) | LLM Infrastructure | Hardware requirements, installation, and usage guide for AirLLM (streaming LLM layers from disk) |
| [Ollama-CMD.md](Ollama-CMD.md) | LLM Infrastructure | Command cheat sheet for Ollama Cloud, Claude Code, and OpenCode integration |
| [Quantum-Computation-Qiskit.md](Quantum-Computation-Qiskit.md) | Quantum Computing | Comprehensive Qiskit guide: circuits, transpilation, primitives, addons, hardware backends |
| [Security-Audit-Prompt.md](Security-Audit-Prompt.md) | Security | Reusable prompt for full security audit & testing of frontend + backend applications |
| [Dynamic-Rank-RBAC-Prompt.md](Dynamic-Rank-RBAC-Prompt.md) | Security/Architecture | Technology-neutral prompt for implementing dynamic rank-based access control |
| [BMAD_NEW_PROJECT_PLAYBOOK.md](BMAD_NEW_PROJECT_PLAYBOOK.md) | Development Workflow | 7-phase playbook for BMAD + Claude Code: discovery → requirements → architecture → stories → implementation → testing → production |
| [Plexo.md](Plexo.md) | Networking | Overview of Plexo network bonding concept for combining multiple connections |
| [FreeWebsites.md](FreeWebsites.md) | Productivity | Privacy-friendly free web apps (video editing, AI, music, screen recording) |
| [FreeSoftware.md](FreeSoftware.md) | Productivity | Open-source desktop software alternatives (video, AI, audio, screen recording) |

---

## 🗂️ By Category

### 🤖 LLM Serving & Inference
- **vLLM** — Production-grade LLM serving with OpenAI-compatible API, PagedAttention, continuous batching, multi-GPU support
- **AirLLM** — Run large models on limited VRAM by streaming layers from disk/storage
- **Ollama Cloud + OpenCode** — Quick commands for cloud model access and IDE integration

### ⚛️ Quantum Computing
- **Qiskit** — End-to-end guide covering circuit construction, transpilation, primitives (Sampler/Estimator), addons, function templates, visualization, and hardware execution on IBM Quantum, IonQ, AWS Braket, Azure Quantum, and more

### 🔒 Security & Architecture
- **Security Audit Prompt** — Complete 3-phase audit methodology: discovery → static analysis → dynamic testing with automated tooling
- **Dynamic Rank RBAC** — Runtime-editable role hierarchy with rank-based visibility, permission intersection, and grant ceiling enforcement

### 🛠️ Development Workflows
- **BMAD Playbook** — Gate-controlled, artifact-driven development from idea to production using BMAD specialist roles (analyst, PM, architect, dev, QA)

### 🌐 Networking
- **Plexo** — Conceptual overview of network bonding for combining Wi-Fi, Ethernet, and mobile data

### 💡 Free Tools & Resources
- **Free Websites** — Browser-based tools requiring no account: OpenCut (video), AnythingLLM (local AI), Handy (speech-to-text), Nuclear (music), Cap (screen recording)
- **Free Software** — Open-source desktop equivalents with GitHub links

---

## 🚀 Quick Start

### For LLM Practitioners
```bash
# vLLM - Start OpenAI-compatible server
pip install vllm
vllm serve meta-llama/Llama-3.1-8B-Instruct

# AirLLM - Run large models on limited VRAM
pip install airllm
python -c "from airllm import AutoModel; m = AutoModel.from_pretrained('Qwen/Qwen2.5-3B-Instruct')"

# Ollama Cloud
ollama signin
ollama run gemma4:31b-cloud
ollama launch opencode --model gemma4:31b-cloud
```

### For Quantum Developers
```bash
pip install qiskit qiskit[visualization] qiskit-ibm-runtime
# See Quantum-Computation-Qiskit.md for complete workflows
```

### For Security Auditors
```bash
# Copy Security-Audit-Prompt.md into a Claude Code session at project root
# Fill in the Parameters block and run
```

### For New Projects
```bash
mkdir my-project && cd my-project
git init
npx bmad-method install
claude
# Paste the Master Prompt from BMAD_NEW_PROJECT_PLAYBOOK.md
```

---

## 📖 How to Use This Repository

1. **Browse by topic** — Each document is self-contained with installation, usage, and reference sections
2. **Copy prompts directly** — Security audit and RBAC prompts are designed to be pasted into Claude Code or similar agents
3. **Reference guides** — vLLM, Qiskit, and AirLLM guides serve as offline documentation
4. **Cheat sheets** — Ollama-CMD for quick command lookup

---

## 🤝 Contributing

This is a personal knowledge repository, but suggestions and corrections are welcome via issues or pull requests.

### Adding New Documents
1. Follow the existing markdown structure (overview → installation → usage → references)
2. Include version dates and official links
3. Add entry to the table above
4. Keep content practical and command-focused

---

## 📄 License

See [LICENSE](LICENSE) for details. Individual documents reference their respective upstream project licenses (Apache 2.0, MIT, etc.).

---

## 🔗 Useful Links

| Resource | Link |
|----------|------|
| vLLM | https://docs.vllm.ai / https://github.com/vllm-project/vllm |
| AirLLM | https://github.com/lyogavin/airllm |
| Ollama | https://ollama.com / https://docs.ollama.com |
| Qiskit | https://quantum.cloud.ibm.com/docs / https://github.com/Qiskit/qiskit |
| BMAD Method | https://docs.bmad-method.org |
| OpenCode | https://opencode.ai/docs |

---

*Last updated: October 2026*
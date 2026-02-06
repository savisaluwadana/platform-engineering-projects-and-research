# Quick Start Guide

Welcome to Platform Engineering! This guide will help you get started quickly with hands-on learning.

## 🚀 Getting Started in 5 Minutes

### Step 1: Set Up Your Environment (Day 1)

#### Minimum Requirements
```bash
# If you're on Windows, install WSL2
wsl --install

# If you're on macOS or Linux, you're ready!

# Install essential tools
# On Ubuntu/Debian:
sudo apt update
sudo apt install -y git curl wget vim

# On macOS:
brew install git curl wget
```

#### Verify Your Setup
```bash
# Check your system
uname -a
git --version

# You should see Linux or Darwin (macOS)
```

### Step 2: Choose Your First Project (Day 1)

Start with **ONE** of these beginner projects:

#### Option A: System Dashboard (Easiest)
- **Module**: [01-linux-fundamentals](./01-linux-fundamentals/)
- **Project**: System Administration Dashboard Script
- **Time**: 2-3 hours
- **Skills**: Basic bash scripting, system commands

#### Option B: Network Scanner (More Challenging)
- **Module**: [02-networking-fundamentals](./02-networking-fundamentals/)
- **Project**: Network Scanner Tool
- **Time**: 4-6 hours
- **Skills**: Networking, scripting, tool usage

#### Option C: Git Helper Scripts (Practical)
- **Module**: [03-version-control-git](./03-version-control-git/)
- **Project**: Git Automation Scripts
- **Time**: 3-4 hours
- **Skills**: Git, automation, scripting

### Step 3: Complete Your First Project (Week 1)

#### Example: System Dashboard Project

```bash
# Create project directory
mkdir -p ~/platform-engineering/01-linux-fundamentals/system-dashboard
cd ~/platform-engineering/01-linux-fundamentals/system-dashboard

# Create your first script
touch dashboard.sh
chmod +x dashboard.sh
```

**dashboard.sh** starter:
```bash
#!/bin/bash

# System Administration Dashboard
echo "================================"
echo "   System Dashboard"
echo "================================"
echo ""

# CPU Usage
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1"%"}'

# Memory Usage
echo ""
echo "Memory Usage:"
free -h | awk '/^Mem/ {print $3 "/" $2}'

# Disk Usage
echo ""
echo "Disk Usage:"
df -h / | awk '/\// {print $3 "/" $2 " (" $5 ")"}'

# Logged in Users
echo ""
echo "Logged in Users:"
who | wc -l

# Top 5 Processes
echo ""
echo "Top 5 CPU Processes:"
ps aux --sort=-%cpu | head -6 | tail -5
```

Run it:
```bash
./dashboard.sh
```

### Step 4: Share Your Progress (Week 1)

```bash
# Initialize git repository
git init
git add dashboard.sh
git commit -m "Add: basic system dashboard script"

# Create a GitHub repository and push
# (Follow GitHub's instructions for creating a new repository)
```

## 📅 Your First Month Plan

### Week 1: Linux Basics
- [ ] Set up environment
- [ ] Complete 1 beginner project from Module 01
- [ ] Learn 10 new Linux commands
- [ ] Practice shell scripting

**Daily Commitment**: 1-2 hours

### Week 2: Networking Fundamentals
- [ ] Read networking theory
- [ ] Complete 1 beginner project from Module 02
- [ ] Understand TCP/IP basics
- [ ] Practice with networking tools (ping, traceroute, netstat)

**Daily Commitment**: 1-2 hours

### Week 3: Version Control
- [ ] Master Git basics
- [ ] Complete 1 beginner project from Module 03
- [ ] Set up GitHub profile
- [ ] Practice branching and merging

**Daily Commitment**: 1-2 hours

### Week 4: Introduction to Containers
- [ ] Install Docker
- [ ] Complete 1 beginner project from Module 04
- [ ] Build your first Docker image
- [ ] Run multi-container applications

**Daily Commitment**: 1-2 hours

## 🎯 Quick Wins (Build Confidence Fast!)

### 5-Minute Wins
- [ ] Create a bash script to show system uptime
- [ ] Set up a Git alias for common commands
- [ ] Run your first Docker container
- [ ] Install a useful CLI tool (htop, ncdu, tldr)

### 30-Minute Wins
- [ ] Build a log file analyzer
- [ ] Create a backup script for your home directory
- [ ] Set up SSH keys for GitHub
- [ ] Deploy a web server in Docker

### 2-Hour Wins
- [ ] Build a service health monitor
- [ ] Create a multi-container app with Docker Compose
- [ ] Set up a local Git server
- [ ] Implement automated file cleanup

## 📚 Learning Resources for Beginners

### Free Online Courses
1. **[Linux Journey](https://linuxjourney.com/)** - Interactive Linux tutorial
2. **[Docker for Beginners](https://docker-curriculum.com/)** - Hands-on Docker guide
3. **[Git Handbook](https://guides.github.com/introduction/git-handbook/)** - GitHub's Git guide

### YouTube Playlists
1. **Linux Basics** - TechWorld with Nana
2. **Git & GitHub** - Traversy Media
3. **Docker Tutorial** - Programming with Mosh

### Practice Platforms
1. **[OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)** - Learn Linux through games
2. **[HackerRank Linux Shell](https://www.hackerrank.com/domains/shell)** - Practice problems
3. **[Play with Docker](https://labs.play-with-docker.com/)** - Free Docker playground

## 🛠️ Essential Tools to Install

### Must-Have Tools (Week 1)
```bash
# Git for version control
sudo apt install git

# Text editors
sudo apt install vim
# Or install VS Code from https://code.visualstudio.com/

# Essential utilities
sudo apt install curl wget htop tree
```

### Nice-to-Have Tools (Week 2+)
```bash
# Docker
# Follow: https://docs.docker.com/engine/install/

# Network tools
sudo apt install net-tools dnsutils

# Modern alternatives to classic tools
sudo apt install bat exa fd-find ripgrep
```

## 💡 Tips for Success

### Do's ✅
- **Start small**: Complete small projects before moving to complex ones
- **Practice daily**: Even 30 minutes a day is better than 5 hours once a week
- **Document everything**: Keep notes of what you learn
- **Break things**: Learn by experimentation and mistakes
- **Ask questions**: Use communities when stuck
- **Share progress**: Post your projects on GitHub

### Don'ts ❌
- **Don't skip fundamentals**: Linux and networking are crucial
- **Don't just watch tutorials**: Build projects yourself
- **Don't compare**: Everyone learns at their own pace
- **Don't give up**: It's normal to feel overwhelmed at first
- **Don't memorize**: Understand concepts instead
- **Don't work in isolation**: Join communities

## 🔥 Challenge Yourself

### Week 1 Challenge: Build a Complete Toolbox
Create a GitHub repository with useful scripts:
- System information script
- File backup automation
- Git helper commands
- Log analyzer

### Week 2 Challenge: Network Lab
Set up a small network lab:
- Multiple VMs or containers
- Custom network configuration
- Simple load balancer
- Network monitoring

### Week 3 Challenge: Automation Project
Build an automation project:
- Automated server setup
- Configuration management
- Deployment automation
- Monitoring setup

## 📊 Track Your Progress

Create a simple checklist:

```markdown
# My Platform Engineering Journey

## Month 1
- [x] Set up development environment
- [x] Completed first Linux project
- [ ] Completed first networking project
- [ ] Completed first Docker project

## Skills Acquired
- [x] Basic Linux administration
- [x] Shell scripting
- [ ] Network troubleshooting
- [ ] Container management

## Projects Completed
1. System Dashboard - [link to GitHub]
2. ...
```

## 🎓 Certification Path (Optional)

If you want to get certified:

**Month 3-4**: Prepare for Linux Foundation Certified System Administrator (LFCS)
**Month 5-6**: Prepare for Docker Certified Associate (DCA)
**Month 7-9**: Prepare for Certified Kubernetes Administrator (CKA)

## 🤝 Get Help

### When You're Stuck
1. **Google the error**: Often someone has faced the same issue
2. **Check documentation**: Official docs are usually the best
3. **Ask in communities**: 
   - [Reddit r/devops](https://reddit.com/r/devops)
   - [Stack Overflow](https://stackoverflow.com/)
   - [CNCF Slack](https://slack.cncf.io/)
4. **GitHub Issues**: Look for similar issues in project repos

### Study Groups
Consider joining or creating a study group:
- Weekly meetups (virtual or in-person)
- Project reviews
- Knowledge sharing
- Motivation and accountability

## 🎉 Celebrate Small Wins!

Remember to celebrate when you:
- ✨ Complete your first script
- ✨ Solve your first bug
- ✨ Deploy your first container
- ✨ Get your first star on GitHub
- ✨ Help someone else learn
- ✨ Complete your first project

## 📞 Next Steps

1. **Today**: Set up your environment
2. **This Week**: Complete one beginner project
3. **This Month**: Complete modules 01-03
4. **This Quarter**: Build your first capstone project
5. **This Year**: Land your first platform engineering role!

## 🚀 Ready to Start?

Choose your path:
1. **Quick Learner**: Start with [PROJECT-IDEAS.md](./PROJECT-IDEAS.md) - Quick Start Projects
2. **Structured Learner**: Follow [ROADMAP.md](./ROADMAP.md) - Complete learning path
3. **Deep Diver**: Begin with [01-linux-fundamentals](./01-linux-fundamentals/) - Start from basics

---

**Remember**: Every expert was once a beginner. The only way to fail is to not start. 

**You've got this! 🎯**

---

*Questions? Check out our [CONTRIBUTING.md](./CONTRIBUTING.md) or open an issue!*
